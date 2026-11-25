# README examples

Three complete READMEs for small invented projects. Read the one closest to your project:

| Your project | Read |
|---|---|
| Command-line tool | Example 1 |
| HTTP service | Example 2 |
| Library | Example 3 |

An example shows proportion and level of detail. Its wording, headings, table columns, and claims are its own and do not transfer: every statement you write needs evidence in the repository at hand. The three are built differently from each other on purpose, because a README's sections follow from what its project needs.

## Example 1. Command-line tool

`````markdown
# tzwhen

tzwhen converts a date and time from one time zone to another, for people who schedule calls across offices and do not want to do the arithmetic.

```console
$ tzwhen "2025-03-10 09:00" Europe/Berlin America/New_York
2025-03-10 04:00 EDT
```

## Install

Rust 1.75 or newer is required.

```bash
cargo install tzwhen
```

## Usage

`tzwhen TIME FROM TO...` takes a time, the zone it is in, and one or more zones to convert to. Zones are IANA names such as `Asia/Tokyo`. With several targets it prints one line for each, in the order given:

```console
$ tzwhen "2025-03-10 09:00" Europe/Berlin America/New_York Asia/Tokyo
2025-03-10 04:00 EDT
2025-03-10 17:00 JST
```

`--format` changes the output; it takes the same patterns as `date`, for example `--format "%H:%M"`.

## Accepted time formats

| Input | Read as |
|---|---|
| `2025-03-10 09:00` | That date and time |
| `09:00` | Today at that time, in the source zone |
| `now` | The current moment |

A time that does not exist in the source zone, such as 02:30 on the night clocks go forward, is an error. A time that occurs twice when clocks go back is read as the earlier one.

## Development

```bash
cargo test
```
`````

What to notice:

- The opening says what the tool does and who uses it, then shows it running. That is already the real task, so Usage goes straight to how the command works.
- The tool has one command, so it gets a sentence, not a subsection of its own.
- "Accepted time formats" is a heading of this project's own, because that is what its users will look for.
- The edge cases around clock changes are two sentences, not two demonstrations.
- There is no Features, Limitations, or Configuration section, because this project has nothing that needs them.

## Example 2. HTTP service

`````markdown
# clipd

A shared clipboard for a home or office network: one machine PUTs a piece of text, another GETs it. Entries are held in memory and expire after ten minutes.

```console
$ CLIPD_TOKEN=change-me ./clipd &
clipd listening on 127.0.0.1:4040
$ curl -s -X PUT localhost:4040/clip/notes -H "Authorization: Bearer change-me" -d "meeting moved to 3pm"
stored notes (20 bytes)
$ curl -s localhost:4040/clip/notes -H "Authorization: Bearer change-me"
meeting moved to 3pm
```

Text is stored and sent unencrypted. Do not put passwords in it, and put it behind TLS before using it across networks.

## Install

Go 1.22 or newer is required.

```bash
go build -o clipd ./cmd/clipd
```

clipd refuses to start without `CLIPD_TOKEN`.

## API

Every request needs the header `Authorization: Bearer <token>`. Without it the response is 401.

| Request | Does | Responses |
|---|---|---|
| `PUT /clip/{name}` | Stores the request body under `name`, replacing any earlier entry | 200; 413 when the body exceeds 64 KiB |
| `GET /clip/{name}` | Returns the stored text | 200; 404 when missing or expired |
| `DELETE /clip/{name}` | Removes the entry | 204; 404 when missing |

## Configuration

| Variable | Default | Meaning |
|---|---|---|
| `CLIPD_TOKEN` | none | Required. The bearer token clients must send |
| `CLIPD_ADDR` | `127.0.0.1:4040` | Address and port to listen on |
| `CLIPD_TTL` | `10m` | How long an entry is kept |

## Running as a service

The repository includes `clipd.service` for systemd. It reads the variables above from `/etc/clipd.env` and restarts the process on failure:

```bash
sudo install -m 755 clipd /usr/local/bin/clipd
sudo install -m 644 clipd.service /etc/systemd/system/clipd.service
sudo systemctl enable --now clipd
```

## Development

```bash
go test ./...
```
`````

What to notice:

- The opening session starts the server and then uses it, so it runs top to bottom. The install section does not repeat the start command.
- What a reader must know before trusting the service with data comes directly after the proof.
- Three small endpoints fit in one table that lists every status each can return. No subsection per endpoint is needed at this size.
- The deployment steps carry what the unit file declares, and the Configuration table has no column that says the same thing in every row.

## Example 3. Library

`````markdown
# wordwrap

Line wrapping for terminal output in Rust, for command-line programs that print help text or messages of unknown length. It breaks at word boundaries and measures width in display columns, so wide CJK characters count as two. It has no dependencies.

```rust
use wordwrap::{fill, wrap};

assert_eq!(wrap("the quick brown fox", 10), ["the quick", "brown fox"]);
assert_eq!(fill("the quick brown fox", 10), "the quick\nbrown fox");
```

## Installation

```toml
[dependencies]
wordwrap = "2"
```

Rust 1.70 or newer is required. Code written for 1.x needs the changes in the [migration guide](https://example.com/wordwrap/blob/main/MIGRATING.md).

## Usage

Print an error message indented under a prefix, wrapped to the terminal width:

```rust
use wordwrap::{wrap_with, Options};

let message = "could not read config.toml: permission denied (os error 13)";
let options = Options { width: 32, indent: "  ", ..Options::default() };
for line in wrap_with(message, &options) {
    println!("{line}");
}
```

```text
  could not read config.toml:
  permission denied (os error
  13)
```

### wrap(text, width) and fill(text, width)

`wrap` returns the lines as a `Vec<String>`; `fill` returns them joined with `\n`. Both apply the same rules:

1. Existing newlines are kept as line breaks.
2. Within a line, a run of whitespace becomes one space, and breaks happen only there.
3. A word longer than `width` gets a line of its own and is not split.

### wrap_with(text, options)

| Field | Type | Default |
|---|---|---|
| `width` | `usize` | `80` |
| `indent` | `&str` | `""` |
| `break_words` | `bool` | `false` |

`indent` is added to every line and counts toward `width`. `break_words` splits a word longer than `width` instead of letting it overflow.

## Not supported

- Tabs count as one column, although a terminal draws them wider. Expand tabs before wrapping.
- ANSI color codes are counted as visible characters.
- Hyphenation.

## Development

```bash
cargo test
```
`````

What to notice:

- The opening says what the library is, who needs it, and what it handles, then shows both main functions with their results as assertions, which is the habit in this language.
- Usage starts with a realistic task and its printed output. The names and data come from that task.
- The rules are numbered because they apply in order, and they are sentences, not one code block per rule.
- What the library does not do is a plain list. Each item was checked before it was written, but none needs a demonstration.
- The import appears where a block introduces new names, and the link to a repository file is an absolute URL because the package is installed from a registry.
