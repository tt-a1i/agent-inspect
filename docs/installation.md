# Installation

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

## Verify the command exists

Check the file is present:

```bash
ls ~/.config/opencode/commands/inspect.md
```

## Optional: project-level override

If you want a repository-specific version, create:

```text
.opencode/commands/inspect.md
```

Project-level commands can override the global command.

## Notes

1. This command is designed to be read-only by default.
2. It asks the model to use multiple subagents in parallel.
3. It is optimized for audit-style output, not automatic remediation.
