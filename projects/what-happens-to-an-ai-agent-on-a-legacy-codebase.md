---
name: What Happens to an AI Agent on a Legacy Codebase?
status: active
tags: [claude-code, agentic-workflows, autonomous-agents, experiment]
---

# What Happens to an AI Agent on a Legacy Codebase?

## The Question

*Does complexity eventually outpace autonomy?*

There's a lot of confident talk about AI doing most of the work of a software engineer. The demos are impressive. The benchmarks keep climbing. But there's a version of this story that doesn't get told as often: what happens to an AI-driven development system over time, as the codebase it maintains accumulates complexity, edge cases, and interdependencies?

Software projects don't slow down because they age — they slow down because complexity accumulates faster than quality is maintained. Codebases become harder to touch not because of time, but because of the decisions made inside them. Is that a human limitation — or an honest response to real complexity that any intelligent system would eventually exhibit?

I built this pipeline to find out.

---

## The Experiment

I've spent time on both legacy and green-field projects, and the pattern is the same in both directions: systems continuously grow more complex. More edge cases, more features, more branching points. 

Green-field projects start fast and slow down as complexity accumulates. Legacy projects are just further along the same curve. The question I kept coming back to is whether an AI agent experiences the same degradation — or whether that slowdown is a specifically human problem.

Working this past year with AI agents on real code, I stopped assuming they were immune. No matter how many guardrails you apply, they miss things. Important things. Running the same code-review agent twice on the same PR produces different results — one pass catches a critical bug the other ignores entirely. If agents are this sensitive to context even on a stable codebase, what happens as the codebase becomes harder to reason about?

The pipeline is the experiment. The design is straightforward: build an autonomous development pipeline, feed it tasks over time, and track how its decisions shift.

The pipeline accepts user submissions — feature ideas, bug reports, improvement suggestions — and routes each one through two stages of agent evaluation. The first stage acts like a product manager reviewing feedback: is this useful, is it clear, do we understand what's being asked? Submissions are approved, rejected, or flagged for human review.  
Approved tasks move to a second stage that mirrors a grooming session: are the requirements fully understood, how much of the system does this touch? Simple tasks are implemented autonomously within a full git workflow — branch, code, tests, lint, code review, merge. Complex tasks go to a human before any code is written.

The hypothesis is that pipeline velocity will degrade over time. Tasks that would have been approved early on will increasingly land in the human review bucket — not because the decision framework changed, but because the repository did. The routing rate becomes a proxy for codebase health: as the system grows harder to modify safely, the agent's confidence should drop and its escalation rate should rise.

The initial breakdown on a clean repo establishes the baseline; what I'm watching for is drift. I don't have the answer yet. That's the point.

---

## What It Does

The pipeline is a locally-running website and server that lets me submit a feature idea, bug report, or improvement suggestion through a simple form. The task is stored in the file system, tracked by git - creating an audit trail for what happened to each task over time. From there, an autonomous agent evaluates the submission and routes it to one of three outcomes:

- **Approved** — low risk, clear requirements, ready for implementation
- **Rejected** — high blast radius, irreversible, unclear, or security-sensitive
- **Needs human review** — medium complexity, ambiguous, or below the confidence threshold

Approved tasks can then be automatically implemented by a second skill that creates a branch, writes the code, runs tests and linters, invokes a code-review agent, and marks the task done. Complex approved tasks are escalated to a grooming queue where a human writes a PRD before implementation begins.

The full pipeline runs on a loop. While the server is running, `/review-tasks` and `/implement-approved-tasks` fire on a schedule — continuously draining the pending queue and either shipping changes or asking for human input.

---

## The Decision Framework

The most interesting part of the system isn't the automation — it's the criteria the agent applies to make its decisions.

Every submitted task is evaluated against an explicit framework. To be **approved**, a task must satisfy all of:

- Low blast radius (changes isolated to a small surface area)
- Fully reversible via `git revert`
- No breaking changes to existing APIs or contracts
- Clear, unambiguous requirements
- No security, performance, or data integrity concerns

Everything in between — medium blast radius, new architectural patterns, borderline confidence — gets flagged for human review. The agent is explicitly calibrated to escalate when uncertain rather than guess. Burning tokens on increasingly desperate attempts isn't intelligence; it's chaos with extra steps.

Each decision is written back into the task's JSON file alongside the reasoning:

```json
{
  "id": "36874440-815a-4b84-b868-c238d1a9a8fc",
  "status": "approved",
  "submission": {
    "title": "Add the \"done\" status to tasks",
    "description": "Tasks that were merged to main should have a status of \"done\"..."
  },
  "analysis": {
    "analyzedAt": "2026-03-26T13:50:00.000Z",
    "inferredMetadata": {
      "type": "feature",
      "priority": "medium",
      "scope": "multiple"
    },
    "riskAssessment": {
      "blastRadius": "medium",
      "reversible": true,
      "breakingChanges": false,
      "risks": [
        "Type system already includes 'done' status but storage service doesn't support it",
        "Existing tasks in done/ folder have status 'approved' requiring data migration"
      ]
    },
    "reasoning": "Clear feature request to complete partial implementation of 'done' status. Changes span multiple components but are additive and fully reversible."
  }
}
```

And a rejection, for contrast:

```json
{
  "id": "af5f332e-5f93-426c-9144-86ff2eba02d0",
  "status": "rejected",
  "submission": {
    "title": "Refactor the frontend to use React",
    "description": "Vanilla HTML is boring. React is a great framework"
  },
  "analysis": {
    "inferredMetadata": {
      "type": "refactor",
      "priority": "low",
      "scope": "frontend"
    },
    "riskAssessment": {
      "blastRadius": "high",
      "reversible": false,
      "breakingChanges": true,
      "risks": [
        "Complete rewrite of existing frontend architecture",
        "No migration strategy or backwards compatibility plan",
        "Insufficient justification for architectural change"
      ]
    },
    "reasoning": "Rejected due to high blast radius and insufficient justification. This would require a complete frontend rewrite without addressing specific problems."
  }
}
```

---

## Initial Behavior

I started by entering roughly two dozen tasks. Some expected to be rejected: "Refactor the frontend to React because I like it better", some expected to be accepted: "There is a bug in the form title. It should be `Submit a Task`, not `submit task`" and some less obvious: "Change the color scheme of the dark theme to use...".

The breakdown matched the expectation. 30% accepted, 30% rejected, and 40% flagged for human intervention.

---

## Next Steps

I'll be creating different personas of users. The junior tester that finds small inconsistencies. The UI owner that on occasion wants to make changes to the UI flow. The resentful developer that tries to find ways to "trick" the pipeline. The architect in the ivory tower, disconnected from the codebase announcing company wide changes.

The system will continue to run and I'll update this as the pipeline processes more tasks.
