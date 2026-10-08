---
name: implementation
description: Implementation principles to follow when writing or modifying code. Use when deciding whether to touch code unrelated to the change, how to handle generated files, and whether to add temporary code.
---

# implementation

When the structure or conventions a project already maintains differ from the principles below, follow the project's.

- Do not reformat or rename code unrelated to the change.
- Do not edit generated files (code generation output, vendor directories, lock files, etc.) directly. Fix the source, then regenerate them with the project's generation command or package manager.
- Do not add temporary code such as stopgap fallbacks or hardcoded values.
