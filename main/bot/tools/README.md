## Available Tools

Refer to each tool's `README.md` (`bot/tools/<tool-name>/README.md`) for execution instructions, invocation syntax, and parameters.

- **start** — Computes the current date/time and prints the directory tree of `main/` (full repo topology, including `bot/`), rendered from the template in `config/templates/output.md`. It is the entry point of every session: it is what `start.md`, in the root, instructs the assistant to execute first — to know what day it is, what time it is, and have the full system layout in view before doing anything else.
- **web-search** — Performs a web search using the Brave Search API and returns formatted markdown results (titles, URLs, snippets) suitable for LLM comprehension.