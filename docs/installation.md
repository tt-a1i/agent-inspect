# Installation

## Prerequisites

1. OpenCode with support for custom markdown commands and skills
2. Writable commands and skills directories on your machine:

```text
~/.config/opencode/commands/
~/.config/opencode/skills/
```

If you are not sure whether your OpenCode version supports custom commands, check the official commands documentation first.

## Install as a Global OpenCode Command

This package has two required parts:

1. `commands/inspect.md` — the slash command wrapper
2. `skills/agent-inspect/SKILL.md` — the inspection engine

Install both from the same release. Installing only the command is an incomplete setup and does not provide the full inspection method.

Copy the command wrapper into your global OpenCode commands directory:

```bash
mkdir -p ~/.config/opencode/commands
cp commands/inspect.md ~/.config/opencode/commands/inspect.md
```

Then copy the skill into your global OpenCode skills directory:

```bash
mkdir -p ~/.config/opencode/skills
cp -R skills/agent-inspect ~/.config/opencode/skills/agent-inspect
```

If OpenCode is already running, restart it once and then run:

```text
/inspect
```

## Quick Verification

After installation:

1. Open OpenCode inside any repository
2. Type `/inspect`
3. Confirm the command appears in the command list with its description
4. Run it once and verify that it loads or follows the `agent-inspect` skill and starts a read-only inspection flow instead of editing files

## Verify the Command File Exists

Check that the file exists:

```bash
ls ~/.config/opencode/commands/inspect.md
ls ~/.config/opencode/skills/agent-inspect/SKILL.md
```

This only proves that the file exists. It does **not** prove that OpenCode has loaded the command, so you should still do the quick verification steps above.

## Optional: Project-Level Override

If you want one repository to use its own version, install both parts under `.opencode/`:

```text
.opencode/commands/inspect.md
.opencode/skills/agent-inspect/SKILL.md
```

Example:

```bash
mkdir -p .opencode/commands .opencode/skills
cp commands/inspect.md .opencode/commands/inspect.md
cp -R skills/agent-inspect .opencode/skills/agent-inspect
```

Project-level commands and skills override global ones. Keep the command and skill from the same release.

## Troubleshooting

### `/inspect` does not appear in OpenCode

1. Confirm that `~/.config/opencode/commands/inspect.md` exists
2. Restart OpenCode
3. Check that the YAML frontmatter block is still present at the top of the file

### The command exists but behaves unexpectedly

1. Open the installed file and confirm that it matches the repository version in `commands/inspect.md`
2. Confirm that the installed skill matches the repository version in `skills/agent-inspect/SKILL.md`
3. Check whether a project-level `.opencode/commands/inspect.md` or `.opencode/skills/agent-inspect/` is overriding the global version
4. Note that if subagents are queued, fail, or time out, `/inspect` may enter degraded mode and the report should explicitly disclose evidence-coverage limitations

### I only want this command for one project

Put the command file here instead:

```text
.opencode/commands/inspect.md
.opencode/skills/agent-inspect/SKILL.md
```

## Notes

1. The command is a thin wrapper; the skill owns the inspection method
2. This workflow is read-only by default
3. It asks the model to use parallel subagents when the host platform supports them
4. It is optimized for audit-style output, not automatic remediation
5. Degraded mode belongs to the skill, not the command wrapper
