# Testing Skills with Subagents (optional)

Heavy method for **discipline-enforcing skills** — skills whose whole point is to make an agent do something it would rationalize away under pressure (test-first, verify-before-done, no-shortcuts). For technique or reference skills this is overkill; a read-through plus one application scenario is enough.

Core idea, **for discipline skills only**: writing the skill IS test-driven development applied to documentation. Same Iron Law — for *this class of skill*, no version is trustworthy until you watch an agent fail WITHOUT it. If you didn't see the baseline failure, you don't know the skill teaches the right thing. (Technique/reference skills don't need this — see SKILL.md.)

## RED — capture the baseline failure

Run a pressure scenario with a subagent that does NOT have the skill. Stack multiple pressures: time ("ship in 5 min"), sunk cost ("you already wrote it"), authority ("the senior dev said skip it"), exhaustion ("last task of the day").

Record **verbatim**:
- What choice did the agent make?
- What exact words did it use to rationalize the violation?
- Which pressure triggered it?

These verbatim rationalizations are your test cases. You are not guessing what to forbid — the agent tells you.

## GREEN — write the minimal skill

Write counters for the **specific** rationalizations you captured. Nothing speculative. Re-run the same scenarios WITH the skill present. The agent should now comply.

## REFACTOR — close new loopholes

The agent will find a new escape hatch. Add an explicit counter, re-run, repeat until bulletproof.

## Bulletproofing techniques

**Close loopholes by name** — don't just state the rule, forbid the specific workaround:

```
❌ "Write code before the test? Delete it."
✅ "Write code before the test? Delete it. Start over.
    No exceptions: don't keep it as reference, don't 'adapt' it,
    don't look at it. Delete means delete."
```

**Cut the spirit-vs-letter argument** — add early:
> Violating the letter of the rule is violating the spirit of the rule.

**Build a rationalization table** from every excuse the baseline produced:

| Excuse | Reality |
|---|---|
| "Too simple to test" | Simple code breaks. Test takes 30s. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Following the spirit, not the ritual" | The ritual IS the spirit. Do it. |

**Red-flags list** so the agent can self-catch mid-rationalization:
> STOP and start over if you think: "I already tested manually" · "tests-after are the same" · "this case is different because…"

## When you're done

The skill is bulletproof when the subagent complies under the maximum combined pressure that broke it in RED. Not before.
