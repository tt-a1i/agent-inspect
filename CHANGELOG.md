# Changelog

## v0.4.1

- Added official **Claude Code** support. The skill body was already platform-neutral; only install paths and docs needed to change.
- Added install steps for Claude Code at `~/.claude/commands/` and `~/.claude/skills/` in README and `docs/installation.md`.
- Added a Platform Support table (OpenCode ✅ / Claude Code ✅ / Codex 🚧 planned) to README and installation docs.
- Removed the `compatibility: opencode` frontmatter field from `skills/agent-inspect/SKILL.md`. The skill is cross-platform by default; the field was informational and misleading.
- Stubbed **Codex** as a planned platform in `docs/installation.md` with a note on what's needed (confirmed paths + native-subagent concurrency verification) for contributors.
- Clarified in Claude Code installation notes that `/inspect` uses the native `Task` tool for its 7 per-dimension subagents, and must not be substituted by the `squad` plugin or other external multi-agent frameworks.

## v0.4.0

- Default subagent count raised from 3 to 7: one subagent per dimension. This keeps each subagent focused on a single concern and gives the Dimension Scorecard independent per-dimension inputs. Added an explicit degradation ladder (7 → 5 → 3 agents) for platforms that cannot sustain 7 concurrent subagents; security, reliability, and AI-risks never collapse into other dimensions silently.
- Replaced `Sampling Rules for Large Repositories` with `Coverage Rules`. Default mode is complete coverage: every file in scope must be fully read by exactly one subagent. Large files are split into overlapping passes handled by the same subagent, not sampled.
- Added **over-budget degraded mode** for repositories too large to cover completely. In this mode, every file that was NOT fully read is listed by absolute path in Residual Risks, and the Executive Summary and Verdict must not claim "complete evaluation". No silent sampling is allowed.
- Removed the 400 LOC per-file cap that v0.3.0 adopted from Google's human-reviewer data. That figure measured human attention fatigue, not LLM attention, and was the wrong constraint for this tool.
- Added explicit scope exclusions (dependency trees, lock files, build output, binary assets). They must be listed at the top of the report, not silently dropped. User-supplied include/exclude arguments to `/inspect` override these defaults and are also disclosed.
- Added a new `Dimension Scorecard` section to the output format, positioned between Executive Summary and Top Findings. Every dimension gets a 1-5 named score (🔴 Critical debt → ✅ Exemplary) with rationale citing at least one finding ID, and a `Scored by` line. Decimals and arbitrary numeric scales are not allowed.
- Subagent-proposed scores may be overridden by the main thread after direct verification; both scores must appear in the scorecard entry with a one-sentence reason, never silently reconciled.
- Scoring guardrails: 🔴 1 and ✅ 5 require at least two distinct findings or source locations as evidence. ✅ 5 is not allowed in any dimension with an open High or Critical finding. Merged-dimension and partial-coverage entries must state their limitations.
- Executive Summary must now state the subagent count, whether complete coverage was achieved, and reference the lowest-scored dimension(s).
- Reconciled Execution Notes with the new coverage model: removed the "prioritize and sample" residue, clarified that sampling is treated as a failure mode.
- Updated `examples/sample-output.md` to demonstrate the 7-dimension scorecard, a main-thread score override disagreeing with a subagent proposal, and the new default scope-exclusions disclosure.

## v0.3.0

- Expanded the finding schema: every finding must now include `ID`, `Title`, `Severity`, `Blocking`, `Location`, `Evidence` (3-15 quoted lines of actual source), `Why it matters`, `Recommendation`, `Confidence`, and `Verification`. `CWE` and `cross_dimension_links` are optional. Prose summaries alone no longer count as evidence.
- Required each finding to quote the source code that triggered it. This is the single strongest defense against hallucinated findings; readers can verify claims without opening the repository.
- Added `Blocking: yes/no` to each finding, separate from severity. A High finding can be non-blocking if well scoped, a Medium can be blocking if it hides real bugs behind broad fallback.
- Added `Confidence` and `Verification` fields so per-finding trust level is explicit instead of being implied only by the section-level degraded-mode disclosure.
- Added an explicit deduplication and merge pass on the triple `(file, line range, category)` in main-thread responsibilities. Dimension-split subagents will overlap; unmerged reports dilute signal.
- Added commit-trailer scanning (`Co-authored-by:`, `Assisted-by:`) so files touched by AI-authored commits get extra correctness scrutiny. Explicitly refused heuristic AI-origin guessing (comment density, emoji, style) as noise.
- Capped per-file sampling at roughly 400 lines of code per pass, based on public defect-detection data. Uncovered ranges must be listed in Residual Risks rather than pretended inspected.
- Explicitly rejected CVSS vector emission. LLM-generated CVSS without runtime, exploit, and deployment context produces plausible-looking but wrong scores, which is exactly the failure mode this skill is written to avoid. CWE IDs are allowed when they map cleanly.
- Cut `Performance and frontend interaction behavior` as a top-level dimension. It was weak in practice and too project-specific for a general audit tool. Performance concerns that matter for a given project can still surface under reliability or maintainability.
- Removed `performance` from the skill's frontmatter trigger list to match the dimension cut.
- Updated `examples/sample-output.md` to demonstrate the new schema end-to-end.

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
