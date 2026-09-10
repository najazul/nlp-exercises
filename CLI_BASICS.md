# CLI Basics & Command Anatomy

A complete beginner-to-intermediate guide to understanding terminal commands, flags, chaining, and discovering how any CLI tool works.

---

## 1. Anatomy of a Command

Every command in the terminal follows this universal structure:

```text
command   [subcommand]   [flags / options]   [arguments]
   │           │                 │                │
  git        config           --list           --global
  ls                             -la            Documents/
  uv           pip            install            pandas
```

* **Command**: The executable program you are running (e.g., `git`, `uv`, `ls`).
* **Subcommand**: An action or module inside that tool (e.g., `git commit`, `uv pip`, `docker compose`).
* **Flags / Options**: Modifiers that change *how* the command behaves (e.g., `--list`, `-a`, `-v`).
* **Arguments**: The target data the command operates on (e.g., a file path, a branch name, a URL).

---

## 2. What are Flags? (`-` vs `--`)

Flags are switches that tweak a command's behavior.

### A. Long Flags (`--`)
* Preceded by **two hyphens** (`--`).
* Written as full, readable words.
* Examples:
  * `--list`: List all items.
  * `--help`: Show documentation.
  * `--version`: Show version number.
  * `--global`: Apply setting system-wide.

### B. Short Flags (`-`)
* Preceded by **one hyphen** (`-`).
* Single character abbreviations of long flags.
* Examples:
  * `-l` is often short for `--list`
  * `-a` is often short for `--all`
  * `-h` is often short for `--help`
  * `-v` is often short for `--verbose` or `--version`

---

## 3. Flag Chaining (Why `-la` works)

In Unix shells (macOS and Linux), single-letter flags can be **bundled together** behind a single hyphen `-` to save typing.

| Chained Flag | Equivalent To | What it means in `ls` |
| :--- | :--- | :--- |
| **`ls -la`** | `ls -l -a` | **`-l`** (long detailed format) + **`-a`** (all files, including hidden `.` files) |
| **`ls -lah`** | `ls -l -a -h` | Long format + all files + **`-h`** (human-readable sizes like KB/MB) |
| **`git branch -la`** | `git branch -l -a` | **`-l`** (list branches) + **`-a`** (all, including remote branches) |
| **`git commit -am "msg"`** | `git commit -a -m "msg"` | **`-a`** (all modified files) + **`-m`** (commit message) |

> [!NOTE]
> The order of chained flags does not matter: `ls -la` is identical to `ls -al`.

---

## 4. How to Discover Commands & Their Purpose in ANY CLI

Never guess what a command or flag does—every tool has built-in ways to teach you:

### 1. The Universal `--help` Flag
Almost every modern CLI program supports `--help` or `-h`:
```bash
# General help for the entire tool
git --help
uv --help

# Specific help for a subcommand
git config --help
git branch -h
uv sync --help
```

### 2. Tab Autocompletion (The Developer Secret)
Type a command, add a space, and press **`Tab`** once or twice:
```bash
git <Tab><Tab>
# Shell lists every available subcommand (checkout, clone, commit, etc.)

git config --<Tab><Tab>
# Shell lists all flags starting with --
```

### 3. The Built-in Manual (`man`)
For traditional Unix commands (`ls`, `curl`, `grep`, `ssh`):
```bash
man ls
```
*(Press **`q`** to exit the manual viewer).*

### 4. Community Cheat Sheets (`tldr`)
Standard manuals (`man`) can be overwhelming. The popular community tool `tldr` gives 5 practical examples:
```bash
# Install once via Homebrew
brew install tldr

# Get instant, practical cheat sheets
tldr git config
tldr ls
tldr tar
```

---

## 5. Summary Cheat Sheet

| Syntax | Type | Example | Purpose |
| :--- | :--- | :--- | :--- |
| `command` | Base Program | `git` | The tool itself |
| `subcommand` | Action | `git commit` | Specific operation |
| `--flag` | Long Option | `--verbose` | Self-documenting setting |
| `-f` | Short Option | `-v` | Fast shorthand |
| `-abc` | Chained Options | `ls -lah` | Combine multiple short options |
| `--flag=value` or `-f value` | Option with Value | `-m "msg"`, `--name=foo` | Supply input to a flag |
| `cmd --help` | Built-in Guide | `uv add --help` | Read documentation on the spot |
