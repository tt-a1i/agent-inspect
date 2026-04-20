# Installation

## Prerequisites

1. OpenCode with support for custom markdown commands
2. A writable commands directory:

```text
~/.config/opencode/commands/
```

If you are not sure whether your OpenCode version supports custom commands, check the official OpenCode commands documentation first.

## Install as a global OpenCode command

Copy the command file into your global OpenCode commands directory:

```bash
mkdir -p ~/.config/opencode/commands
cp commands/inspect.md ~/.config/opencode/commands/inspect.md
```

Then restart OpenCode if needed and run:

```text
/inspect
```

## Quick Verification

After installation:

1. Open OpenCode in any repository
2. Type `/inspect`
3. Confirm the command appears in the command list with its description
4. Run it once and check that it starts a read-only audit flow instead of editing files

## Verify the command exists

Check the file is present:

```bash
ls ~/.config/opencode/commands/inspect.md
```

This only proves the file exists. It does **not** prove OpenCode has loaded the command, so use the quick verification steps above as well.

## Optional: project-level override

If you want a repository-specific version, create:

```text
.opencode/commands/inspect.md
```

Project-level commands can override the global command.

## Troubleshooting

### `/inspect` does not appear in OpenCode

1. Confirm the file exists at `~/.config/opencode/commands/inspect.md`
2. Restart OpenCode
3. Check that the file keeps the YAML frontmatter block at the top

### The command exists but behaves unexpectedly

1. Open the installed file and verify it matches the repository version in `commands/inspect.md`
2. Check whether a project-level `.opencode/commands/inspect.md` is overriding the global version

### I only want this command for one project

Put the command file here instead:

```text
.opencode/commands/inspect.md
```

## Notes

1. This command is designed to be read-only by default.
2. It asks the model to use multiple subagents in parallel.
3. It is optimized for audit-style output, not automatic remediation.
