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

To ensure the best ergonomics, ease of execution for agents, and full audibility for humans without code duplication, the recommended layout is:

```
main/bot/tools/<tool-name>/
├── README.md     ← tool documentation & behavioral contract (mandatory)
├── run           ← executable script with optional --dry-run (chmod +x, recommended)
├── history.jsonl ← append-only telemetry of runs and metrics (optional)
└── ...           ← any auxiliary scripts, configs, templates, DAGs, etc.
```

### 1. Documentation & Contract: `README.md` (Mandatory)

The `README.md` is the single source of truth for both humans and agents. It serves as both user documentation and the behavioral contract (Inputs, Outputs, Side Effects).

We recommend structuring it with these **4 standard sections**:

```markdown
## Description
Clearly explains what the tool does.
Note: parsed by the start script to populate main/bot/tools/README.md (if omitted, the tool is still indexed by name).

## How to use it
Explains invocation syntax, parameters, and flags (such as --dry-run).
Agents rely on this section to know how to execute the tool!

## Examples
```bash
bot/tools/<tool-name>/run arg1 arg2
bot/tools/<tool-name>/run arg1 --dry-run
```

## Details
Specifies the behavioral contract:
- **Inputs**: expected parameters, environment variables, secrets.
- **Outputs**: STDOUT format, STDERR diagnostics, exit codes.
- **Side Effects**: network calls, file reads/writes/deletions, subprocesses.
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

#### Native `--dry-run` Support
To make tools inspectable without code duplication, tools are encouraged to support a `--dry-run` flag directly in `run`. In dry-run mode, the script:
1. Validates inputs and environment without making destructive or external changes.
2. Prints an execution plan (what network calls or file mutations *would* happen).
3. Prints an output format preview.
4. Measures execution telemetry (duration, peak RAM) and records the receipt to `history.jsonl`.

### 3. Telemetry & Auditing: `history.jsonl` (Optional)

Tools can record execution receipts (timestamp, mode, exit code, duration in ms, peak RSS memory in MB, and effect verification) to `history.jsonl` in JSON Lines format for auditing and debugging.

### 4. Permissions and Testing

Whenever an executable script (like `run`) is created or modified:
1. **Make it executable:**
   ```bash
   chmod +x main/bot/tools/<tool-name>/run
   ```
2. **Test execution:**
   Run both the real execution and `--dry-run` to verify that output, exit codes, and side effects match expectations.

### Automatic Tools Index

When `bot/tools/start/run` is executed, the script scans all subfolders in `bot/tools/`, extracts the `## Description` section from each tool's `README.md` (if present), and automatically regenerates the [main/bot/tools/README.md](main/bot/tools/README.md) index file.

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