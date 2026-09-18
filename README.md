# yai — folder-based agent harness

## What is yai

**yai** is a folder-based system for orchestrating AI agents. The agent's world is divided into 3 areas, designed to be inspected and modified similarly by both the agent and the human operator:

| Folder (inside `main/`) | Purpose |
|---|---|
| `tasks/` | things to do |
| `memory/` | persistent memory |
| `objects/` | produced artifacts |

The agent operates inside `main/`. Upon starting, it reads `main/start.md`, which instructs it to run `bot/tools/start/run` to acquire the session context.

## Quick Setup

Clone the repository and run the initial setup script to verify or automatically install required prerequisites:

```bash
git clone <repo>
cd yai
./setup
```

## Prerequisites

The `./setup` script automatically manages requirements, but if you prefer to configure them manually:
- **Python 3.11+**
- **uv**: extremely fast Python package manager and runner. Required for isolated, self-contained execution of tool scripts.
  ```bash
  # macOS (Homebrew)
  brew install uv

  # Linux / macOS (official installer)
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```

## Adding a New Tool

Tools are executable commands that the agent can launch to perform actions. Each tool lives in its own subfolder inside `main/bot/tools/`.

### Required Tool Structure

Every tool folder must strictly follow this structure:

```
main/bot/tools/<tool-name>/
├── README.md     ← tool documentation (4 standard sections)
├── run           ← executable script (chmod +x)
└── config/       ← auxiliary resources and configuration
```

Place any files supporting the tool inside `config/` (e.g. text templates, configuration files, static data, etc.). For example, for the `start` tool, template files live in `config/templates/`.

### 1. Documentation: `README.md`

Every tool's `README.md` must contain exactly these **4 standard sections**:

```markdown
## Description
Clearly explains what the tool does to determine if it is the right tool to use.
Note: this section is automatically parsed by the start script to regenerate the index in main/bot/tools/README.md.

## How to use it
Explains the invocation syntax, expected arguments, and any available flags.

## Examples
```bash
bot/tools/<tool-name>/run arg1 arg2
```

## Details
Explains the internal mechanics of the script in detail:
- Assumptions (e.g. expected cwd)
- Resources read or written (side effects)
- Algorithmic logic or templates used
```

### 2. Executable script: `run`

Tools can be written in any language (Python, bash, Node.js...), as long as the executable bit is set.

For **Python** tools, the project standard is to use **`uv` with inline script metadata (PEP 723)**:

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.11"
# dependencies = [
#     # Add any external libraries here, e.g.:
#     # "httpx",
#     # "beautifulsoup4",
# ]
# ///

# Your Python code here
```

#### How the `uv` mechanism works
- **Zero manual virtualenvs:** no need to create or activate virtual environments in the repository.
- **Isolated environment per tool:** each tool gets its own separate virtual environment (managed in `~/.cache/uv/environments-v2/`). Dependencies of one tool will never conflict with another.
- **Automatic cache and instant startup:** on first execution, `uv` downloads declared dependencies in milliseconds. Subsequent runs reuse the local cache, launching in ~10ms without hitting the network.
- **Transparent invocation:** the agent (or user) simply runs `bot/tools/<tool-name>/run`, without needing to know what happens under the hood.

### 3. Permissions and testing

After creating or modifying a `run` file:

1. **Make it executable:**
   ```bash
   chmod +x main/bot/tools/<tool-name>/run
   ```
2. **Test execution:**
   Run the script and verify that stdout/stderr output and any side effects (file writes) match expectations.

> **⚠️ Warning**: whenever you edit a `run` file, verify that it has not lost its executable bit (`chmod +x`).

### Automatic Tools Index

When `bot/tools/start/run` is executed, the script scans all subfolders in `bot/tools/`, extracts the `## Description` section from each tool's `README.md`, and automatically regenerates the [main/bot/tools/README.md](main/bot/tools/README.md) file.

## Full Repository Structure

```
yai/
├── CONTRIBUTING.md    ← guiding principles and conventions (read first)
├── README.md          ← this file (overview and tool development guide)
├── setup              ← automated prerequisites setup script
├── LICENSE
├── .gitignore
└── main/              ← agent runtime (the harness operates in here)
    ├── start.md       ← entry point for the agent
    ├── memory/        ← persistent memory
    ├── tasks/         ← tasks to accomplish
    ├── objects/       ← produced artifacts
    └── bot/
        └── tools/     ← tools executable by the agent
            ├── README.md  ← auto-generated index of available tools
            ├── start/     ← session bootstrap tool
            └── ...        ← other tools
```