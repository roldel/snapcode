# 📸 snapcode

**Turn your codebase into a single, LLM-friendly text file.**

`snapcode` is a lightweight shell script that creates a snapshot of your project (directory tree + file contents) to share with Large Language Models (ChatGPT, Claude, etc.).

It is designed to be **transparent** and **safe**: it strictly respects your existing `.gitignore` rules and does not hide or exclude files "magically" (other than the `.git` directory and binary files).

## ✨ Features

*   **Strict Ignore Handling:** Respects the `.gitignore` of the target directory. If you haven't ignored a file in git, `snapcode` will include it.
*   **Performance:** Checks ignore rules *before* recursing into directories (safe for large projects with ignored `node_modules`).
*   **Binary Safety:** Automatically detects and skips binary files (images, compiled binaries) to keep the text output clean.
*   **Multi-Folder Support:** Snapshot multiple distinct directories in a single run.
*   **Portable:** Single-file script. Runs on any POSIX shell (`sh`, `bash`, `zsh`).

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

## 📖 Usage

### Basic Snapshot
Run it in your project root. It will generate `snapshot_report.txt`.

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

1.  **It uses your `.gitignore`:** If you have `.env` inside your `.gitignore`, `snapcode` will skip it.
2.  **No Magic Hiding:** If you have **not** ignored `.env` (or other secrets), `snapcode` **will dump them**.
3.  **Safety Stop:** If no ignore rules are detected at all, `snapcode` will pause and ask for confirmation before dumping files.

### Custom Ignore Files
You can add extra ignore rules just for the snapshot without modifying your project.

```bash
# Apply rules from .llmignore ON TOP of the standard .gitignore
./snapcode --ignore .llmignore --include-root-gitignore
```

### The "Dump Everything" Mode
To ignore all rules and dump every text file found:

```bash
./snapcode --no-ignore --yes
```

## ⚙️ Command-Line Options

| Option | Description |
| :--- | :--- |
| `--path <DIR>` | Add a directory to the snapshot (optional, can just list dirs). |
| `--output <FILE>` | File to write to. Use `-` for stdout. |
| `--ignore <FILE>` | Add a custom ignore file (repeatable). |
| `--include-root-gitignore` | Merge the current directory's `.gitignore` with custom ignores. |
| `--no-ignore` | Disable all filtering (dumps everything except .git and the snapshot script itself). |
| `--yes` | Non-interactive mode (auto-accepts prompts). |
| `-h, --help` | Show help message. |

## 📦 Requirements

*   **Shell:** Standard `sh` (bash/zsh compatible).
*   **Git:** Required for accurate ignore checking.
*   **Grep:** Required for binary file detection.
*   **Tree:** (Optional) If installed, generates a better visual map.

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
├── src
│   ├── main.js
│   └── utils.js
└── README.md

===== File Contents (/my/project) =====

===== File: src/main.js =====
console.log("Hello World");


===== File: src/utils.js =====
export const add = (a, b) => a + b;
```