---
name: review-tasks
description: Process all pending task submissions by spawning parallel task-reviewer agents. Each agent analyzes a task and makes an approve/reject/needs-review decision.
---

You are tasked with orchestrating autonomous analysis of pending task submissions.

## Input

The user will invoke this skill with:
```
/review-tasks
```

No additional parameters are required.

## Your Process

### 1. List Pending Tasks

Use the Glob tool to find all pending task files:
```
Pattern: packages/backend/suggestions/pending/*.json
```

If no pending tasks exist, inform the user with a friendly message and stop.

### 2. Spawn Task-Reviewer Agents in Parallel

For each pending task file:
- Spawn a task-reviewer agent using the Agent tool
- Pass the task file path as context: "Analyze the task submission in {filepath}"
- **Spawn all agents in parallel** (not sequentially) for efficiency

### 3. Wait for Agent Completion

The Agent tool will wait for all agents to complete. Each agent will:
- Read the task JSON file
- Analyze the submission
- Infer metadata (type, priority, scope)
- Assess risks (blast radius, reversibility, breaking changes)
- Make a decision (APPROVED, REJECTED, NEEDS_REVIEW)
- Write analysis to the JSON file
- Move the file to the appropriate status directory

### 4. Aggregate Results

After all agents complete, use Glob again to check how many files are in each status directory:
- `packages/backend/suggestions/pending/` (should be empty)
- `packages/backend/suggestions/approved/`
- `packages/backend/suggestions/rejected/`
- `packages/backend/suggestions/needs-review/`

### 5. Commit Task Decisions

After processing all tasks, commit the changes to git:

1. Check if there are changes in the suggestions directory:
   ```bash
   git status packages/backend/suggestions/
   ```

2. If there are changes, stage and commit them:
   ```bash
   git add packages/backend/suggestions/
   git commit -m "$(cat <<'EOF'
   chore: process task review batch - {X} approved, {Y} rejected, {Z} needs review

   Autonomous task-reviewer agents analyzed {N} pending submissions:
   - {X} tasks approved and ready for implementation
   - {Y} tasks rejected due to high risk or unclear requirements
   - {Z} tasks flagged for human review

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
   EOF
   )"
   ```

3. Replace `{X}`, `{Y}`, `{Z}`, and `{N}` with actual counts from the aggregate results

This creates an audit trail showing:
- When tasks were reviewed
- How many tasks were processed
- What decisions were made

**Note**: If the agents had to manually move files (when Bash tool wasn't available), ensure those moves are included in the commit.

### 6. Generate Summary Report

Present a report in this format:

```markdown
## Task Review Complete

Processed **{N}** tasks

### Results:
- ✓ **{X} Approved** - Low risk, ready for implementation
- ✗ **{Y} Rejected** - High risk or unclear requirements
- ? **{Z} Needs Review** - Medium complexity, human input needed

### Details:
{For each task, show filename and decision with brief reasoning}

### Next Steps:
- Review approved tasks in `suggestions/approved/`
- Check needs-review tasks in `suggestions/needs-review/`
- View rejected tasks and reasoning in `suggestions/rejected/`
```

## Error Handling

### No Pending Tasks
If the pending directory is empty:
```
No pending tasks to review. Submit tasks via the web UI at http://localhost:5173
```

### Agent Failures
If an agent fails to complete:
- Log the error in your report
- Continue processing remaining tasks
- Report which tasks succeeded and which failed

### Partial Success
If some agents succeed and others fail:
```
Processed {X} of {Y} tasks successfully.
Failed tasks: {list filenames}
Check logs for details.
```

## Usage Documentation

### When to Use
Run this skill when:
- Task submissions have accumulated in the pending queue
- After users submit tasks via the web UI
- As part of a regular review cadence (e.g., daily)

### Expected Behavior
- **Parallel Processing**: All agents run simultaneously for efficiency
- **Filesystem Operations**: Files are moved between status directories
- **Analysis Persistence**: Each task JSON gets an `analysis` field added
- **Atomic Operations**: File moves use atomic rename operations
- **Git Audit Trail**: Changes are committed automatically after processing

### Output Format
The summary report shows:
- Total tasks processed
- Count by decision type (approved/rejected/needs-review)
- Per-task details with decision and reasoning
- Clear next steps for the user

### Troubleshooting

**Issue: "No files found in pending/"**
- Verify the backend server created the directory structure
- Check that tasks were submitted via the web UI
- Ensure you're in the correct working directory

**Issue: "Agent failed to analyze task"**
- Check file permissions on suggestions/ directories
- Verify task JSON is valid and complete
- Review agent logs for specific errors

**Issue: "File already exists in destination"**
- Multiple agents may have processed the same task
- Check for duplicate task IDs
- Verify filesystem concurrency handling

**Issue: "Analytics not updating"**
- This skill only moves files, frontend must refresh
- Analytics component reads filesystem on each load
- Try refreshing the browser at http://localhost:5173

## Example Usage

```
User: /review-tasks
Assistant: I'll process all pending tasks in parallel using task-reviewer agents.

[spawns agents for each pending task]
[agents complete analysis and move files]
[commits changes to git]

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
- ✗ task-jkl012.json: Refactor auth system (security implications, core system)
- ? task-mno345.json: Integrate analytics library (new dependency, medium risk)

### Next Steps:
- Review approved tasks in `suggestions/approved/`
- Check needs-review tasks in `suggestions/needs-review/`
- View rejected tasks and reasoning in `suggestions/rejected/`

Changes committed to git with audit trail.
```

## Technical Notes

**Parallel Agent Spawning**: Use multiple Agent tool calls in a single response to spawn agents in parallel. This is significantly faster than spawning them sequentially.

**File Concurrency**: Each agent works on a unique file, so there should be no race conditions. The task-reviewer agent uses atomic file operations (write to temp, then rename).

**Git Audit Trail**: All task files are tracked in git. After each review batch:
- Task movements between directories are captured
- Analysis metadata is preserved in the JSON files
- Commit messages document the decisions made
- Full history is available via `git log packages/backend/suggestions/`

This provides transparency and accountability for autonomous decision-making.

**Decision Framework Iteration**: As you observe agent decisions over time, the decision criteria in the task-reviewer agent definition can be tuned based on false positives/negatives.
