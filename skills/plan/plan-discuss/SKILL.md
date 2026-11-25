---
name: plan-discuss
description: Plan a non-trivial change with the user, settling goal, scope, and approach before any files.
---

# plan-discuss

Settle the frame of one change with the user through discussion. This phase produces agreement, not files: write no plan files and no code until it ends.

Agents default to depth-first: they reach a node and expand it down to implementation detail. Planning works breadth-first: agree on the top, look at each lower node only long enough to see that it is solvable, and go deep only once the top is fixed. Detail written under a frame that later changes is wasted, and it buries the decisions the user has to make.

**Not for:** a change whose frame is already agreed and only needs writing down (use `plan-write`), or a trivial single-step change.

## The frame

The discussion ends when the user has agreed on all five items:

| Item | Settles |
|---|---|
| Goal | What changes, and why |
| Non-goals | What stays out, especially what a reader would assume is included |
| Architecture | Which modules are touched, who owns what, where the boundaries are |
| Technology | What is kept and what is introduced |
| Acceptance | The observable criteria that show the change is done |

An item may be settled as "unchanged" or "not applicable", but the user says so; do not assume it.

## How to discuss

1. **Breadth-first.** Work on the five items before anything below them. Look into a lower node only to answer "is there a viable path?", then return.
2. **One point per turn.** Raise one question, decision, or piece of analysis, with your recommendation when you have one, and wait for the answer. Several independent points in one message cannot be answered separately.
3. **Short turns.** The slow pace is intended; do not compensate with long messages.
4. **Look before asking.** Read the repository for anything it can answer. Ask the user only for decisions and for facts only they hold.
5. **Probe before drilling.** Before expanding a branch into deeper detail, check it against the five questions below. This is a check you run yourself: tell the user the conclusion, not the worksheet. If any answer fails, stay at the current level and name one to three blockers and the minimum information that would clear them; do not speculate about implementation.
6. **Fold when the top changes.** When a frame item changes, compress the detail already discussed under it to a few lines (confirmed constraints, key trade-offs, open risks) and continue from there. Do not rewrite the detail.

```text
Goal           What is this branch, in one sentence?
Necessity      Which decision stalls if it is not expanded now?
Solvability    Is there at least one clearly viable path? What is missing:
               data, an interface, a permission, a resource, a user decision?
Verifiability  Can the next level's output be checked, and by what criteria?
Reversibility  If the top changes, can this expansion fold back to a summary?
```

These rules apply to the planning discussion only. A factual question asked along the way gets a normal, complete answer.

## Handoff

When all five items are settled:

1. Write the frame out in full in one message: the five items, the constraints confirmed along the way, and the risks still open. This is the one long message of the phase. `plan-write` works from it, possibly after the earlier conversation is no longer available, so fragments are not enough.
2. Ask the user to confirm it. A correction reopens the discussion at that item.
3. After confirmation, continue with `plan-write`, unless the user asked only for the discussion.
