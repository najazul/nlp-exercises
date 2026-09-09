# uv Basics & Cheatsheet

A practical, quick-reference guide to managing Python projects, virtual environments, dependencies, and Python versions using [`uv`](https://docs.astral.sh/uv/).

---

## 1. Project Management (Recommended Workflow)

`uv` uses standard `pyproject.toml` and a deterministic `uv.lock` file to manage project dependencies.

| Task                             | Command                             | Description                                                                     |
| :------------------------------- | :---------------------------------- | :------------------------------------------------------------------------------ |
| **Initialize project**     | `uv init`                         | Creates a new project with`pyproject.toml`                                    |
| **Add dependency**         | `uv add <package>`                | Adds to`pyproject.toml`, updates `uv.lock`, and installs into `.venv`     |
| **Add specific version**   | `uv add "pandas>=2.0,<3.0"`       | Adds package with version constraints                                           |
| **Add dev dependency**     | `uv add --dev pytest`             | Adds to development dependency group                                            |
| **Remove dependency**      | `uv remove <package>`             | Removes from`pyproject.toml`, updates lockfile, and uninstalls from `.venv` |
| **Sync environment**       | `uv sync`                         | Reconciles`.venv` with `uv.lock` (auto-uninstalls removed packages)         |
| **Upgrade dependencies**   | `uv lock --upgrade`               | Upgrades all packages within allowed version constraints                        |
| **Upgrade single package** | `uv lock --upgrade-package <pkg>` | Upgrades only the specified package                                             |

> [!TIP]
> **Manual `pyproject.toml` Edits:**
> If you manually add or remove a line in `pyproject.toml`, just run `uv sync`. It detects the change, re-locks `uv.lock`, and automatically adds or uninstalls packages from `.venv`.

---

## 2. Running Code & Commands

You do **not** need to manually activate the virtual environment if you use `uv run`.

```bash
# Run a Python script inside the project environment
uv run python script.py

# Run a command or tool installed in .venv
uv run pytest
uv run jupyter lab

# Run a one-off script with temporary dependencies (without adding to pyproject.toml)
uv run --with httpx --with rich python fetch_data.py
```

---

## 3. Python Version Management (Replaces `pyenv`)

`uv` can download, install, and switch Python versions automatically without external tools.

```bash
# List available and installed Python versions
uv python list

# Install a specific Python version
uv python install 3.11 3.12

# Pin the Python version for the current project (.python-version file)
uv python pin 3.11

# Run a script with an arbitrary Python version on-the-fly
uv run --python 3.10 python script.py
```

---

## 4. Virtual Environments & Pip Drop-in Mode

If you are working on a traditional non-project workflow (similar to `python -m venv` and `pip`):

### Creating and Activating `.venv`

```bash
# Create a virtual environment using default Python
uv venv

# Create a virtual environment with a specific Python version
uv venv --python 3.11

# Activate on macOS / Linux
source .venv/bin/activate

# Activate on Windows (PowerShell)
.venv\Scripts\Activate.ps1
```

### Fast `pip` Replacements

`uv pip` commands are 10-100x faster drop-in replacements for standard `pip`:

```bash
# Install packages
uv pip install -r requirements.txt
uv pip install torch torchvision

# Uninstall packages
uv pip uninstall requests

# Inspect installed packages
uv pip list
uv pip tree

# Compile requirements (replaces pip-compile / pip-tools)
uv pip compile pyproject.toml -o requirements.txt
```

---

## 5. Tool Management (Replaces `pipx`)

Run or install standalone CLI tools in isolated environments without cluttering your system or project venv:

```bash
# Run a tool ephemerally without installing
uvx ruff check .
uvx black .

# Install a CLI tool globally/user-wide
uv tool install ruff
uv tool install black

# List installed tools
uv tool list

# Upgrade installed tools
uv tool upgrade --all
```

---

## 6. Jupyter Notebooks & VS Code Setup

When working with Jupyter notebooks in VS Code:

1. **Install ipykernel in the project:**
   ```bash
   uv add ipykernel
   ```
2. **Select the Kernel in VS Code:**
   - Open your notebook (`.ipynb`).
   - Click **Select Kernel** in the top right corner.
   - Choose **Python Environments...**
   - Select the interpreter under `.venv/bin/python`.

---

## 7. Useful Flags & Maintenance

| Command            | Purpose                                                      |
| :----------------- | :----------------------------------------------------------- |
| `uv cache clean` | Clears all cached wheels and downloads to free up disk space |
| `uv tree`        | Displays a dependency tree of the current project            |
| `uv self update` | Updates the`uv` CLI itself to the latest release           |
