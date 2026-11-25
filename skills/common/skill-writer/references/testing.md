# Checking a skill

A skill that loads is not a skill that works. Check triggering and effect separately, and always in a fresh session: the session that wrote the skill already holds the context the skill is meant to supply, so it hides the gaps in the written text.

## 1. Triggering

Write a handful of realistic requests of two kinds:

- **Should trigger**: include indirect phrasings, and every language the users write in.
- **Should not trigger**: near-misses that share keywords, and requests that belong to a neighboring skill.

Run each in a fresh session and note whether the skill was loaded. Fix misses by editing the description only, within its 100-byte budget:

| Result | Fix |
|---|---|
| A request that should trigger did not | Swap in the user's actual wording or symptom |
| A near-miss triggered | Narrow the capability clause, or add a "not for" boundary naming the neighbor |
| The skill should never trigger on its own | That is an invocation setting of the agent, where it has one, not a wording problem. Check the agent's documentation; for Claude Code see [claude-code.md](claude-code.md) |

## 2. Effect

Pick a few realistic tasks. Run each twice in fresh sessions, once with the skill and once with it turned off, and compare the results.

| Observation | Meaning |
|---|---|
| A rule makes no difference | The agent already does it. Cut the line |
| The run without the skill fails in a way the skill does not address | That is the missing line. Add it in terms of the observed failure, nothing speculative |
| The run with the skill ignores a rule | The rule is buried, placed late, or phrased as a one-time step. Move it up or reword it before making it louder |

To turn a skill off for the comparison run, move its directory out of the agent's skills folder. Some agents have a setting for this instead; Claude Code's is in [claude-code.md](claude-code.md).

Repeat both checks in each agent the skill is meant for: triggering in particular depends on how that agent lists and shortens descriptions.

## Rules that keep getting skipped

When the agent follows a rule at first and drops it later, or skips it under pressure, stronger wording is the wrong tool. Repetition, capitals, and lists of forbidden excuses add context cost on every turn and still guarantee nothing.

- **Must hold every time** → enforce it outside the skill: a hook, a CI check, a linter.
- **Should hold with judgment** → word it as a standing instruction that covers the whole task, and put it near the top.

## Automating the loop

When the agent has a skill-evaluation tool, use it once there are more than a few cases. One example is the `skill-creator` skill: it stores test prompts in `evals/evals.json`, runs the with/without comparison in isolated subagents, measures should-trigger and should-not-trigger hit rates, and proposes description edits.
