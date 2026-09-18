The user opens the `main/` directory, starts their harness in that directory, and tells it: "read start.md".
The agent reads `start.md` and runs the specified command.
From that point on, the agent will know how to proceed.

The agent's world consists of 3 parts, with a folder for each: `tasks`, `memory`, `objects`.
These are designed to be inspected and modified similarly by both the agent and the human operator.

## Guiding Principle

When modifying anything in this system, put yourself in the shoes of the agent that will use it: read the prompts and imagine what the agent would do step-by-step. If in doubt, ask the human operator what happened in the session that prompted the changes or additions.

## Tools (`run`)

Every tool available to the agent lives in `main/bot/tools/<name>/` and must contain:
- **`README.md`** — tool documentation. It must contain exactly 4 sections:
  1. `## Description`: what the tool does (used by the `start` script to generate the tools index).
  2. `## How to use it`: invocation syntax, parameters, and flags.
  3. `## Examples`: practical command-line examples.
  4. `## Details`: internal mechanics, assumptions (e.g. expected cwd), resources read/written.
- **`run`** — executable script (with `+x` permissions).
  - For Python scripts, the standard is to use `uv` with PEP 723 inline script metadata (`#!/usr/bin/env -S uv run --script` and a `# /// script` block) to declare dependencies in a self-contained, isolated manner.
- **`config/`** — folder for auxiliary resources needed by the tool (templates, static data, configuration files).

**Core rules:** every time you create or modify a `run` script:
1. **Make it executable:** `chmod +x main/bot/tools/<name>/run`
2. **Test it:** execute it directly from within `main/` and verify that output and side effects match expectations.

The `start` tool is the session bootstrap: it regenerates the tools index in `main/bot/tools/README.md` and prints the current date, time, and directory tree of `main/`.