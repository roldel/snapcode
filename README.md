# 📸 snapcode

**Turn your codebase into a single, LLM-friendly text file.**

`snapcode` is a lightweight shell script that creates a snapshot of your project (directory tree + file contents) to share with Large Language Models (ChatGPT, Claude, etc.).

It is designed to be **transparent** and **safe**: it strictly respects your existing `.gitignore` rules and does not hide or exclude files "magically" (other than the `.git` directory and binary files).

## ✨ Features

*   **Strict Ignore Handling:** Respects git ignore rules for any target inside a git worktree. If you haven't ignored a file in git, `snapcode` will include it.
*   **Performance:** Checks ignore rules *before* recursing into directories (safe for large projects with ignored `node_modules`).
*   **Binary Safety:** Automatically detects and skips binary files (images, compiled binaries) to keep the text output clean. Whitespace-only files (e.g. `.gitkeep`) are correctly treated as text.
*   **Symlink Safety:** Symlinks to files are never followed — they are noted in the output but their targets are not read, preventing accidental leaks outside the project tree.
*   **Multi-Folder Support:** Snapshot multiple distinct directories in a single run.
*   **Safe Output:** Writes to a temp file in the same directory as the destination, then renames atomically — an existing snapshot is never corrupted by a failed run.
*   **Stable Output:** File and directory entries are sorted alphabetically, so snapshots are consistent and diffable across runs.
*   **Portable:** Single-file script. Runs on any POSIX shell (`sh`, `bash`, `zsh`). Works correctly when installed via symlink.

## 🚀 Installation

Download the script and make it executable.

**Using wget:**
```bash
wget -O snapcode https://github.com/roldel/snapcode/raw/main/snapcode
chmod +x snapcode
```

**Using curl:**
```bash
curl -L https://github.com/roldel/snapcode/raw/main/snapcode -o snapcode
chmod +x snapcode
```

*Optional: Move it to your path to use it globally:*
```bash
mv snapcode /usr/local/bin/snapcode
```

*and update .gitignore file:*
```bash
# snapcode 
snapcode
snapcode.txt
```


## 📖 Usage

### Basic Snapshot
Run it in your project root. It will generate `snapcode.txt`.

```bash
./snapcode
```

### Snapshot Multiple Folders
You can create a snapshot that combines different parts of your system. `snapcode` will switch contexts correctly for each folder.

```bash
./snapcode frontend/src backend/api
```

### Custom Output
```bash
./snapcode --output context.md
```

### Pipe to Clipboard
```bash
./snapcode --output - | pbcopy
```

## 🛡️ Ignore Rules & Transparency

`snapcode` relies on **your** configuration to keep secrets safe.

1.  **It uses your `.gitignore`:** Ignore rules are applied via `git check-ignore`, so the target must be inside a git worktree. If you have `.env` in your `.gitignore`, `snapcode` will skip it.
2.  **No Magic Hiding:** If you have **not** ignored `.env` (or other secrets), `snapcode` **will dump them**.
3.  **Safety Stop:** If the target is not inside a git repository, `snapcode` will warn you that no filtering will be applied and ask for confirmation before continuing.

> **Note:** `snapcode` requires `git` to be installed. If git is not available and `--no-ignore` is not set, the script will exit with an error rather than silently dumping unfiltered files.

### Custom Ignore Files
You can add extra ignore rules just for the snapshot without modifying your project.

```bash
# Apply rules from .llmignore ON TOP of the standard .gitignore
./snapcode --ignore .llmignore
```

### Respecting the Current Directory's `.gitignore`
If your current working directory differs from the snapshot target and has its own `.gitignore` rules you want applied, use `--include-root-gitignore`:

```bash
./snapcode --include-root-gitignore /some/other/project
```

This is additive — the target's own `.gitignore` is always respected regardless.

### The "Dump Everything" Mode
To ignore all filtering rules and dump every text file found (binary files are still skipped, and `.git` is always excluded):

```bash
./snapcode --no-ignore --yes
```

## ⚙️ Command-Line Options

| Option | Description |
| :--- | :--- |
| `--path <DIR>` | Add a directory to the snapshot (optional, can just list dirs). |
| `--output <FILE>` | File to write to. Use `-` for stdout. |
| `--ignore <FILE>` | Add a custom ignore file (repeatable). |
| `--include-root-gitignore` | Also respect `.gitignore` in the current working directory, in addition to each target's own rules. Useful when CWD differs from the snapshot target. |
| `--no-ignore` | Disable all filtering (dumps everything except `.git` and binary files). |
| `--yes` | Non-interactive mode (auto-accepts prompts). |
| `--version` | Print version and exit. |
| `-h, --help` | Show help message. |

## 📦 Requirements

*   **Shell:** Standard `sh` (bash/zsh compatible).
*   **Git:** Required. Used for ignore rule evaluation (`git check-ignore`). Without git, the script will refuse to run unless `--no-ignore` is passed.
*   **Grep:** Used for binary file detection. Falls back to `od` or `tr` if unavailable (e.g. BusyBox environments).
*   **Tree:** (Optional) If installed with `--gitignore` support, generates a better visual directory map when no custom ignore flags are active.

## 📝 Output Format Example

The generated report is structured to be easily parsed by LLMs:

```text
===== snapcode Report =====
Generated: Sat Dec 13 14:00:00 GMT 2025

################################################
# SECTION ROOT: /my/project
################################################

===== Directory Tree (/my/project) =====
.
├── README.md
├── logo.png
├── secret_link
└── src
    ├── main.js
    └── utils.js

===== File Contents (/my/project) =====

===== File: /my/project/README.md =====
# My Project


===== File: /my/project/logo.png =====
[Binary file skipped]


===== Symlink: /my/project/secret_link -> /etc/passwd (skipped) =====


===== File: /my/project/src/main.js =====
console.log("Hello World");


===== File: /my/project/src/utils.js =====
export const add = (a, b) => a + b;
```

When writing to a file, a summary is printed to the terminal:

```text
Done! Snapshot saved to: snapcode.txt
  4 file(s) processed — 3 included, 1 skipped as binary
```