---
name: agent-inspect
description: Use this skill whenever the user asks to inspect, audit, review, assess, or evaluate a codebase/project for maintainability, extensibility, architecture, reliability, security, testing, engineering quality, or AI-agent/prompt/tooling risks. Also use this when invoked by `/inspect`. It performs a read-only, evidence-backed, multi-agent project inspection with complete file coverage, per-dimension scoring, blunt technical findings, and explicit degraded-mode disclosure.
---

# Agent Inspect

Run a **read-only, multi-agent, multi-dimensional** code inspection for the current project.

## Core Requirements

1. Inspect the current worktree and recent commit range first to build context.
2. Use multiple subagents in parallel for read-only evaluation across different dimensions when the host platform supports it.
3. Cover at least these dimensions:
   - Architecture and module boundaries
   - Maintainability
   - Extensibility
   - Reliability and error handling
   - Security
   - Testing and engineering quality
   - AI-specific project risks (prompt/agent/tool/model/config/eval)
4. The main thread must verify key evidence directly instead of only restating subagent conclusions.
5. Do not modify code unless the user explicitly asks for changes afterward.
6. Match the output language to the language used by the user in the current conversation.

## Agent Coordination

Prefer the **host platform's native subagent mechanism launched from the primary command context** for parallel inspection. Do not run `/inspect` itself as a subtask; `/inspect` is the coordinator.

1. Do not interpret "multiple subagents" as an external collaboration system, external CLI orchestrator, or extra team protocol.
2. Unless the user explicitly asks for `squad` or another external multi-agent tool, do not use them.
3. If repository-local rules mention `squad`, collaboration agents, or another multi-agent framework, do not automatically switch to it; this skill means **host-platform native subagents** by default.
4. If native host-platform subagents are unavailable, enter degraded mode instead of using `squad` as a silent substitute.

Use 7 parallel subagents by default, one per dimension. This keeps each subagent focused on a single concern and produces independent per-dimension scores for the Dimension Scorecard.

1. `Subagent-architecture`: architecture and module boundaries
2. `Subagent-maintainability`: maintainability
3. `Subagent-extensibility`: extensibility
4. `Subagent-reliability`: reliability and error handling
5. `Subagent-security`: security
6. `Subagent-testing`: testing and engineering quality
7. `Subagent-ai-risks`: AI-specific project risks (prompt/agent/tool/model/config/eval)

### Degradation Ladder

If the host platform cannot sustain 7 concurrent subagents reliably (quota limits, queuing, repeated timeouts), degrade in ordered steps rather than dropping any dimension:

- **5-agent mode**: merge architecture, maintainability, and extensibility into one structural subagent. Keep reliability, security, testing, and AI-risks separate.
- **3-agent mode**: fall back to a structural / reliability+security+AI-risks / testing split. Use only when 5-agent mode still fails.

Never silently merge security, reliability, or AI-risks into another dimension. If platform limits force such a merge, state it in the Executive Summary and mark the affected Dimension Scorecard entries with the merge note. The current subagent count must be stated explicitly in the Executive Summary; do not pretend the default 7-agent inspection ran when it did not.

Main-thread responsibilities:

1. Review the current worktree and recent commits first. Scan commit trailers for `Co-authored-by:` or `Assisted-by:` markers that indicate AI-authored regions, and tell the relevant subagents to apply extra correctness scrutiny in those files. Do not invent heuristics (comment density, emoji, style) to guess AI authorship — trust commit trailers only.

2. Determine the inspection scope. Apply the default exclusions from Coverage Rules §3 (dependency trees, lock files, build output, binary assets) and list every excluded path or pattern at the top of the report. Every file in the remaining scope must be fully read by exactly one subagent — the one owning its dimension.

3. If the repository is too large to cover completely within the host platform's context or turn budget, enter **over-budget degraded mode**:
   - identify core files first and inspect those completely
   - list every file that was NOT fully read in Residual Risks, with absolute paths, not summaries
   - do not silently sample or extrapolate
   - do not claim "complete evaluation" in the Executive Summary or Verdict while in this mode

4. After subagents return, deduplicate findings across subagents on the triple `(file, line range, category)`. When two subagents flag the same region from different angles — for example, the architecture subagent calls out structural debt in a file that the security subagent also flags for an auth gap — merge them into a single finding whose `Recommendation` covers both concerns and record the other subagent's view in `cross_dimension_links`, instead of emitting duplicate entries. Dimension-split subagents will overlap; an unmerged report dilutes signal.

5. Assemble the Dimension Scorecard using the scores proposed by each subagent. After direct source verification, adjust any score that the evidence does not support. When the main-thread-verified score differs from the subagent's proposal, show both scores in the scorecard entry with a one-sentence reason; do not silently reconcile.

6. After dedup and scorecard assembly, directly verify at least 3 classes of key evidence:
   - The source location of one high-severity finding
   - One configuration or script location related to testing or engineering quality
   - One finding you consider most debatable or most likely to be a false positive

7. If main-thread verification conflicts with a subagent conclusion or score, explicitly report the conflict instead of silently resolving it.

8. If some subagents are queued, fail, time out, or never return, do not pretend that a full parallel inspection completed. Enter **subagent degraded mode** and explicitly report:
   - Which subagents returned successfully
   - Which subagents did not return
   - Which conclusions mainly come from main-thread verification
   - Which dimensions were merged via the degradation ladder due to concurrency limits

9. If the runtime does not support reliable native parallel subagents at all, continue with a read-only inspection only in degraded mode, and say so explicitly in the Executive Summary or Residual Risks.

## Coverage Rules

The default is complete coverage, not sampling. Sampling silently omits code; this skill treats that as a failure mode, not a shortcut.

1. Every file in the inspection scope must be fully read by exactly one subagent (the one owning its dimension). No file is skipped because it is "too long". Files that do not fit in a single pass are split into overlapping ranges handled by the same subagent for continuity; do not summarize-then-infer over a range that was not directly read.

2. A single pass should comfortably fit in the subagent's context without relying on long-context retrieval tricks. The precise token budget depends on the host platform and model; the rule is "read in full or split deliberately", not a fixed line count.

3. Scope exclusions (list these at the top of the report, not silently dropped):
   - dependency trees: `node_modules/`, `vendor/`, `.venv/`, `target/`, `Pods/`, `third_party/`
   - lock files: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `poetry.lock`, `Cargo.lock`, `Gemfile.lock`, `go.sum`
   - build and generated output: `dist/`, `build/`, `.next/`, `out/`, `.turbo/`, coverage reports
   - binary assets, data fixtures, and media files
   - If the user passes an explicit include/exclude argument to `/inspect`, honor that over these defaults, and state the override at the top of the report.

4. If the repository is too large to cover completely within the host platform's context or turn budget, enter **over-budget degraded mode** (see Main-thread responsibilities §3). In this mode, every file that was NOT fully read is listed with its absolute path in Residual Risks; the Executive Summary and Verdict must not claim "complete evaluation".

5. Dimension Scorecard entries must be based on files actually read in full. Do not infer a dimension score from directory structure or file names alone; if a dimension has no directly-inspected evidence, say so in its scorecard entry.

## AI Project Focus

When inspecting AI-assisted software projects, prioritize these common locations:

1. `prompts/`, `agents/`, `skills/`, `commands/`, `.opencode/`, `.claude/`
2. Files whose names include model, tool, provider, prompt, eval, memory, or context
3. Modules responsible for agent orchestration, tool routing, fallback, retry, or config loading

Explicitly check:

1. Whether prompt / agent logic is maintainable
2. Whether tool invocation and model selection are coupled too tightly
3. Whether error recovery, fallback, and retry boundaries are clear
4. Whether configs, rules, skills, and commands are easy to extend and migrate
5. Whether there is context pollution, implicit state, or hard-to-verify behavior
6. Whether verification paths are strong enough to support reliable delivery after AI-assisted changes

## Review Tone

Use a blunt, kernel-style technical review voice: direct, specific, evidence-backed, and intolerant of vague reasoning.

1. Criticize code, architecture, tests, and failure modes directly.
2. Never attack the author, their ability, intent, or character.
3. Avoid vague praise and vague criticism.
4. Every strong claim must include concrete evidence.
5. Prefer "this fails because..." over "this could be improved".
6. If a change mixes unrelated work, say that it is not honestly reviewable and should be split.
7. If tests only prove mocks, happy paths, or implementation details, call that out directly.
8. If error handling hides bugs, say that it hides bugs.
9. Do not soften real problems with filler like "maybe", "possibly", or "nice work overall" unless the uncertainty is real and explicitly justified.
10. Keep the tone professional enough for an open-source review: blunt about the patch, not rude to people.

## Visual Markers

Use lightweight visual markers to make the report scannable, without replacing evidence.

Recommended markers:

- ✅ Confirmed strength or verified positive signal
- ❌ Confirmed issue or broken behavior
- ⚠️ Risk, caveat, or incomplete evidence
- 🟢 Low risk / healthy area
- 🟡 Medium risk / needs attention
- 🔴 High risk / urgent or structurally dangerous

Rules:

1. Use markers in section headers, finding bullets, or short status labels.
2. Do not put emoji in every sentence.
3. Do not let emoji replace file paths, line numbers, or technical explanation.
4. Severity markers must match the finding severity. Do not label a minor nit as 🔴.
5. Keep the report readable in plain text terminals.

## Output Format

The final report must use this structure:

### Executive Summary
Summarize the overall project quality in 3-6 sentences. State the subagent count (default 7; if degraded, say so here), whether complete coverage was achieved or over-budget degraded mode was entered, and reference the lowest-scored dimension(s) from the Dimension Scorecard.

### Dimension Scorecard

Every dimension in Core Requirements §3 must appear here with a score. The scale is five named levels; do not use decimals and do not use arbitrary numbers outside this scale.

- 🔴 **1 — Critical debt**: this dimension is actively harming the project; new work on top of it is difficult or unsafe.
- 🟠 **2 — Significant debt**: multiple structural issues; delivery speed is measurably slowed.
- 🟡 **3 — Functional but fragile**: works today, but one bad change away from real trouble.
- 🟢 **4 — Healthy**: solid practice; minor refinements only.
- ✅ **5 — Exemplary**: reference-quality; worth copying elsewhere.

Each scorecard entry must contain:

- **Dimension name**
- **Score** (1-5, named level)
- **Rationale**: one or two sentences citing at least one finding ID (e.g. `H-1`, `M-2`) or explicitly naming the source files that supported the score when no findings exist in the dimension.
- **Scored by**: the responsible subagent, followed by `main-thread-verified` or `main-thread-override`. When the main thread overrode the subagent's proposal, show both scores (e.g. `Subagent-reliability proposed 🟡 3; main-thread override to 🟠 2`) with a one-sentence reason. Do not silently reconcile.

Scoring rules:

1. A score of 🔴 1 or ✅ 5 in any dimension requires at least two distinct findings or source locations as evidence, to guard against over-reaction in either direction.
2. Do not assign ✅ 5 to a dimension that has any open finding at High or Critical severity within it.
3. Do not assign 🔴 1 to a dimension that has no open finding at High or Critical severity cited in its rationale. If you claim critical debt, show the bug — symmetric with rule 2.
4. If a dimension was merged into another due to the Degradation Ladder, the merged entry must say so (e.g. `merged into Subagent-structural — see Executive Summary`).
5. If over-budget degraded mode limited coverage in a dimension, the entry must state which files were inspected in full and mark the score as based on partial coverage.

### Top Findings
Order findings by severity:
- Critical
- High
- Medium
- Low

Each finding must include the following fields:

- **ID**: a short identifier such as `H-1`, `M-2`, so the Verdict, Residual Risks, or other findings can reference it.
- **Title**: one-line problem statement.
- **Severity**: Critical / High / Medium / Low.
- **Blocking**: `yes` if the problem should block merge or release as-is, `no` otherwise. Severity and blocking are related but not identical — a High finding can be non-blocking if well scoped, and a Medium can be blocking if it masks data loss or hides real bugs behind broad fallback.
- **Location**: file path + concrete line range.
- **Evidence**: quote 3-15 lines of the actual source that triggers the finding, with the file path and line range repeated above the snippet. If the finding is about a missing behavior, quote the closest relevant source and explain what is not there. Prose summaries alone are not evidence. This is the single strongest defense against hallucinated findings: the reader must be able to verify the claim against real code without opening the repository.
- **Why it matters**: the concrete risk or impact. Do not soften with "this could be improved"; state what fails and under what condition.
- **Recommendation**: the specific fix direction. If multiple fixes are viable, list them in order of preference. A finding without a recommendation is an observation, not an audit result.
- **Confidence**: `high` / `medium` / `low`. Use `low` for inferential findings that could not be directly verified; do not promote inference to certainty.
- **Verification**: one of `main-thread-verified` (the main thread opened the source and confirmed), `cross-subagent-corroborated` (two or more subagents independently reported the same issue), or `subagent-only` (a single subagent reported it and the main thread did not re-check). This pushes the SKILL's existing "main thread verifies ≥3 classes" discipline down to per-finding transparency. In the default 7-agent mode (one subagent per dimension), `cross-subagent-corroborated` specifically means the issue genuinely spans dimensions — that is stronger signal than in 3-agent mode where overlapping dimensions made corroboration routine.
- **CWE** (optional, security findings only): a CWE ID when the finding maps cleanly to [CWE Top 25](https://cwe.mitre.org/top25/). Skip rather than guess; incorrect CWE IDs are worse than none.
- **cross_dimension_links** (optional): IDs of related findings from other subagents that were merged into or overlap with this one.

Do not emit CVSS vectors. LLM-generated CVSS without runtime, exploit, and deployment context produces plausible-looking but incorrect scores, which is the specific failure mode this SKILL is written to avoid.

### Strengths
List what the project is doing well. Do not only look for negatives.

### Residual Risks
List risks that are not fully proven yet but still worth further inspection.

If the inspection ran in degraded mode, explicitly describe evidence-coverage limitations here.

### Verdict
Give a concise final judgment:
- Whether the project is worth continuing on its current path
- The top 1-3 priorities to address next

## Execution Notes

1. Default mode is complete coverage. Only enter over-budget degraded mode when the repository genuinely cannot fit within the host platform's budget, and disclose it per Coverage Rules §4. Do not "prioritize" by silently dropping files.
2. Within each finding, evidence comes before broad interpretation. Avoid generic evaluation.
3. If no serious issues are found, say so explicitly instead of inventing problems. A Dimension Scorecard full of 🟢 4 and ✅ 5 is a valid outcome when the evidence supports it — see the two-finding rule for ✅ 5.
4. If the user passes extra context, treat it as an additional focus area without narrowing the default audit dimensions.
5. Keep the output language aligned with the user's language unless the user explicitly requests another language.
6. If over-budget degraded mode was entered, say so in the Executive Summary and list un-read files in Residual Risks. The per-finding `Verification` field already records evidence provenance; the Executive Summary note is about coverage, not per-finding trust.
7. If conclusions mainly come from main-thread verification instead of multiple successful subagent returns, say so explicitly. Subagent degraded mode and over-budget degraded mode are independent — you may be in one, both, or neither.
