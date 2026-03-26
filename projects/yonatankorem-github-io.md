---
name: yonatankorem.github.io
status: active
repo: https://github.com/yonatankorem/yonatankorem.github.io
tags: [claude-code, agentic-workflows, github-pages]
---

# Agentic Website Development with Claude Code

## What This Project Is

This is a GitHub Pages website built with an unusual constraint: **every meaningful development decision passes through a structured agentic workflow** powered by Claude Code. The website itself is unremarkable — static content, standard tooling. The interesting part is the scaffolding around it.

Most agentic coding setups optimize for speed: let the agent run, ship fast, clean up later. This project optimizes for something different — **quality, pushback, and genuine engineering judgment**. The agents here are configured to behave like cautious senior engineers, not eager interns.

---

## The Problem This Solves

Agentic coding workflows tend to fail in predictable ways:

- The agent takes the path of least resistance instead of the right path
- It never questions whether the requirement makes sense
- It builds confidently in the wrong direction, then needs expensive correction
- It handles ambiguity by guessing silently, not by asking
- It produces code that works today but creates debt for tomorrow

The configuration in this project attacks each of these failure modes directly.

---

## The Core Laws (CLAUDE.md)

Everything in this project flows from four laws defined in `CLAUDE.md`. They are **priority-ordered** — when they appear to conflict, the lower-numbered law wins.

### Law I — It is OK to be wrong. It is not OK to keep going when wrong.

The agent is explicitly told to **stop and backtrack** when it hits unexpected resistance. Choosing a direction and being wrong is normal. Continuing down a wrong path after encountering friction is not. A wrong assumption abandoned at step 2 costs far less than one discovered at step 10.

This law exists because the default behavior of most LLMs under uncertainty is to keep going. Momentum feels like progress. This law breaks that pattern.

### Law II — Simpler is always better.

No abstractions that don't earn their place. No defensive edge-case handling for things that haven't happened yet. No over-engineering for hypothetical futures. The agent is explicitly told: **the best code is the code that doesn't need to exist**.

This law is a direct counterweight to the tendency of LLMs to add layers, helpers, and indirection as a form of apparent thoroughness.

### Law III — Question the brief before executing it.

Before building, the agent is expected to surface hidden assumptions, push back on ambiguous requirements, and make tradeoffs explicit. A clarifying question now is cheaper than a wrong deliverable later.

This is the most counter-cultural law. Most prompting practices reward responsiveness — just do the thing. This law rewards restraint and skepticism.

### Law IV — Asking for help is not failure. Silent struggle is.

When the path forward is genuinely unclear, the agent escalates — either to the user or to a specialist subagent. The preference for appearing capable should never produce a worse outcome than a brief check-in would have.

---

## The Workflow: From Idea to Shipped Code

The development workflow has four distinct stages, each with its own tool.

```
Idea → Deep Dive → PRD → Implementation Plan → Phase Execution
```

### Stage 1: Deep Dive (Stress-Testing the Idea)

Before anything is written down, the `deep-dive` skill interviews the developer **relentlessly**. It walks down every branch of the decision tree, one question at a time, providing its recommended answer for each.

The key design decisions in this skill:

**Two-phase structure.** The interview explicitly separates *what/why* (feature discovery) from *how* (implementation discovery). Jumping to implementation before requirements are clear is a common mistake. The skill signals the phase transition explicitly, giving the developer a chance to catch gaps.

**One question at a time.** This is an interview, not a questionnaire. The conversational format forces real engagement rather than bulk-answering a list.

**Codebase exploration.** If a question can be answered by looking at the code, the agent looks at the code instead of asking. Questions are reserved for genuine unknowns.

**The goal is shared understanding, not documentation.** The interview surfaces hidden assumptions on both sides — the developer's and the agent's. Once those are resolved, a PRD is drafted.

---

### Stage 2: PRD (Product Requirements Document)

The deep dive produces a PRD. This is the artifact that captures *what* is being built and *why*, with enough specificity to drive planning but without locking in implementation details prematurely.

The PRD is the handoff document between "we understand the problem" and "we know how to solve it."

---

### Stage 3: PRD to Implementation Plan

The `prd-to-plan` skill takes the PRD and produces a detailed, **phase-based implementation plan**. This is not a flat task list — it's a structured breakdown where:

- Every **phase** is independently shippable and testable
- Every **task** is a single, focused commit
- Phase boundaries are chosen deliberately: to de-risk the riskiest parts early, to enable parallel work, or to deliver incremental value
- The plan reflects the **actual state of the codebase**, not an idealized architecture

The skill does codebase exploration before writing the plan. It maps existing patterns, identifies affected modules, spots integration points, and assesses complexity. The output plan is grounded in reality.

Each phase in the plan includes:
- A clear goal and rationale for why it comes now
- Atomic tasks with file-level specificity
- Acceptance criteria and verification steps (where warranted)
- Rollback procedures for risky changes

The key design principle: **phases are hard checkpoints**. Each one is a stopping point where work is reviewed before proceeding. This is not a continuous pipeline.

---

### Stage 4: Phase Execution

The `implement-phase` skill takes the plan and implements one phase at a time. It:

1. **Verifies prerequisites** — checks that the phase isn't already complete and that dependency phases are marked done
2. **Creates a dedicated branch** — each phase gets its own branch (`phase-{n}-{slug}`)
3. **Implements tasks with individual commits** — each task becomes one focused, reviewable commit
4. **Reports completion and stops** — the agent presents a completion report and **waits for explicit human approval** before marking the phase done

This last point is critical. The agent does not mark phases complete on its own. It does not start the next phase automatically. It stops and asks. This keeps humans in the review loop at every meaningful checkpoint, while the changes are small and manageable.

---

## Why This Architecture Works

### The Laws Govern Everything

The four Core Laws in `CLAUDE.md` aren't suggestions. They apply at every stage — during the deep dive, during planning, during execution. When the agent encounters an unexpected complexity mid-implementation, Law I tells it to stop. When it's tempted to add a helpful abstraction, Law II tells it not to. When a requirement is underspecified, Law III tells it to push back before building.

The laws create a consistent decision framework that persists across the entire workflow, not just at task boundaries.

### Quality Is Upstream

Most quality problems in software originate in requirements and design, not in implementation. By front-loading the deep dive and PRD stages, the hardest questions get answered before any code is written. The implementation plan exists to execute a well-understood design, not to figure out the design while building.

### Small Surface Area, High Confidence

Atomic tasks and phase-isolated branches mean every code change is narrow and reviewable. There are no "big bang" merges. There's no state where the codebase is half-way through a feature and broken. Each phase leaves the system in a shippable state.

### The Agent Pushes Back

This is the most unusual part. The agents here are not configured to maximize helpfulness in the naive sense. They're configured to surface problems, ask hard questions, and refuse to proceed when proceeding would be wrong. This is closer to how a good senior engineer behaves than how most AI coding assistants behave.

The deep dive skill challenges the developer's assumptions. The planning skill grounds the plan in codebase reality. The execution skill verifies prerequisites and waits for approval. At every stage, the agent is working *with* the developer to produce the right outcome — not racing to complete the task.

---

## Summary

| Component | Purpose |
|---|---|
| `CLAUDE.md` Core Laws | Defines the agent's judgment framework and priority rules |
| `deep-dive` skill | Stress-tests ideas through structured interviewing before anything is built |
| `prd-to-plan` skill | Converts requirements into a grounded, phase-based execution plan |
| `implement-phase` skill | Executes one phase at a time with checkpoints and human approval gates |

The website is just content. The system around it is an experiment in building agentic workflows that behave like careful engineers — not because they're incapable of moving fast, but because they've been explicitly configured to know when not to.