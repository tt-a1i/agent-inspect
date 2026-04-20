---
name: agent-inspect
description: Use this skill whenever the user asks to inspect, audit, review, assess, or evaluate a codebase/project for maintainability, extensibility, architecture, reliability, security, testing, engineering quality, performance, or AI-agent/prompt/tooling risks. Also use this when invoked by `/inspect`. It performs a read-only, evidence-backed, multi-agent project inspection with blunt technical findings and explicit degraded-mode disclosure.
compatibility: opencode
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
   - Performance and frontend interaction behavior
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

Use 3 parallel subagents by default:

1. Subagent A: architecture, module boundaries, maintainability, extensibility
2. Subagent B: reliability, error handling, security, AI-specific risks
3. Subagent C: testing, engineering quality, performance, frontend interaction, developer experience

Main-thread responsibilities:

1. Review the current worktree and recent commits first, then provide that context to subagents.
2. After collecting subagent conclusions, directly verify at least 3 classes of key evidence:
   - The source location of one high-severity finding
   - One configuration or script location related to testing or engineering quality
   - One finding you consider most debatable or most likely to be a false positive
3. If main-thread verification conflicts with a subagent conclusion, explicitly report the conflict instead of silently resolving it.
4. If some subagents are queued, fail, time out, or never return, do not pretend that a full parallel inspection completed. Enter **degraded mode** and explicitly report:
   - Which subagents returned successfully
   - Which subagents did not return
   - Which conclusions mainly come from main-thread verification
5. If the runtime does not support reliable native parallel subagents at all, continue with a read-only inspection only in degraded mode, and say so explicitly in the Executive Summary or Residual Risks.

## Sampling Rules for Large Repositories

1. Prioritize directories with the highest recent change volume and core source directories.
2. If there are obvious large files, primary entrypoints, main service files, or main component files, sample them directly.
3. Even if recent changes are concentrated in one area, do not completely skip tests, config files, or engineering-related files.

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
Summarize the overall project quality in 3-6 sentences.

### Top Findings
Order findings by severity:
- Critical
- High
- Medium
- Low

Each finding must include:
- A concise problem statement
- File path
- As-specific-as-possible line numbers
- Why it matters

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

1. If the repository is large, prioritize key directories and the most recently changed areas.
2. Within each finding, evidence comes before broad interpretation. Avoid generic evaluation.
3. If no serious issues are found, say so explicitly instead of inventing problems.
4. If the user passes extra context, treat it as an additional focus area without narrowing the default audit dimensions.
5. Keep the output language aligned with the user's language unless the user explicitly requests another language.
6. If your conclusions mainly come from sampling instead of broad coverage, say so in Residual Risks or Verdict.
7. If your conclusions mainly come from main-thread verification instead of multiple successful subagent returns, say so explicitly.
