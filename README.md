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

Tools are commands, scripts, or systems that the agent can launch to perform actions. Each tool lives in its own subfolder inside `main/bot/tools/<tool-name>/`.

### The Tool Contract

The **only mandatory requirement** for a tool is its documentation file:

```
main/bot/tools/<tool-name>/
└── README.md     ← mandatory (4 standard sections)
```

Everything else inside the tool folder is flexible: a tool can be a single standalone script, a multi-file Python module, an external CLI/Docker wrapper, an Airflow DAG, or any arbitrary file structure.

Because tools can take varied forms, **the agent does not assume a tool is invoked via `run`**. Instead, the agent follows a **two-tier discovery protocol**:
1. At session start, the agent inspects [main/bot/tools/README.md](main/bot/tools/README.md) to see what tools exist and their general descriptions.
2. When the agent needs to use a specific tool, it reads that tool's `main/bot/tools/<tool-name>/README.md`, where the **`## How to use it`** section provides the exact invocation syntax and prerequisites.

---

### Recommended Tool Layout (The Gold Standard)

To ensure the best ergonomics, ease of execution for agents, and full audibility for humans, the recommended layout is:

```
main/bot/tools/<tool-name>/
├── README.md     ← tool documentation (mandatory)
├── run           ← executable script (chmod +x, recommended)
├── verify/       ← verification & audit chamber (recommended)
│   ├── dry-run   ← executable: simulation mode with zero mutations
│   └── history.jsonl ← append-only telemetry of runs and metrics
└── ...           ← any auxiliary scripts, configs, templates, DAGs, etc.
```

### 1. Documentation: `README.md` (Mandatory)

Every tool's `README.md` must contain exactly these **4 standard sections**:

```markdown
## Description
Clearly explains what the tool does to determine if it is the right tool to use.
Note: this section is automatically parsed by the start script to regenerate the index in main/bot/tools/README.md.

## How to use it
Explains the invocation syntax, expected arguments, flags, and any environment/dependency requirements.
Agents rely on this section to know how to execute the tool!

## Examples
```bash
bot/tools/<tool-name>/run arg1 arg2
# or: python -m tool_package --flag
```

## Details
Explains the internal mechanics of the tool in detail:
- Assumptions (e.g. expected cwd)
- Resources read or written (side effects)
- Algorithmic logic, templates, or external services used
```

### 2. Executable script: `run` (Recommended)

When possible, provide a `run` entry point script with `chmod +x`.

For **Python** tools, the project standard is to use **`uv` with inline script metadata (PEP 723)**:

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.11"
# dependencies = [
#     # Add any external libraries here, e.g.:
#     # "httpx",
# ]
# ///

# Your Python code here
```

#### Why `uv` is recommended:
- **Zero manual virtualenvs:** no need to create or activate virtual environments in the repository.
- **Isolated environment per tool:** each tool gets its own separate virtual environment (managed in `~/.cache/uv/environments-v2/`). Dependencies of one tool will never conflict with another.
- **Automatic cache and instant startup:** downloads declared dependencies on first run and launches in ~10ms on subsequent runs.
- **Transparent invocation:** the agent simply executes the command declared in `README.md`.

### 3. Verification & Auditing: `verify/` (Recommended)

To make vibecoded tools understandable, testable, and auditable without forcing humans to read through hundreds of lines of code:
- **`verify/dry-run`**: A non-destructive executable that simulates execution, logging what inputs were parsed and what side-effects (file writes, reads, network calls) would have happened without actually mutating anything.
- **`verify/history.jsonl`**: An append-only log tracking telemetry from runs (wall-clock time, peak RSS memory, CPU time, exit code, and detected mutations).

### 4. Permissions and Testing

Whenever an executable script (like `run` or `verify/dry-run`) is created or modified:
1. **Make it executable:**
   ```bash
   chmod +x main/bot/tools/<tool-name>/run
   ```
2. **Test execution:**
   Run the script and verify that stdout/stderr output and any side effects match expectations.

### Automatic Tools Index

When `bot/tools/start/run` is executed, the script scans all subfolders in `bot/tools/`, extracts the `## Description` section from each tool's `README.md`, and automatically regenerates the [main/bot/tools/README.md](main/bot/tools/README.md) index file.

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