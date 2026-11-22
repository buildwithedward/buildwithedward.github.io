---
layout: single
title: "Forward Deployed Engineering 101: Selling Outcomes, Not Software"
date: 2026-11-22
categories: [today-i-learned]
redirect_from:
  - /watched-and-learned/forward-deployed-engineering-101/
tags: [forward-deployed-engineering, fde, go-to-market, palantir, platform-engineering, ai-engineer, youtube]
excerpt: "Why Palantir sends engineers to sit with customers, the one situation where that model actually makes sense, and why agentic AI might put a lot more of us in that situation. Notes from Kevin Bai's talk at AI Engineer."
header:
  teaser: https://img.youtube.com/vi/KwhgfwOSToQ/maxresdefault.jpg
---

## Introduction

"Forward Deployed Engineer" is showing up in a lot of job descriptions lately, especially at AI companies. I've done enough customer-facing ML work to know the shape of the job - sitting with a client, understanding their mess, building something that actually moves their numbers - but I'd never seen a clear explanation of *why* a company would choose to sell this way, or when it's a bad idea.

This talk is a short, sharp explanation of exactly that. It walks through where the FDE role came from, the specific business situation that makes it necessary, the one ingredient without which it falls apart, and why AI is making the model relevant to far more companies than before.

---

## The Talk

**Forward Deployed Engineering 101** - Kevin Bai, Member of Technical Staff on the Applied AI team at Anthropic. Previously the first FDE hire at Rippling (where the team grew to around 25 in a year) and before that at Palantir. Published by the [AI Engineer](https://www.youtube.com/@aiDotEngineer) channel.

{% include video id="KwhgfwOSToQ" provider="youtube" %}

---

## Key Takeaways

<div class="notice--info" markdown="1">

- **FDE = selling an outcome.** The customer isn't buying software or someone's hours - they're buying the business result that engineers build for them on top of a platform.
- **You only need FDE in one quadrant:** selling a *technically complex* product to a *non-technical* buyer.
- **FDE is a design partnership at enterprise scale.** The same close collaboration startups use to find product-market fit, kept going as a permanent go-to-market motion.
- **No platform, no FDE.** If engineers build every solution from scratch, you have a dev shop, and maintenance costs will eat you alive.
- **Ask "do I *need* this?", not "do I want it?"** Then ask whether you have (or will build) a platform.
- **Agentic AI expands the quadrant.** When every product is customizable, customers increasingly don't know what you actually do - and leaving implementation to them gets risky.

</div>

---

## The Problem With Selling a Platform

The story starts with Palantir's product, Foundry. Foundry lets an organization centralize its data and build an *ontology* on top of it - turning "table one, table two, table three" into proper nouns like a single source-of-truth table for *warehouses*. On top of that, companies build applications.

The trouble is how that sounds to an industry leader: "Cool, you organized my data. What does that do for my business?" Someone in consumer goods cares about shelf placement and sales throughput. How the data is organized is, rightly, an implementation detail to them.

And there's a second, hidden cost. An app-building platform only succeeds if customers can use it well. So the customer pays for the platform *and* pays to train their people on it - and only then can they build anything valuable. Kevin's verdict: that is a terrible way to do business.

Palantir's answer was to stop choosing between selling products and selling services, and sell both as one thing: **an outcome**. Smart engineers go to the customer, learn how the business works, build the solution on the platform, and the customer gets the result.

Think of it like the waiter at a fine dining restaurant (Kevin's analogy). You don't walk into the kitchen and cook from the restaurant's ingredients yourself. Someone attentive figures out what you actually want and makes sure it arrives.

---

## The 2×2: When You Actually Need FDE

The most useful part of the talk is a simple grid: *what you sell* against *who buys it*.

| | **Technical buyer** | **Non-technical buyer** |
|---|---|---|
| **Complex, technical product** | Developer tools (GitHub, Datadog). Buyers are CTOs/CIOs, users are engineers who can absorb the complexity. | **FDE territory.** Palantir selling Foundry to a Fortune 500 oil & gas company. |
| **Configurable, less technical product** | - | Tools like Rippling, Jira, Slack. They can be complex, but they're configured, not developed on. |

Only the top-right corner needs forward deployed engineers. Big tech companies and AI labs have engineers who can build whatever apps they need, so a platform like Foundry isn't that interesting to them. A Fortune 500 company in oil and gas is different: as Kevin puts it, their pipelines aren't data pipelines. Rather than hoping they hire, train, and keep the right engineers, you loan them yours - people they don't have to recruit, manage, or retain, who already know the platform inside out.

Does it work? Kevin's numbers, measuring public SaaS companies' average contract value (ACV) across the Fortune 500: Palantir at around $4M (as of when he last checked), ServiceNow next at about $1.2M, Workday at roughly $600K, and no other public SaaS company even reaching $500K.

---

## FDE Is a Design Partnership That Never Ends

If you've worked at an early-stage B2B startup, you've seen a design partnership. Neither you nor the customer knows exactly what the product is yet, so you work very closely together: they give you context on their problem, you put your time, engineering, and technology into solving it. That's how most B2B startups find product-market fit.

Palantir's core assertion was: who says design partnerships are only for the early stages? Why not run that same motion **at enterprise scale**?

---

## The Non-Negotiable: A Platform

The obvious objection is maintenance. Build something custom for every customer and you end up herding cats - dozens of repos, inconsistent code, and engineers who quit rather than learn 55 codebases.

Kevin agrees completely. If every FDE builds from scratch, *you don't have an FDE function - you have a dev shop*. Nothing wrong with dev shops; they can be very profitable. But they're a different business.

What makes FDE work is that engineers **never write software from scratch**. They assemble an existing set of platform primitives into a workflow or application that's valuable to that particular customer. Without that shared base, you're reinventing the wheel each time, and the maintenance costs will wreck your P&L - assuming your engineers don't leave first. Even with a solid platform, he stresses, the maintenance burden is real.

---

## Two Questions Before You Build an FDE Team

Kevin's advice for anyone considering the model comes down to two questions:

1. **Do I *need* an FDE function?** Not want - need. It's easy to want whatever is in fashion. Do you have a real case where you must sell something technically complicated to a non-technical buyer? If not, there are better options: a developer relations / developer engagement team for a technical go-to-market, or a sales-led motion for more traditional SaaS.
2. **Do I have a platform - or am I willing to invest in building one?** If FDEs aren't building on shared primitives, you're in for a very bad time.

---

## Why AI Changes the Picture

Palantir started this motion back around 2004-2005. So why is everyone talking about FDEs now?

Kevin's personal hypothesis isn't that the industry suddenly realized Palantir was right. It's that **the software business itself has changed**. AI has made it very easy to write code and to build sophisticated, customizable software for customers - think of all the "agent for insurance" or "agent for legal" startups. Nearly every platform is now agentic, which means nearly every platform is customizable.

And that pushes a lot of companies into the FDE quadrant: your customers increasingly have no idea what your product actually does. If success depends on their ability to implement it themselves, selling upmarket or expanding into new verticals gets hard.

---

## My Take

This framing clicked for me because it matches what I've seen on the ML side. The projects that went well were rarely the ones where we handed over a model and an API. They were the ones where someone sat with the business team, understood the metric they actually cared about, and shaped the solution around it - like the explainable model I built for [pharma sales call optimization](/hcp-call-optimization/), where "no black boxes" was a business requirement before it was a technical one.

The platform point is the one I'd underline. In AI work it's tempting to build a fresh RAG pipeline or agent per client because it's now cheap to write the code. But cheap to write isn't cheap to maintain. The question I'd ask of any customer-facing AI team: what are your shared primitives - your eval harness, your retrieval layer, your deployment template on AWS - and how much of each customer engagement is just configuration on top of them?

---

## Credits

All ideas, frameworks, and figures in this post come from Kevin Bai's talk *Forward Deployed Engineering 101*, published by [AI Engineer](https://www.youtube.com/watch?v=KwhgfwOSToQ). Any mistakes in summarizing them are mine. The "My Take" section is my own perspective.
