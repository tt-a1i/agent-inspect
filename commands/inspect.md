---
description: Run a multi-agent code inspection and return a structured audit report
---

# /inspect

Use the `agent-inspect` skill. If the skill is unavailable, fails to load, or cannot be invoked, do not run a full `/inspect` review.

Run a read-only project inspection for the current repository.

User-supplied extra focus: $ARGUMENTS

## Requirements

1. Treat this command as the entrypoint, not the inspection engine.
2. The `agent-inspect` skill owns the full inspection method: subagent coordination, degraded mode, evidence verification, review tone, visual markers, and output structure.
3. If the skill is unavailable, stop and report the install/load error instead of producing a low-confidence substitute audit. Include this minimum checklist for the user to run once the skill is available: read-only execution, complete coverage or explicit degraded mode, quoted evidence for every finding, Critical/High findings must be main-thread-verified or cross-subagent-corroborated, Dimension Scorecard, and Residual Risks.
4. Do not modify files unless the user explicitly asks for remediation after the inspection.
5. Match the output language to the language used by the user in the current conversation.
