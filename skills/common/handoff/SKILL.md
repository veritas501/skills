---
name: handoff
description: Use when context is getting long, quality is degrading, or you want a fresh session while preserving context — or when the user says "handoff", "context summary", etc. Not for status reports, debugging, or codegen.
argument-hint: "[--brief]"
allowed-tools: Read, Grep, Glob, Bash
---

# Agent Handoff Skill

You are generating a context handoff that a NEW AGENT will consume to resume work in a fresh session.

CRITICAL — this one fact changes how you write everything: the output is FOR A NEW AGENT, not for the user. Every line must be an actionable instruction or concrete state another agent can directly use or verify. No human-readable narrative, no status report, no explanatory prose. This is counter-intuitive — your default is to summarize for a human; resist it.

Arguments: $ARGUMENTS

---

## GIT CONTEXT (pre-collected)

- Current branch: !`git branch --show-current 2>/dev/null || echo "(detached or no branch)"`
- Recent commits:
!`git log --oneline -15 2>/dev/null || echo "(no commits yet)"`
- Recent file changes:
!`git diff --stat HEAD~10..HEAD 2>/dev/null || echo "(no recent commits)"`
- Uncommitted changes:
!`git status --short 2>/dev/null || echo "(not a git repo)"`
- Current commit SHA: !`git rev-parse --short HEAD 2>/dev/null || echo "N/A"`

---

## Execution Flow

### PHASE 1: Validate Request

Before proceeding, confirm:
- There is meaningful work or context in this session worth preserving
- If the session is nearly empty or has no meaningful context, inform the user there is nothing to hand off, then stop
- If all tasks are complete and there is nothing pending, warn the user that a handoff may not be necessary, but proceed if they confirm

### PHASE 2: Gather Deep Context

On top of the pre-collected git context, perform additional collection:

1. **Task status**: Call TaskList or read TODO files for current task progress (if any active tasks exist)
2. **Project instructions**: Check if `CLAUDE.md` or `.claude/CLAUDE.md` exists in the project, read key instructions if found
3. **Loaded skills**: Identify every skill that has been activated or invoked in this session. Sources to scan:
   - Skill tool invocations in the conversation (e.g. `Skill(skill="X")`)
   - Skill names surfaced as `<command-name>` / slash-command markers (e.g. `/X`)
   - Entries listed under "Skills relevant to your task:" system reminders
   - Skill references the user mentioned by name
   Record the exact skill name (as it appears in the Skill tool) and a one-line note on what it was used for or why the next agent needs it. If no skills were loaded, note "None".
4. **Session context review**: Review the entire conversation history and extract:
   - User's original requests (verbatim)
   - Work that was completed
   - Technical decisions made and their rationale
   - Constraints, limitations, and caveats discovered
   - Key files discussed or modified

### PHASE 3: Extract and Structure

Transform human-style summaries into agent-actionable instructions. The pattern (apply to every line you write):
- BAD: "The project uses a hybrid approach combining lightweight and full protocol features"
- GOOD: "When adding new features, only auto-detect CLAUDE.md and TaskList. Do not add SCOPE.yml or STATUS.yml dependencies."
- BAD: "Future work may include testing the handoff output quality"
- GOOD: "Next task: run /handoff in a session with active work, verify the output is agent-consumable"

Extraction focus:
- Convert decisions into rules the new agent must follow
- Convert completed work into current state the new agent can verify
- Convert constraints into direct imperatives
- Only include files the new agent can actually access (not gitignored or external references)
- USER REQUESTS must be quoted verbatim, never paraphrased
- If user requests reference temporary or session-scoped files (e.g. ~/.claude/plans/, /tmp/, session-generated files), inline the key content into TASK or PENDING — the new agent will NOT have access to those files
- LOADED SKILLS must be listed by their exact Skill tool name so the new agent can reload them in one shot; do not paraphrase or rename skill identifiers
- If there are no pending tasks, state what was last completed and what logical next steps would be
- Review your own mistakes during this session — wrong approaches tried, dead ends hit, incorrect assumptions made — and record them in GOTCHAS so the new agent does not repeat them
- Preserve the last 3-5 concrete operations (exact commands, API calls, file edits) as LAST OPERATIONS — this gives the new agent few-shot examples of how things are done in this project, preventing it from guessing at operational patterns

### PHASE 4: Format Output

Check if `$ARGUMENTS` contains `--brief`.

**If `--brief` mode**, output the condensed version only:

```
HANDOFF CONTEXT (BRIEF)
=======================

TASK
----
[Concrete next action for the new agent. If no pending task, state "No pending task. Last completed: X"]

STATE
-----
- Branch: [branch name] @ [commit SHA]
- [Build/test status if applicable]
- [Key state the new agent needs to know]

SKILLS
------
- [exact-skill-name] - [one-line: what it does / why it is needed]
- (List every skill loaded in this session by its exact Skill tool name; the new agent MUST reload each via the Skill tool before continuing. If none, write "None")

PENDING
-------
- [Concrete tasks with enough detail to execute]
- [Blockers with specific error messages or file paths]

FILES
-----
- [path/to/file1] - [what to do with it or why it matters]
- [path/to/file2] - [what to do with it or why it matters]
(Maximum 10 files, only files the new agent can access)
```

**If full mode (default)**, output the complete version:

```
HANDOFF CONTEXT
===============

USER REQUESTS (AS-IS)
---------------------
- [Exact verbatim user requests - NOT paraphrased]

TASK
----
[Concrete next action for the new agent. If no pending task, state "No pending task. Last completed: X"]

COMPLETED
---------
- [What was done, stated as verifiable facts: "file X now contains Y", "endpoint Z returns W"]
- [Include file paths so the new agent can verify]

LAST OPERATIONS (few-shot for new agent)
-----------------------------------------
- [Last 3-5 concrete operations with exact commands/steps, so the new agent can see HOW things were done in this project]
- Example format:
  - "Ran: pnpm build -> output: 3 packages built, 0 errors"
  - "Tested: curl http://localhost:3000/api/healthz -> {status: ok}"
  - "Edited: apps/api/src/routes/auth.ts:42 — changed bcrypt rounds from 10 to 12"
- This serves as few-shot examples for the new agent to follow the same operational patterns

STATE
-----
- Branch: [branch name] @ [commit SHA]
- Uncommitted changes: [yes/no, what files]
- [Build/test status if applicable]
- [Environment or configuration state that affects work]

PENDING
-------
- [Concrete tasks with enough detail to execute without asking the user]
- [Blockers with specific error messages or file paths]
- [Current task status from TaskList if available]

FILES
-----
- [path/to/file1] - [what to do with it or why it matters]
- [path/to/file2] - [what to do with it or why it matters]
(Maximum 10 files, only files the new agent can access)

SKILLS
------
- [exact-skill-name] - [one-line: what it does / why it is needed]
- (List every skill loaded in this session by its exact Skill tool name; the new agent MUST reload each via the Skill tool before continuing. If none, write "None")

RULES
-----
- [Direct imperatives the new agent must follow, derived from decisions and constraints]
- [User preferences stated as rules: "Always do X", "Never do Y"]
- [Codebase conventions: "Commit messages use conventional commits", "Tests go in __tests__/"]
- If none, write: None

GOTCHAS
-------
- [Specific technical pitfalls: "git log fails on empty repo without 2>/dev/null fallback"]
- [Non-obvious behaviors: "plugin install changes command name to /agent-handoff:handoff"]
- [Things that look wrong but are intentional]
- [Mistakes I made during this session — format as: "Tried X, failed because Y, fix was Z"]
  Examples:
  - "Tried using @hono/node-ws v1.2 upgradeWebSocket(ws => ...), got type errors. v1.3 changed signature to (c) => ({ onOpen(evt, ws) }). Use v1.3 signature."
  - "Initially put skills/ directory for plugin structure, but this project is a command not a skill. Restructured to commands/ directory."
  - "Ran git push to main branch, but project uses master. Had to delete remote main and re-push to master."
```

### Output Rules

- Plain text with bullets format
- Use `===` and `---` underline style for headers, no `#` markdown headers
- No bold, italic, or code fences within content
- Use workspace-relative paths; exclude files the new agent cannot access (gitignored dirs, external references, temp files); FILES ≤ 10 entries
- Every bullet must be actionable or verifiable by the new agent
- USER REQUESTS (AS-IS) must contain verbatim quotes only
- Self-contained: must work without access to the current session. Do NOT include secrets (API keys, credentials). Do NOT attempt to programmatically create new sessions

---

## Post-Output Instructions

After the summary, append these usage instructions:

```
---

TO CONTINUE IN A NEW SESSION:

1. Start a new Claude Code session (run `claude` in terminal)
2. Paste the HANDOFF CONTEXT above as your first message
3. Add your request: "Continue from the handoff context above. [Your next task]"

The new session will have all context needed to continue seamlessly.
```

---

## Execute Now

Begin by gathering context, then synthesize the handoff summary.
