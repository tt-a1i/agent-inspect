# Contributing

Thanks for contributing to `agent-inspect`.

## Scope

This project is intentionally small. Keep changes focused on:

1. Improving the `/inspect` command prompt
2. Improving installation and usage docs
3. Improving examples and output clarity
4. Fixing distribution or installation issues

Avoid turning this repo into a general-purpose framework unless there is a clear need.

## Workflow

1. Open an issue first for larger prompt or behavior changes
2. Keep pull requests small and specific
3. Explain why the change improves audit quality, stability, or usability

## Style

1. Prefer precise language over marketing language
2. Keep the command read-only by default
3. Preserve the audit-report style output
4. Avoid adding unnecessary files, tooling, or dependencies

## What to include in a PR

1. What changed
2. Why it helps
3. Any behavior change users should expect

## PR Checklist

Before requesting review, confirm:

1. README English and Chinese sections stay in sync when user-facing behavior, install paths, or output structure changes.
2. `docs/installation.md` is updated for any install, update, uninstall, troubleshooting, or platform-support change.
3. `CHANGELOG.md` records user-visible changes under `Unreleased` or the correct release section.
4. `examples/sample-output.md` still reverse-proves any `skills/agent-inspect/SKILL.md` output or methodology change.
5. OpenCode, Claude Code, and Codex paths have been checked, or the PR states which platform could not be verified and why.
6. The read-only inspection constraint remains intact: `/inspect` must not edit files unless the user explicitly asks for remediation.
