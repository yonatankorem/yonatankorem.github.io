# Decision Framework Iteration Guide

This guide explains how to improve the task-reviewer agent's decision-making based on real-world outcomes.

## Overview

The decision framework will need continuous refinement as it processes actual task submissions. This document provides a systematic approach to tracking performance, identifying issues, and updating criteria.

## Metrics to Track

### 1. Decision Distribution

Track the percentage of tasks in each category:

```bash
# Count tasks in each status directory
ls -1 suggestions/pending/ | wc -l
ls -1 suggestions/approved/ | wc -l
ls -1 suggestions/rejected/ | wc -l
ls -1 suggestions/needs-review/ | wc -l
```

**Target Ranges** (from PRD):
- **Approved**: 30-40% of total submissions
- **Rejected**: 20-30% of total submissions
- **Needs Review**: <50% of total submissions

**Red Flags**:
- >50% needs-review: Framework is too uncertain, needs more decisive criteria
- <20% needs-review: Framework may be overconfident, missing edge cases
- Very low approval or rejection rates: Criteria may be imbalanced

### 2. False Positives (Auto-Approved but Risky)

When a task is approved but shouldn't have been:

**Example Scenario**: Agent approves "Refactor authentication system" → Task turns out to be high-risk security change

**How to Track**:
1. Review approved tasks manually (sample or full review)
2. Identify tasks that should have been rejected or flagged
3. Document in `docs/metrics/false-positives.md`

**Calculate Rate**:
```
False Positive Rate = False Positives / Total Approved
Target: <5%
```

**Action**: If >5%, tighten approval criteria

### 3. False Negatives (Auto-Rejected but Safe)

When a task is rejected but should have been approved:

**Example Scenario**: Agent rejects "Add dark mode CSS variables" → Task is actually low-risk styling change

**How to Track**:
1. Review rejected tasks manually
2. Identify safe, low-risk tasks that were incorrectly rejected
3. Document in `docs/metrics/false-negatives.md`

**Calculate Rate**:
```
False Negative Rate = False Negatives / Total Rejected
Target: <10%
```

**Action**: If >10%, loosen rejection criteria or add exceptions

### 4. Common Rejection Reasons

Analyze the reasoning field in rejected task analysis files:

```bash
# Extract rejection reasons
find suggestions/rejected/ -name "*.json" -exec jq -r '.analysis.reasoning' {} \;
```

**Look for Patterns**:
- Are certain phrases triggering rejection?
- Are legitimate low-risk tasks using terminology that sounds risky?
- Are there task types consistently rejected that should be approved?

### 5. Needs-Review Quality

For tasks flagged as needs-review, track:
- What percentage are ultimately approved after human review?
- What percentage are ultimately rejected?
- What additional context helped make the decision?

**Pattern Recognition**:
- If most needs-review tasks get approved → Criteria too cautious
- If most needs-review tasks get rejected → Criteria too permissive
- If distribution is balanced → Criteria are appropriately calibrated

## When to Adjust Criteria

### Trigger Conditions

Update the decision framework when:

1. **False positive rate >5%** for 2+ weeks
   - High-risk tasks are being auto-approved
   - Risk: Production incidents from approved tasks

2. **False negative rate >10%** for 2+ weeks
   - Safe tasks are being unnecessarily rejected
   - Risk: Bottleneck from over-conservative framework

3. **Needs-review rate >50%** consistently
   - Agent is too uncertain about most tasks
   - Risk: No efficiency gain from automation

4. **New task patterns emerge** not covered by current criteria
   - Example: Team adopts new tool or framework
   - Example: New category of tasks (performance, accessibility)

5. **Repeated human overrides** on specific task types
   - If humans consistently disagree with agent decisions on a category
   - Example: Agent rejects all "optimize X" tasks, but humans approve them

### Don't Adjust When

Avoid changing criteria if:
- Only 1-2 weeks of data (insufficient sample size)
- Recent changes haven't been tested adequately
- Edge case occurs once (outliers are expected)
- Metrics are within target ranges
- Team is still learning the system

**Rule of Thumb**: Wait for at least 20-30 task submissions before making framework changes.

## How to Update the Framework

### Step 1: Identify the Issue

Document the specific problem:
- What type of tasks are being misclassified?
- What reasoning is the agent using?
- What decision should have been made?

**Example**:
```
Issue: Agent approves all "Add [UI feature]" tasks
Reality: Some UI features touch authentication (risky)
Problem: Agent doesn't check for security implications in UI changes
```

### Step 2: Locate Relevant Criteria

Find the section in `.claude/agents/task-reviewer.md` that governs these decisions:

- Approval criteria (APPROVED section)
- Rejection criteria (REJECTED section)
- Review criteria (NEEDS REVIEW section)
- Risk assessment guidance
- Decision process steps

### Step 3: Draft Updated Criteria

Write new or modified criteria that address the issue:

**Before**:
```markdown
**APPROVED**:
- UI styling changes with no functional impact
```

**After**:
```markdown
**APPROVED**:
- UI styling changes with no functional impact
- Exception: UI changes touching authentication, authorization, or
  payment flows require NEEDS REVIEW (security boundary)
```

### Step 4: Test with Historical Data

Before deploying, test the updated criteria:

1. Copy 10-20 past task submissions back to `pending/`
2. Run `/review-tasks` with updated agent definition
3. Compare new decisions with old decisions
4. Verify the change fixes the issue without introducing new problems

**Check**:
- Do false positives decrease?
- Do false negatives stay low?
- Does needs-review rate remain reasonable?

### Step 5: Deploy and Monitor

1. Commit the updated `.claude/agents/task-reviewer.md`
2. Document the change in commit message with rationale
3. Monitor metrics for next 20-30 submissions
4. Be prepared to revert if metrics worsen

**Commit Template**:
```bash
git commit -m "refactor(agent): tighten approval criteria for UI changes

Add exception for UI features touching auth boundaries after observing
false positives in security-sensitive UI tasks.

Metrics before: 8% false positive rate
Expected after: <5% false positive rate
"
```

### Step 6: Document the Change

Update this guide or create a changelog entry:
- What was changed and why
- What metrics triggered the change
- What the expected impact is
- Date of change

## Common Adjustments

### Scenario 1: Too Many Needs-Review

**Symptom**: >50% of tasks flagged for human review

**Diagnosis**: Decision criteria are too vague or conservative

**Solution**:
1. Review needs-review tasks to find common themes
2. Create more specific approval/rejection criteria for common patterns
3. Add examples to agent definition showing clear approve/reject cases

**Example Fix**: Add specific guidance for testing tasks
```markdown
## Testing Tasks
- Unit tests for existing features: APPROVED
- Integration tests without new infra: APPROVED
- E2E tests requiring new services: NEEDS REVIEW
```

### Scenario 2: False Positives (Risky Approvals)

**Symptom**: Agent approves tasks that cause production issues

**Diagnosis**: Approval criteria are too broad or miss edge cases

**Solution**:
1. Add exclusions to approval criteria
2. Strengthen risk assessment guidance
3. Add examples of similar tasks that should be rejected

**Example Fix**: Add security boundary check
```markdown
## Risk Assessment - Security Implications
Check for:
- Authentication/authorization changes
- Data access control modifications
- Input validation bypasses
- Cryptographic operations
If present → REJECTED or NEEDS REVIEW
```

### Scenario 3: False Negatives (Safe Rejections)

**Symptom**: Agent rejects low-risk tasks like documentation or styling

**Diagnosis**: Rejection criteria are too broad or keyword-based

**Solution**:
1. Add explicit exceptions for safe task categories
2. Clarify that certain keywords don't automatically mean high risk
3. Add examples of "looks risky but isn't" scenarios

**Example Fix**: Whitelist safe operations
```markdown
## Safe Operations (Generally Approved)
- Documentation updates (README, comments, guides)
- CSS styling without logic changes
- Adding unit tests with mocked dependencies
- Fixing typos or formatting
- Adding logging/observability (non-PII)
```

### Scenario 4: New Task Pattern Emerges

**Symptom**: Team starts submitting new type of tasks not covered by criteria

**Diagnosis**: Criteria don't account for new patterns

**Solution**:
1. Create new criteria section for this task category
2. Define approval/rejection rules specific to the pattern
3. Add 2-3 examples of each decision type

**Example**: Team adopts Storybook for component documentation
```markdown
## Storybook/Component Documentation
- Adding stories for existing components: APPROVED
- Adding visual regression tests: APPROVED
- Major Storybook version upgrade: NEEDS REVIEW
- Integrating Storybook with CI/CD: NEEDS REVIEW
```

## Iteration Cycle

Follow this rhythm for continuous improvement:

### Weekly Review (Light)
- Check decision distribution metrics
- Spot-check 5-10 recent decisions
- Note any obvious errors for next monthly review

### Monthly Review (Deep)
1. Calculate all metrics (false positive/negative rates, distribution)
2. Review all flagged errors from weekly checks
3. Identify patterns requiring framework updates
4. Test proposed changes with historical data
5. Deploy changes if validated
6. Document changes and expected impact

### Quarterly Assessment (Strategic)
1. Analyze trends over time (are metrics improving?)
2. Evaluate if framework is achieving efficiency goals
3. Consider major criteria overhauls if needed
4. Update this iteration guide based on learnings
5. Share findings with team for feedback

## Tools and Scripts

### Calculate Metrics Script

Create `scripts/calculate-metrics.sh`:
```bash
#!/bin/bash

TOTAL=$(find suggestions/{approved,rejected,needs-review} -name "*.json" | wc -l)
APPROVED=$(find suggestions/approved -name "*.json" | wc -l)
REJECTED=$(find suggestions/rejected -name "*.json" | wc -l)
NEEDS_REVIEW=$(find suggestions/needs-review -name "*.json" | wc -l)

echo "Decision Distribution:"
echo "  Approved: $APPROVED / $TOTAL ($(( 100 * APPROVED / TOTAL ))%)"
echo "  Rejected: $REJECTED / $TOTAL ($(( 100 * REJECTED / TOTAL ))%)"
echo "  Needs Review: $NEEDS_REVIEW / $TOTAL ($(( 100 * NEEDS_REVIEW / TOTAL ))%)"
```

### Extract Common Rejection Reasons

```bash
# Find most common rejection keywords
find suggestions/rejected -name "*.json" -exec jq -r '.analysis.reasoning' {} \; \
  | tr '[:upper:]' '[:lower:]' \
  | grep -oE '\b\w+\b' \
  | sort | uniq -c | sort -rn | head -20
```

### Find High-Risk Approvals

```bash
# Find approved tasks with "high" risk indicators in description
find suggestions/approved -name "*.json" -exec sh -c '
  jq -r ".description" "$1" | grep -iE "(delete|remove|refactor|migration|upgrade|auth)" && echo "$1"
' _ {} \;
```

## Versioning the Framework

Track framework versions in `.claude/agents/task-reviewer.md`:

```markdown
---
name: task-reviewer
version: 2.1.0  # Add this line
lastUpdated: 2026-03-25
---
```

**Semantic Versioning**:
- **Major** (2.0.0): Complete criteria overhaul
- **Minor** (2.1.0): New criteria section or significant change
- **Patch** (2.1.1): Small clarification or example addition

Keep a changelog in the agent definition:
```markdown
## Changelog

### v2.1.0 (2026-04-15)
- Added security boundary checks for UI changes
- Reduced false positive rate from 8% to 4%

### v2.0.0 (2026-03-25)
- Initial framework deployment
```

## Success Indicators

The framework is working well when:

1. **Metrics are stable** within target ranges for 4+ weeks
2. **Human overrides are rare** (<10% of decisions)
3. **Team trusts the system** (no complaints about obvious errors)
4. **Iteration frequency decreases** (fewer adjustments needed)
5. **Efficiency gain is measurable** (time saved vs manual review)

## Red Flags

Stop and reassess if:

1. **Metrics oscillate wildly** week to week
2. **Frequent framework changes** (>2 per month)
3. **High human override rate** (>20%)
4. **Team loses confidence** in agent decisions
5. **False positive rate increases** despite adjustments

In these cases, consider:
- Reverting to earlier framework version
- Consulting with team on criteria philosophy
- Switching to human-only review temporarily
- Redesigning criteria from scratch

## Conclusion

Framework iteration is about continuous learning, not perfection. Start conservative, collect data, identify patterns, make targeted changes, and validate outcomes. Over time, the system will become increasingly accurate and valuable.

Remember: The goal is to augment human decision-making, not replace it. Some tasks will always need human judgment—that's by design.
