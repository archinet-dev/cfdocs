# cfdoc — ColdFusion Documentation CLI

`cfdoc` is a command-line tool for looking up ColdFusion/CFML documentation directly from your terminal. It queries the same data that powers [cfdocs.org](https://cfdocs.org), but without needing a browser, a web server, or an internet connection.

## Why cfdoc?

### Built for coding agents

Coding agents (Claude, Codex, Copilot, Cursor, etc.) already know how to use CLI tools. They understand `--help`, subcommands, and flags out of the box. There's no special integration, no MCP server to configure, and no custom instructions needed — just point the agent at `cfdoc` and it will figure out the rest.

### Low token overhead

`cfdoc brief arrayAppend` returns a six-line summary; `cfdoc arrayAppend` returns the full entry in clean, structured plain text. Compare this to alternatives:

- **Grepping JSON files** requires the agent to parse raw JSON, figure out the schema, and filter out noise — burning tokens on structure instead of content.
- **MCP servers** add protocol overhead, require configuration, and introduce latency from tool negotiation and JSON-RPC framing.
- **Web searches** are slow, unreliable, and return pages of HTML that need to be parsed down to the useful bits.

### Fast

`cfdoc` reads a single JSON file from disk for lookups. There's no database, no server startup, no network request.

### Targeted queries

Instead of dumping an entire documentation page, agents can request only what they need:

- `cfdoc brief queryExecute structKeyExists` — one-screen summaries, several names per call
- `cfdoc syntax cfquery` — just the syntax
- `cfdoc params cfhttp` — just the parameters
- `cfdoc examples arrayFilter` — just the code examples
- `cfdoc engines arrayMap` — just which CF/Lucee/BoxLang versions support it
- `cfdoc search "encrypt"` — find the right function name before looking it up
- `cfdoc list deprecated` — everything removed, deprecated or discouraged, for code review
- `cfdoc since 2025` — what a ColdFusion version added

## Requirements

- **[jq](https://jqlang.github.io/jq/)** — a lightweight JSON processor
  - macOS: `brew install jq`
  - Debian/Ubuntu: `apt install jq`
  - Windows: `choco install jq`
- bash 3.2 or newer (the macOS default works)

## Setup

Clone the repo and the tool is ready to use:

```bash
git clone https://github.com/foundeo/cfdocs.git
cd cfdocs
./cfdoc --help
```

To make `cfdoc` available globally, symlink it into your PATH or add the checkout to PATH:

```bash
ln -s "$(pwd)/cfdoc" /usr/local/bin/cfdoc
```

To keep the dataset current when working from a fork:

```bash
git remote add upstream https://github.com/foundeo/cfdocs.git
git pull --ff-only upstream master
cfdoc --version      # prints the dataset date and counts
```

## Usage

### Look up a function or tag

```bash
cfdoc brief arrayAppend          # one-screen summary (cheapest)
cfdoc brief arrayAppend arrayMap # several at once
cfdoc arrayAppend                # full entry
cfdoc cfhttp
cfdoc http                       # automatically resolves to cfhttp
```

### Search by keyword

```bash
cfdoc search "json"              # name matches listed first
cfdoc search "encrypt"
cfdoc search "hash" --engine boxlang
```

### List available items

```bash
cfdoc list functions
cfdoc list tags
cfdoc list categories            # e.g. array-functions, string-functions
cfdoc list guides                # e.g. script, closures, security
cfdoc list deprecated            # removed/deprecated/discouraged on ColdFusion
cfdoc list deprecated --engine lucee
```

### Browse a category or a version

```bash
cfdoc category array-functions
cfdoc category security-functions
cfdoc since 2025                 # added in ColdFusion 2025
cfdoc since 6 --engine lucee     # added in Lucee 6
```

### Targeted lookups (less output, fewer tokens)

```bash
cfdoc syntax queryExecute        # just the syntax
cfdoc params cfhttp              # just the parameters
cfdoc examples arrayFilter       # just the examples
cfdoc engines arrayAppend        # just engine compatibility
```

### Read a guide

```bash
cfdoc guide script               # CFScript syntax guide
cfdoc guide closures
cfdoc guide security
```

### Flags

```bash
cfdoc arrayAppend --json         # raw JSON output (for piping or parsing)
cfdoc arrayAppend --engine lucee # filter to a specific engine
cfdoc arrayAppend --engine cf    # aliases: cf/acf = coldfusion, bl = boxlang
```

Engines: `coldfusion`, `lucee`, `boxlang`, `railo`, `openbd`, `rustcfml`.

## Coverage

`cfdoc` covers the full cfdocs.org dataset. Run `cfdoc --version` for the exact counts and the date of the data in your checkout. At the time of writing: 905 functions, 170 tags, 40 categories, 33 guides and the Application.cfc component.

Each entry includes description, syntax, parameters with types and defaults, engine compatibility, related items, and code examples.

## For agent developers

If you're configuring a coding agent to use `cfdoc`, the only instruction it needs is:

> Use the `cfdoc` CLI to look up ColdFusion/CFML documentation. Start with `cfdoc brief <name>`; run `cfdoc --help` for usage.

The `--help` output is self-documenting, and the plain-text output format is optimized for LLM consumption — no HTML, no ANSI colors, no decorative formatting.

## Environment variables

| Variable | Description |
|----------|-------------|
| `CFDOCS_DATA` | Override the path to the `data/en/` directory |
| `CFDOCS_GUIDES` | Override the path to the `guides/en/` directory |
