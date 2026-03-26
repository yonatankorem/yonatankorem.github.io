---
name: implement-phase
description: Implement a specific phase from an implementation plan. Verifies prerequisites, creates a branch, implements all tasks with commits, and reports completion for user approval.
---

You are tasked with implementing a specific phase from an implementation plan.

## Input

The user will provide:
1. **Path to the implementation plan** (typically in `docs/plans/`)
2. **Phase number** (e.g., "1", "2", "Phase 3")

## Your Process

### 1. Read and Validate the Phase

Read the implementation plan and:
- Extract the specified phase details (goal, tasks, acceptance criteria, verification steps)
- **Verify the phase is not already completed**: Check if the phase header contains `**COMPLETED**`
- If already completed, inform the user and stop

### 2. Check Dependencies

From the plan's "Dependencies Summary" or "Build Order" section:
- Identify any prerequisite phases
- **Verify all dependency phases are marked as completed**
- If dependencies are incomplete, inform the user which phases must be completed first and stop

### 3. Ensure Dedicated Branch

Check the current git branch:
- If not on a dedicated branch for this phase, create one with format: `phase-{number}-{short-slug}`
  - Example: `phase-1-shared-types`, `phase-2-backend-api`
  - Slug should be kebab-case, max 3-4 words from phase name
- If already on an appropriately named branch for this phase, use it
- Push the branch to remote if it doesn't exist there yet

### 4. Implement Phase Tasks

For each task in the phase:
1. **Announce the task**: Tell the user what you're working on
2. **Implement the task**: Make the necessary code changes
3. **Verify locally**: Run any relevant verification commands (tests, type checks, linters)
4. **Commit**: Create a focused commit for this task
   - Follow the repository's commit message conventions
   - Reference the phase in the commit message (e.g., "feat: create Task interface [Phase 1]")
5. **Mark as done**: Update your task list

**Important Implementation Guidelines:**
- Follow the project's existing patterns and conventions (from CLAUDE.md)
- Adhere to Law II (Simpler is always better) - don't over-engineer
- If a task is unclear or blocks progress, ask the user before proceeding
- If you discover missing information or issues, document them but don't let it block the phase

### 5. Phase Completion Verification

After all tasks are complete:
1. Run the phase's verification commands (from the "Verification" section)
2. Check acceptance criteria (if present)
3. Run any test strategy commands specified in the phase
4. Summarize what was accomplished

### 6. Report to User

Present a completion report:

```
## Phase {N} Implementation Complete

**Phase**: {Phase Name}
**Branch**: {branch-name}
**Commits**: {number of commits}

### Tasks Completed:
- ✓ {Task 1}
- ✓ {Task 2}
- ✓ {Task 3}
...

### Verification Results:
{Results of running verification commands}

### Acceptance Criteria:
{Status of each criterion, if applicable}

### Issues Encountered:
{Any problems, blockers, or deviations from the plan}

### Next Steps:
1. Review the changes in the branch
2. Test the implementation
3. If approved, I will mark this phase as **COMPLETED** in the implementation plan
4. {Suggest next phase if applicable}

Ready for your review. Should I mark this phase as complete?
```

### 7. Mark Phase as Complete (User Approval Required)

**STOP and wait for user approval.** Do not proceed to marking the phase complete or starting the next phase without explicit approval.

Once the user approves:
1. Edit the implementation plan file
2. Add `**COMPLETED**` to the phase header
   - Before: `### Phase 1: Shared Types Foundation`
   - After: `### Phase 1: Shared Types Foundation **COMPLETED**`
3. Commit the plan update: `docs: mark Phase {N} as complete`
4. Inform the user that the phase is now marked complete

## Key Principles

1. **Verification before action**: Always check if a phase is already done and if dependencies are met before starting work

2. **One phase at a time**: Complete the current phase fully before moving to the next

3. **Commit discipline**: Each task should result in one focused commit - never bundle unrelated changes

4. **Branch isolation**: Each phase gets its own branch to enable parallel work and easier review

5. **Stop on completion**: After implementing a phase, ALWAYS stop and wait for user approval before marking it complete

6. **Transparency**: Report issues clearly - don't hide problems or make assumptions

7. **Follow the plan**: Stick to the tasks outlined in the phase. If changes are needed, discuss with the user first

## Error Handling

- **Phase already completed**: Inform user and suggest the next phase
- **Missing dependencies**: List which phases must be completed first
- **Task blocked**: Ask user for clarification or guidance
- **Verification failures**: Report failures and ask user how to proceed
- **Branch conflicts**: Ask user to resolve conflicts before proceeding

## Example Usage

```
User: /implement-phase docs/plans/implementation-plan-task-queue.md 1
```

You would then:
1. Read the plan
2. Verify Phase 1 is not completed
3. Check no dependencies exist (or that they're met)
4. Create branch `phase-1-shared-types`
5. Implement each task, committing as you go
6. Run verification
7. Report completion and wait for approval
8. Mark phase complete after approval