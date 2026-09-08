---
title: "Asynchronous processing in production backend"
author: "Muaz Wazir"
date: "2026-08-30"
toc: true
summary: "How does an asyncrhonous processing looks like in the backend"
readTime: true
tags: ["backend", "software engineering"]
showTags: true
hideBackToTop: true
---

## Preface

There was a point in time in my study where I started dabbling with tools like Express JS and trying to learn about backend systems. I actually stumbled upon a question, in a small system, the most you would do in a single API call is just read or write a few rows in the DB. How about for big production systems like banks, Twitter (yes Twitter) and E commerce, do they rely on REST API? if yes then how did they do heavier stuff in that API, there's a strict timeout tho? But my banking app a lot of times can take more than 5 second to process? why didn't it timeout?

Yes my knowledge were very scattered and fragmented, 
- I knew that a typical REST API would need to give a response ASAP and I knew that you'll usually setup timeouts (I didn't know why back then)
- I knew a lot of processing would need to take a few second or even minutes/hours, if they're building APIs how are they handling it.
- What's this fuss about kafka and rabbitmq, events and message broker? The publisher and consumer diagram made sense but I don't understand how that's useful.


Well this was me 5 years ago and let's try to learn about this

## What is Asynchronous processing and why do I need to care about it

Synchronous processing: the caller hits an API and wait, the server processes the request, returns a response. While processing is ongoing, your request is on hold.
Asynchronous processing: the caller doesn't wait. The API does the bare minimum, replies with an `acknowledgement`, and the actual work happens elsewhere.

Syncrhonous processing means that the client will have to wait for a response to know processing has been completed/failed, Asyncrhonous on the other hand, replies to the client and basically says `got it`, we'll process in later.

Some things are synchronous by nature, and that's fine:
- Reading a profile
- Checking an account balance
- Validating a password

These are a single DB read, fast, and the caller can't move on without the answer. Making them async is just complexity for no reason.

Some things are heavy and nobody is waiting on them:
- Resizing an uploaded video
- Generating a statement PDF
- Sending 10,000 emails

Cram these into a single request and your API hangs. Keep in mind, there are timeouts everywhere, load balancers, proxies, the client itself. Something gives up before processing completes.

Now, don't confuse async with `slow batch job`. Async doesn't mean the work itself is slow or tolerant to failures, it means the caller isn't waiting on it. Some async work is time-sensitive. a user places an order, a fraud check kicks off immediately, but the user isn't stuck staring at a spinner. The work needs to happen *now*, just not in their face.

So split it. The API does the light part inline (validate the request, save the intent, return "accepted") and hands the rest to a background worker to process later — whether that's in five milliseconds or five hours.

That's exactly why your banking app doesn't time out on a transfer. It gets a quick "accepted", while the actual moving of money happens in the background.

And that background worker? That's where queues and message brokers come in. Next section.

## Syncrhonous vs Asyncrhonous

// TODO: we'll come up with a dummy example of what happens when we try to process heavy workfload as sync and async

// TODO: write the diagrams to illustrate the components involved

## Demo Project time

// TODO: write a mini fun project, monolith app that could startup an API server or a worker

// TODO: API publish message to kafka, worker consumes it
// screenshot the codebase and link the github repo
