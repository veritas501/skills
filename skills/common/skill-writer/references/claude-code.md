# Claude Code notes

Claude Code is one agent among several. This file covers how it loads skills and what it adds on top of the Agent Skills spec. None of it can be assumed of another agent.

Using any extension below ties the skill to Claude Code: claude.ai uploads, the Skills API, and `package_skill.py` reject frontmatter that contains them, and other agents give them no defined meaning.

Limits quoted here come from the Claude Code skills documentation (<https://code.claude.com/docs/en/skills>, v2.1.33x). Re-check there when an exact number matters.

## How Claude Code loads a skill

- Every frontmatter field is optional. `name` defaults to the directory name; without `description`, the first non-empty body line is used.
- A field name that does not match exactly is ignored without an error. If the YAML does not parse, the skill loads with no fields set: `/name` still works, but nothing triggers it.
- `description` plus `when_to_use` is cut at 1,536 characters per skill in the listing. The whole listing gets about 1% of the context window; on overflow, the descriptions of the least-invoked skills are dropped first.
- `<` and `>` in `description` are displayed escaped. Write `[branch]`, or move the placeholder to `argument-hint`.
- Once invoked, the rendered body stays in context and the file is not read again. After compaction only the first 5,000 tokens of each invoked skill are re-attached, within 25,000 tokens shared by all of them.

## Names that misbehave

| Skill name | Effect |
|---|---|
| `synced`, `anthropic-skills`, `anthropic-skills:*` | Never loads |
| `ship`, `ship-local`, `steward` | Not loaded where the bundled `/ship` is available |
| Same as a bundled skill or built-in command (`code-review`, `usage`, …) | Replaces it, though not its aliases |
| `verify`, `simplify` | Claude is told to run it before every commit, unless model invocation is disabled |

## Who invokes the skill

| Situation | Setting | Cost |
|---|---|---|
| Side effects, or timing the user must control (commit, release, deploy, send) | `disable-model-invocation: true` | The description leaves the context, so natural-language requests no longer trigger it; the user types `/name` |
| Background knowledge that is not an action | `user-invocable: false` | Hidden from the `/` menu |
| Self-contained task that needs no conversation history | `context: fork`, optionally `agent:` | Runs as a subagent that sees only the skill body, in the background unless `background: false`. Pointless for a guidelines-only skill: it has no task to perform |

`when_to_use` is appended to the description in the listing. In a Claude Code-only skill it is the place for trigger phrases that do not fit the 100-byte description.

## Turning a skill off, and evaluating it

- For a with/without comparison, set the skill to `"off"` under `skillOverrides` in settings. Plugin skills are not affected by that setting.
- The `skill-creator` plugin automates the comparison: `/plugin install skill-creator@claude-plugins-official`.

## Tool permissions

- `allowed-tools` pre-approves tools for the turn that invokes the skill. It restricts nothing: unlisted tools stay callable under the normal permission rules, and the grant ends at the user's next message.
- A bare `Bash` pre-approves every shell command for that turn. Scope it: `Bash(git status *)`, `Bash(${CLAUDE_SKILL_DIR}/scripts/render.sh *)`.
- To take tools away while the skill is active, use `disallowed-tools`.

## Arguments and substitution

These are replaced in the body before the agent sees it:

| Placeholder | Becomes |
|---|---|
| `$ARGUMENTS` | Everything typed after the skill name |
| `$0`, `$1`, … or `$ARGUMENTS[N]` | The argument at that 0-based position, split with shell-style quoting |
| `$name` | The argument named in the `arguments:` field; an empty string when it is missing |
| `${CLAUDE_SKILL_DIR}` | The directory holding this `SKILL.md`. Use it for bundled script paths, because the working directory is the project |
| `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_SESSION_ID}`, `${CLAUDE_EFFORT}` | Project root, session id, current effort level |

- A literal `$0`, `$1`, or `$ARGUMENTS` in prose or in an inline shell or awk snippet is a placeholder too, and is replaced whenever the skill is invoked with that many arguments. Write `\$1`, or move the snippet into a script file.
- An indexed placeholder with no matching argument is left as typed.
- When the body has no placeholder, the arguments are appended as `ARGUMENTS: <value>`.
- Set `argument-hint` (for example `[issue-number]`) so autocomplete shows what is expected.

## Shell injection

A line containing `` !`command` ``, or a fenced block opened with ```` ```! ````, runs before the skill loads, and its output replaces the placeholder. Use it to ground the skill in live state such as the current diff or branch.

- A command that exits non-zero aborts the whole invocation, and the agent never sees the skill. Give every command that may fail a fallback: `|| echo "(none)"`. Exit code 1 from search and comparison commands is treated as a normal result; 2 or higher still fails.
- Injected commands never prompt. Outside auto mode, one that is not already allowed aborts the invocation, so pre-approve it in `allowed-tools`.
- `!` is recognized only at the start of a line or after whitespace.
- Commands run in the session's current directory, which moves when the agent runs `cd`. Anchor paths with `${CLAUDE_SKILL_DIR}` or `${CLAUDE_PROJECT_DIR}`.
