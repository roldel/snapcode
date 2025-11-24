# snapcode

Create an LLM‑friendly text snapshot of your codebase in a single file: a directory tree plus the contents of your source files, while respecting ignore rules.

## What it does

`snapcode` walks your project directory and produces a plaintext report that contains:

- A **directory tree** of your project (using `tree` if available, otherwise a `find` fallback).
- The **full contents of each file**, one after another, clearly delimited.
- Respect for **ignore rules** (like `.gitignore` and custom ignore files), so you don’t accidentally dump huge build artifacts, dependencies, or secrets.
- Automatic skipping of:
  - The `.git` directory
  - The `snapcode` script itself
  - The output report file (to avoid self‑reference loops)
- Explicit inclusion of `.env.sample` and `.env.example` files, even if they would otherwise be ignored.

This makes it ideal for sharing your codebase context with LLMs, code review tools, or search/indexing systems.

## Installation

You only need a POSIX shell. `git` is recommended (for ignore handling), and `tree` is optional.

From your project root, download and make it executable:

```bash
wget -O snapcode https://github.com/roldel/snapcode/raw/main/snapcode
chmod +x snapcode
```

Or with `curl`:

```bash
curl -L https://github.com/roldel/snapcode/raw/main/snapcode -o snapcode
chmod +x snapcode
```

You can keep it in your project root or move it somewhere on your `PATH` (e.g. `~/bin`).

## Basic usage

From inside your project:

```bash
./snapcode
```

This will:

- Use the **current directory** as the root.
- Write the snapshot to `./snapshot_report.txt`.
- Use the root `.gitignore` (if present) to filter files.

You can change the root and the output file:

```bash
./snapcode --path ./my-app --output context.txt
```

To write the snapshot directly to stdout (e.g. to pipe into another tool):

```bash
./snapcode --output -
```

## Ignore rules & safety

By default, `snapcode` tries to avoid dumping everything blindly:

- If you don’t specify any `--ignore` files and there is a `.gitignore` in the root, that `.gitignore` is used automatically.
- If **no** ignore rules are active and you haven’t passed `--no-ignore`, `snapcode` will print a warning and ask for confirmation before dumping every file.

### Custom ignore files

You can supply one or more custom ignore files (using gitignore syntax):

```bash
./snapcode --ignore .gitignore --ignore ~/.config/llm-ignore
```

To combine custom ignore files with the root `.gitignore` in the target directory:

```bash
./snapcode --ignore ~/.config/llm-ignore --include-root-gitignore
```

### Disabling ignores

If you truly want a full dump:

```bash
./snapcode --no-ignore --yes
```

- `--no-ignore` disables all filtering.
- `--yes` auto‑answers “yes” to prompts (useful for non‑interactive usage).

> **Note:** Even with `--no-ignore`, `snapcode` still skips `.git`, its own script file, and the output report file.

## Command‑line options

```text
--path <DIR>                 Snapshot starting from this directory (default: current)

--output <FILE>              Write report to this file (default: <DIR>/snapshot_report.txt) Use "-" to output to stdout

--ignore <RULEFILE>          Add ignore file (gitignore syntax). Repeatable.
--include-root-gitignore     Also respect <DIR>/.gitignore when using custom ignores

--no-ignore                  Disable all filtering (full dump, skips safety prompt)
--yes                        Auto-answer yes to prompts (non-interactive)

-h, --help                   Show help
```

## Typical workflows

### 1. Generate a snapshot for a support / LLM session

```bash
./snapcode --output project_snapshot.txt
```

Then upload `project_snapshot.txt` to your tool / chat for analysis.

### 2. Use a custom global ignore file

```bash
./snapcode --ignore .gitignore --ignore ~/.config/llm-ignore --output context.txt
```

### 3. Pipe into another process

```bash
./snapcode --output - | some-other-tool
```

## Requirements & compatibility

- **Shell:** POSIX `sh`
- **Recommended:** `git` (for ignore handling)
- **Optional:** `tree` (for a nicer directory tree view). If `tree` is missing, `snapcode` falls back to `find`.

`snapcode` is designed to be self‑contained and portable: just one script, no external Python/Node/whatever dependencies.
