# Repository Guidelines Checker

This skill verifies that the repository adheres to the rules and standards defined

## When to Use This Skill

- Before creating a pull request
- During code reviews
- Periodically to ensure ongoing compliance
- When you need to verify repository standards are being followed
- When explicitly invoked by the user with `/repository-guidelines`

## Checklist

Run through each item in this checklist and report findings:

### 1. Git History Compliance

**Commit Message Format:**
- ✓ All commit messages follow Conventional Commits specification
- ✓ Commit types are one of: `feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `test:`, `chore:`
- ✓ Commit messages are descriptive and explain the "why" not just the "what"

**Single Responsibility Principle:**
- ✓ Each commit addresses one topic or change
- ✓ No commits mixing multiple unrelated changes
- ✓ Commits are atomic and could be reverted independently

**Linear History:**
- ✓ The `main` branch has a linear history (no merge commits)
- ✓ Feature branches use rebase workflow
- ✓ No "Merge branch" commits on main

**Human-Readable History:**
- ✓ Commit history tells a clear story
- ✓ Commit sequence shows logical progression of work
- ✓ Someone reading the history can understand what happened and why

## How to Execute

1. **Analyze Recent Commits:**
   ```bash
   git log --oneline -20
   git log --graph --oneline main --max-count=30
   ```

2. **Check for Merge Commits on Main:**
   ```bash
   git log main --merges --oneline
   ```

3. **Review Commit Message Format:**
   - Check that each commit starts with a valid type
   - Verify messages are descriptive
   - Look for patterns that violate single responsibility

4. **Report Findings:**
   Create a clear report with:
   - ✅ Items that pass
   - ⚠️ Items that need attention
   - ❌ Items that fail
   - Specific examples for any issues found
   - Recommendations for fixes

## Example Output Format

```
# Repository Guidelines Check

## Git History Compliance

### ✅ Commit Message Format
All recent commits follow Conventional Commits:
- feat: add Core Laws section to frontend
- chore: add mise run permission
- docs: update CLAUDE.md

### ⚠️ Single Responsibility
Most commits are atomic, but found 1 commit mixing concerns:
- abc1234: "feat: add feature and fix bug" (should be 2 commits)

### ✅ Linear History
Main branch has clean linear history with no merge commits.

### ✅ Human-Readable History
Commit history shows clear progression of features and refactoring.

---

**Overall:** 3/4 checks passed. Address single responsibility issue.
```

## Future Checklist Items

Additional checks will be added here as the repository evolves:
- Code formatting standards
- Test coverage requirements
- Documentation completeness
- Dependency management
- Security best practices
