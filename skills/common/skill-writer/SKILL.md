---
name: skill-writer
description: Write, edit, or review an agent skill (SKILL.md), or fix one that is bloated or mis-triggers.
---

# Skill Writer

You already know YAML, markdown, and clear prose. This skill adds the decisions that keep a skill **complementary to the agent rather than redundant with it**, and the format rules an agent cannot guess.

## The test every line must pass

A loaded skill stays in context for the rest of the session, so each line is a recurring cost and each redundant line buries one that matters. Hold every line to one question:

> **"Will the agent do this wrong if it does NOT read this line?"**

- **Yes → write it in full.** Domain knowledge, gotchas, counter-intuitive constraints, exact tool or API behavior, order dependencies.
- **No, but a choice must be pinned → one line.** The agent knows *how*; pin *which*: which option, what order, where the boundary is.
- **No → cut it.** Generic programming, common sense, steps the agent gets right unprompted.

| Content | Tier | Why |
|---|---|---|
| "Wrap the call in try/except" | Cut | Agent knows error handling |
| "Write clear variable names" | Cut | Agent knows clean code |
| Step-by-step of an obvious CLI flow | Cut | Agent can drive standard tools |
| "On failure, fail closed, not open" | One line | Agent knows *how*; pin *which* choice |
| "Use the repo's existing logger, don't add a new dep" | One line | Known skill, but the boundary must be anchored |
| "This API returns 200 with an `error` field on failure, not 5xx — check the body" | Full | Counter-intuitive, agent would miss it |
| "Run migrations before seeding or the FK constraint fails" | Full | Order-dependent gotcha |
| Exact schema / enum values / version-specific flags | Full | Agent can't guess these |

When unsure, ask whether a competent agent that never saw this skill would get it right anyway. If yes and no choice needs pinning, cut.

## Skill, or something else?

| The content is… | Put it in |
|---|---|
| A short fact or convention needed in every session | The agent's always-loaded instruction file (`AGENTS.md`, `CLAUDE.md`, or its equivalent) |
| A procedure, checklist, or reference needed only for some tasks, project-specific ones included | A skill, project-scoped when it applies to one repo |
| A rule that must hold every time, or that a tool can check | A hook, linter, or CI check; wording cannot guarantee it |
| A one-off fix | Nothing |

Reference skills (knowledge applied alongside the work) and task skills (a procedure, often with bundled scripts) are both valid. A skill is worth writing when it will be reused and part of it passes the line test.

## Layout

```
skill-name/
  SKILL.md        # required, under 500 lines
  references/     # documents read on demand
  scripts/        # deterministic tools, executed rather than read
  assets/         # templates and data files the output uses
```

- Keep inline what every use needs. Move out large API or syntax references and long example sets.
- Every pointer to a supporting file says what the file contains and when to read it. A bare link is either never followed or always followed.
- Keep references one level deep: `SKILL.md` points to files, and those files do not chain to further files.
- Link with relative paths from the skill root. Avoid `@`-prefixed file references: some agents attach the file on every load.

## Frontmatter

The opening `---` must be the first line of the file. Write for any agent that loads skills; no single agent is the default reader.

- **Spec fields only**, unless the skill is deliberately tied to one agent: `name`, `description`, and optionally `license`, `compatibility`, `metadata`, `allowed-tools`. These are the Agent Skills spec fields, which is all a skill can count on across agents. `allowed-tools` is experimental in the spec and its effect differs between agents.
- **Anything else is one agent's extension.** Other agents ignore it, and some loaders reject the file. Use an extension only when the skill is meant for that agent alone: say so in `compatibility`, and take the semantics from that agent's documentation, never from the field's name. The same holds for body features such as argument placeholders and shell injection.
- **Per-agent notes** live in `references/`, one file per agent. [references/claude-code.md](references/claude-code.md) covers Claude Code's extensions, loading limits, and reserved names; read it when a skill targets Claude Code or will be installed there. For any other agent, check its own documentation instead of carrying assumptions over.

**`name`**: 1–64 characters of lowercase letters, digits, and hyphens, with no leading, trailing, or doubled hyphen. Make it identical to the directory name: the spec requires it and installers key on the directory. Avoid a name that a built-in command of a target agent already uses.

**`description`**: the only text the agent sees when deciding whether to load the skill, and it is loaded in every session alongside every other skill's description.

- **Stay within 100 bytes.** Some agents compress a longer description when loading the skill and silently lose part of it. Count bytes, not characters: a CJK character costs three. Measure with `wc -c` instead of estimating. The spec's 1024-character limit is only what validators accept.
- Shape: one sentence that opens with the key use case, meaning what the skill acts on and the situation that calls for it. Spend leftover bytes on a boundary against a confusable sibling skill, not on synonyms or lists of trigger phrases.
- Name the capability, never the steps. An agent that reads the workflow in the description follows the description and skips the body.
- Use the one term users actually type for the thing (`SKILL.md`, "release", an error code) rather than a paraphrase of it. A translated copy of the same trigger rarely fits the budget; add one only when users name the thing differently in that language.
- Compare against the descriptions of sibling skills before finalizing. The boundary between two skills has to be drawn here, because the body is read only after the choice is made.
- Quote the value when it contains `: ` or ` #`. A strict YAML parser rejects it otherwise, and a skill whose frontmatter fails to parse can load with no description and no error. Misspelled field names are ignored just as silently.

```yaml
# Bad: lists steps, so the agent follows the description and skips the body
description: Use when writing skills — set frontmatter, write body, then test with subagents

# Bad: vague, so nothing tells it apart from its neighbors
description: Use when working with skills

# Bad: over 100 bytes, so some agents compress it and part is silently lost
description: Use when creating, writing, editing, or reviewing an agent skill. Triggers on "write a skill", "create a skill", "新建 skill", or when authoring agent capabilities.

# Good: 82 bytes, key use case first, then the boundary
description: Write, edit, or review an agent skill (SKILL.md). Not for always-on project rules.
```

## Writing the body

- **Most important first.** After context compaction an agent may keep only the start of a loaded skill, so rules placed last are the first lost. Order: one-line framing, core principle, the non-obvious rules, then pointers and common mistakes.
- **Write for any agent.** Name the action, not one agent's tool: "read the file", "search the repo", "ask the user", rather than a specific tool or command name. Where a step truly needs one agent's feature, tie the skill to that agent in `compatibility`.
- **Standing instructions, not one-time steps.** The body is read once and then sits in context. For anything that must hold through the whole task, write "run the tests after every edit", not "run the tests".
- **State what to do.** Add a reason only where it marks the point at which the rule stops applying; do not narrate background.
- **Exit conditions, not triggers.** Triggers belong in the description. In the body, say when to stop and hand back: "if the file is generated, leave it".
- **Match strictness to fragility.** Exact commands for fragile, order-dependent operations; principles for open-ended work.
- **Keep placeholder-like tokens out of the body.** Some agents substitute text in the body before reading it, for example a dollar sign followed by a digit when the skill is invoked with arguments, which corrupts inline shell or awk snippets. Put such code in a script file.
- Tables for reference data, numbered lists for ordered steps, a flowchart only for a non-obvious decision point.
- No emoji. Label examples with words such as Bad and Good.

## Editing an existing skill

- An oddly specific line is usually there because the agent once got it wrong. Before cutting one, look for its origin in the commit history; when none can be found, flag it instead of deleting it.
- A change to what the skill does also changes its description and every catalog entry that summarizes it, such as README tables. Update them in the same change.
- When reviewing, use the Common mistakes table as the checklist, and separate verified defects from judgment calls.

## Checking that it works

Reading a skill does not show whether it works. Two questions need separate answers, each from a fresh session, because the session that wrote the skill already holds the context the skill is supposed to supply:

1. Does it trigger on the requests it should, and stay quiet on near-misses?
2. With the skill loaded, is the result better than without it?

Read [references/testing.md](references/testing.md) for the procedure when the skill is about to be shared, when its description changed, when it triggers wrongly, or when the agent keeps skipping one of its rules. For a small edit to a reference skill, a read-through against the table below is enough.

## Common mistakes

| Mistake | Fix |
|---|---|
| Re-explaining what the agent already knows | Apply the line test and cut what fails |
| Description lists the workflow steps | Name the capability and the situations; the steps stay in the body |
| Description is vague | Open with the key use case: what the skill acts on and when |
| Description over 100 bytes | Cut synonyms and trigger lists; keep the key use case and, if bytes remain, the boundary |
| A "when to use" section in the body | Move it to the description; the body keeps exit conditions only |
| The key rule sits at the bottom | Move it up; the start of the body is what survives compaction |
| One-time phrasing for a rule that must hold throughout | Reword as a standing instruction: "after every edit" |
| Bare link to a supporting file | Say what it contains and when to read it |
| A field, tool name, or body feature only one agent understands | Strip to the spec fields and name actions generically, or tie the skill to that agent in `compatibility` |
| Louder wording for a rule the agent skips | Enforce it with a hook or CI, or move it up |
| Unquoted `: ` in the description | Quote the value |
| Padding for completeness | A skill is a delta, not a textbook |
