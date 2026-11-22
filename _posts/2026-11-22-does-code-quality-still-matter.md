---
layout: single
title: "Does Code Quality Still Matter in the Age of AI? Disposable vs Durable Software"
date: 2026-11-22
categories: [today-i-learned]
tags: [code-quality, software-engineering, ai-coding, disposable-software, testing, standardization, ai-engineer, youtube]
excerpt: "\"It depends\" is every senior engineer's answer - but depends on what? Max Kanat-Alexander (Capital One) gives a real framework: code quality exists for understanding, AI is creating a flood of disposable software, and the real goal is making rigor cheap."
header:
  teaser: https://img.youtube.com/vi/bU5gDVyu_28/maxresdefault.jpg
---

## Introduction

Every week I see a new hot take: "AI writes the code now, so code quality is dead." Followed by the opposite: "AI-generated slop is going to bury us in tech debt." Both sides sound confident. Neither explains *when* quality matters and when it doesn't.

This talk is the best answer I've seen. Instead of "yes" or "no", it gives a framework: why code quality mattered in the first place, a new category of software that AI is making extremely common, and the four factors that tell you which kind you're building. And then a twist - the most useful thing you can do isn't deciding how much rigor to apply, it's making rigor cheaper.

---

## The Talk

**Does Code Quality Still Matter? It Depends on This** - Max Kanat-Alexander, Capital One. Published by the [AI Engineer](https://www.youtube.com/@aiDotEngineer) channel.

{% include video id="bU5gDVyu_28" provider="youtube" %}

---

## Key Takeaways

<div class="notice--info" markdown="1">

- **All software success comes down to two things:** understanding, and executing on that understanding. Code quality practices exist only to serve those two.
- **What matters is structure.** Readability and testability were always proxies for "does this system have an understandable structure?"
- **Every change carries two risks:** causing a problem now, and making future changes harder. Both come from understanding, and they compound.
- **AI is making disposable software extremely common.** When consequences, replacement cost, dependencies, and maintenance are all low, code quality doesn't matter.
- **Durable software isn't going away** - there will probably be more of it, because AI makes it easier to build.
- **The real lever is making rigor cheap:** standardization, tooling that just runs, easy testing, and maximum strictness.

</div>

---

## Why Code Quality Ever Mattered

Max starts by throwing out the weakest justification: "it made me feel good." That's not enough.

His foundation: everything you want to achieve with software - aligning a model, scaling a system, driving adoption - depends on just two things: **understanding**, and **executing on that understanding**. Debuggers, observability, evals, even tests all exist to improve one of those two. And, he argues, everything we've ever done in the name of code quality is for those two things only - not for correctness or anything else.

For a software engineer, two understandings matter most:

1. **What is the actual behavior of the system?** Not what the docs say - what it really does.
2. **How do I correctly and safely change that behavior?**

On a tiny system, this is trivial. It prints "hello world"; you want "hi world"; done. But real systems are enormously complex. Max shows a map of the Chromium codebase - a system with essentially infinite possible behaviors. Nobody can describe Chromium's behavior as a whole in words.

What you *can* describe is a **component**: Chromium's URL parser, for example, which figures out the different parts of a URL. That's the whole reason code has structure at all.

---

## It Was Always About Structure

Max's example is two programs that both print Hello World on Linux. One clearly *is* Hello World - it has an identity and a recognizable structure. The other is just a series of instructions with no identity you can see. At that size both are understandable. At millions or billions of lines, only one approach stays understandable.

Here's my own small Python version of the same idea:

```python
import os

# Version A: has an identity. You can tell what it is and where to change it.
def greet(name: str = "World") -> str:
    return f"Hello {name}"

print(greet())

# Version B: same output, no identity - just instructions.
os.write(1, bytes([72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100, 10]))
```

Both print `Hello World`. Now ask for "Hi World" instead. In version A it's obvious where the change goes. In version B you have to reverse-engineer what the bytes mean first. Scale that up and you have the difference between a maintainable codebase and an unmaintainable one.

His point: readability and testability always sounded like human preferences, but **what actually mattered most was that the system had an understandable structure.**

---

## The Two Risks of Every Change

Structure matters most when you want to change the system. Every modification carries two kinds of risk:

1. **You introduce a problem.** Not just a bug - anything bad: the project takes much longer than expected, it doesn't behave the way users wanted, or it fails from budget overruns.
2. **You make future changes harder** - which makes future problems more likely.

The first comes from the system being hard to understand. The second comes from making it harder to understand. Max says he can reduce every postmortem he's ever read to one of these two: a production config that was too hard to understand, someone making something impossible to understand, emergent behavior nobody could understand.

And the second feeds the first in a **vicious cycle that compounds non-linearly**. Systems tend to eventually reach a point where *every* change causes some kind of problem - think of the giant mainframe codebases with hundreds of millions of lines.

Will future models be able to handle unlimited complexity? Maybe, he says - but that's not the interesting part right now.

---

## Disposable Software: The New Majority

What *is* interesting is that AI models are creating a category of software that has always existed but is about to become vastly more common than any other kind: **disposable software**.

Imagine software where:

- the **consequences of failure** are minimal or non-existent,
- very few people and almost no other systems **depend** on it,
- the **cost of replacement** is near zero - you could rebuild it in hours or days,
- and the **cost of maintenance** is basically nothing.

In that world, **code quality doesn't matter**, because the compounding risks it protects against don't exist. Don't like your OS calculator? Have an agent make your own. Planning a party for your kids? Generate a party planner for that one party, then throw it away. If it breaks, nobody cares.

Traditionally we've built **durable** software: it lives for years, thousands or millions of people depend on it, failure can be catastrophic for the business, and it needs constant maintenance. Most engineers spend most of their time maintaining exactly these systems.

Here's the part Max says social media gets wrong: **durable software isn't going away.** Durable systems will probably remain the most important systems in the world, and there will likely be *more* of them, because AI coding tools make durable software easier to build too. Disposable software will simply be the most *numerous*.

---

## The Four Factors: Where Does Your Software Sit?

Disposable vs durable isn't black and white - it's a spectrum. At one end, none of our engineering rigor matters. At the other, everything we've learned over decades matters a lot. Where you sit is set by four factors:

| Factor | Disposable | Durable |
|---|---|---|
| Consequences of failure | Minimal | Potentially catastrophic |
| Cost of replacement | Rebuild in hours | Years of work |
| Dependencies | One person, no systems | Thousands of users, many systems |
| Cost of maintenance | Near zero | Enormous |

All low means disposable; all high means durable. Max adds an important caveat from experience: **err on the side of durable.** These factors are usually higher than you expect.

To make it concrete, here's an illustrative way you might score a project on the four factors. The scoring scale and thresholds are my own, not from the talk:

```python
FACTORS = ["consequences", "replacement_cost", "dependencies", "maintenance_cost"]

def classify(project: dict) -> str:
    """Each factor scored 1 (low) to 5 (high). Thresholds are illustrative."""
    scores = [project[f] for f in FACTORS]
    if max(scores) <= 2:
        return "disposable - skip the rigor, ship it, throw it away"
    if min(scores) >= 4:
        return "durable - full engineering rigor"
    # Mixed: following the talk's advice, err on the side of durable
    return "in between - treat as durable unless you can lower the high factors"

projects = {
    "party planner for one party":  dict(consequences=1, replacement_cost=1, dependencies=1, maintenance_cost=1),
    "payments API":                 dict(consequences=5, replacement_cost=5, dependencies=5, maintenance_cost=4),
    "internal dashboard for my team": dict(consequences=2, replacement_cost=2, dependencies=3, maintenance_cost=2),
}

for name, p in projects.items():
    print(f"{name:32} -> {classify(p)}")
```

---

## Making Software More Disposable

Here's the new move: instead of just measuring where software sits on the spectrum, you can **deliberately push it toward disposable**. Max's examples:

- **Isolated environments** - a deployment environment with access only to your own data: no customer data, no personally identifiable information.
- **Enforced independence** - no inbound API calls from other production systems into that environment, so everything deployed there is disposable by design.
- **Redundancy** - keep the old, clunky, durable system as a backup. If people want to vibe code their own more efficient front end on top, let them. When it fails (and it will), the clunky system is still there.
- **Limited scope** - a calculator for one person, a planner for one party.

He thinks this is a whole new discipline of software engineering: not improving the rigor of durable systems, but designing systems so that more things are *allowed* to be thrown away.

This connects for me with how platform teams think about sandboxes. A sandbox AWS account with no production data and no inbound connections is exactly this kind of environment - anything built there is disposable by construction.

---

## The Real Win: Make Rigor Cheap

The other axis is the one Max cares about most. You'll never estimate perfectly where something sits on the spectrum, and things change - the disposable prototype suddenly becomes something people depend on. **The cheaper rigor is, the less it matters where you are on the scale**, and the easier it is to change your mind or be wrong.

These aren't rigor practices themselves - they're practices that lower the *cost* of rigor:

### Standardization

Standards remove possibilities. If there's no PHP in your system, you never have to figure out PHP linters, PHP practices, or PHP security holes. The same goes for limiting the number of front-end and back-end frameworks, and standardizing RPC protocols. Yes, a coding agent could build tooling for every framework - but you'd be doing it every single time.

### Inward-flow tooling

Tooling that just happens. The programmer doesn't choose to run it or need to understand how it runs. It feels free, it's fast, and it's reliable - meaning not many false positives. When tooling is that cheap, you can apply it to disposable software too.

### Making testing easy

Not "how do we test better" but "how do we make it so nobody has to *think* about testing better?" Every test should be fast and deterministic without anyone being a testing expert. That takes real investment in test infrastructure.

Max's long-time dream as a test owner: every team provides **in-memory fake versions of their services** that anyone can run. That used to be a lot of work to ask for. Now a coding agent can write them - you just have to decide to do it and make them available to everyone.

Here's an illustrative example of the pattern - a service interface, a fake that runs in memory, and a test that's fast and deterministic because it uses the fake:

```python
from typing import Protocol

class PaymentService(Protocol):
    def charge(self, customer_id: str, amount_cents: int) -> str: ...
    def balance(self, customer_id: str) -> int: ...

class InMemoryPaymentService:
    """Fake that behaves like the real service, with no network or database."""
    def __init__(self):
        self._balances: dict[str, int] = {}
        self._next_id = 0

    def charge(self, customer_id: str, amount_cents: int) -> str:
        if amount_cents <= 0:
            raise ValueError("amount must be positive")
        self._balances[customer_id] = self._balances.get(customer_id, 0) + amount_cents
        self._next_id += 1
        return f"txn-{self._next_id}"

    def balance(self, customer_id: str) -> int:
        return self._balances.get(customer_id, 0)

def checkout(payments: PaymentService, customer_id: str, items: list[int]) -> str:
    return payments.charge(customer_id, sum(items))

# Fast, deterministic test - no mocks to configure, no service to spin up
fake = InMemoryPaymentService()
assert checkout(fake, "cust-1", [500, 250]) == "txn-1"
assert fake.balance("cust-1") == 750
print("checkout test passed")
```

### Maximum strictness

Turn strictness up everywhere: every lint rule at maximum, no compromises. Like standardization, it eliminates a lot of conversations. Sometimes the linter will be wrong, but as long as false positives are rare and not catastrophic, it doesn't matter. Ask anyone who has tried to switch on a linter in a million-line codebase and got a 100,000-line change - it's far easier to have it on for everyone from the start.

---

## The Answer

So, does code quality still matter?

- **For disposable software: no.**
- **For durable software: yes.**
- **And does it have to be expensive? No.**

---

## My Take

This framework fits ML work surprisingly well. A notebook to answer one question about a dataset is disposable - nobody should be writing unit tests for it. A feature pipeline feeding a production model, like the [leak-proof preprocessing pipelines](/machine-learning/feature-engineering-pipelines/) I wrote about earlier, is durable: other systems depend on it, failures are expensive, and it gets maintained for years. Most of the arguments I've seen about "do data scientists need to write clean code?" are really arguments about which of these two things you're building.

The idea I'll take into my own work is the **cheap rigor** side. Strict linting from day one, a standard project template, and in-memory fakes for services like model endpoints or feature stores cost very little up front - and they mean the prototype that unexpectedly ends up in production doesn't need a rewrite.

---

## Credits

All ideas, frameworks, and examples in this post come from Max Kanat-Alexander's talk *Does Code Quality Still Matter? It Depends on This*, published by [AI Engineer](https://www.youtube.com/watch?v=bU5gDVyu_28). The code examples are my own illustrations of the concepts. The "My Take" section is my own perspective.
