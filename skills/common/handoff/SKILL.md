---
name: handoff
description: Write a handoff summary so a new agent session can continue this work. Not for status reports.
---

# Agent Handoff

You are writing a context handoff that a NEW AGENT will consume to resume this work in a fresh session.

The one fact that changes how you write everything: the output is for a new agent, not for the user. Every line must be an actionable instruction or a concrete state that another agent can use or verify directly. No narrative, no status report, no explanatory prose. Your default is to summarize for a human; resist it.

## Phase 1: Validate the request

- If the session is nearly empty or holds no meaningful context, tell the user there is nothing to hand off, then stop.
- If every task is complete and nothing is pending, warn the user that a handoff may not be necessary, and proceed only if they confirm.

## Phase 2: Gather context

1. **Git state** (skip when the workspace is not a Git repository): current branch, short HEAD SHA, the last 15 commits in one-line form, the diff stat of the last 10 commits, and the short status. Each command must tolerate failure: an empty repository, fewer than 10 commits, and a detached HEAD are all normal.
2. **Task status**: read the agent's task or todo list if it has one, and any TODO files, for current progress.
3. **Project instructions**: read the key rules from the project's instruction files (`AGENTS.md`, `CLAUDE.md`, or the agent's equivalent) when they exist.
4. **Loaded skills**: identify every skill activated or invoked in this session, however this agent surfaces them: tool calls, slash commands, skill listings, and names the user mentioned. Record the exact skill name as the agent identifies it, plus one line on what it was used for or why the next agent needs it. If none were loaded, note "None".
5. **Session review**: go through the entire conversation and extract:
   - the user's original requests, verbatim
   - work that was completed
   - technical decisions and their rationale
   - constraints, limitations, and caveats discovered
   - key files discussed or modified

## Phase 3: Extract and structure

Turn human-style summaries into agent-actionable instructions. Apply this pattern to every line:

- BAD: "The project uses a hybrid caching approach combining two strategies"
- GOOD: "When adding endpoints, cache only GET responses in Redis. Do not add an in-process cache."
- BAD: "Future work may include testing the export quality"
- GOOD: "Next task: run the export on a repository with active work and verify the output parses as JSON"

Extraction rules:

- Convert decisions into rules the new agent must follow.
- Convert completed work into current state the new agent can verify.
- Convert constraints into direct imperatives.
- Include only files the new agent can actually access, not gitignored or external ones.
- USER REQUESTS are quoted verbatim, never paraphrased.
- When a user request refers to temporary or session-scoped files (agent-private plan directories, `/tmp/`, files generated in this session), inline the key content into TASK or PENDING. The new agent will not have access to those files.
- LOADED SKILLS are listed by exact name so the new agent can reload them in one shot. Do not paraphrase or rename skill identifiers.
- With no pending tasks, state what was last completed and what the logical next steps are.
- Review your own mistakes in this session (wrong approaches, dead ends, incorrect assumptions) and record them in GOTCHAS so the new agent does not repeat them.
- Preserve the last 3-5 concrete operations (exact commands, API calls, file edits) as LAST OPERATIONS. They are few-shot examples of how things are done in this project and keep the new agent from guessing at operational patterns.

## Phase 4: Format the output

Brief mode applies when the user asks for a brief handoff or passes `--brief`. Otherwise use full mode.

**Brief mode** outputs the condensed version only:

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
- (List every skill loaded in this session by its exact name; the new agent must load each before continuing. If none, write "None")

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

**Full mode** (default) outputs the complete version:

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
- [Current task status from the task list if available]

FILES
-----
- [path/to/file1] - [what to do with it or why it matters]
- [path/to/file2] - [what to do with it or why it matters]
(Maximum 10 files, only files the new agent can access)

SKILLS
------
- [exact-skill-name] - [one-line: what it does / why it is needed]
- (List every skill loaded in this session by its exact name; the new agent must load each before continuing. If none, write "None")

RULES
-----
- [Direct imperatives the new agent must follow, derived from decisions and constraints]
- [User preferences stated as rules: "Always do X", "Never do Y"]
- [Codebase conventions: "Commit messages use conventional commits", "Tests go in __tests__/"]
- If none, write: None

GOTCHAS
-------
- [Specific technical pitfalls: "git log fails on an empty repo without a fallback"]
- [Non-obvious behaviors: "installing as a plugin changes the command name"]
- [Things that look wrong but are intentional]
- [Mistakes I made during this session — format as: "Tried X, failed because Y, fix was Z"]
  Examples:
  - "Tried using @hono/node-ws v1.2 upgradeWebSocket(ws => ...), got type errors. v1.3 changed signature to (c) => ({ onOpen(evt, ws) }). Use v1.3 signature."
  - "Ran git push to main branch, but project uses master. Had to delete remote main and re-push to master."
```

Output rules:

- Plain text with bullets.
- Use `===` and `---` underlines for headers, no `#` markdown headers.
- No bold, italic, or code fences within the content.
- Use workspace-relative paths. Exclude files the new agent cannot access (gitignored directories, external references, temporary files). FILES holds at most 10 entries.
- Every bullet must be actionable or verifiable by the new agent.
- USER REQUESTS (AS-IS) contains verbatim quotes only.
- The handoff is self-contained: it must work without access to the current session. Do not include secrets such as API keys or credentials. Do not try to create a new session programmatically.

## After the summary

Append these usage instructions:

```
---

TO CONTINUE IN A NEW SESSION:

1. Start a new agent session in this workspace
2. Paste the HANDOFF CONTEXT above as your first message
3. Add your request: "Continue from the handoff context above. [Your next task]"

NEW AGENT — DO THIS BEFORE ACTING ON THE TASK:
- Load every skill in SKILLS by its exact name — do not proceed without them
- Read every file in FILES, and any file path referenced in TASK/PENDING/GOTCHAS
- Explore the repo to verify STATE and COMPLETED against the actual current code — trust the files over this summary if they diverge
- Only then start on TASK
```
