---
name: readme-writer
description: Write, rewrite, or review a project README.md for first-time readers.
---

# README Writer

This skill covers what an agent gets wrong in a README: claims the repository does not support, text written for nobody in particular, and a document that reads like an audit of the repository instead of an introduction to it.

Three commitments hold for every line:

- **Evidence.** Each claim is backed by something in the repository, or it is left out.
- **Reader.** It is written for someone who has never seen the project and decides within seconds whether to keep reading.
- **Show.** The reader sees the project doing the job it exists for: a realistic command with its real output, realistic code with its real result. Checking a claim is your work; only what helps someone use the project is shown.

**Not for:** a standalone API reference site, architecture docs, changelogs, contribution guides.

## Hard rules

1. **Never leak secrets.** No real tokens, passwords, credentials, private URLs, or paths containing a username. Example config files may hold real values by mistake, so treat every scanned value as untrusted and document only the name, purpose, and a safe default: `API_KEY=sk-...` becomes `API_KEY=<your-api-key>`. When a value looks like a live credential, tell the user in the reply which file and variable hold it. Never repeat the value, in the README or in the reply.
2. **No claim without evidence.** The table under "What may be written" names the evidence each kind of claim needs.
3. **Stay inside the mode** the user's intent sets.
4. **The README describes the project, never the repository or your work on it.** It says what a user will observe: "Requires Go 1.22 or newer", "Setting `CACHE_DIR` has no effect". It never says what the repository contains or lacks, where a fact was found, what was verified, or what is still to do: not "as declared in `go.mod`", not "no code reads `CACHE_DIR`", not "the tests cover this". Those belong in the reply to the user.
5. **Closing sections belong to the owner.** License, contributing, security policy, support channels, acknowledgements, and sponsorship are the project owner's decisions. Never write, link, or reword them, even when the files exist. Carry existing sections over unchanged, and in the reply remind the user which are missing, license first.

## Decide the mode first

The request sets the mode, and the README's condition never upgrades it. Judge the intent from what the request means, in whatever language it is written.

| The user wants | Mode | What to do |
|---|---|---|
| A README where none exists, or the existing one replaced | `write` | Write the whole file. A request to replace is the authorization; do not ask again. Read the old file first |
| The existing README improved | `patch` | Assess it, then fix only what the assessment flags. Keep the author's structure, voice, badges, credits, and images. Do not rename, reorder, or merge existing sections. When most of it is stale, replace the stale sections and say in the reply what was replaced |
| An opinion on the README | `report` | Assess it and stop. Edit nothing |
| An opinion and the fixes | `report`, then `patch` | Present the assessment, then apply the fixes in the same turn |
| Something unclear | `report` | Assess it and let the user choose the next step |

## Get real output before writing

Before writing, obtain the real output of the commands and examples you intend to show, by the first route that works:

1. Run the tool or the example, when an environment for it already exists.
2. Run the project's own code directly with the system interpreter or toolchain, for example by pointing the import path at the source directory. This needs no install.
3. Run the project's documented install and then the tool, when everything it creates stays inside the project's own ignored directories.
4. Take the values asserted in the tests.

Limits on what you run:

- Never run anything that publishes, deploys, sends data to an external service, needs credentials, or changes files outside the project.
- Start a service only when it binds to the loopback address, on a port nothing else is using. Stop it afterwards, confirm that the port is free again, and delete the data files it created.
- Afterwards, the README must be the only tracked file that changed. Restore anything else a command rewrote, such as a lockfile.

Never invent or hand-derive output. When no route works, show the code alone and ask the user for the output in the reply; do not describe in prose what the output would look like.

In the reply, state which you did: **executed** (name the route) or **static** (nothing was run). Never say "works" for something that was not run.

## What may be written

| Claim | Evidence | Without it |
|---|---|---|
| Install, run, test, or build command | Tier 1 or 2 in the next table | Leave it out and name the missing evidence in the reply |
| Install from a registry, such as `pip install <name>` | A publish workflow, an install line already documented, or a registry link in the repository; the name comes from the manifest | Document the from-source path only |
| Feature or capability | Code that implements it | Leave it out. Plans, TODOs, and issues are not features |
| Identifier in an example: function, flag, option, config key | It exists in the source with that name and signature | Adapt an example from `examples/` or the tests |
| Shown output or result | Obtained by one of the routes above | Show the code alone and ask for the output in the reply |
| Behavior or limitation you inferred from the code | You reproduced it on a concrete input, or a test asserts it | Leave it out |
| Advice or a rule of thumb: "set X to at least N" | You ran the case at the boundary and the advice held | Say only what you observed, without the advice |
| Something the project does not do | The repository says so, a note in the code included, or a check you ran shows it | Leave it out. Not noticing something is not evidence |
| Supported platform or version | The CI matrix, the manifest's version constraint, or a language or shell feature the code uses that exists only from a known version | Leave it out |
| Performance or comparison | A benchmark in the repository or a cited source | Leave it out |
| What another file contains | You read that file and it says so | Link to the file without summarizing it |
| Link, badge, or image | The target exists; a badge reflects something configured; the repository URL comes from the Git remote | Leave it out and ask for the asset in the reply |
| Configuration variable | Code reads it | Leave it out and mention it in the reply. Exception: when an example config lists a setting that a reader would rely on for safety and it does nothing, say in one sentence that setting it has no effect |

Commands come in three tiers:

| Tier | Source | Use |
|---|---|---|
| 1 Explicit | Manifest scripts, Makefile or Justfile targets, Dockerfile CMD, CI config | As written. Leave one out, and report it, only when you ran it and it failed for a reason in the project itself. A tool that is merely not installed on this machine is no reason to drop the command |
| 2 Ecosystem-standard | Implied by the manifest and lockfile | Yes: convention, not fiction |
| 3 Unknown | Neither of the above | Never guess |

- **The lockfile picks the package manager**: `pnpm-lock.yaml`, `yarn.lock`, `bun.lock`, `uv.lock`, and `poetry.lock` each name their own tool. Never fall back to `npm` or `pip` when another lockfile exists.
- **An entry-point file shows what the project does, not how to run it.** `src/main.py` existing does not make `python src/main.py` correct. Promote it to a command only when CI, scripts, docs, a shebang, or a CLI parser confirms the invocation.
- **A manifest entry names the command.** A `bin` or `[project.scripts]` entry is evidence that the tool is run by that name, and installing the checkout with the ecosystem's standard command, such as `npm install -g .` or `pipx install .`, is tier 2. Write the tool by its command name in the synopsis and in every session. You may run the script file directly to capture output, but the README shows the command a user types. Never write that there is nothing to install.
- **Every shell command the README gives runs as pasted** from the state the README has set up. When the tool lives in a project environment, each command carries the runner prefix, such as `uv run`, or the README activates the environment first.
- **Example input is shown plainly.** Use files that exist in the repository, or show a small input file in its own fence with its name above it. Never build input with process substitution, heredocs, or `printf` pipelines.

## Build it for the reader

Pick the README's spine from the reader's **first action**, not from the tech stack: run a command, import a library, start a service, open a UI, install into a host (plugin, extension, skill), or use as a template. With two first actions, lead with the primary one.

A first-time reader must be able to do four things, in this order. These are goals to check the result against, not section names:

1. Say what the project is, from the opening.
2. See it working, from real output on the first screen.
3. Get it running, from the install and first command.
4. Finish a real task, from the usage or API section.

**Take inventory before writing.** Read the body of every command and public function, and every install and deployment file. List:

- the options, special cases, defaults, and rules each one applies;
- what the install and run commands need from the machine: the interpreter or shell version the code actually relies on, privileges, directories that must already exist, a second terminal;
- what happens on the inputs a user is likely to get wrong: a missing value, an unknown name, zero, an empty file.

Try each documented option the way a user would type it. Each item you find is either documented or deliberately left out; none is missed, and no summary sentence contradicts something shown later.

Use only the sections the project needs:

| Section | Include when | Contents |
|---|---|---|
| Opening (no heading) | Always | What the project is, who uses it and when, and what sets it apart when the code backs that up. The manifest's description is the author's pitch, not evidence: do not take an audience or a quality from it. Then the proof, then anything the reader must know before use |
| Installation or Quick start | Always | Every prerequisite the inventory found, with versions, then the steps from nothing to the state in which the opening example works as written: getting the code, installing, and for a service starting it. The first command shown must succeed for a typical user, so it carries `sudo` or a writable prefix when the default location needs privileges. Name the section for what it holds: "Running" or "Quick start" when nothing is installed |
| Usage, or API for a service | Always | The realistic task the project exists for, done end to end, unless the opening already showed it. Then the reference: commands, public functions, or endpoints, each with its synopsis or signature, its rules as plain sentences, and its options. A tool with a single command needs no subsection for it. An example shows typical use; an edge case is a sentence, not a demonstration |
| How it works | The user must tune or reason about a mechanism | The mechanism in a few sentences, one demonstrated case, and what the user should adjust |
| Configuration | Code reads several settings from the environment or a file | A table of the settings. A single setting is a sentence where it is used, not a section |
| Limitations | A reader would reasonably expect something the project does not do, or correct, typical use runs into a restriction | A short list of plain sentences about what the project does not do or handle, the way a maintainer would note it. Include only what someone using the project as intended will meet. What you found by feeding it mistaken input, such as zero, a negative number, a wrong type, or a missing value, is a finding for the reply, not a limitation: leave it out of the README. Check each item before writing it, and state it no more broadly than what you checked |
| Development | Contributors need commands | Test, lint, and build commands only |

- **What the reader must know before use comes first.** A condition you verified that can lose data or expose the user, even one reached through a slip such as a mistyped number, gets one plain sentence directly after the opening proof. It is then not repeated in Limitations; the only second mention allowed is at an install or deployment step that triggers it.
- **The opening shows the project as installed.** Its commands are written the way a user types them once the install section has been followed, and nothing in the opening sets things up. A service is the exception: its opening session starts the server and then sends a request. Any later session works from the state the lines above it have set up.
- **Be complete where readers script against it.** List the exit statuses a script would branch on and the status codes an endpoint returns for valid and invalid requests. Do not enumerate crashes. A deployment command carries what the deployment file declares, such as its ports and volumes.
- **Otherwise do not repeat yourself.** The opening may summarize what a later section details, but no sentence, link, or example appears twice, and no section refers the reader to another section. A fact belongs in How it works or in Limitations, never both.
- **Features is optional.** Write it only for capabilities a reader would not learn from the usage section.
- **Every example earns its place.** The opening shows the main entry points, two to four calls for a library. Examples use names and data from a realistic use of the project, not the placeholder names of the test suite. An example that passes an option shows a result that differs because of that option.
- **When documentation exists elsewhere**, in a `docs/` directory or on a site, the README is an entry point of about a hundred lines: the opening, the install, the realistic task, and a link to the documentation. When that documentation already holds the reference, leave the reference out of the README; when it does not, keep the reference as one compact table. Do not copy the documentation in, and do not create new doc files unless asked.
- **With more than about ten commands**, cover the main ones in full and list the rest in one table.
- **Link companion documents that exist**, each once: documentation files in the repository in preference to a site address you cannot check, a migration guide beside the install or compatibility note it concerns, a changelog as a single last line.
- **Users before contributors.** When a registry install is evidenced, it is the install path and building from source belongs under Development.
- **Do not state the project's own version number.**

## Assessing an existing README

Used by `report` and `patch`.

1. Score each of the four reader goals as Clear, Vague, or Missing.
2. Mark every command and claim as Evidenced, Contradicted, or Unevidenced, noting the file that settles it.
3. List the fixes in priority order: contradicted claims first, then missing goals, then vague ones.

`patch` fixes what this assessment flags and nothing else.

## Writing the text

Before writing or editing README text:

- Read [references/style.md](references/style.md) in full: the first screen, the wording that marks a README as machine-written, and code-block forms.
- In [references/examples.md](references/examples.md), read the one complete example closest to the project: a command-line tool, an HTTP service, or a library. Take its proportions and level of detail. Its sentences, headings, and table columns are its own; yours should fit your project.
- Read [references/special-cases.md](references/special-cases.md) only when one applies: images or badges, sibling translations, or editing an existing README in `patch` mode.

**Language**: use the language the user names. Otherwise match the existing README's dominant language. With neither, use the user's language for a private or internal project and English for a public library or tool.

## Before finishing

Read the README once as a stranger would, then check:

- no secret-like value survived;
- real output appears on the first screen, or the reply asks for it; later output is shown only where it tells the reader something;
- a reader who follows the install section and then the opening example gets the output shown, and every later session works top to bottom;
- every documented option was tried as a user would type it, and nothing found only through mistaken input is in the README unless it loses data;
- every shell command runs as pasted, and every identifier in an example exists in the source;
- no sentence describes the repository or your work instead of the project;
- no sentence, link, or example appears twice, apart from a must-know warning;
- every link and image path resolves.

Then tell the user: whether output was executed or the check was static, every behavior that looks like a defect in the code, with the input that triggers it, the commands and claims left out for lack of evidence, the file and variable of any value that looks like a live credential, the closing sections that are missing and theirs to add, and what else only a human can supply, such as a screenshot.
