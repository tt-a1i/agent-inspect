# Installation

`agent-inspect` ships as two files that must be installed together:

1. `commands/inspect.md` — the `/inspect` slash command wrapper
2. `skills/agent-inspect/SKILL.md` — the inspection engine

Installing only the command or prompt wrapper is an incomplete setup and does not provide the full inspection method. The skill body is platform-neutral; global install paths differ per platform, and Codex currently has no project-scoped slash prompt discovery.

| Platform | Status | Command path | Skill path |
|----------|--------|--------------|------------|
| [OpenCode](#install-on-opencode) | ✅ supported | `~/.config/opencode/commands/inspect.md` | `~/.config/opencode/skills/agent-inspect/` |
| [Claude Code](#install-on-claude-code) | ✅ supported | `~/.claude/commands/inspect.md` | `~/.claude/skills/agent-inspect/` |
| [Codex](#install-on-codex) | ⚠️ partial: global prompt path supported; project-level prompts are not supported ([codex#9848](https://github.com/openai/codex/issues/9848)) | `~/.codex/prompts/inspect.md` | `~/.codex/skills/agent-inspect/` |

## Install on OpenCode

### Prerequisites

1. OpenCode with support for custom markdown commands and skills
2. Writable command and skill directories:

```text
~/.config/opencode/commands/
~/.config/opencode/skills/
```

If you are unsure whether your OpenCode version supports custom commands, check the official commands documentation first.

### Steps

Run from the repository root after `git clone` (or set `$REPO=/path/to/agent-inspect` and prefix source paths with `$REPO/`).

```bash
mkdir -p ~/.config/opencode/commands ~/.config/opencode/skills
cp commands/inspect.md ~/.config/opencode/commands/inspect.md
cp -R skills/agent-inspect ~/.config/opencode/skills/agent-inspect
```

If OpenCode is already running, restart it once, then run `/inspect` in any repository.

### Project-level override (optional)

```bash
mkdir -p .opencode/commands .opencode/skills
cp commands/inspect.md .opencode/commands/inspect.md
cp -R skills/agent-inspect .opencode/skills/agent-inspect
```

Project-level commands and skills override global ones. Keep the command and skill from the same release.

## Install on Claude Code

### Prerequisites

1. Claude Code CLI installed and logged in
2. Writable command and skill directories:

```text
~/.claude/commands/
~/.claude/skills/
```

### Steps

Run from the repository root after `git clone` (or set `$REPO=/path/to/agent-inspect` and prefix source paths with `$REPO/`).

```bash
mkdir -p ~/.claude/commands ~/.claude/skills
cp commands/inspect.md ~/.claude/commands/inspect.md
cp -R skills/agent-inspect ~/.claude/skills/agent-inspect
```

Open a new Claude Code session (or run `/reload-plugins` / restart) and run `/inspect` in any repository.

On Claude Code, `/inspect` uses the native `Task` tool to dispatch its 7 per-dimension subagents. The skill's "host-platform native subagents" requirement is satisfied by `Task` — do not substitute the `squad` plugin or other external multi-agent frameworks.

### Project-level override (optional)

```bash
mkdir -p .claude/commands .claude/skills
cp commands/inspect.md .claude/commands/inspect.md
cp -R skills/agent-inspect .claude/skills/agent-inspect
```

Project-level commands and skills override user-level ones.

## Install on Codex

### Prerequisites

1. Codex installed and logged in
2. Writable prompt and skill directories:

```text
~/.codex/prompts/
~/.codex/skills/
```

### Steps

Run from the repository root after `git clone` (or set `$REPO=/path/to/agent-inspect` and prefix source paths with `$REPO/`).

```bash
mkdir -p ~/.codex/prompts ~/.codex/skills
cp commands/inspect.md ~/.codex/prompts/inspect.md
cp -R skills/agent-inspect ~/.codex/skills/agent-inspect
```

Restart Codex, then run `/inspect` in any repository.

On Codex, the command wrapper is installed as a native custom prompt at `~/.codex/prompts/inspect.md`. The inspection method still lives in the `agent-inspect` skill. If your Codex setup uses shared skill discovery via `~/.agents/skills/`, a symlinked install there can also work, but `~/.codex/skills/` is the documented path for this project.

Codex support depends on Codex native collaboration/subagent features for the default 7-agent inspection. If your local Codex setup cannot sustain that concurrency, `/inspect` must enter the skill's documented degraded mode instead of silently substituting an external multi-agent framework.

## Quick Verification

After installing on your chosen platform:

1. Open the CLI inside any repository
2. Type `/inspect`
3. Confirm the command appears in the command list with its description
4. Run it once and verify that it loads or follows the `agent-inspect` skill and starts a read-only inspection flow instead of editing files

## Verify the Files Exist

```bash
# OpenCode
ls ~/.config/opencode/commands/inspect.md
ls ~/.config/opencode/skills/agent-inspect/SKILL.md

# Claude Code
ls ~/.claude/commands/inspect.md
ls ~/.claude/skills/agent-inspect/SKILL.md

# Codex
ls ~/.codex/prompts/inspect.md
ls ~/.codex/skills/agent-inspect/SKILL.md
```

This only proves that the files exist. It does **not** prove that the CLI has loaded them, so still do the Quick Verification above.

## Update

Update by pulling the latest repository state, then reinstalling the command wrapper and skill using the same platform block you used for installation.

```bash
git pull
```

For OpenCode:

```bash
cp commands/inspect.md ~/.config/opencode/commands/inspect.md
rm -rf ~/.config/opencode/skills/agent-inspect
cp -R skills/agent-inspect ~/.config/opencode/skills/agent-inspect
```

For Claude Code:

```bash
cp commands/inspect.md ~/.claude/commands/inspect.md
rm -rf ~/.claude/skills/agent-inspect
cp -R skills/agent-inspect ~/.claude/skills/agent-inspect
```

For Codex:

```bash
cp commands/inspect.md ~/.codex/prompts/inspect.md
rm -rf ~/.codex/skills/agent-inspect
cp -R skills/agent-inspect ~/.codex/skills/agent-inspect
```

Restart or reload the host CLI after updating. Keep `commands/inspect.md` and `skills/agent-inspect/` from the same release or commit. If release tags lag behind the changelog, install from `main` when you need the latest documented behavior.

## Uninstall

Remove both the command wrapper and the skill. Removing only one leaves a confusing partial install behind.

For OpenCode:

```bash
rm -f ~/.config/opencode/commands/inspect.md
rm -rf ~/.config/opencode/skills/agent-inspect
```

For Claude Code:

```bash
rm -f ~/.claude/commands/inspect.md
rm -rf ~/.claude/skills/agent-inspect
```

For Codex:

```bash
rm -f ~/.codex/prompts/inspect.md
rm -rf ~/.codex/skills/agent-inspect
```

If you installed project-level overrides for OpenCode or Claude Code, also remove the matching project files:

```bash
rm -f .opencode/commands/inspect.md
rm -rf .opencode/skills/agent-inspect
rm -f .claude/commands/inspect.md
rm -rf .claude/skills/agent-inspect
```

## CI / Non-Interactive Use

`/inspect` is an interactive slash command that expects a host CLI session capable of loading the command wrapper, loading the `agent-inspect` skill, and dispatching native subagents. It is not currently packaged as a standalone non-interactive binary or script.

For CI, the practical option today is to wrap the host CLI if that CLI supports non-interactive slash-command execution in your environment. The wrapper should fail the job when `/inspect` cannot load the skill, enters an unacceptable degraded mode, or reports blocking findings. This repository does not provide a supported CI wrapper yet, so treat CI usage as platform-specific glue rather than a guaranteed project feature.

## Debug Failed Runs

Common symptoms and checks:

1. `/inspect` does not appear: check the command or prompt path for your platform, then restart or reload the CLI.
2. The command appears but the skill is not found: check that `skills/agent-inspect/SKILL.md` exists under the matching platform skill directory.
3. Codex project-level install does not work: Codex currently discovers custom slash prompts from `$CODEX_HOME/prompts`, not project `.codex/prompts` directories; see [codex#9848](https://github.com/openai/codex/issues/9848).
4. The report claims degraded mode: inspect the report's degraded-mode disclosure before trusting coverage-sensitive conclusions.
5. Output structure is missing the scorecard or evidence fields: compare the installed skill with this repository's `skills/agent-inspect/SKILL.md`, then reinstall both files from the same release or commit.

## Troubleshooting

### `/inspect` does not appear in the CLI

1. Confirm the command file exists at the correct path for your platform
2. Restart the CLI or run its reload command (`/reload-plugins` on Claude Code)
3. Check that the YAML frontmatter block is still present at the top of the command file

### The command exists but behaves unexpectedly

1. Open the installed file and confirm it matches the repository version in `commands/inspect.md`
2. Confirm the installed skill matches the repository version in `skills/agent-inspect/SKILL.md`
3. Check whether a project-level override (`.opencode/` or `.claude/`) is shadowing the global version
4. If subagents are queued, fail, or time out, `/inspect` may enter degraded mode — the report should explicitly disclose evidence-coverage limitations. See the skill's Degradation Ladder for the 7 → 5 → 3 agent fallback.

### I only want this command for one project

Use project-level paths instead of global paths for OpenCode and Claude Code:

```text
.opencode/commands/inspect.md   +  .opencode/skills/agent-inspect/SKILL.md     # OpenCode
.claude/commands/inspect.md     +  .claude/skills/agent-inspect/SKILL.md       # Claude Code
```

Codex currently has no project-scoped slash prompt discovery; install the prompt globally at `~/.codex/prompts/inspect.md`.

## Notes

1. The command is a thin wrapper; the skill owns the inspection method.
2. The workflow is read-only by default. It does not edit files unless the user explicitly asks for remediation.
3. The skill asks the host CLI to use native parallel subagents. On OpenCode, Claude Code, and Codex this means the host platform's built-in mechanism — not external multi-agent tools.
4. It is optimized for audit-style output, not automatic remediation.
5. Degraded mode (both subagent-degraded and over-budget-degraded) belongs to the skill, not the command wrapper.
6. Codex currently has no project-scoped slash prompt discovery; install the prompt globally.
