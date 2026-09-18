The user opens the `main/` directory, starts their harness in that directory, and tells it: "read start.md".
The agent reads `start.md` and runs the specified command.
From that point on, the agent will know how to proceed.

The agent's world consists of 3 parts, with a folder for each: `tasks`, `memory`, `objects`.
These are designed to be inspected and modified similarly by both the agent and the human operator.

## Guiding Principle

When modifying anything in this system, put yourself in the shoes of the agent that will use it: read the prompts and imagine what the agent would do step-by-step. If in doubt, ask the human operator what happened in the session that prompted the changes or additions.

## Tools

Every tool available to the agent lives in `main/bot/tools/<name>/`.

### The Contract
The **only mandatory invariant** for any tool is its `README.md`. It must contain exactly 4 sections:
1. `## Description`: what the tool does (used by the `start` script to generate the tools index).
2. `## How to use it`: invocation syntax, parameters, flags, and prerequisites. **The agent reads this section to learn how to run the tool.**
3. `## Examples`: practical command-line examples.
4. `## Details`: internal mechanics, assumptions (e.g. expected cwd), resources read/written.

Tools can have arbitrary files, scripts, or structures inside their folder. The agent does not assume a tool is named `run`: it reads `bot/tools/<name>/README.md` first to know how to execute it.

### Recommended Standards
- **`run`** — executable script (`chmod +x`). For Python scripts, use `uv` with PEP 723 inline script metadata (`#!/usr/bin/env -S uv run --script` and a `# /// script` block) for frictionless execution.
- **`verify/`** — verification suite containing `dry-run` (non-destructive execution simulator) and `history.jsonl` (append-only telemetry of runs).

**Core rule:** every time you create or modify an executable script:
1. **Make it executable:** `chmod +x <path-to-script>`
2. **Test it:** execute it directly from within `main/` and verify that output and side effects match expectations.

The `start` tool is the session bootstrap: it regenerates the tools index in `main/bot/tools/README.md` and prints the current date, time, and directory tree of `main/`.