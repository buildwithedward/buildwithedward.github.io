---
layout: single
title: "The Self-Improving Software Factory: Treating Coding Agents as Infrastructure"
date: 2026-11-22
categories: [today-i-learned]
redirect_from:
  - /watched-and-learned/self-improving-software-factory/
tags: [coding-agents, agentic-development, software-factory, llm-as-judge, evals, platform-engineering, ai-engineer, youtube]
excerpt: "Most teams use coding agents interactively on laptops. Zach Lloyd (Warp) argues the next step is a cloud software factory - defined in code, measured on your own data, and improved by agents themselves. Here's what that means and how the pieces fit."
header:
  teaser: https://img.youtube.com/vi/nKjPBAw_N_0/maxresdefault.jpg
---

## Introduction

Right now, the way most of us use coding agents is pretty personal: open a terminal, prompt Claude Code or Codex, review what comes back, repeat. It works, and it makes individual engineers faster. But I've been wondering what happens when a whole team does this. Who sees what the agents are doing? Who controls the spend? How do you know whether things are getting better or worse?

This talk gives a clear answer: stop thinking of coding agents as a personal tool and start treating them as **infrastructure** - a "software factory" running in the cloud, defined in code, and measured well enough that agents can improve it themselves.

---

## The Talk

**How to Build a Software Factory That Improves Itself** - Zach Lloyd, founder and CEO of Warp (previously a principal engineer at Google, where he led engineering on Google Docs). Published by the [AI Engineer](https://www.youtube.com/@aiDotEngineer) channel.

{% include video id="nKjPBAw_N_0" provider="youtube" %}

---

## Key Takeaways

<div class="notice--info" markdown="1">

- **Interactive prompting doesn't scale to a team.** It's bottlenecked on a person, hard to cost-control, and hard to govern.
- **Sandbox agents and move them to the cloud.** A compromised local agent can act as the signed-in user, and per-laptop setups can't be audited.
- **Define the factory in code.** Workflows, agent definitions, skills, MCPs, where agents run - all version-controlled, testable, and editable by agents.
- **Stay open:** any model, any harness, your own infrastructure and inference. The best models and harnesses keep changing.
- **Be measurement-first on your own data.** Record every run, score runs with LLM-as-a-judge, and build benchmarks from your real tasks instead of trusting public leaderboards.
- **Close the loop.** Agents find recurring failure patterns in scored runs and open PRs against the factory definition to fix them.

</div>

---

## From Interactive Prompting to Automated Development

Zach describes where most companies are today as the **interactive prompting** phase: engineers have Claude Code or Codex on their laptops, maybe with several sessions running, prompting agents to build features and fix bugs.

Where companies want to get to is **automated development** - using agents to get the most business value at the lowest cost, in a way that's governed and secure. Interactive agents run into three limits:

1. **Throughput** - you're bottlenecked by a person running each agent.
2. **Cost** - hard to control when everyone has their own setup.
3. **Governance** - everyone configures their own MCPs, skills, and tools, so you can't see or audit what's happening.

There's also a security problem specific to local agents: if one is compromised, it can effectively take over whatever the signed-in user on that laptop can do.

His fix for both the security and governance problems: **sandbox the agents and move them to the cloud.** The emerging product category for this is what he calls a *cloud software factory*.

---

## What a Software Factory Is

The term is overloaded, so here's his definition: every company will set up **a loop in the cloud where humans and agents do work together**. Humans put work in. Agents handle different parts of the software lifecycle - triage, working with a human on a spec, implementation, and verification using computer use.

The business's job is to define the rules for how humans and agents interact, then measure and improve the loop over time. Zach's claim is that getting this right will become the main driver of how an organization ships software.

The analogy I find helpful is a CI/CD pipeline. Nobody runs builds by hand from their laptop anymore; the pipeline is shared infrastructure, defined in config, with logs and metrics everyone can see. A software factory applies the same idea to the *writing* of the code, not just building and deploying it.

---

## Factories as Infrastructure: Open, Adaptable, Defined in Code

Warp's belief is that the factory should be thought of as infrastructure - not a product or an "AI teammate", but a stack you build on and scale. Three properties matter:

- **Open** - support different models (open-weight or frontier) and different harnesses, because the best ones keep changing.
- **Adaptable** - work with your existing infrastructure.
- **Defined in code** - a strict definition of the workflows, skills, MCPs, and where everything runs, kept in version control. That gives you the ability to test different versions of your factory, and lets coding agents update the factory itself.

In the demo, the default workflow is an orchestrator agent that decides whether a piece of work goes next to triage, spec, implementation, or code review. Each step is an agent definition: its prompt, model, MCPs, and secrets. Behind all of it sits a repo with a definition of the entire factory at any point in time - agent definitions, automations, benchmarks, where agents run, skills, webhooks.

To make that concrete, here's roughly what such a definition could look like. **This is my own illustrative sketch, not Warp's format** - the point is that everything is plain, diffable config:

```yaml
# factory.yaml - hypothetical factory definition (illustrative only)
runtime:
  sandbox: cloud-container
  inference: self-hosted-gateway        # bring your own inference

intake:                                  # work comes in through existing tools
  - slack
  - issue-tracker
  - github-pr-comments

workflow:
  orchestrator: { model: frontier-large }
  steps:
    triage:         { model: open-weight-small, skills: [repo-map] }
    spec:           { model: frontier-large, requires_human: true }
    implementation: { model: frontier-large, mcps: [github, docs-search],
                      skills: [read-related-skills-first, testing-conventions] }
    verification:   { model: frontier-large, tools: [computer-use] }
    code_review:    { model: frontier-mid }

scorers:
  - name: redundant-tests
    judge: llm
    prompt: "Did the agent write tests that duplicate existing coverage?"

benchmarks:
  - name: frontend-tasks
    source: past-runs
```

Once it's a file in a repo, an agent can propose a change to it like any other code change - which is the key to the self-improvement loop below.

Zach also stresses **AI sovereignty**: own the data going in and out, run agents on your own infrastructure and compute, provide your own inference, and keep the freedom to use any model and harness. He says this is what most larger enterprises Warp works with want.

---

## Two Kinds of Engineering

In a factory world, Zach sees engineering splitting in two:

- **Product engineering** - engineers still build features and fix bugs for customers, but they do it *through* the factory, putting work in.
- **Factory engineering** - a platform team (or engineers in general) improving the factory itself: refining skills, setting context, changing the model mix.

As engineers spend more time building the system that builds the software, the second kind matters more and more.

Importantly, product engineers don't need a new tool to put work in. On Warp's team, work enters through what people already use - Slack, the task tracker (Linear for them, Jira works the same way), and GitHub PR comments. It all lands in one central place, which is what makes the next part possible.

---

## Record Everything, Then Score It

The manager's view of the factory shows how automated it is, cycle time (DORA-style metrics), and spend. You get those views because **every run is recorded** - every agent trace, conversation, and artifact, for every engineer and every automation. Zach's point: what your coding agents are doing is the raw material for improving them.

On top of that sit **scores**, which you can think of as online evals. A set of agents reviews the runs flowing through the system and uses **LLM-as-a-judge** to decide whether each run was good. His example is a scorer that looks for agents writing redundant tests - a common failure mode for coding agents. Most runs pass, but each flagged failure is an opportunity to improve the system. You could have a person grade these, but it's much more efficient to have agents do it.

Here's a minimal, illustrative version of a scorer in Python. The `judge` function is a stand-in so the example runs on its own - in a real system you'd call an LLM with the trace and a grading prompt:

```python
import random
from collections import Counter

random.seed(42)

# Simulated agent runs - in a real factory these are stored traces
runs = [{"id": i, "task_type": random.choice(["frontend", "backend", "infra"]),
         "tests_added": random.randint(0, 6), "tests_needed": random.randint(0, 4)}
        for i in range(200)]

def judge(run):
    """Stand-in for an LLM-as-a-judge call.
    Real version: send the trace + a rubric to a model and parse a verdict."""
    redundant = run["tests_added"] - run["tests_needed"] >= 3
    return {"score": "fail" if redundant else "pass",
            "reason": "wrote redundant tests" if redundant else None}

results = [{**run, **judge(run)} for run in runs]
failures = [r for r in results if r["score"] == "fail"]

print(f"Pass rate: {1 - len(failures)/len(results):.1%}")
print("Failures by task type:", Counter(r["task_type"] for r in failures))
```

The output tells you not just *that* agents fail, but *where* - which is exactly what the next step needs.

---

## The Self-Improvement Loop

This is the core idea of the talk. Once runs are scored, **agents look across all the failures, aggregate them, find patterns, and propose changes to the factory definition** so the same failure doesn't happen again.

In the demo, one suggestion is essentially "read more relevant skills before implementing", backed by the observation that this failure happened **37 times**. The agent then proposes the fix as a **PR to the factory definition**. That's the loop closing: a problem is detected automatically, and a change that should improve results is proposed automatically, through the same code review process as everything else.

This is why defining the factory in code matters so much. If your agent setup lives in people's local configs, there's nothing for an agent to open a PR against.

---

## Benchmark on Your Own Data, Not Twitter

On any given day, someone online is claiming some model is the best, and public benchmarks like SWE-bench or Terminal-Bench add more numbers. Zach's argument is that none of this tells you how a model or harness will do on *your* workflows. You want to be **measurement-first on your own data**.

Because every past run is recorded, you can turn them into a benchmark: pick representative runs - say, a suite of front-end tasks - and **replay them against different model configurations**. The output is a cost-versus-quality chart like the ones published with public benchmarks, but built from your code and your stack.

In his example, a cheaper model got about 93% of the tasks right at a cost of $2, while a more expensive model got about 95% right at roughly three times the cost. With that data you can build a **model routing strategy** you're confident in for your actual codebase, not a generic one.

Here's an illustrative version of that routing decision. The numbers are made up apart from the 93%/$2 vs 95%/~3x comparison from the talk:

```python
# Results from replaying a benchmark suite of past tasks (illustrative)
benchmark = {
    "frontend": [
        {"model": "cheap-model",    "accuracy": 0.93, "cost_usd": 2.0},   # from the talk
        {"model": "frontier-model", "accuracy": 0.95, "cost_usd": 6.0},   # ~3x the cost
    ],
    "backend": [
        {"model": "cheap-model",    "accuracy": 0.78, "cost_usd": 2.4},
        {"model": "frontier-model", "accuracy": 0.94, "cost_usd": 7.1},
    ],
}

def route(task_type, min_accuracy=0.90):
    """Pick the cheapest model that clears the quality bar for this task type."""
    ok = [r for r in benchmark[task_type] if r["accuracy"] >= min_accuracy]
    if not ok:  # nothing clears the bar - fall back to the most accurate
        return max(benchmark[task_type], key=lambda r: r["accuracy"])["model"]
    return min(ok, key=lambda r: r["cost_usd"])["model"]

for task in benchmark:
    print(f"{task:>9} -> {route(task)}")
```

With a 90% bar, front-end tasks go to the cheap model and back-end tasks go to the frontier model - a decision you couldn't make from a public leaderboard.

---

## The Full Stack

Zach closes with the layers he expects any software factory to have, however you build it:

| Layer | What it does |
|---|---|
| **Definition in code** | The whole factory - workflows, agents, skills, MCPs, benchmarks - version-controlled at the bottom of the stack |
| **Data** | What goes in and out, all recorded and owned by you |
| **Runtime** | Where agents run - sandboxed |
| **Inference** | Flexible: any model, including your own |
| **Self-improvement** | Benchmarks and scoring that feed improvements back into the definition |
| **Orchestration** | Multi-step workflows (triage → spec → implement → verify → review) |
| **Access** | Work comes in from existing tools: Slack, issue trackers, GitHub |

---

## My Take

What struck me is how much of this is just good MLOps applied to coding agents. Record everything, define your system declaratively, evaluate on your own held-out data rather than public benchmarks, and close the loop from monitoring back to training - or here, back to the factory definition. It's the same discipline we use for production ML models, now pointed at the agents writing our code.

The part I'd adopt first, even without a full factory platform, is **benchmarking on our own past tasks**. Even a small, hand-picked set of representative tasks replayed against two or three models gives you a cost-quality picture no leaderboard can. The routing decision alone can save real money.

The part I'd be most careful with is LLM-as-a-judge scoring. It's what makes the self-improvement loop practical, but the judge's rubric is now part of your system, and a judge that's wrong in a consistent way will happily steer the factory in the wrong direction. I'd want a small human-graded sample to check the judge against before trusting it to drive PRs.

---

## Credits

All ideas, demo details, and figures in this post come from Zach Lloyd's talk *How to Build a Software Factory That Improves Itself*, published by [AI Engineer](https://www.youtube.com/watch?v=nKjPBAw_N_0). The code examples and the sample factory definition are my own illustrations, not Warp's implementation. The "My Take" section is my own perspective.
