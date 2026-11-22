---
layout: single
title: "From Microservices to Agents: Designing for Decisions, Not Just Services"
date: 2026-11-22
categories: [today-i-learned]
tags: [agentic-ai, distributed-systems, microservices, observability, reliability, architecture, infoq]
excerpt: "Microservices taught us to decompose functionality. Agentic AI asks us to decompose decisions - and that changes how systems fail, how we observe them, and how we keep them reliable. Notes on Mallika Rao's InfoQ article."
---

## Introduction

I've spent a good part of the last year moving between two worlds: deploying microservices on AWS ECS, and building LLM-based systems that pick their own tools and steps at runtime. On paper they look similar - lots of components, network calls, orchestration. In practice they break in very different ways, and I didn't have a clean way to explain why.

This article gave me the vocabulary. Its central idea is simple: **microservices decomposed functionality; agents decompose decisions**. Once you see it that way, it becomes clear why the reliability playbook from microservices is necessary but no longer enough.

---

## The Source

**From Microservices to Agents: the Next Evolution of Distributed Systems** - Mallika Rao, published on [InfoQ](https://www.infoq.com/articles/distributed-systems-microservices-agentic-ai) on October 19, 2026, as part of InfoQ's *Agentic AI Architecture* series. The author grounds the article in experience with search, recommendation, and serving infrastructure.

*This post summarizes the article's main ideas in my own words and adds my own examples. Read the original for the full argument and its diagrams.*

---

## Key Takeaways

<div class="notice--info" markdown="1">

- **The unit of design is shifting** from orchestrating services to orchestrating decisions made at runtime.
- **Execution paths are no longer fixed.** Agents choose tools, data sources, and strategies based on context, so two requests can take completely different paths.
- **Decision quality is a reliability metric**, alongside latency, availability, and throughput.
- **The scariest failures are silent:** infrastructure looks healthy while the quality of reasoning quietly drops.
- **Production agentic systems need guardrails:** bounded autonomy, evaluation loops, and deterministic fallbacks.

</div>

---

## Service Decomposition vs Decision Decomposition

Microservices solved a specific problem: how to break up a monolith while keeping the system reliable. You split by business capability, give each service a bounded context and an API, and get independent deployments, clearer ownership, and better scaling. But the execution is still **deterministic** - a given request follows a path someone designed in advance.

Agentic systems keep the distribution but change what's being distributed. Instead of fixed call graphs, components decide at runtime which tools to call, which data sources to use, and which strategy to follow. The article describes this as reasoning spread across components in a dynamic decision loop. In a search or recommendation system, that might mean choosing different candidate sources depending on context, adjusting ranking on the fly, and re-evaluating based on intermediate results.

The trade-off is the theme of the whole article: you gain flexibility and lose predictability. Tracing and debugging get much harder.

An analogy that helped me: a microservice architecture is like a train network. Routes are fixed, so you can publish a timetable and know exactly where delays will show up. An agentic system is more like a taxi fleet - each driver chooses a route based on traffic, the passenger, and what they saw a minute ago. Usually that's faster and more flexible. But "why did this trip take 40 minutes?" is a much harder question to answer.

---

## Three Things That Change: Control Flow, State, Failure

The article frames the difference along three dimensions.

**Control flow.** Execution paths are decided at runtime, so different requests can follow entirely different paths. You can't draw one call graph and call it "the architecture".

**State.** Agents carry an evolving execution context: conversation history, plans, retrieved documents, ranking signals, tool outputs, and evaluation results. The article points to remedies like shared context stores, memory scoped to a single execution, and structured reasoning artifacts. In long chains, keeping this state consistent starts to look like a classic distributed state coordination problem.

**Failure semantics.** A traditional failure is binary - the call succeeded or it threw an error. An agentic failure is usually a matter of degree: flawed reasoning, a poor tool choice, stale context, or an incomplete reading of the request. Nothing crashes; the answer is just worse.

---

## Four Patterns That Keep Showing Up

The author is careful to call these recurring structures rather than formal standards.

1. **Separate planning from execution.** A planner decides the strategy; the execution layer does deterministic work like retrieval, transformation, and ranking. This keeps the non-determinism in one place and makes the rest traceable.
2. **Specialized agents.** Separate agents for retrieval, ranking, evaluation, and orchestration - essentially microservices for decisions. The cost is familiar: coordination overhead, variable latency, and failures that spread from one agent to the next.
3. **Evaluation inside the execution loop.** The system judges its own intermediate output and decides whether to refine, retry, or adjust. This needs hard controls - latency budgets, iteration limits, and confidence thresholds - or it turns into runaway execution.
4. **Tool abstraction layers.** Standardized interfaces for finding and calling tools, which over time grow into a platform of capabilities.

The thread running through all four: complexity doesn't disappear, it moves **from service orchestration to decision orchestration**.

---

## Failure Modes You Won't See on a Dashboard

This was the most useful section for me. The article lists five failure modes:

| Failure mode | What happens | Why it's hard to catch |
|---|---|---|
| **Cascading decision degradation** | An early weak choice (e.g. poor candidate sources) compounds through later steps | Infrastructure is perfectly healthy |
| **Retry amplification** | Evaluation loops retry based on subjective quality judgments | Retries are hard to bound; they add latency and backend load in unpredictable patterns |
| **Silent quality degradation** | Quality drops while latency and error rates stay normal | Infrastructure monitoring sees nothing wrong |
| **Coordination overhead** | More agents means more latency variability and more partial failures | Each agent looks fine in isolation |
| **State drift** | Context becomes inconsistent across steps | It shows up as drifting reasoning, not as inconsistent data |

The common thread: **traditional observability tells you the system is up, not that it's right.** The article's answer is monitoring that is aware of evaluation results, not just infrastructure metrics.

---

## Observability and Reliability, Rebuilt for Decisions

Latency, throughput, and error rate are still needed - the article calls them necessary but insufficient. On top of them, it proposes:

- **Decision-level tracing** - record tool selections, reasoning steps, intermediate outputs, and evaluation outcomes, not just spans and status codes.
- **Behavioral monitoring** - watch tool usage, how often each strategy is chosen, fallback rates, and retry behavior. A sudden jump in retries or a shift toward one strategy can be an early warning, often before engagement or latency metrics move.
- **Evaluation as runtime infrastructure** - but with constrained loops.
- **Bounded autonomy** - latency budgets, retry limits, and limits on reasoning depth.
- **Deterministic fallbacks** - when confidence is low, drop to a predictable path. The article calls hybrid architectures a pragmatic balance.
- **Progressive rollout** - experimentation, shadow execution, and controlled deployment.

The mindset it asks for: design systems that **degrade gracefully in decision quality** while keeping the user experience predictable.

### What bounded autonomy looks like in code

The article has no code, so here's my own illustration of several of these ideas together: an agent loop with an evaluation step, a retry limit, a latency budget, a confidence threshold, a deterministic fallback, and a decision-level trace. The "agent" and "evaluator" are simulated so it runs on its own:

```python
import random
import time

random.seed(7)

MAX_ATTEMPTS = 3          # retry limit
LATENCY_BUDGET_S = 0.5    # latency budget for the whole request
MIN_CONFIDENCE = 0.75     # confidence threshold

STRATEGIES = ["semantic_search", "keyword_search", "popular_items"]

def agent_step(query, attempt):
    """Stand-in for an LLM choosing a strategy and producing a result."""
    strategy = random.choice(STRATEGIES)
    time.sleep(random.uniform(0.05, 0.2))          # simulated model + tool latency
    return {"strategy": strategy, "result": f"{strategy} results for '{query}'"}

def evaluate(output):
    """Stand-in for an evaluator (LLM-as-judge or a learned quality model)."""
    return round(random.uniform(0.4, 0.95), 2)

def deterministic_fallback(query):
    """Predictable path: e.g. a plain keyword search or a cached ranking."""
    return f"keyword_search results for '{query}' (fallback)"

def answer(query):
    trace = []                                     # decision-level trace
    start = time.monotonic()
    for attempt in range(1, MAX_ATTEMPTS + 1):
        if time.monotonic() - start > LATENCY_BUDGET_S:
            trace.append({"event": "latency_budget_exceeded", "attempt": attempt})
            break
        out = agent_step(query, attempt)
        score = evaluate(out)
        trace.append({"attempt": attempt, "strategy": out["strategy"], "confidence": score})
        if score >= MIN_CONFIDENCE:
            return out["result"], trace
    trace.append({"event": "fallback"})
    return deterministic_fallback(query), trace

for q in ["wireless headphones", "running shoes", "standing desk"]:
    result, trace = answer(q)
    print(result)
    for t in trace:
        print("   ", t)
```

Every request ends within a known number of attempts and a known time budget, always returns *something*, and leaves a trace of which strategy was tried and how confident the evaluator was. That trace is the raw material for behavioral monitoring.

### Turning traces into an early warning

Here's a second illustration: watching **fallback rate** and **strategy mix** across windows of requests, so a shift shows up before any latency or error alert would fire. The data is simulated, with a quality regression injected in the last window:

```python
import random
from collections import Counter

random.seed(42)

def simulate_window(n, degraded=False):
    """Simulated per-request summaries from decision traces."""
    rows = []
    for _ in range(n):
        if degraded:   # e.g. a prompt change makes the agent favour one strategy and fail more
            strategy = random.choices(["semantic_search", "keyword_search", "popular_items"], [0.2, 0.2, 0.6])[0]
            fallback = random.random() < 0.18
        else:
            strategy = random.choices(["semantic_search", "keyword_search", "popular_items"], [0.6, 0.3, 0.1])[0]
            fallback = random.random() < 0.05
        rows.append({"strategy": strategy, "fallback": fallback})
    return rows

windows = [simulate_window(500) for _ in range(3)] + [simulate_window(500, degraded=True)]
baseline_fallback = sum(r["fallback"] for w in windows[:3] for r in w) / (3 * 500)

for i, w in enumerate(windows, 1):
    fallback_rate = sum(r["fallback"] for r in w) / len(w)
    mix = Counter(r["strategy"] for r in w)
    top, top_n = mix.most_common(1)[0]
    alert = "  <-- ALERT" if fallback_rate > 2 * baseline_fallback else ""
    print(f"window {i}: fallback={fallback_rate:.1%}  top strategy={top} ({top_n/len(w):.0%}){alert}")
```

Latency and error rate would look identical across all four windows. The fallback rate and strategy mix are what reveal that something changed.

---

## The Road Ahead

The article closes by framing agents as a natural evolution: microservices decomposed functionality, agents decompose decision-making. That brings adaptability, personalization, and dynamic execution - and new problems in observability, reliability, and design. The principles that let microservices scale still apply: clear boundaries, observability, reliability engineering, and platform thinking. The author's closing note is that we're still early in this transition.

---

## My Take

What I like about this framing is that it stops treating LLM systems as something exotic. They're distributed systems, and most of what we learned from running microservices still applies - timeouts, retry budgets, circuit breakers, gradual rollouts. The new part is that "correct" is no longer binary, so **quality has to become a monitored signal** the same way latency is.

That matches what I've seen deploying AI applications on ECS. CloudWatch can tell you the container is healthy and p95 latency is fine; it can't tell you the model started giving worse answers after a prompt change. The two things I'd add to any agentic system from day one are the ones in the code above: a hard cap on attempts and time with a deterministic fallback, and a decision trace you can aggregate into fallback rates and strategy mix. Neither is expensive, and both turn silent degradation into something you can actually alert on.

---

## Credits

The ideas, patterns, and failure modes in this post come from Mallika Rao's article *[From Microservices to Agents: the Next Evolution of Distributed Systems](https://www.infoq.com/articles/distributed-systems-microservices-agentic-ai)* on InfoQ. The analogies and code examples are my own illustrations, and the "My Take" section is my own perspective.
