---
title: "How resilient is your consumer?"
author: "Muaz Wazir"
date: "2025-11-11"
toc: true
summary: "How well can your consumer recover from failure"
readTime: true
tags: ["kafka", "consumer", "sqs", "resiliency"]
showTags: true
hideBackToTop: true
draft: true
---


## The Problem
---

In modern systems, you'll find it pretty common where we'd have asynchronous components. Mechanism such as `queues` and `message broker` are an important piece to building a resilient and scalable systems. Regardless of a monolithic backend or a microservices architecture, you'll see this mechanism being used for asyncrhonous processing, event driven architecture and etc.

Despite all the advantages of this architecture decision. It comes with some tricky nuances when it comes to handling errors that are both transient and non transient. Which begs the question

- How resilient is your consumers? 
- How well does your consumer operates in a degraded state?
- Can your consumers keep up under heavy load, do you have mechanism to shed some of the loads?
- Do you have enough visibility to any poison pills your consumers attempted to process?
- How well can your system recovers from lost messages? 

For this topic, we'll specifically cover the usecases with Kafka as the message broker.


## Production system setup with kafka
---

To illustrate how a kafka is being used in systems. Let's start with a diagram

![image](consumer_1.png "Typical kafka pub sub setup")

**Publishers**
- Kafka publishers fire events into a topic. They're the upstream producers — whenever something happens in your system (a user signed up, a payment was processed, an order was fulfilled), the publisher emits that event to the relevant topic so downstream systems can react to it.

**Consumers**
- Kafka consumers subscribe to topics and process whatever events come through. A consumer pulls messages off the queue, does something useful with them (updates a database, triggers a notification, runs business logic), and then moves on to the next message. In a typical setup, you might have multiple consumer instances all reading from the same topic in parallel.

**Topics**
- A topic is a named channel — think of it as a logical inbox. Publishers drop messages in here, consumers pick them up from here. What makes Kafka different from a regular queue is that messages in a topic are retained for a configurable period (usually days), so consumers can re-read old messages if needed, and multiple independent consumers can subscribe to the same topic without interfering with each other.

**Partitions**
- Under the hood, every topic is split into partitions. Partitions are the secret sauce behind Kafka's parallelism — each partition can be consumed by a different consumer in your consumer group simultaneously. Within a single partition, messages are strictly ordered and immutable. This is powerful, but it comes with a catch: if a consumer gets stuck on one message in a partition, everything behind that message waits its turn. That clawback is what we'll get into next.


## A production system handling thousands - millions messages per day
---

One of the benefit of kafka is that you can scale up your partitions alongside the number of consumers you have to increase the message processing throughput (partitions created can't be scaled down, ideal starting point for partition is 3)

Let's imagine if one of the consumer's downstream dependency faces some issue (transient failures, permanent failures)

![image](consumer_2.png "Clogged partition")

Doesn't look pretty right. What the diagram is trying to illustrate is that assuming that one consumer is facing an issue processing messages in one of the partition queue, mind you that `kafka offers ordered message guarantee within partition level`, which in this case would mean that unless the consumer ack() the message, the partition will forever be clogged as messages being queued in that partition won't be processed due to being held back by this one message and your consumer lag will shoot through the roof.

Let's explore the side effect of that.
- Reduced throughput (one clogged partitions meaning you're left with only 2 good partition and consumers)
- Messages that were queued within the clogged partition won't be processed unless someone intervenes or system recovers (assuming transient failures, network blip, rate limit, load shed)

Well that does look nasty, but that's not really the worst thing that could happen

![image](consumer_3.png "Just pray bro")

well ........ gg, now what?


## The Naive Options (and why they suck)
---

Well, from the previous chapter. We explored one of the failure modes that could catastrophically halt your system

In such situation you're left with several `naive` options


### Best effort retry and move on
---

Assuming the problem was intermittent, some dependency has degraded that's causing the an impact to the consumer. We could setup a best effort retry with a simple backoff (retry n number of times and just move on if that fails). This is better than nothing but you're now left with a partially failed transactions and a broken state (assuming you system is eventually consistent).

To be fair this is better than no retry mechanism at all and a lot of system start with this approach to balance speed of development and system reliability. Assming you're only observing some intermittent failures and only 1% of message retries are being exhausted and dropped, it's probably fine to just manually intervene to patch the broken state or repubished the drop message. 

But what happens when one of your dependencies has degraded and was rejecting all new connections for 2 hours straight. 100% of messages were dropped for that 2 hours. Good luck to oncall engineer that has to figure out how to reconcile those messages


### Halt and retry in place
---

This option is as explained in earlier discussions. When a failure happens we let the consumer crash. What would happen is that the consumer comes back up, re-poll the same message on the next cycle and fall again. As you can imagine, this is a vicious loop

```
poll -> fail -> don't ack() -> poll -> fail ....
```

However, this method does ensures that all messages gets processed ...... even poison messages (though it might crash the consumer, yikes)

The downside is clear here.
- Consumer lags increases
- Any transactions and any user journey will get stuck
- It'll be worse for poison messages (no way to self recover)

Both options doesn't look so good right? Best effort is the most viable here if you're considering the two options here


## The Retry Pattern: Decouple and Defer
---

So what do we do about it? At work, we had this exact problem. The moment we realized our consumer was failing synchronously that the Kafka partition was blocked while we retried in place, we knew we had to change the design.

The core premise of the solutuion is to **stop handling failure inside the consumer.** The moment you treat a failed message as a problem the consumer needs to solve right now, you're introducing a mode in your consumer where it needs to handle the failure at the expense of throughput. Instead, park it somewhere else, commit the offset like nothing happened, and move on.

That's the "decouple and defer" principle. The failed message gets handed off to a separate system whose entire job is handling retries. Kafka stays clean and fast it's the hot path for happy path processing. The retry system is the warm path where problem messages go to be tried again later.

If you think about it, this is naturally a mechanism to handle load shedding. Imagine there's a surge in messages in your system and your system is facing issues keeping up. 


### Enter SQS
---

At work, we settled on SQS as that warm path. Two reasons made it the right fit:

**Visibility timeout.** 
- When a message lands in SQS, you can hide it for N number of seconds from other SQS consumers in your fleet. After those N seconds, it reappears automatically, no cron jobs, no polling logic, no custom retry scheduler to build and maintain. SQS handles the timing for you.

**Redrive policy.** 
- This was the killer feature. You configure SQS to say "if a message is received X times and still not deleted, move it here." That "here" is your dead letter queue. We didn't have to write a single line of code to get automatic DLQ behavior. it's built into the SQS queue configuration. It's considered best practice with SQS to also provision an SQS DLQ alongside your main queue. 

The mental model we use internally is three tiers:
- **Kafka** = hot path (real-time, happy path, clean)
- **SQS** = warm path (retries, managed, waiting)
- **DLQ** = cold path (manual intervention, something has gone horribly wrong)


### The Key Behavior
---

There's one rule that makes this all work, **always commit the Kafka offset, even when you fail.**

This was the biggest mental shift for the team. When a message fails, you publish it to SQS and then commit the offset anyway. You're not ignoring the failure, you're deferring it. The message isn't lost. It's sitting in SQS, waiting to be retried. Kafka moves on, your consumer stays healthy, and your partition never clogs.

### Load Shedding and Throttling
---

This pattern does more than just retry failed messages. It also naturally sheds load when things go wrong.

Here's what we discovered in practice, when our downstream dependency is struggling, the consumer fails fast on those messages, pushes them to SQS, and keeps working through the partition. We're not piling more requests onto an already struggling system, we're deferring them. The consumer stays productive, processing whatever it can handle right now.

The visibility timeout also acts as a built-in throttle. A failed message hides for N seconds before reappearing, giving the downstream system room to recover. With exponential backoff, the retry cadence slows down naturally and this all comes with no custom rate-limiting code needed.

This turned out to be one of the biggest operational wins for us. Not only did we stop losing messages, but we also stopped making bad situations worse (retry storm, self inflicted ddos).

This allows our consumer to still operate under a degraded state

## Architecture Walkthrough
---

<!-- Diagram idea: Kafka topic → Consumer → (success → commit offset + ack) | (failure → publish to SQS → commit offset) then SQS → Retry Worker → (success → delete from SQS) | (failure after max attempts → DLQ) -->

<!-- Step by step:
1. Consumer polls Kafka, gets batch of messages
2. For each message, attempt to process
3. On success: commit offset (or rely on batch commit), nothing else
4. On failure: publish the message (or just key + payload + metadata) to the SQS retry queue, then commit the Kafka offset anyway
5. A separate retry worker (could be a Lambda, a cron-based process, or a long-lived service) polls the SQS queue
6. Retry worker attempts processing. If it succeeds, delete the message from SQS.
7. If it fails, SQS's visibility timeout makes the message reappear after a delay -->

<!-- Important detail: commit the Kafka offset even on failure. This is the critical mental shift. The message isn't lost — it's in SQS. You're not ignoring the failure, you're deferring it to a system purpose-built for retries. -->

<!-- Data flow considerations:
- What exactly do you put in SQS? The full Kafka message body? Just an ID and a reference? Include metadata like original topic, partition, offset, failure reason, retry count?
- Message size limits: SQS max 256KB, so large payloads may need S3 with SQS carrying a pointer -->

## Retry Logic & Thresholds

<!-- The retry policy — how many times, how long between attempts, when to give up -->

<!-- SQS redrive policy:
- `maxReceiveCount`: the number of times a message can be received before being sent to DLQ
- How to pick this number: too low = messages die too early, too high = messages sit in limbo for too long
- Anecdote: we started with 3, moved to 5 after seeing transient DB failovers take ~2 minutes to resolve -->

<!-- Visibility timeout and backoff:
- SQS visibility timeout: how long the message stays hidden after being received
- Fixed vs exponential backoff: fixed = simple but can hammer a recovering system, exponential = gentler
- SQS doesn't natively support exponential backoff per-message (visibility timeout is queue-level). Workaround: set queue timeout to the max, and your retry worker can change the visibility timeout per message as it sees more retries.
- Mention `ChangeMessageVisibility` API for implementing custom backoff -->

<!-- When to give up: the DLQ
- After maxReceiveCount is hit, SQS automatically moves the message to the configured DLQ
- DLQ is your "manual intervention" inbox
- Set up a CloudWatch alarm on DLQ depth — if there's anything in there, someone needs to look
- DLQ messages should carry original context: what was the original Kafka message? What error did it hit on each attempt? This makes debugging actually possible -->

<!-- The human loop:
- Engineer gets paged (or checks dashboard) → sees DLQ has messages → inspects payload and error context → fixes root cause (bug, config, external dependency) → redrives messages from DLQ back to the main SQS queue -->

## Trade-offs & Gotchas

<!-- Nothing comes for free. Be honest about the costs. -->

<!-- Ordering:
- Kafka guarantees ordering within a partition. Routing through SQS breaks that — messages may be retried and re-processed out of order.
- Is that okay? For payment processing: maybe not. For notification delivery: probably fine.
- If ordering matters: mention this pattern isn't a great fit, alternatives like pausing the partition or using a state store -->

<!-- Duplicates and idempotency:
- SQS Standard queues are at-least-once. You WILL get duplicate messages.
- Your processing logic must be idempotent — use a unique message key, check a database before re-applying side effects
- Mention idempotency techniques: storing processed message IDs in DynamoDB/Redis with a TTL -->

<!-- Latency:
- The happy path is still fast (Kafka → process → done). But a failed message will take visibility_timeout * max_receive_count seconds before hitting DLQ.
- At 30s timeout + 5 retries, that's ~2.5 minutes before a message is declared dead. Adjust based on your SLA. -->

<!-- Cost:
- SQS is dirt cheap (first 1M requests free, then $0.40 per million)
- The real cost is operational complexity: you now have two queues to monitor, a retry worker to maintain, a DLQ to watch
- Worth it for the resiliency gain, but don't pretend it's zero overhead -->

<!-- When NOT to use this pattern:
- Strict ordering requirements
- Sub-second latency requirements
- Very high throughput where SQS costs become material (unlikely for most teams)
- Synchronous request-response patterns where the caller expects an immediate answer -->

## Closing

<!-- Wrap it up, bring it back to the reader -->

<!-- What we built:
- A consumer that doesn't lose data and doesn't block on failure
- Three tiers: Kafka (hot) → SQS (warm retry) → DLQ (cold, manual intervention)
- The key insight: commit the offset early, defer the retry to a system built for it -->

<!-- What this pattern gave us at work:
- Confidence: no more 2am panic about lost messages
- Operational peace: fewer on-call pages, clear escalation path through DLQ
- Observability: you know exactly when and why messages are failing -->

<!-- Broader principle:
- Design for failure, not just the happy path. Your consumers will fail — it's not a question of if, but when.
- Build the retry path into your architecture from day one, not as an afterthought.
- The real measure of a system isn't how it performs when everything works — it's how it degrades when things break. -->

<!-- Call to action:
- Look at your own consumers today. What happens when a message fails? If the answer makes you uncomfortable, consider the SQS retry pattern. -->
