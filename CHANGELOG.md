# Changelog

## v0.2.0

- Added explicit degraded-mode rules when subagents are queued, unavailable, or do not return
- Required final reports to disclose when findings mainly come from main-thread verification instead of successful parallel subagents
- Updated README and installation docs to explain degraded mode and make setup guidance clearer
- Explicitly required host-platform native subagents instead of external multi-agent tools like squad by default
- Removed `subtask: true` so `/inspect` runs in the primary context and can orchestrate subagents correctly
- Added a blunt, kernel-style technical review tone that criticizes code directly without attacking people
- Added lightweight visual markers for clearer audit reports
- Converted `/inspect` into a thin command wrapper backed by the `agent-inspect` skill

## v0.1.0

- Added the `/inspect` OpenCode command for multi-agent project inspection
- Added installation documentation
- Added sample audit output
- Added language-following output behavior
- Added clearer agent coordination and evidence-checking rules
