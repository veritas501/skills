---
name: skill-writer
description: Use when creating, writing, editing, or reviewing an agent skill (SKILL.md). Triggers on "write a skill", "create a skill", "新建 skill", "写个 skill", "skill 怎么写", or when authoring reusable agent capabilities, behavior guidance, or domain references. Also use when a skill is bloated with content the agent already knows.
---

# Skill Writer

Constraints and decisions for authoring an agent skill. You already know YAML, markdown, and how to write clear prose — this skill only adds the non-obvious rules that keep a skill **complementary to the agent, not redundant with it**.

**Not for:** project conventions (→ CLAUDE.md), one-off solutions, mechanically-enforceable rules (→ lint/validation/regex).

## The one principle that decides everything

**A skill is a supplement to the agent's existing ability and a guide for its behavior — NOT a re-statement of what the agent already knows.**

The agent already knows general programming, common workflows, standard tools, and obvious steps. Writing those down wastes tokens and buries the few lines that actually matter. Every line must earn its place by passing one test:

> **"Will the agent do this wrong if it does NOT read this line?"**

- **Yes → write it in full.** The agent doesn't know: domain knowledge, gotchas, counter-intuitive constraints, specific tool/API behavior, discipline it would skip under pressure.
- **No, but it still needs anchoring → compress to a minimal constraint.** The agent already knows *how*, but you still want to pin the *choice* (which option, what order, where the boundary is). Don't delete — shrink to one line. Example: don't explain how to handle errors, but do say "fail closed, not open" if that decision matters.
- **No, and no constraint needed → cut it entirely.** Pure redundancy: generic programming, common sense, self-evident steps the agent gets right unprompted.

This is what "详略得当" (right level of detail) means in practice — not a binary keep/delete, but three tiers: **full detail** for the unknown, **minimal one-line constraint** for known-but-must-be-pinned, **nothing** for pure redundancy. Write only the delta between what the agent knows and what the task needs.

### Detail calibration table

| Content | Tier | Why |
|---|---|---|
| ❌ "Wrap the call in try/except" | Cut | Agent knows error handling |
| ❌ "Write clear variable names" | Cut | Agent knows clean code |
| ❌ Step-by-step of an obvious CLI flow | Cut | Agent can drive standard tools |
| ⚠️ "On failure, fail closed, not open" | Min constraint | Agent knows *how* to handle errors; pin *which* choice |
| ⚠️ "Use the repo's existing logger, don't add a new dep" | Min constraint | Known skill, but the boundary must be anchored |
| ✅ "This API returns 200 with an `error` field in body on failure, not 5xx — check the body" | Full | Counter-intuitive, agent would miss it |
| ✅ "Run migrations before seeding or the FK constraint fails" | Full | Order-dependent gotcha |
| ✅ "Delete the code if you wrote it before the test — no exceptions" | Full | Discipline the agent skips under pressure |
| ✅ Exact schema / enum values / version-specific flags | Full | Agent can't guess these |

When unsure, ask: *would a competent agent who never saw this skill get it right anyway?* If yes and no choice needs pinning, delete. If yes but a specific choice matters, keep one line.

## When to write a skill

Write when ALL hold:
- The technique/knowledge is **non-obvious** to a capable agent.
- It will be **reused** across tasks or projects.
- It encodes a **judgment call** (not mechanically enforceable — if regex/lint can catch it, automate instead).

Don't write for: one-off fixes, well-documented standard practice, project-specific conventions, or anything a validator could enforce.

## Skill structure

```
skill-name/
  SKILL.md            # required — the skill itself
  references/         # optional — heavy reference, loaded on demand
  scripts/            # optional — deterministic tools the agent would otherwise re-generate
```

**Keep inline:** principles, judgment calls, short patterns, the detail-calibration the agent needs every time.
**Split out:** large API/syntax references, reusable scripts. Reference them by name; never `@`-link (force-loads and burns context).

## Frontmatter

Two required fields, max 1024 chars total:

- `name`: letters, numbers, hyphens only. Active, verb-first where natural (`skill-writer`, `condition-based-waiting`).
- `description`: **third person, triggering conditions ONLY.** Start with "Use when…". Pack in concrete keywords the agent would search for: error messages, symptoms, synonyms, file types, both English and 中文 triggers if relevant.

**Critical trap — never summarize the workflow in the description.** If the description describes the *process*, the agent follows the description and skips the skill body. State *when to load*, not *what to do*.

```yaml
# ❌ summarizes workflow → agent skips the body
description: Use when writing skills — set frontmatter, write body, then test with subagents

# ✅ triggers only
description: Use when creating, writing, or editing an agent skill (SKILL.md)
```

## What goes in the body

Recommended sections (drop any that don't apply — don't pad):

1. **One-line framing** — what the agent already knows vs. what this skill adds (like the paragraph under the title here).
2. **Core principle** — the single insight, stated once, sharply.
3. **When to use / when NOT** — symptoms and triggers; the "not for" list prevents misfire.
4. **The non-obvious rules** — tables, gotchas, constraints. This is the payload.
5. **Common mistakes** — what goes wrong + the fix.

Use a small flowchart ONLY for a non-obvious decision point. Reference material → tables/lists, never flowcharts. Linear steps → numbered list.

## Testing (optional, for discipline skills)

For skills that **enforce discipline under pressure** (e.g. "always test first", "always verify before claiming done"), agents rationalize their way out. Optionally pressure-test: run a scenario WITHOUT the skill to capture the agent's natural rationalizations, write counters for those exact excuses, then re-run WITH the skill. Build a rationalization table from what you find. See [references/testing-optional.md](references/testing-optional.md) for the full RED-GREEN-REFACTOR subagent method.

For technique/reference skills, this is usually overkill — a careful read-through and one application scenario suffice.

## Common mistakes

| Mistake | Fix |
|---|---|
| Re-explaining what the agent already knows | Apply the "will it do this wrong without this line?" test; cut anything that fails. |
| Description summarizes the workflow | Rewrite to triggering conditions only — "Use when…". |
| Vague description ("helps with skills") | Add concrete keywords, symptoms, file types, 中文 triggers. |
| Everything inline, 600-line SKILL.md | Split heavy reference to a separate file, load on demand. |
| Flowchart for reference data | Use a table. Flowcharts only for non-obvious decisions. |
| Padding for completeness | A skill is a delta, not a textbook. Short and load-bearing beats long and obvious. |
