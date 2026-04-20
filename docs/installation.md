# Installation

## Prerequisites

1. OpenCode with support for custom markdown commands
2. A writable commands directory on your machine:

```text
~/.config/opencode/commands/
```

If you are not sure whether your OpenCode version supports custom commands, check the official commands documentation first.

## Install as a Global OpenCode Command

Copy the command file into your global OpenCode commands directory:

```bash
mkdir -p ~/.config/opencode/commands
cp commands/inspect.md ~/.config/opencode/commands/inspect.md
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
4. Run it once and verify that it starts a read-only inspection flow instead of editing files

## Verify the Command File Exists

Check that the file exists:

```bash
ls ~/.config/opencode/commands/inspect.md
```

This only proves that the file exists. It does **not** prove that OpenCode has loaded the command, so you should still do the quick verification steps above.

## Optional: Project-Level Override

If you want one repository to use its own version, create:

```text
.opencode/commands/inspect.md
```

Project-level commands override the global command.

## Troubleshooting

### `/inspect` does not appear in OpenCode

1. Confirm that `~/.config/opencode/commands/inspect.md` exists
2. Restart OpenCode
3. Check that the YAML frontmatter block is still present at the top of the file

### The command exists but behaves unexpectedly

1. Open the installed file and confirm that it matches the repository version in `commands/inspect.md`
2. Check whether a project-level `.opencode/commands/inspect.md` is overriding the global version
3. Note that if subagents are queued, fail, or time out, `/inspect` may enter degraded mode and the report should explicitly disclose evidence-coverage limitations

### I only want this command for one project

Put the command file here instead:

```text
.opencode/commands/inspect.md
```

## Notes

1. This command is read-only by default
2. It asks the model to use parallel subagents when the host platform supports them
3. It is optimized for audit-style output, not automatic remediation
4. If subagents are unavailable, the command must enter degraded mode explicitly instead of pretending a full parallel inspection completed
