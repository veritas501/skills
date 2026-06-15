---
name: readme-writer
description: Use when writing, rewriting, or reviewing a project README.md. Triggers on "write README", "generate README", "improve README", "README 不够好", "写个 README", or when a project lacks documentation for first-time users. Also use when onboarding docs feel unclear or quickstart instructions are missing.
---

# README Writer

Constraints and decisions for authoring a README from the **first-time reader's** perspective. You already know markdown and standard README layout (Title → Overview → Features → Quickstart → Usage → Development → Config → License) — this skill only adds the non-obvious rules that keep a README honest, scoped, and safe.

**Not for:** API reference, architecture docs, changelog, contribution guides. The Development section stays minimal (clone/install/test/build); branch/PR/style/commit conventions → CONTRIBUTING.md.

## Step 0 — Decide mode from user intent (before anything else)

Intent decides **whether to ask** and **how aggressively to change**; README state decides scope. Don't equate "big gaps" with "must ask", and don't equate "improve" with "rewrite".

| User intent | Mode |
|-------------|------|
| Write new / no README exists | `rewrite` |
| Rewrite / 重写 / 直接替换 / 推倒重来 | `rewrite` — authorized, even if it deletes a large stale README; do **not** ask |
| Improve / 优化 / 改好一点 / "不够好" + action verb | `patch` by default; escalate to `rewrite` only if existing README is mostly stale or structurally broken |
| Review and improve / 审查并改 | `report+patch` — assess first, then apply concrete changes |
| Review / assess only ("审查", "review", "够好吗") | `report` — assessment only |
| Ambiguous / no action verb (bare trigger) | `report` — assess, let user decide next |

Mode stop rules:
- `report` — assessment only; **stop, do not write the README**.
- `report+patch` — present assessment, then apply concrete patches in the same turn.
- `patch` / `rewrite` — go straight to editing.

## Assess existing README (report / patch modes)

Score each of the 5 reader questions (What does it do? / Why need it? / How to run fastest? / How to configure+use? / How to develop+test?) as **Clear / Vague / Missing** (run-question uses Works / Broken / Missing). This feeds the report and tells `patch` what to fix — it does **not** decide whether to ask (Step 0 does). `patch` → fix Missing/Vague/Broken, preserve good parts. `rewrite` → score informs gaps to close.

## Classify by user's first action (not tech form)

Tech forms are an open set (AI/ML, IaC, MCP server, mobile, template…) — never fully listable. Pick the README's spine from the **first action** instead, a closed set: run a command (CLI) · import a library (SDK) · start a service · open a UI · install into a host (plugin/extension/MCP) · use as a template. Multi-type (CLI + library): Quickstart shows the primary action, Usage splits into subsections.

## Command tiers — what you may write

| Tier | Source | Use? |
|------|--------|------|
| 1 Explicit | package.json scripts, Makefile/Justfile targets, Dockerfile CMD, CI config | Yes, as-is |
| 2 Ecosystem-standard | manifest implies it (see install detection below); plus an entry point **only when** CI/scripts/docs/shebang/CLI parser confirms how it's invoked | Yes — convention, not fiction |
| 3 Unknown | no explicit + no manifest standard; or a bare entry-point file with no confirmed invocation | **Never guess** |

**Entry points are evidence of what the project does, not automatically a run command.** `src/main.py` existing ≠ `python src/main.py` — real invocation may be `python -m pkg`, a console script after install, or need env/cwd/args/`uv`/`poetry`. Promote to a runnable command only when CI, scripts, docs, shebang, or a CLI parser confirms it; otherwise Tier 3.

**Install/run command — detect by lockfile/manifest, never default blindly:**

| Evidence | Command |
|----------|---------|
| `pnpm-lock.yaml` | `pnpm install` |
| `yarn.lock` | `yarn install` |
| `bun.lockb` / `bun.lock` | `bun install` |
| `package-lock.json` | `npm install` (or `npm ci`) |
| `uv.lock` | `uv sync` |
| `poetry.lock` | `poetry install` |
| `Pipfile` | `pipenv install` |
| `pyproject.toml` / `setup.py` (no lock) | `pip install -e .` |
| `go.mod` | `go build ./...`, `go test ./...` |
| `Cargo.toml` | `cargo build`, `cargo test` |

Do **not** fall back to `npm`/`pip` when `pnpm`/`yarn`/`bun`/`uv`/`poetry` evidence exists — Quickstart breaks on the first line.

Tier 2 `inferred` is an **internal label only** — don't annotate it in the README. Self-Check verifies the manifest exists; doesn't require the command string to appear in files.

Tier 3: never put `TODO:` inside a code block (fakes a runnable command). Keep the heading, write TODO as prose naming the missing evidence:
```markdown
### Run

TODO: confirm run command — no run script in package.json, Makefile, or CI config.
```

## Verification claims must match what you did

Static check can't prove the quickstart runs (deps fail, external services, stale Dockerfile, CI green but local incomplete). Never manufacture certainty.

- **Static (default):** commands from trusted source, paths/config files exist → claim "statically verified from project files". Do **not** say "works".
- **Dynamic:** you actually ran install/test/run and they succeeded → claim "verified by execution".

If unsure which level, you're at static.

## Scale — README is an entry point, not an encyclopedia

Standard sections are a **maximum, not a checklist**. Default to smallest that fits.

| Size | Sections |
|------|----------|
| Small | What it is · Quickstart · Usage · Development/Test · License |
| Medium | + Configuration · Project Structure · API overview |
| Large | Entry point only — move details to `docs/`, link out |

Adding sections a project doesn't need is bloat. Drop, don't fill with filler.

## Reader timeline (acceptance test)

A good README lets a first-timer hit each mark; if not, that section needs work.

| Time | Reader can… | Served by |
|------|-------------|-----------|
| 10s | know what it is | Title + one-liner |
| 1min | judge if it fits | Overview / Features |
| 5min | get it running | Quickstart |
| 15min | complete one real use case | Usage example |
| 30min | know how to develop/test | Development |

## Hard rules

0. **Never leak secrets** (highest priority). No real tokens, passwords, private URLs, credentials. `.env.example`/config may hold real values by mistake — treat all scanned values as untrusted, document **names + purpose + safe default only**. Redact secret-like values: `API_KEY=sk-...` → `API_KEY=<your-api-key>`. When in doubt, redact.
1. **No fiction** — every command/flag/config key is tier 1 or 2. Tier 3 → prose `TODO:`, never as fact.
2. **Copy-paste ready** — every code block directly executable; no `TODO:` inside code blocks.
3. **Real values over placeholders** — except secrets (Rule 0), which are always redacted.
4. **Shortest path first** — Quickstart = minimum steps to working state.
5. **Verify step** — after Run, state expected output ("you should see…").
6. **Language** — if user names a target language, use it. Else match the existing README's dominant language. If mixed/unclear with no preference: default to user's language for private/internal projects, English for public/open-source libraries/tools. Ask only when publication language genuinely matters — don't stall on it.

## Self-check before finishing

Confirm the Hard rules hold, plus two easy-to-miss items: prerequisites listed (runtime version, system deps), and the Development section didn't bloat into a contribution guide.
