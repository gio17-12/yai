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

## Why should i install UV?

The only standard tool implemented by me is the start/ tool, which helps the agent get some context at the start of the session. That tool requires UV. If you don't want to use my tool, you can avoid installing UV.

## Other notes (still to organize - don't rely on them)

The user opens the `main/` directory, starts their harness in that directory, and tells it: "read start.md".
The agent reads `start.md` and runs the specified command.
From that point on, the agent will know how to proceed.

The agent's world consists of 3 parts, with a folder for each: `tasks`, `memory`, `objects`.
These are designed to be inspected and modified similarly by both the agent and the human operator.

## Guiding Principle (still to organize - don't rely on them)

When modifying anything in this system, put yourself in the shoes of the agent that will use it: read the prompts and imagine what the agent would do step-by-step. If in doubt, ask the human operator what happened in the session that prompted the changes or additions.

these are some notes that still need to be verified, so don't reply on them.
"
The user opens the `main/` directory, starts their harness in that directory, and tells it: "read start.md".
The agent reads `start.md` and runs the specified command.
From that point on, the agent will know how to proceed.

The agent's world consists of 3 parts, with a folder for each: `tasks`, `memory`, `objects`.
These are designed to be inspected and modified similarly by both the agent and the human operator.

## Guiding Principle (still to organize - don't rely on them)

When modifying anything in this system, put yourself in the shoes of the agent that will use it: read the prompts and imagine what the agent would do step-by-step. If in doubt, ask the human operator what happened in the session that prompted the changes or additions.

## Tools (still to organize - don't rely on them)

For now, only the start/ tool has been implemented. Here are described the rules for that specific tool.
It's not decided that these rules will apply also to other tools.
"
the tool lives in `main/bot/tools/<name>/` and contains:
- **`README.md`** — tool documentation. Recommended sections:
  1. `## Description`: what the tool does (parsed by `start` to build the tools catalog).
  2. `## How to use it`: invocation syntax, arguments, and options.
  3. `## Examples`: practical command-line examples.
  4. `## Details`: internal mechanics, assumptions, resources read or written.
- **`run`** — executable script (`chmod +x`).
  - For Python scripts, use `uv` with PEP 723 inline script metadata (`#!/usr/bin/env -S uv run --script` and a `# /// script` block) to declare dependencies cleanly.

Supporting files or templates needed by the tool can live alongside `run` or in subdirectories within the tool folder.

**Core rules:** every time you create or modify a tool:
1. **Make it executable:** `chmod +x main/bot/tools/<name>/run`
2. **Test it:** execute it directly from within `main/` and verify that output and behavior match expectations.

The `start` tool is the session bootstrap: it regenerates the tools index in `main/bot/tools/README.md` and prints the current date, time, and directory tree of `main/`.
"
"
.