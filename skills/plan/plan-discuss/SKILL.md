---
name: plan-discuss
description: Use when discussing or designing a plan with the user, before writing any plan files or code — exploring approaches, refining requirements, deciding architecture. Use when the user wants to think through an approach and a large one-shot design dump would overload them or pre-commit to detail too early.
---

# plan-discuss

## Overview

How to conduct the **discussion phase** of planning: think breadth-first, surface one point at a time, drill down only after probing feasibility. This phase is for *thinking clearly together*, not for producing files.

**Core principle:** Agents default to depth-first — they hit a node and expand it all the way to implementation detail. Humans plan breadth-first — confirm the top frame, glance at each lower node only to check it's solvable, return up, and only drill deep once the top is frozen. Premature detail wastes effort (the frame will change) and overloads the user.

**Terminal state:** When the top-level frame is agreed and frozen, hand off to `plan-write` to lay it out as files. Do NOT write plan files or code during discussion.

## When to Use

- User wants to discuss an approach, feature, or architecture before building
- Requirements are still fluid; multiple approaches are on the table
- Any time you'd otherwise dump a long multi-point design in one message

**When NOT to use:** the plan is already agreed and you're writing it down (use `plan-write`); trivial single-step changes.

## The Five Behaviors

### 1. Breadth-first
Define the top frame first. A lower node is examined only to answer "is there a viable path?" — if yes, return up immediately, don't expand it. Drill into detail only after the top is locked. The frame will be revised repeatedly; detail written under an unstable frame is wasted.

### 2. One point at a time
A turn raises ONE thing to discuss — one question, one decision, one piece of analysis. Never merge multiple independent points into one message. The user needs to respond point-by-point; merged points can't be answered separately.

### 3. Ping-pong rhythm
Short turns, frequent back-and-forth. Raise → wait for response → next point. Avoid one-shot long output. The slow pace is intentional — discussion is about getting it right, not maximizing information per message.

### 4. Probe before drilling
Before expanding any branch into deeper detail, run the five-question feasibility probe. If any answer fails, stay at the current layer, return "blocker + minimal missing info", do NOT speculate implementation.

```text
[Goal]          What is this branch, in one sentence?
[Necessity]     Why expand it now? Which decision stalls if we don't?
[Solvability]   Is there at least one clearly viable path? What's missing — data, interface, permission, resource, or user confirmation?
[Verifiability] Can the next layer's output be checked/accepted? By what criteria?
[Reversibility] If the top changes, can this expansion compress back to a summary?

[Conclusion]    drill / don't drill. If drill: which layer. If not: return up + 1-3 blockers.
```

### 5. Fold when top changes
When the top frame is adjusted, compress already-discussed deep detail back to a 3–5 line summary — keep confirmed constraints, key trade-offs, open risks, next entry point. Drop implementation minutiae. Don't rewrite detail.

## Scope

Applies ONLY during plan discussion. It does NOT constrain: writing code, executing, exploring a repo, factual Q&A — those get normal complete answers.

## Handoff

Once the top-level frame is user-approved and frozen:

1. State that the frame is frozen.
2. Note: discussion is for thinking; execution needs full context, so the frozen decisions will be written out completely — not left as fragments.
3. Invoke `plan-write` to lay the agreed frame into a layered multi-file plan directory.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Dumping the whole design in one message | One point per turn. Wait for response. |
| Drilling into a branch before the frame is agreed | Confirm the top first; only glance at branches for feasibility. |
| Expanding a branch without a probe | Run the five questions; if any fail, return up with blockers. |
| Rewriting detail after the frame changes | Fold it to a summary instead. |
| Writing plan files during discussion | Discussion produces agreement, not files. Hand off to plan-write. |
