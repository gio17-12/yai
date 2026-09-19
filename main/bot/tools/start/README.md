## Description
Computes the current date/time and prints the directory tree of `main/` (full repo topology, including `bot/`), rendered from the template in `templates/output.md`. It is the entry point of every session: it is what `start.md`, in the root, instructs the assistant to execute first — to know what day it is, what time it is, and have the full system layout in view before doing anything else.

## How to use it
<!-- SYNTAX:START -->
No arguments. Run: `bot/tools/start/run`
<!-- SYNTAX:END -->

## Examples
```bash
bot/tools/start/run
```

## Details
The script performs two tasks in sequence: builds the directory tree, then injects it into a template along with the current date/time.

**Behavior:**
- **Input**: filesystem layout of `main/`
- **Reads**: `templates/index.md`, `templates/output.md`, `bot/tools/*/README.md`
- **Writes**: `bot/tools/README.md` (regenerated tool index)
- **Output on stdout**: session context (date, time, tree)

**Tree construction (`tree()`):** starts from the `main/` root directory (resolved dynamically relative to the script's location) and recursively descends into each subdirectory, skipping hidden files and folders (starting with `.`) and any names specified in `EXCLUDE` (a set defined at the top of `run`). Entries at each level are sorted alphabetically with directories before files, and the visual connectors (`├──`, `└──`, indentation with `│`) replicate standard shell `tree` output. There is no depth limit: it descends as long as subdirectories exist.

**Output composition:** the file `templates/output.md` contains static text plus four placeholders — `{{DATE}}`, `{{WEEKDAY}}`, `{{TIME}}`, `{{TREE}}`. The script reads the file as a string and performs direct string replacement (no external templating engine, just sequential `.replace()`) before printing the result to stdout.

**Path resolution:** the script dynamically resolves `main/` and `bot/tools/` relative to `__file__` (`TOOL_DIR.parents[2]`). It works seamlessly whether executed from inside `main/`, from the repository root (`main/bot/tools/start/run`), or from an absolute path.