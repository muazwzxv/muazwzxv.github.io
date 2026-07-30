---
title: "For Aspiring Engineers"
author: "Muaz Wazir"
date: "2026-07-30"
toc: true
summary: "Advice I'd give to my junior self"
readTime: true
tags: ["career", "software engineering", "growth"]
showTags: true
hideBackToTop: true
---

My hope for this article is that if you're an aspiring engineer or going through internship, or Uni, are able to take some lessions with you.

# Junior Engineer: The Optimistic Beginner

Being a Junior Engineer is probably one of the most exciting times in your career. You're fresh out of uni and ready to utilise your data structures, algorithms, SQL queries, and whatever else you learnt in Uni.

## What you learn in Uni VS industry

First thing to keep in mind is that, industry practice are very different than academia. Uni optimizes for clarity and proof, you normalise schemas, enforce invariants, hand-roll the data structure, and write tests to show the code works. Industry optimizes for time-to-change, blast radius, and the next engineer being able to read it at 2am. They're not opposites, they trade different things.

In Uni you went through a data structure course, handrolled 5-6 of them in a semester, and scored an A. At work, your senior engineer is picking up new knowledge on a Thursday because of a production issue in a system that was built 12 years ago. Sounds daunting, but the job is the same: figure out which constraint matters in *this* room.

A few examples to make it concrete.

| Uni                                                         | Industry                                                                                                                       |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Normalise schemas to 3NF, FKs enforce referential integrity | Denormalise hot paths to avoid joins, FKs get dropped on large tables to dodge row-locking contention, not because FKs are bad |
| Write tests to prove the code works                         | Tests are a safety net for the *next* change, cover the bugs that already bit you in prod, not the happy path                  |
| Hand-roll a data structure to understand it                 | Use the stdlib one. Re-implement when profiling says so, not before                                                            |
| There's one correct answer                                  | Several workable answers, pick the one that's cheapest to delete in six months                                                 |

## People's expectations on you

A Junior Engineer joining a team will always be a fresh breath of air, they're usually very energetic and eager to learn, be one of those engineers.

- Be teachable and ask a lot of questions
- It's okay if you can't solve it, but show me your thought process and run me through the things you've tried
- Internalise and apply the first principle thinking [read](https://fs.blog/first-principles/)

## Thinking like an engineer

### First Principle Thinking

The idea of first principle thinking stripping down a problem right into it's core fundamental and build your solutions from there. 

Example, given an API that is slowing down after 6 months, P95 latency went from 150ms to 1200ms 

Reasoning by analogy: 
- It's slow. 
- Usually, people add a Redis cache to speed things up. 
- Let's spin up Redis

First principles thinking: 
- Why is it slow? The database query takes 2 seconds. 
- Why does it take 2 seconds? Because it's doing a full table scan on 5 million rows. 
- Why is the database doing a full table scan, because there's no indexing that our query pattern can leverage 
- Can we just add a B-tree index? Yes, let's do that

Notice the vast difference in these 2 ways of thinking, by probing it until the fundamental core of the issue, we managed to figure out the root cause and figure out the ideal path to solution. Real life engineering will get a lot more complicated than this, but with the right way of thinking and doing it together with other brilliant engineers, you can figure out almost anything

### Validate your assumptions

Unvalidated assumptions are the silent villains of software engineering. I see a lot of juniors fall into the trap of building entire features on top of a *hunch* about how a certain system behaves, only to be wrong and have to go back to the drawing board.

Ruling out assumptions isn't an innate talent. As you spend more time in the industry, you'll realize it is a muscle you have to actively train. An assumption is any time your brain silently whispers:

1. I'm sure the other team has the API I need to retrieve the customer data.
2. This third-party API will always return the status.
3. I thought that API already had rate limiting configured.

I mean, if your assumptions are right, there's usually no problem.

But from experience, a lot of people are wrong most of the time (if they didn't validate their assumptions).

- They don't have the API apparently, and I didn't scope it in our planning, so the project timeline might stretch :(
- Apparently there's an edge case in the API where it responds with an empty `status` but populates the `reason` field instead. A code change was required.
- You deployed to production and you were wrong, the extra traffic from your system degraded the service in production.

My biggest takeaway: making assumptions has to be followed up with the muscle to validate them as soon as possible. Regardless of whether it came from you or someone else on your team, any assumption that hasn't been validated shouldn't be in your planning. I'd repeat it as loud as I need it, never make decisions over an assumption.

## Mistakes are expected, leverage that

Being a junior is the best time to make mistakes. Well, I'm not saying to go out of your way and intentionally make mistakes. But as a junior you'll have more leeway to screw up, and the expectation is that you bounce back and learn from it.

Personally, as a junior, I once modified the response payload of an API in a way that broke the API contract and took our app down. The mistake was called out, and I sat down with my mentor to figure out why that was the wrong approach and what I should've looked out for to support backwards compatibility [read](https://zuplo.com/learning-center/api-versioning-backward-compatibility-best-practices).

Making mistakes is one of the best ways to learn, and being a junior gives you plenty of room for trial and error. In an established company where the development process has matured, it's usually safe to let these rascals loose, they probably have good safety nets to catch the more catastrophic errors.

## Take charge of your career

In Software Engineering, or tech generally, there are a lot of archetypes, roles, and domains. Site Reliability Engineer (SRE), Software Engineer (SWE), Mobile Engineers (iOS, Android, React Native/Flutter), Data Engineer (DE), Quant Engineer, Systems Engineer (low-level), to name a few.

Early on in your career, you're not expected to 100% commit to a niche. However, it's your responsibility to figure out what you want to pursue. Sure, you can try multiple roles and see which one sticks; you can self-study a separate role while working as another. Whatever you do, take control of your career direction.

My takeaway: take charge of your career, whether your goal is to achieve FIRE (Financial Independence, Retire Early), climb your way to CTO, or retire and become a farmer.
