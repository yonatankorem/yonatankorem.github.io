# Task Suggestion System: Technical Overview

This document describes the autonomous task suggestion system, covering the end-to-end flow from user submission through analysis, decision-making, and implementation.

## System Overview

The task suggestion system enables users to submit feature ideas, bug reports, and improvements through a web UI. Tasks are autonomously analyzed by AI agents that assess feasibility, risk, and scope, then make approve/reject/needs-review decisions. Approved tasks can be automatically implemented; complex or ambiguous tasks are routed to humans.

```
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│                              TASK SUGGESTION PIPELINE                                    │
│                                                                                          │
│  ┌──────────┐     ┌──────────┐     ┌──────────────────────────────────────┐              │
│  │  Web UI  │────▶│ Backend  │────▶│           Filesystem                 │              │
│  │  (Form)  │     │  (API)   │     │  suggestions/pending/task-{id}.json  │              │
│  └──────────┘     └──────────┘     └───────────────┬──────────────────────┘              │
│                                                    │                                     │
│                                    ┌───────────────▼───────────────┐                     │
│                                    │      /review-tasks skill      │                     │
│                                    │  (spawns parallel agents)     │                     │
│                                    └───────────────┬───────────────┘                     │
│                                                    │                                     │
│                      ┌─────────────────────────────┼────────────────┐                    │
│                      │                             │                │                    │
│                      ▼                             ▼                ▼                    │
│  ┌─────────────────────────┐  ┌─────────────────────────┐  ┌───────────────┐             │
│  │   task-reviewer agent   │  │   task-reviewer agent   │  │     ...       │             │
│  │   (analyzes task-001)   │  │   (analyzes task-002)   │  │               │             │
│  └───────────┬─────────────┘  └───────────┬─────────────┘  └──────┬────────┘             │
│              │                            │                       │                      │
│              └────────────┬───────────────┴───────────────────────┘                      │
│                           ▼                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐               │
│  │                        Decision Framework                             │               │
│  │  ┌─────────┐         ┌─────────────┐         ┌────────────┐           │               │
│  │  │APPROVED │         │NEEDS-REVIEW │         │ REJECTED   │           │               │
│  │  │ 30-40%  │         │   <50%      │         │  20-30%    │           │               │
│  │  └────┬────┘         └──────┬──────┘         └─────┬──────┘           │               │
│  └───────┼─────────────────────┼──────────────────────┼──────────────────┘               │
│          │                     │                      │                                  │
│          ▼                     ▼                      ▼                                  │
│   suggestions/           suggestions/            suggestions/                            │
│   approved/              needs-review/           rejected/                               │
│          │                     │                                                         │
│          │            ┌────────▼────────┐                                                │
│          │            │  HUMAN REVIEW   │                                                │
│          │            │  (manual step)  │                                                │
│          │            └────────┬────────┘                                                │
│          │                     │                                                         │
│          ◀─────────────────────┘                                                         │
│          │                                                                               │
│          ▼                                                                               │
│  ┌───────────────────────────────────┐                                                   │
│  │  /implement-approved-tasks skill  │                                                   │
│  └───────────────┬───────────────────┘                                                   │
│                  │                                                                       │
│                  ▼                                                                       │
│         ┌───────────────────┐                                                            │
│         │ Assess Complexity │                                                            │
│         └────────┬──────────┘                                                            │
│                  │                                                                       │
│      ┌───────────┴───────────┐                                                           │
│      │                       │                                                           │
│      ▼                       ▼                                                           │
│ ┌─────────┐          ┌───────────────┐                                                   │
│ │ COMPLEX │          │STRAIGHTFORWARD│                                                   │
│ └────┬────┘          └───────┬───────┘                                                   │
│      │                       │                                                           │
│      ▼                       ▼                                                           │
│ ┌─────────────┐    ┌─────────────────────────────────────────────────────────────┐       │
│ │  GROOMING   │    │              Implementation Flow (Automated)                │       │
│ │  (needs PRD)│    │  ┌────────┐   ┌───────┐   ┌──────┐   ┌──────┐   ┌──────┐   │       │
│ │             │    │  │ Branch │──▶│Build &│──▶│Lint &│──▶│ Code │──▶│ Done │   │       │
│ │ Human writes│    │  │        │   │ Test  │   │Format│   │Review│   │      │   │       │
│ │ PRD first   │    │  └────────┘   └───┬───┘   └──────┘   └──┬───┘   └──┬───┘   │       │
│ └──────┬──────┘    │                   │                     │          │       │       │
│        │           │                   ▼ fail                ▼ critical │       │       │
│        │           │            ┌─────────────┐       ┌─────────────┐   │       │       │
│        │           │            │ needs-review│◀──────│ needs-review│   │       │       │
│        │           │            └─────────────┘       └─────────────┘   │       │       │
│        │           └────────────────────────────────────────────────────┼───────┘       │
│        │                                                                │               │
│        └────────────────────────────────────────────────────────────────┤               │
│                                                                         ▼               │
│                                                                  suggestions/           │
│                                                                  done/                  │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```

## User Flow

### 1. Task Submission

Users access the web UI and fill out a submission form with:
- **Title** (1-100 characters): Brief description of the task
- **Description** (1-1000 characters): Detailed explanation of what should be done

The frontend sends a POST request to the backend API:

```typescript
// POST /api/tasks
const response = await fetch('/api/tasks', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    title: "Add dark mode toggle",
    description: "Add a toggle button in the header that switches between light and dark themes..."
  })
});
```

### 2. Backend Processing

The backend validates the submission and creates a task file:

```typescript
// packages/backend/src/routes/tasks.ts
fastify.post<{ Body: TaskSubmissionBody }>(
  '/api/tasks',
  {
    schema: {
      body: {
        type: 'object',
        required: ['title', 'description'],
        properties: {
          title: { type: 'string', minLength: 1, maxLength: 100 },
          description: { type: 'string', minLength: 1, maxLength: 1000 },
        },
      },
    },
  },
  async (request, reply) => {
    const { title, description } = request.body;
    const task = await storageService.createTask({ title, description });
    return reply.code(201).send({ success: true, id: task.id });
  }
);
```

### 3. Viewing Analytics

The frontend fetches and displays task statistics:

```typescript
// GET /api/tasks/stats returns TaskStats
interface TaskStats {
  total: number;
  pending: number;
  approved: number;
  rejected: number;
  needsReview: number;
  done: number;
}
```

## Backend File Storage

Tasks are stored as JSON files in a directory structure that reflects their status:

```
packages/backend/suggestions/
├── pending/           # New submissions awaiting analysis
├── approved/          # Approved by task-reviewer, ready for implementation
├── rejected/          # Not suitable for implementation
├── needs-review/      # Flagged for human review
├── grooming/          # Complex tasks needing PRD
└── done/              # Implemented and merged
```

### Storage Service

The `StorageService` class manages task persistence with atomic file operations:

```typescript
// packages/backend/src/services/storage.ts

export class StorageService {
  private readonly baseDir: string;
  private readonly statusDirs: Record<TaskStatus, string>;

  async createTask(submission: TaskSubmission): Promise<Task> {
    const task: Task = {
      id: uuidv4(),
      submittedAt: new Date().toISOString(),
      status: 'pending',
      submission,
      analysis: null,
      version: '1.0',
    };

    const filePath = this.getTaskPath(task.id, 'pending');
    await this.writeTaskFile(filePath, task);
    return task;
  }

  async moveTask(id: string, fromStatus: TaskStatus, toStatus: TaskStatus): Promise<void> {
    const task = await this.getTask(id, fromStatus);
    task.status = toStatus;

    const toPath = this.getTaskPath(id, toStatus);
    await this.writeTaskFile(toPath, task);
    await fs.unlink(this.getTaskPath(id, fromStatus));
  }

  // Atomic write: temp file + rename to prevent partial writes
  private async writeTaskFile(filePath: string, task: Task): Promise<void> {
    const tempPath = `${filePath}.tmp`;
    await fs.writeFile(tempPath, JSON.stringify(task, null, 2), 'utf-8');
    await fs.rename(tempPath, filePath);
  }
}
```

### Why Filesystem Storage?

1. **Git-tracked history**: All task files are version-controlled, providing audit trail
2. **Human-readable**: JSON files can be inspected and edited directly
3. **Claude Code integration**: Agents can read/write files natively without database setup
4. **Atomic operations**: Directory moves and file renames are atomic on most filesystems

## JSON Task Representation

### TypeScript Types

```typescript
// packages/shared/src/types/task.ts

export type TaskStatus = 'pending' | 'approved' | 'rejected' | 'needs-review' | 'done';

export interface TaskSubmission {
  title: string;       // 1-100 characters
  description: string; // 1-1000 characters
}

export interface InferredMetadata {
  type: 'feature' | 'bug' | 'refactor' | 'docs' | 'test' | 'chore';
  priority: 'low' | 'medium' | 'high';
  scope: 'frontend' | 'backend' | 'shared' | 'infrastructure' | 'multiple';
}

export interface RiskAssessment {
  risks: string[];           // Specific identified risks
  blastRadius: 'low' | 'medium' | 'high';
  reversible: boolean;       // Can changes be undone via git revert?
  breakingChanges: boolean;  // Does this alter APIs or contracts?
}

export interface TaskAnalysis {
  analyzedAt: string;              // ISO 8601 timestamp
  inferredMetadata: InferredMetadata;
  riskAssessment: RiskAssessment;
  reasoning: string;               // Agent's explanation for the decision
}

export interface Task {
  id: string;                      // UUID v4
  submittedAt: string;             // ISO 8601 timestamp
  status: TaskStatus;
  submission: TaskSubmission;
  analysis: TaskAnalysis | null;   // Null until analyzed
  version: string;                 // Schema version ("1.0")
  // Added after implementation:
  implementedAt?: string;          // When task was completed
  branch?: string;                 // Implementation branch name
}
```

### Example: Pending Task (Before Analysis)

```json
{
  "id": "36874440-815a-4b84-b868-c238d1a9a8fc",
  "submittedAt": "2026-03-26T13:44:08.796Z",
  "status": "pending",
  "submission": {
    "title": "Add the \"done\" status to tasks",
    "description": "Tasks that were merged to `main` should have a status of \"done\" and the files moved to the \"done\" folder. Also add it to the analytics part."
  },
  "analysis": null,
  "version": "1.0"
}
```

### Example: Approved Task (After Analysis)

```json
{
  "id": "36874440-815a-4b84-b868-c238d1a9a8fc",
  "submittedAt": "2026-03-26T13:44:08.796Z",
  "status": "approved",
  "submission": {
    "title": "Add the \"done\" status to tasks",
    "description": "Tasks that were merged to `main` should have a status of \"done\"..."
  },
  "analysis": {
    "analyzedAt": "2026-03-26T13:50:00.000Z",
    "inferredMetadata": {
      "type": "feature",
      "priority": "medium",
      "scope": "multiple"
    },
    "riskAssessment": {
      "risks": [
        "Type system already includes 'done' status but storage service doesn't support it",
        "Existing tasks in done/ folder have status 'approved' requiring data migration"
      ],
      "blastRadius": "medium",
      "reversible": true,
      "breakingChanges": false
    },
    "reasoning": "Clear feature request to complete partial implementation of 'done' status. Changes span multiple components but are additive and fully reversible."
  },
  "version": "1.0"
}
```

### Example: Rejected Task

```json
{
  "id": "af5f332e-5f93-426c-9144-86ff2eba02d0",
  "submittedAt": "2026-03-25T22:18:03.008Z",
  "status": "rejected",
  "submission": {
    "title": "Refactor the frontend to use React",
    "description": "Vanilla HTML is boring. React is a great framework"
  },
  "analysis": {
    "analyzedAt": "2026-03-26T00:00:00.000Z",
    "decision": "rejected",
    "inferredMetadata": {
      "type": "refactor",
      "priority": "low",
      "scope": "frontend"
    },
    "riskAssessment": {
      "risks": [
        "Complete rewrite of existing frontend architecture",
        "Breaking changes to current UI implementation",
        "No migration strategy or backwards compatibility plan",
        "Insufficient justification for architectural change"
      ],
      "blastRadius": "high",
      "reversible": false,
      "breakingChanges": true
    },
    "reasoning": "Rejected due to high blast radius and insufficient justification. This would require a complete frontend rewrite without addressing specific problems or providing technical rationale."
  },
  "version": "1.0"
}
```

### Example: Needs-Review Task

```json
{
  "id": "919b4728-500f-43f3-8aa4-ad929688fc58",
  "submittedAt": "2026-03-26T16:25:37.016Z",
  "status": "needs-review",
  "submission": {
    "title": "The UI should have two views: Tasks, and \"How it works\"",
    "description": "The UI is too busy now. It should have a view dedicated for tasks, and another for \"How it works\" where users can see the rules, laws, subagents, etc."
  },
  "analysis": {
    "analyzedAt": "2026-03-26T16:27:15.000Z",
    "decision": "needs-review",
    "inferredMetadata": {
      "type": "feature",
      "priority": "medium",
      "scope": "frontend"
    },
    "riskAssessment": {
      "risks": [
        "Requires navigation/routing pattern decision",
        "May need refactoring of existing component structure"
      ],
      "blastRadius": "medium",
      "reversible": true,
      "breakingChanges": false
    },
    "reasoning": "Medium blast radius UI restructuring. While the functional requirement is clear, this introduces new navigation architecture that would benefit from human review."
  },
  "version": "1.0"
}
```

## The review-tasks Skill

The `/review-tasks` skill orchestrates the analysis of pending task submissions.

### Skill Definition

```yaml
# .claude/skills/review-tasks/SKILL.md
---
name: review-tasks
description: Process all pending task submissions by spawning parallel task-reviewer agents.
---
```

### Workflow

1. **List Pending Tasks**: Glob for `packages/backend/suggestions/pending/*.json`
2. **Spawn Parallel Agents**: For each pending task, spawn a `task-reviewer` agent
3. **Aggregate Results**: Count tasks in each status directory after processing
4. **Commit Changes**: Create git commit with audit trail
5. **Generate Report**: Summary of decisions made

### Example Output

```markdown
## Task Review Complete

Processed **5** tasks

### Results:
- ✓ **3 Approved** - Low risk, ready for implementation
- ✗ **1 Rejected** - High risk or unclear requirements
- ? **1 Needs Review** - Medium complexity, human input needed

### Details:
- ✓ task-abc123.json: Add dark mode toggle (UI styling, fully reversible)
- ✓ task-def456.json: Update documentation (no code changes)
- ✓ task-ghi789.json: Add unit tests (improves quality)
- ✗ task-jkl012.json: Refactor auth system (security implications)
- ? task-mno345.json: Integrate analytics library (new dependency)

Changes committed to git with audit trail.
```

## The task-reviewer Subagent

The `task-reviewer` agent is a specialized AI agent that analyzes individual task submissions and makes autonomous decisions.

### Agent Definition

```yaml
# .claude/agents/task-reviewer.md
---
name: task-reviewer
description: Analyze task submissions and make approve/reject/needs-review decisions
tools: Glob, Grep, Read, Edit, Write, Bash, ...
model: sonnet
---
```

### Responsibilities

1. **Task Analysis**: Read task file, extract title and description, understand requirements
2. **Metadata Inference**: Determine type, priority, and scope
3. **Risk Assessment**: Evaluate blast radius, reversibility, breaking changes
4. **Decision Making**: Apply decision framework criteria
5. **File Operations**: Write analysis, move file to appropriate directory

### Agent Process

```
1. Read Task File
   ↓
2. Parse Requirements
   ↓
3. Infer Metadata
   • type: feature/bugfix/refactor/docs/test/chore
   • priority: low/medium/high
   • scope: frontend/backend/shared/infra/multiple
   ↓
4. Assess Risks
   • blastRadius: low/medium/high
   • reversible: true/false
   • breakingChanges: true/false
   • risks: [specific concerns]
   ↓
5. Apply Decision Framework
   ↓
6. Write Analysis to JSON
   ↓
7. Move File to Status Directory
   ↓
8. Report Results
```

## Decision Framework

The decision framework defines criteria for autonomous approve/reject decisions.

### APPROVED Criteria

Tasks must meet **ALL** of these:

| Criterion | Description |
|-----------|-------------|
| Low blast radius | Changes isolated to single component or small set of files |
| Fully reversible | Can be undone via `git revert`, no data changes |
| No breaking changes | Doesn't alter existing APIs, signatures, or contracts |
| Clear requirements | Well-defined with unambiguous intent |
| Low risk | Minimal security, performance, or reliability concerns |

**Common approved scenarios:**
- UI styling changes (colors, spacing, layout)
- Documentation updates
- Test additions
- Small utility functions
- Non-breaking feature additions
- Logging/monitoring improvements

### REJECTED Criteria

Tasks matching **ANY** of these are rejected:

| Criterion | Description |
|-----------|-------------|
| High blast radius | Affects core systems, multiple components, or architecture |
| Irreversible | Data deletion, third-party API changes, permanent state changes |
| Breaking changes | Alters public APIs without migration strategy |
| Unclear requirements | Contradictory, vague, or missing critical details |
| Security implications | Touches auth, crypto, or permissions |
| Data integrity risks | Could cause data loss or corruption |

**Common rejected scenarios:**
- Auth system refactoring
- Database schema changes without migration plan
- Removing/renaming public API endpoints
- Tasks with insufficient detail
- Bypassing security mechanisms

### NEEDS-REVIEW Criteria

Flag for human review when:

| Criterion | Description |
|-----------|-------------|
| Medium blast radius | Not isolated, but not core system either |
| New patterns | Introduces unfamiliar libraries or architectural approaches |
| Ambiguous requirements | Multiple interpretations possible |
| Uncertain confidence | <80% confident in approve/reject |
| Critical refactoring | Changes important code paths |
| Borderline complexity | Between "simple" and "complex" |

**Important distinction**: Questions about WHERE or HOW (implementation details) are not ambiguous requirements. Only flag if the WHAT is unclear.

```
❌ NEEDS_REVIEW: "Add some kind of task visualization" (unclear WHAT)
✅ APPROVED: "Add task detail view. Maybe not in analytics?" (clear WHAT, questions WHERE)
```

## The implement-approved-tasks Skill

The `/implement-approved-tasks` skill automates implementation of approved tasks.

### Skill Definition

```yaml
# .claude/skills/implement-approved-tasks/SKILL.md
---
name: implement-approved-tasks
description: Implement approved tasks by creating branches, running tests, invoking code review.
---
```

### Workflow

1. **List Approved Tasks**: Glob for `packages/backend/suggestions/approved/*.json`
2. **Assess Complexity**: Determine if task needs PRD or is straightforward
3. **For Straightforward Tasks**:
   - Create implementation branch: `task-{id-prefix}-{slug}`
   - Implement changes following CLAUDE.md guidelines
   - Run build and tests: `mise run build && mise run test`
   - Run lint and format: `mise run lint && mise run format`
   - Invoke code-reviewer agent
   - Address feedback, mark task as done
4. **For Complex Tasks**: Move to `grooming/` for PRD creation
5. **Generate Summary Report**

### Task Status Flow

```
pending → approved → grooming (complex)
                  ↘  done (straightforward)
```

### Quality Gates

Every implementation must pass:
1. Build succeeds
2. Tests pass
3. Lint passes
4. Format passes
5. Code review passes

## Human-in-the-Loop Points

The system requires human intervention at specific points:

### 1. Needs-Review Tasks

Tasks flagged `needs-review` require a human to:
- Review the analysis and reasoning
- Make a final approve/reject decision
- Optionally add clarifying notes

**Location**: `packages/backend/suggestions/needs-review/`

### 2. Grooming Tasks

Complex approved tasks moved to grooming require:
- PRD creation to define requirements
- Architecture decisions
- Implementation planning

**Location**: `packages/backend/suggestions/grooming/`

### 3. PR Review

Even auto-implemented tasks require:
- Code review before merge
- Manual testing verification
- Final approval to merge to main

### 4. Framework Iteration

Humans must review and adjust decision criteria when:
- False positive rate >5%
- False negative rate >10%
- Needs-review rate >50%
- New task patterns emerge

## Analytics and Decision Framework Metrics

### Target Decision Distribution

| Decision | Target Range | Red Flag |
|----------|--------------|----------|
| Approved | 30-40% | Very low (<20%) or very high (>50%) |
| Rejected | 20-30% | Very low (<10%) |
| Needs Review | <50% | >50% (framework too uncertain) or <20% (overconfident) |

### Key Metrics

#### 1. Decision Distribution

```bash
# Count tasks in each status directory
APPROVED=$(find suggestions/approved -name "*.json" | wc -l)
REJECTED=$(find suggestions/rejected -name "*.json" | wc -l)
NEEDS_REVIEW=$(find suggestions/needs-review -name "*.json" | wc -l)
TOTAL=$((APPROVED + REJECTED + NEEDS_REVIEW))

echo "Approved: $((100 * APPROVED / TOTAL))%"
echo "Rejected: $((100 * REJECTED / TOTAL))%"
echo "Needs Review: $((100 * NEEDS_REVIEW / TOTAL))%"
```

#### 2. False Positive Rate

Auto-approved tasks that should have been rejected or flagged.

```
False Positive Rate = False Positives / Total Approved
Target: <5%
```

**Action**: If >5% for 2+ weeks, tighten approval criteria.

#### 3. False Negative Rate

Auto-rejected tasks that should have been approved.

```
False Negative Rate = False Negatives / Total Rejected
Target: <10%
```

**Action**: If >10% for 2+ weeks, loosen rejection criteria.

#### 4. Human Override Rate

How often humans disagree with agent decisions on needs-review tasks.

**Patterns**:
- Most needs-review → approved: Framework too cautious
- Most needs-review → rejected: Framework too permissive
- Balanced distribution: Well-calibrated

### Frontend Analytics Display

```typescript
// packages/frontend/src/components/analytics.ts

export function renderAnalytics(stats: TaskStats): string {
  return `
    <div class="analytics-grid">
      <div class="stat-card stat-card--total">
        <div class="stat-card__count">${stats.total}</div>
        <div class="stat-card__label">Total Submissions</div>
      </div>
      <div class="stat-card stat-card--pending">
        <div class="stat-card__count">${stats.pending}</div>
        <div class="stat-card__label">Pending Review</div>
      </div>
      <div class="stat-card stat-card--approved">
        <div class="stat-card__count">${stats.approved}</div>
        <div class="stat-card__label">Approved</div>
      </div>
      <div class="stat-card stat-card--rejected">
        <div class="stat-card__count">${stats.rejected}</div>
        <div class="stat-card__label">Rejected</div>
      </div>
      <div class="stat-card stat-card--needs-review">
        <div class="stat-card__count">${stats.needsReview}</div>
        <div class="stat-card__label">Needs Human Review</div>
      </div>
      <div class="stat-card stat-card--done">
        <div class="stat-card__count">${stats.done}</div>
        <div class="stat-card__label">Done</div>
      </div>
    </div>
  `;
}
```

### Framework Iteration Cycle

| Cadence | Activity |
|---------|----------|
| Weekly | Check decision distribution, spot-check 5-10 decisions |
| Monthly | Calculate all metrics, review flagged errors, test and deploy changes |
| Quarterly | Analyze trends, evaluate efficiency gains, update iteration guide |

### When to Adjust Criteria

**Do adjust when:**
- False positive/negative rates exceed thresholds for 2+ weeks
- Needs-review rate consistently >50%
- New task patterns emerge not covered by criteria
- Repeated human overrides on specific task types

**Don't adjust when:**
- Only 1-2 weeks of data (insufficient sample)
- Edge case occurs once (outliers expected)
- Metrics are within target ranges

**Rule of thumb**: Wait for 20-30 task submissions before making framework changes.

## Summary

The task suggestion system provides an autonomous pipeline for processing user-submitted tasks:

1. **Submission**: Users submit via web form, stored as JSON files
2. **Analysis**: `task-reviewer` agents evaluate each task in parallel
3. **Decision**: Framework criteria determine approve/reject/needs-review
4. **Human Loop**: Flagged tasks and final PR reviews require human judgment
5. **Implementation**: Approved tasks can be auto-implemented with quality gates
6. **Iteration**: Metrics track decision quality, enabling framework refinement

The system balances automation efficiency with appropriate human oversight, using conservative defaults (flag when uncertain) and explicit risk identification.
