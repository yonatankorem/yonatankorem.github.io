# Deep-Dive Interview Guidelines

## Two-Phase Approach

### Phase 1: Feature Discovery (WHAT/WHY)
Focus on understanding **requirements** before discussing **implementation**.

**Questions to explore:**
- What are we building and why?
- What does it need to do?
- What decisions need to be made?
- What are the safety boundaries?
- What's the user experience?
- What are the success criteria?

**Follow the decision tree naturally.** Any answer might reveal critical dependencies. Don't pre-order questions by "importance" - let the conversation reveal what matters.

### Phase 2: Implementation Discovery (HOW)
Once requirements are clear, drill into technical approach.

**Questions to explore:**
- How is this technically built?
- What's the tech stack?
- How do components connect?
- What's the data flow?
- What are the edge cases?

**Many implementation questions answer themselves** once requirements are well-understood. If requirements are unclear, implementation decisions are premature.

### Explicit Transition
When moving from Phase 1 → Phase 2, say it explicitly:

> "I think we've covered the feature requirements. Ready to drill into implementation, or any gaps in what we're building?"

This gives the user a chance to:
- Catch missed feature questions
- Confirm we're ready to shift gears
- Redirect if we jumped to implementation too early

## Example Flow

```
[Feature Phase]
Q1: What's the core goal?
   → Answer reveals need for analytics
Q2: What analytics matter?
   → Answer reveals need for decision framework
Q3: What criteria should decisions use?
   → Answer reveals safety boundaries
Q4: Where are those boundaries?
   → Answer reveals need for human review category
Q5: When does agent need human help?
   → Feature requirements clear

[Transition]
"I think we've covered the feature requirements.
Ready to drill into implementation, or any gaps?"

[Implementation Phase]
Q6: How should user submit tasks?
   → Answer reveals need for backend
Q7: What storage approach?
   → Answer reveals need for filesystem structure
Q8: How should agent access files?
   → Implementation clear

```

---

## Remember

- **Follow the tree**, don't pre-plan the route
- **Context from prior answers** often makes later questions obvious
- **User knows their goals** - drill to expose hidden assumptions, not to impose structure
- **Two phases** (feature → implementation) with explicit transition