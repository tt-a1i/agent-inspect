---
description: Run a multi-agent code inspection and return a structured audit report
---

# /inspect

Use the `agent-inspect` skill if it is available.

Run a read-only project inspection for the current repository.

User-supplied extra focus: $ARGUMENTS

## Requirements

1. Treat this command as the entrypoint, not the inspection engine.
2. The `agent-inspect` skill owns the full inspection method: subagent coordination, degraded mode, evidence verification, review tone, visual markers, and output structure.
3. Do not modify files unless the user explicitly asks for remediation after the inspection.
4. Match the output language to the language used by the user in the current conversation.
