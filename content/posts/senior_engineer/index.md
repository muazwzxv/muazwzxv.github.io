---
title: "Being a senior engineer"
author: "Muaz Wazir"
date: "2026-08-30"
toc: true
summary: "My experience being a senior engineer"
readTime: true
tags: ["career", "software engineering"]
showTags: true
hideBackToTop: true
---

Note: I'll refer to Software Engineers as Engineer in this blog post as this is the convention recognised world wide. Take your Engineer and non Engineer debate elsewhere.

I'm currently employed as a Senior Software Engineer at a renowned Digital Bank in Malaysia. What I hoped for this article is that it'll be able to articulate and describe the experience, expectations and the day to day of being a Senior Engineer as compared to a Junior/Mid level Engineer. I'm still consider a Junior Senior whereby I haven't been holding the title and responsibility for too long so there's a lot of room for improvement on my side, however it doesn't change the expectations and the work I've been spearheading.

## The type of problems

As a mid level engineer, the type of problems that was assigned to me are often very well defined problems and a clear path to solutions. It'd look something like this

```
Problem: 

We have to enhance our underwriting systems to use a new data source 
as part of the portfolio evaluation process for a customer's loan

Solution: 

Build the integration point with the new data source and include the data point as part of the decisioning engine.
```

Take some time to digest, what this problem statement entails is that I basically know that I need to hook up a new integration point (API, Kafka topic, Webhooks) to capture this new data source and pass the new data to my decision engine system. Well defined problem, clear path to solutions.

From the example given, it's very refined and clear because a Senior Engineer probably have gone through the requirement and figure out the clear path to the solution and the solution is clear enough to be hand off to a mid/junior engineer for execution.

From the Senior's perspective, this was probably how the problem looks like when they receive it.

```
Problem: 

We need a way to review customers' portfolios to evaluate all the customers in the bank with 
their credit exposure and credit history to decide whether they are worthy of higher exposures 
or to reduce their exposure to manage risk

Ambiguity for the problem
- Should we build a a new portfolio service or reuse any service we had?
- What data point should we leverage for the review? and do we have the integration setup?
- How frequent will we run these reviews
- How would we increase or reduce their credit exposures? is there a maximum and minimum range for credit exposure allowed after review?
- What's product expectation on triggering the review?
- What's business and product's expectation on the output of these review? do we need to store them for downstream use? 
- Do we run these reviews for all customers all at once or only for selective customers at a time? How do we choose? 
- What's the non functional requirement here? (expected load, P95, uptime)

Note that you have to consider existing mechanism and ensure no regression to the system that's already in production.
```

A lot to take in right? 

One of the expected quality of a Senior Engineer is the ability to break down problems into actionable pieces, I've mentioned about it here [link](). 

A Senior Engineer would need to 
- come up with the tech design 
- consider trade offs
- ensure other engineers and stakeholders are aligned
- raise questions and close gaps in details
- potentially start communicating with other teams if there's dependency or alignment needed
- plan the rollout strategy

## Shifting nature of work

As a junior/mid level engineer, it's expected for you to spend most of your time writing code and actively building. Their programming skills, ability to write maintainable code, having the maturity and foresight to produce quality and maintainable output, execution should be the focus in such role, you're expected to be able to execute and deliver the build with the highest quality possible, this includes
- writing code
- unit tests
- E2E testing
- load test when necessary

Of course, some mid level engineer are closer to the senior role than others, perhaps they can also pick up scopes that are expected of seniors. Having the ability to do that proves that said engineer is being groomed to level up.

For seniors, your role has shifted into a completely different kind of problem solving. It’s less about how fast you can write the code, and more about ensuring the right system gets built in the first place. You spend your time aggressively clearing out vague requirements, talking to product managers to define edge cases, and managing the consequences of technical trade offs. The goal is rarely to find the `perfect` theoretical architecture anymore. Instead, your job is to find the most pragmatic, manageable solution that fits the reality of the business and won't blow up in everyone's face six months down the line.

## Technical competencies as a Senior

Another noteworthy point is that, stepping into a senior role is not an excuse to let your core programming skills slip. You still need to be exceptionally sharp at writing code. Instead of just writing your own features, your technical excellence becomes a tool to elevate the rest of the team. You need that deep, hands on experience to conduct rigorous code reviews, spot structural flaws early, and mentor other engineers through complex implementations. If you aren't writing and reading code at a high level yourself, you lose the ability to effectively guide others.

The defining difference ultimately comes down to scope. Mid level and junior engineers naturally operate with tunnel vision on their immediate deliverables, getting their specific feature across the finish line. A senior engineer however, by contrast, must evaluate the broader ecosystem. You need the technical maturity to evaluate long term architectural trade offs against immediate business goals. 

This means knowing when to reject shiny new technologies and architecture when a boring, battle tested tool will be able to deliver, and having the ability to ruthlessly cut scope creep before it morphs into architectural debt. The aim here isn’t just to build the system right, it’s to ensure the team isn't building things they don't actually need.


## Production battle scars

The jump to senior is also about what you've seen break in production. When you're a junior or mid level engineer, it's pretty normal to focus on the happy path and just getting the feature to work. But as a senior, you've been around long enough to see things go sideways. You've probably dealt with 
- database replication lag
- a poison pill message breaking your Kafka consumers
- retry storms
- race conditions
- betrayed by your rate limiter
- weird timeout bugs
- idempotency issues

Nobody expects a mid level engineer to know every single failure modes. But a senior is expected to catch these traps before the code even gets merged. Because you’ve spent hours debugging broken systems, you also get way better at using your tools. You stop treating logs, traces, and spans as an afterthought because you know you'll need them during an incident. You also start caring a lot more about database fundamentals like indexes, partitions, and schema design because you know exactly what happens to a poorly designed table under heavy load.

## Fin

I’ve officially had the Senior title for about seven months now. What I've realized is that becoming a senior isn't about knowing everything; it's about knowing how to navigate the unknown. There is still a massive amount of room for me to grow, and the content of this article is just a snapshot of my current experience, heavily shaped by the incredible leads and engineers I work with. The imposter syndrome might still creep in from time to time, but at the end of the day, our job is just to clear the fog, write resilient code, and follow the boy scout rule where we leave the system better than we found it.
