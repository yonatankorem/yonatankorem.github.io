---
name: prd-to-plan
description: Transform a PRD into a detailed, phase-based implementation plan. Use when the user has a PRD and needs an execution roadmap broken into shippable phases with atomic tasks.
---

You are tasked with transforming a Product Requirements Document (PRD) into a detailed implementation plan.

## Input

The user will provide a path to a PRD file, typically in `docs/prd/`.

## Your Process

### 1. Read and Analyze the PRD

Read the PRD thoroughly and extract:
- Core requirements and system changes
- Components and subsystems involved
- Technical constraints and considerations
- Success criteria and validation needs
- Any suggested phases (if present, use as guidance but don't treat as final)

### 2. Explore the Codebase

Use the Agent tool with subagent_type=Explore to understand:
- **Existing architecture**: What patterns are already established?
- **Affected modules**: Which files/directories will need changes?
- **Integration points**: Where do new components connect to existing systems?
- **Dependencies**: What relies on what? What must be built first?
- **Complexity assessment**: Is this a simple addition or complex refactor?
- **Technical debt**: Are there existing issues that will complicate implementation?

Request "very thorough" exploration when the PRD involves:
- Core system changes or refactors
- Multiple subsystems interacting
- Breaking changes or migration scenarios

Request "medium" exploration for:
- New feature additions with existing pattern guidance
- UI changes that follow established component patterns

### 3. Design Phases

Each phase must be:
- **Independently shippable**: Can be deployed without breaking the system
- **Testable**: Has clear verification criteria
- **Complete**: Advances the system meaningfully, even if incrementally
- **Foundation-building**: Later phases can build on it

**Phase boundaries should optimize for the appropriate goal:**
- **Risk mitigation**: Early phases validate the riskiest assumptions or most complex parts
- **Fast feedback**: Early phases enable user testing or validation of core concepts
- **Parallel work**: Phases can be designed so multiple developers work simultaneously
- **Incremental value**: Each phase delivers something usable

### 4. Break Phases into Atomic Tasks

Each task should be:
- **One commit**: A single, focused change
- **Independently reviewable**: Can be understood in isolation
- **Actionable**: Clear what needs to be done

Good task examples:
- "Create Task interface in shared/types.ts"
- "Implement POST /api/tasks endpoint with validation"
- "Add TaskSubmission component with form fields"
- "Write integration test for task submission flow"

Bad task examples:
- "Build the backend" (too broad)
- "Add features and fix bugs" (multiple concerns)
- "Make it work" (unclear)

### 5. Add Validation Requirements (Optional per Phase)

For each phase, determine if it needs:
- **Acceptance criteria**: What must be true for this phase to be "done"?
- **Test strategy**: How do we verify it works? (unit tests, integration tests, manual testing)
- **Rollback procedure**: If something goes wrong, how do we revert? (Only for risky phases)
- **Verification steps**: Specific commands or checks to run

Not every phase needs all of these. Use judgment:
- Infrastructure setup: needs verification steps
- Database changes: needs rollback procedure
- UI polish: may just need visual QA
- API endpoints: needs test strategy

## Output Format

Write the implementation plan to: `docs/plans/implementation-plan-{feature-name}.md`

Use this structure:

```markdown
# Implementation Plan: {Feature Name}

**PRD**: [Link to PRD file]
**Created**: {Date}
**Status**: Ready for Implementation

---

## Overview

[1-2 paragraph summary of what will be built and the overall approach]

## Codebase Context

**Existing Patterns Identified:**
- [Pattern 1 and where it's used]
- [Pattern 2 and where it's used]

**Modules to Modify:**
- `path/to/module1` - [why]
- `path/to/module2` - [why]

**New Modules to Create:**
- `path/to/new/module1` - [purpose]
- `path/to/new/module2` - [purpose]

**Dependencies:**
- [Dependency 1]: [why needed]
- [Dependency 2]: [why needed]

**Integration Points:**
- [Where new code connects to existing systems]

**Complexity Assessment:**
- Overall complexity: Low/Medium/High
- Risk areas: [List any concerning areas]
- Technical debt impact: [Any existing issues that complicate this work]

---

## Implementation Phases

### Phase 1: {Phase Name}

**Goal**: [What does this phase accomplish?]

**Why Now?**: [Explain the rationale - risk mitigation, foundation building, etc.]

**Shippable State**: [What can be deployed/tested after this phase?]

**Tasks**:
1. [Atomic task 1]
   - File(s): `path/to/file.ts`
   - Description: [Brief clarification if needed]

2. [Atomic task 2]
   - File(s): `path/to/file.ts`
   - Description: [Brief clarification if needed]

[Continue for all tasks...]

**Acceptance Criteria** (optional):
- [ ] [Criterion 1]
- [ ] [Criterion 2]

**Test Strategy** (optional):
- [How to verify this phase works]
- Commands to run: `npm test ...`

**Rollback Procedure** (optional):
- [Steps to revert if this phase causes issues]

**Verification**:
```bash
# Commands to verify phase completion (always prefer mise tasks or package.json scripts)
```

---

### Phase 2: {Phase Name}

[Same structure as Phase 1...]

---

[Repeat for all phases...]

---

## Dependencies Summary

**External Dependencies to Install**:
- `package-name@version` - [why needed]

**Build Order**:
1. Phase 1 must complete first
2. Phases 2 and 3 can be done in parallel
3. Phase 4 requires Phases 2 and 3

---

## Risk Assessment

**High Risk Areas**:
- [Risk 1]: [Mitigation strategy]
- [Risk 2]: [Mitigation strategy]

**Known Unknowns**:
- [What we don't know yet that might affect the plan]

---

## Success Criteria

The implementation is complete when:
1. [Overall success criterion 1]
2. [Overall success criterion 2]
3. All phase acceptance criteria are met
4. [PRD success criteria from original document]

---

## Notes

[Any additional context, warnings, or suggestions for implementers]
```

## Key Principles

1. **Phases are hard checkpoints**: Each phase is a stopping point where you must pause, ask for human intervention, deploy, test, or pivot

2. **Tasks are commits**: Granularity should match git commit best practices - atomic, focused, reviewable

3. **Codebase-driven**: The plan must reflect the actual state of the code, not idealized architecture

4. **Flexible optimization**: Don't force a single philosophy (e.g., "always de-risk first") - optimize each phase for what matters most in that context

5. **Optional validation**: Not every phase needs extensive testing or rollback procedures - use judgment

6. **Clear rationale**: Explain WHY phases are ordered this way so implementers understand the thinking

## After Completion

Report to the user:
- Path to the implementation plan file
- Number of phases created
- Total task count
- Any risks or concerns identified
- Suggested next steps
