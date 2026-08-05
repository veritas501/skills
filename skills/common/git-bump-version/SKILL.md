---
name: git-bump-version
description: Use when preparing a Git repository release, bumping a SemVer version, creating a release commit, or tagging a version. Triggers on "bump version", "release commit", "git tag", "发版", "版本号递增", "打版本标签", and requests to summarize changes since the previous release.
---

# Git Bump Version

This skill pins the judgment and safety rules around a version release. Git mechanics are standard; the load-bearing parts are choosing the correct SemVer increment, finding the true release baseline, and keeping commit, version files, and tag consistent.

## Core Principle

Treat a release as one local transaction:

**inspect repository → commit existing work if dirty → require clean state → establish baseline → classify released changes → choose one bump → update canonical version sources → validate → create one version-only release commit → create one annotated tag**

Existing work and the version bump are always separate commits. The version bump commit contains only canonical version metadata and required generated lock/manifests. Never stash, reset, discard files, amend history, delete tags, or push as an implicit part of this workflow.

## Normalize Existing Work

1. Before release analysis or version edits, run `git status --porcelain=v1 --untracked-files=all`.
2. If the output is non-empty, create a separate pre-release commit before bumping the version:
   - inspect staged, unstaged, and untracked changes together; do not commit from the status summary alone;
   - require an attached branch and refuse unresolved conflicts;
   - check untracked files for secrets, credentials, large generated artifacts, or accidental local files before staging them;
   - stage all intended existing changes, including reviewed untracked files, so the commit leaves no hidden release input;
   - follow the repository's commit style and write a message that summarizes only these pending changes, not the upcoming version bump.
3. After that commit, require `git status --porcelain=v1 --untracked-files=all` to produce no output. If files remain, inspect and commit the remaining intended work separately before continuing.
4. The pre-release commit is part of the baseline-to-`HEAD` range and must be included in bump selection and the categorized release summary.

Do not silently omit suspicious files merely to obtain a clean status. If repository evidence cannot establish whether a file belongs in source control, stop before staging it and ask one focused question.

## Release Preconditions

1. Work on an attached branch. Respect the repository's release branch and existing release conventions; do not switch branches implicitly.
2. Identify the release line before editing:
   - authoritative version declaration(s), not every version-looking string;
   - the existing release commit subject style;
   - the tag format (`v1.2.3`, `1.2.3`, package-prefixed tags, signed tags);
   - monorepo policy: one shared version or independently versioned packages.
3. The current authoritative version and the latest reachable release tag for that release line must agree. If they do not, stop and explain the mismatch.
4. Require at least one effective change since the baseline. Never create an empty release only to advance a number.

## Find the Baseline

"Since the last version bump" means changes after the commit representing the current released version, not an arbitrary date or the lexicographically largest tag.

Use this precedence:

1. latest **reachable** release tag for the current release line;
2. if the repository has no release tags, the commit that last changed the authoritative version declaration to the current version;
3. if this is the first release, the repository root commit, and state that the summary covers the initial release.

Exclude commits outside the baseline-to-`HEAD` ancestry. Account for reverted changes: summarize and bump for the net behavior being released, not both the original and its full revert.

## Choose the SemVer Increment

Use the highest-impact effective change in the complete range. Existing repository policy overrides this table.

| Effective released change | Increment |
|---|---|
| Breaking public API, data format, configuration, CLI, or documented behavior change; `BREAKING CHANGE:` or `type!:` | **major** |
| Backward-compatible capability; Conventional Commit `feat` | **minor** |
| Backward-compatible bug, security, or performance correction; `fix` or `perf` | **patch** |
| Refactor, dependency, build, CI, test, documentation, or chore-only release with no public contract change | **patch** |

Rules:

- Conventional Commit types are evidence, not truth. Inspect the diff and released behavior when the message is missing, vague, or wrong.
- A change labeled `refactor` that breaks callers is major; a `feat` hidden behind an unchanged internal interface may be maintenance rather than minor.
- Dependency updates are patch by default, unless they expose a new capability or impose a breaking requirement.
- For `0.x`, follow an explicit repository policy. Without one, use the table literally: a breaking release becomes `1.0.0`.
- Prerelease/build metadata rules come from the repository. Do not invent an alpha/beta/rc sequence.

Record the winning reason, for example: `minor because the range contains backward-compatible features and no breaking changes`.

## Build the Release Summary

Read both commit history and the aggregate diff from baseline to `HEAD`. Write outcome-oriented bullets; do not paste commit subjects mechanically.

Use only non-empty categories, in this order:

1. Breaking Changes
2. Features
3. Fixes
4. Security
5. Performance
6. Maintenance — refactors, dependencies, build, CI, chores
7. Documentation
8. Tests

Collapse commits that implement one outcome into one bullet. Separate unrelated outcomes. Mention the affected component when it improves clarity. Omit merge noise, intermediate fixups, and fully reverted work. Do not claim behavior that the diff does not support.

## Update the Version

- Prefer the repository's existing versioning or package-manager command so lockfiles and generated metadata stay synchronized.
- Update every canonical declaration for this release line, including required lockfiles or generated manifests. Do not bump examples, compatibility declarations, dependency constraints, vendored files, or unrelated packages merely because they contain the old number.
- The release commit must contain only the intended version metadata changes. If the version command changes unrelated files, stop and inspect the repository-specific release mechanism.
- Run the repository's configured version/package validation before committing. Also require `git diff --check` and inspect the staged diff.

## Commit Message

Create exactly one release commit after validation. Follow the repository's existing release subject style; otherwise use:

```text
chore(release): vX.Y.Z
```

The body must summarize the entire baseline-to-`HEAD` range, not merely say "bump version":

```text
Release changes since vOLD:

Features:
- Add ...

Fixes:
- Correct ...

Maintenance:
- Update ...

Version: OLD -> NEW
Bump: minor (backward-compatible feature)
```

Use the repository's actual tag prefix in the subject and baseline text. Keep category headings stable and omit empty categories.

## Tag and Verify

1. After the commit succeeds, require the worktree and index to be clean.
2. Refuse to overwrite an existing tag.
3. Create an **annotated** tag using the repository's tag format. Reuse the categorized release summary in the tag message; sign it only when the repository already requires signed release tags.
4. Verify all three identities agree:
   - authoritative version = new version;
   - release commit subject/body = new version and selected bump;
   - tag name resolves to the release commit at `HEAD`.
5. Do not push the branch or tag unless the user explicitly asks. If pushing is requested, push only after all local checks pass.

## Failure Boundaries

- Pre-release commit failure: stop before version analysis or version edits; create no release commit or tag.
- Validation failure: leave changes uncommitted and create no tag.
- Commit failure: create no tag.
- Tag failure after commit: report the release commit SHA and exact tag error; do not rewrite or roll back history automatically.
- Any ambiguity about release line, authoritative version, tag convention, or major/minor policy that repository evidence cannot resolve: stop before mutation and ask one focused question.

## Common Mistakes

| Mistake | Correct action |
|---|---|
| Treating `feat` as a major bump | `feat` is minor; only a breaking change is major. |
| Using the newest tag by name or timestamp | Use the latest reachable tag for the active release line. |
| Summarizing only commit subjects | Reconcile commits with the aggregate diff and net released behavior. |
| Bumping every occurrence of the old version | Change only canonical declarations and required generated metadata. |
| Lightweight tag or tag before commit validation | Commit first, verify clean state, then create an annotated tag at `HEAD`. |
| Mixing pending code changes into the version bump commit | Commit existing work first; the later release commit contains only version metadata and required generated files. |
| Automatically pushing a successful local release | Stop after local commit and tag unless push was explicitly requested. |
