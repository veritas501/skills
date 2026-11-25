# README style

How the text of a README is ordered and worded. `SKILL.md` decides what may be said; this file decides how to say it.

## First screen

Top to bottom, skipping only what does not exist:

1. H1 with the project name.
2. What it is: the category of thing, what it does, and who needs it or when, in one or two sentences. Name what sets it apart only when the repository backs that up.
3. The proof: a command with its real output, a short example with its results, or an existing screenshot. For a service, the start command and then one request with its response, in that order. Never skipped for a tool, library, or service.
4. Anything the reader must know before use, in one sentence.
5. A link to the documentation site when one exists.

## Machine-written voice

Each pattern below comes from following a rule mechanically instead of writing for the reader.

| Pattern | Instead |
|---|---|
| Describing output in prose: "The command prints the address and the time it waited." | Show the output |
| Describing the repository: "Requires Go 1.22, as declared in `go.mod`"; "the config sample lists `CACHE_DIR`, but no code reads it" | Describe the project: "Requires Go 1.22 or newer."; "Setting `CACHE_DIR` has no effect." |
| Naming where an example came from: "This is `examples/wait.go`:" | Say what it does: "Wait for a database before running migrations:" |
| Telling readers how to run their own code: "Save this to a file and run it with ..." | Show the code and its result |
| Pointers between sections: "See Limitations.", "described below", "as shown above" | Nothing. Each section stands on its own |
| The same file linked for several items | One sentence, one link |
| A feature list that repeats the options table | Remove the feature list |
| Headings taken from these instructions: "Proof", "Evidence", "Must know" | Headings a maintainer would write |
| Demonstrating every edge case with its own code block, so the README reads like a test log | One sentence per rule; code blocks for typical use |
| Output that tells the reader nothing: a block of "done" lines | Leave the output out |
| A list of what happens on bad input, so Limitations reads like a bug report | Report those findings in the reply |
| Echo sentences: "`make test` does the same." | Give one way to do each thing |
| A section or table holding a single item | Fold it into the sentence where it is used |
| Placeholder names from the test suite: `ping`, `foo`, `a`, `b` | Names from a realistic use of the project |
| Every bullet in the form "**Bold term:** sentence" | Plain sentences; bold only where a term is defined |
| A section that exists only to say something is absent | Leave the section out |
| Marketing words with nothing behind them: powerful, robust, seamless, blazing fast, modern, simple, comprehensive | Cut the word, or show the fact that justifies it in the same line |
| Opening or closing filler: "Welcome to", "This README explains", "Feel free to", "Happy coding" | Start with what the project is; end when the content ends |
| Emoji as decoration | None |

## Prose

- An evaluative word stays only when the same line demonstrates it: "Intuitive syntax: `fd PATTERN` instead of `find -iname '*PATTERN*'`".
- The first sentence under a heading adds information. It does not restate the heading.
- Instructions in the imperative ("Run", "Set"); descriptions in the present tense.
- One name per thing. Use the term the code uses and keep it throughout.
- State a function's or command's rules as short sentences saying what it does to its input. Use a numbered list only for steps that happen in a fixed order; unordered facts are bullets or prose.
- A comparison names the alternative and states the difference factually, including where the alternative is the better choice.

## Sections

- Use the headings readers scan for: Installation, Quick start, Usage, API, How it works, Configuration, Limitations, FAQ, Troubleshooting, Development. When the project has content of its own that readers will look for by name, give it a heading that names it precisely.
- Reference data such as options and settings goes in a table. Include only columns whose values differ between rows: a Default column in which every row says "off" is removed.
- Put each prerequisite and its minimum version in the sentence before the first command. A command that only works inside the checkout says so in the sentence that introduces it.
- When the command name differs from the package or repository name, say so in the first line of the install section.
- Lead with the install method the project itself recommends.
- A table of contents only when the README runs to several hundred lines.

## Code blocks

Tag every fence with a language. Two forms for shell:

| Form | Fence | Content |
|---|---|---|
| Commands to copy | `bash` or `sh` | No shell prompt, so the block runs as pasted |
| A session that shows results | `console` | A `$ ` prompt before each command, the real output below it |

- For a command-line tool or a service, show the main commands or requests as `console` sessions. Use the copy form for setup, test, and build steps, whose output does not matter.
- Break a shell command longer than about a hundred characters across lines with trailing backslashes.
- For a library, show code in the library's language with the results you obtained by running it: as trailing comments or assertions where that is the language's habit, or as printed output in a `text` fence after the block. Never wrap library code in a shell one-liner to make it a console session.
- Write the import line in the opening example. Later blocks in the same README omit it unless they import something new.
- A library's install snippet uses the ecosystem's dependency syntax, with the version from the manifest when that syntax requires one.
- A placeholder is in angle brackets and names what goes there, as in `<your-api-key>`.

## Staying true over time

- No dates, no version number of the project itself, and no "coming soon".
- Do not pin a version in an install command unless the project requires that exact version.
- Do not copy documentation or other prose files into the README. Link to them. Showing a small input file that an example uses is fine.
- For a package that users install from a registry, links to files in the repository are absolute URLs built from the Git remote, because registry pages do not resolve relative paths.
