---
name: implement-approved-tasks
description: Implement straightforward approved tasks by creating branches, running tests, and invoking code review. Move complex tasks to grooming.
---

You are tasked with implementing approved task submissions autonomously.

## Input

The user will invoke this skill with:
```
/implement-approved-tasks
```

No additional parameters are required.

## Your Process

### 1. List Approved Tasks

Use the Glob tool to find all approved task files:
```
Pattern: packages/backend/suggestions/approved/*.json
```

If no approved tasks exist, inform the user with a friendly message and stop.

### 2. Assess Task Complexity

For each approved task, read the JSON file and assess whether it:
- **Needs a PRD** (complex): Tasks that involve:
  - Multiple components or services
  - Architectural changes or new patterns
  - Security-sensitive modifications
  - Database schema changes
  - Breaking changes to existing APIs
  - High blast radius (per risk assessment)
  - Unclear requirements or ambiguous scope

- **Is straightforward** (ready to implement): Tasks that:
  - Have clear, well-defined requirements
  - Are low-to-medium risk (per risk assessment)
  - Are additive/reversible (per risk assessment)
  - Have no breaking changes
  - Fit existing patterns

Use the `analysis.riskAssessment` data in the JSON to inform this decision.

### 3. Process Each Task

#### For Straightforward Tasks: Implement

1. **Create Implementation Branch**
   - Branch from `main`
   - Format: `task-{first-8-chars-of-id}-{slug}`
   - Example: `task-36874440-add-done-status`
   - Slug should be kebab-case, max 3-4 words from task title

2. **Implement the Task**
   - Read the task description and requirements
   - Implement the changes following CLAUDE.md guidelines (Law II: Simpler is always better)
   - Make focused, atomic commits following Conventional Commits format
   - Reference the task ID in commit messages: `feat: add done status [task-36874440]`

3. **Run Build and Tests**
   ```bash
   mise run build && mise run test
   ```
   - If tests fail, fix the issues and commit the fixes
   - Do not proceed until build and tests pass

4. **Run Lint and Format**
   ```bash
   mise run lint && mise run format
   ```
   - Fix any linting issues
   - If changes were made, commit them: `style: fix linting issues [task-{id}]`

5. **Invoke Code Reviewer**
   - Use the Agent tool with `subagent_type: code-reviewer`
   - Pass the context: "Review changes for task {title} (ID: {id})"
   - Wait for the agent to complete its review

6. **Address Code Review Feedback**
   - Fix any issues identified by the code-reviewer
   - Commit the fixes: `fix: address code review feedback [task-{id}]`
   - Re-run tests if needed

7. **Mark Task as Done**
   - Update the task JSON file:
     - Change `status` from `"approved"` to `"done"`
     - Add `implementedAt` timestamp
     - Add `branch` field with the branch name
   - Move the file from `approved/` to `done/`
   - Commit the status update: `chore: mark task {id} as done`

#### For Complex Tasks: Move to Grooming

1. **Update Task Status**
   - Update the task JSON file:
     - Change `status` from `"approved"` to `"grooming"`
     - Add `groomedAt` timestamp
     - Add `reason` field explaining why it needs a PRD

2. **Move to Grooming Folder**
   - Move the file from `approved/` to `grooming/`

### 4. Generate Summary Report

After processing all tasks, create a comprehensive report:

```markdown
## Approved Tasks Implementation Complete

### Implemented Tasks: {N}
{For each implemented task:}
- ✓ **{Task Title}** (ID: {short-id})
  - Branch: `{branch-name}`
  - Commits: {commit-count}
  - Status: Ready for PR

### Tasks Moved to Grooming: {M}
{For each grooming task:}
- ⚠ **{Task Title}** (ID: {short-id})
  - Reason: {reason}
  - Next Step: Create PRD

### Implementation Branches:
{List all branches created, one per line}

### Next Steps:
- Review implementation branches and create PRs
- Write PRDs for grooming tasks
- Run `git branch -a` to see all implementation branches
```

### 5. Commit Changes

After processing all tasks, commit the final changes:

```bash
git add packages/backend/suggestions/
git commit -m "$(cat <<'EOF'
chore: implement approved tasks - {X} completed, {Y} to grooming

Autonomous implementation of approved task submissions:
- {X} tasks implemented in dedicated branches
- {Y} tasks moved to grooming (require PRD)

Branches created:
{list branch names}

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

Replace placeholders with actual values.

## Key Principles

1. **Branch Isolation**: Each task gets its own branch for independent review and merging

2. **Quality Gates**: Build, test, lint, format, and code review must all pass before marking done

3. **Commit Discipline**: Follow Conventional Commits and reference task IDs

4. **Law I Compliance**: If implementation hits unexpected difficulty, stop and reassess. Mark the task for grooming rather than forcing a solution.

5. **Law II Compliance**: Keep implementations simple. Don't add features beyond what's requested.

6. **Transparency**: Report all decisions and outcomes clearly

## Error Handling

### No Approved Tasks
If the approved directory is empty:
```
No approved tasks to implement. Run /review-tasks to process pending submissions.
```

### Build/Test Failures
If builds or tests fail after reasonable attempts to fix:
- Document the issue in the task JSON
- Move the task back to needs-review with a note about the failure
- Continue with remaining tasks

### Code Review Blocker
If the code-reviewer identifies critical issues that can't be easily fixed:
- Document the issues in the task JSON
- Move the task to grooming for more detailed planning
- Continue with remaining tasks

### Git Issues
If branch creation fails or there are conflicts:
- Ensure you're on `main` branch first
- Pull latest changes
- Report the issue and ask for user intervention if needed

## Example Usage

```
User: /implement-approved-tasks
Assistant: I'll implement all approved tasks. Let me check what's ready.

[reads approved tasks]
[assesses complexity]
[implements straightforward tasks]
[moves complex tasks to grooming]

## Approved Tasks Implementation Complete

### Implemented Tasks: 2
- ✓ **Add dark mode toggle** (ID: abc12345)
  - Branch: `task-abc12345-dark-mode-toggle`
  - Commits: 3
  - Status: Ready for PR

- ✓ **Update documentation** (ID: def67890)
  - Branch: `task-def67890-update-docs`
  - Commits: 1
  - Status: Ready for PR

### Tasks Moved to Grooming: 1
- ⚠ **Refactor auth system** (ID: ghi11111)
  - Reason: High blast radius, security-sensitive, requires architectural design
  - Next Step: Create PRD

### Implementation Branches:
- task-abc12345-dark-mode-toggle
- task-def67890-update-docs

### Next Steps:
- Review implementation branches and create PRs
- Write PRD for auth system refactor
- Run `git branch -a` to see all implementation branches

Changes committed to git.
```

## Technical Notes

**Task Complexity Assessment**: Use the risk assessment data from the task-reviewer agent's analysis to make informed decisions. Don't be afraid to move tasks to grooming if they seem more complex than initially assessed.

**Branch Management**: Keep implementation branches separate. This allows:
- Independent review of each task
- Parallel merging (if tasks are independent)
- Easy rollback if needed
- Clear git history

**Quality Assurance**: The multi-stage verification (build, test, lint, format, code review) ensures high-quality implementations. Don't skip steps.

**Grooming Process**: Tasks moved to grooming should later be processed by creating a PRD, which can then be converted to an implementation plan using the prd-to-plan skill.

**Task Status Flow**:
```
pending → approved → [grooming | done]
```

**Filesystem Structure**:
```
packages/backend/suggestions/
├── pending/       # New submissions
├── approved/      # Approved by task-reviewer
├── grooming/      # Need PRD before implementation
├── done/          # Implemented and ready for PR
├── needs-review/  # Flagged for human review
└── rejected/      # Not suitable for implementation
```
