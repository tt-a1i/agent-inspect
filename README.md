# agent-inspect

[English](#english) | [中文](#中文)

## English

`agent-inspect` is a global slash command (`/inspect`) for **OpenCode**, **Claude Code**, and **Codex**.

Its goal is simple:

- trigger a **multi-agent, parallel** code inspection in any project
- return a **structured, evidence-oriented** audit report
- work especially well for staged reviews of AI-assisted software projects

Unlike a generic code review prompt, `/inspect` checks the current repository across architecture, maintainability, extensibility, reliability, security, testing, engineering quality, and AI-specific risk areas. It also requires the main thread to verify key evidence instead of only repeating subagent conclusions.

The slash command is intentionally thin. The `agent-inspect` skill owns the inspection method, review tone, degraded mode, visual markers, and output structure.

It is designed to run in the primary command context so the command itself can coordinate subagents instead of being wrapped as a subtask first.

If you want a repeatable command that answers the question, "After a lot of AI-assisted changes, is this project still maintainable and extensible?", this project is built for that.

### Platform Support

| Platform | Status | Install paths |
|----------|--------|---------------|
| OpenCode | ✅ supported | `~/.config/opencode/commands/` + `~/.config/opencode/skills/` |
| Claude Code | ✅ supported | `~/.claude/commands/` + `~/.claude/skills/` |
| Codex | ✅ supported | `~/.codex/prompts/` + `~/.codex/skills/` |

The skill body is platform-neutral: it does not depend on any OpenCode-specific API. Only the install paths differ.

### Quick Start

Install both the command wrapper and the skill from the same release. Pick the block that matches your platform.

**OpenCode**

```bash
mkdir -p ~/.config/opencode/commands ~/.config/opencode/skills
cp commands/inspect.md ~/.config/opencode/commands/inspect.md
cp -R skills/agent-inspect ~/.config/opencode/skills/agent-inspect
```

**Claude Code**

```bash
mkdir -p ~/.claude/commands ~/.claude/skills
cp commands/inspect.md ~/.claude/commands/inspect.md
cp -R skills/agent-inspect ~/.claude/skills/agent-inspect
```

**Codex**

```bash
mkdir -p ~/.codex/prompts ~/.codex/skills
cp commands/inspect.md ~/.codex/prompts/inspect.md
cp -R skills/agent-inspect ~/.codex/skills/agent-inspect
```

If you install only the command or prompt wrapper, the setup is incomplete and `/inspect` will not have access to the full inspection method.

Then run it in your CLI:

```text
/inspect
```

### Features

1. Single command entrypoint: `/inspect`
2. Thin command wrapper; the `agent-inspect` skill owns the inspection method
3. Parallel subagent inspection instead of a single-threaded generic review
4. Audit-style output focused on findings, evidence, risks, and verdicts
5. Output language follows the current user conversation
6. Explicit degraded mode when subagents are unavailable
7. Host-platform native subagents by default, not external collaboration frameworks
8. Primary-context orchestration by the command itself
9. Blunt but technical review voice: direct, evidence-first, critical of code but not people
10. Lightweight visual markers such as ✅/❌/⚠️/🟢/🟡/🔴 for scannability
11. AI-specific risk coverage for prompts, agents, tools, models, configs, and evals

### What `/inspect` Produces

Running:

```text
/inspect
```

produces a structured audit report with:

1. Executive Summary
2. Top Findings
3. Strengths
4. Residual Risks
5. Verdict

If some subagents are queued, fail, or never return, `/inspect` does not pretend that a complete parallel inspection finished. It enters degraded mode and explicitly states which conclusions mainly come from main-thread verification and which evidence-coverage limits apply.

### Use Cases

1. Post-AI-development inspection passes
2. Re-assessing maintainability after a large change set
3. Finding structural, testing, and error-handling debt
4. Auditing prompt / agent / tool / model / config risk in AI-heavy projects

### Design Principles

1. Evidence before conclusions
2. Findings before summaries
3. Read-only inspection by default
4. Multi-dimensional coverage, not one subsystem at a time
5. Explicit disclosure when subagent failures reduce evidence coverage
6. Direct criticism of code and failure modes without reviewer fluff
7. Visual markers improve scanability but never replace file paths, line numbers, or evidence

### Maintenance

- Contribution guide: `CONTRIBUTING.md`
- Changelog: `CHANGELOG.md`
- Detailed installation notes: `docs/installation.md`

### File Layout

```text
.
├── README.md
├── LICENSE
├── CHANGELOG.md
├── CONTRIBUTING.md
├── commands/
│   └── inspect.md
├── skills/
│   └── agent-inspect/
│       └── SKILL.md
├── docs/
│   └── installation.md
└── examples/
    └── sample-output.md
```

## 中文

`agent-inspect` 是一个全局 slash command（`/inspect`），支持 **OpenCode**、**Claude Code** 和 **Codex**。

它的目标很简单：

- 在任意项目里触发一次**多子代理并行**的代码质量审计
- 输出一份**结构化、证据导向**的审计报告
- 特别适合 AI 辅助开发项目的阶段性复评

和普通 code review prompt 不同，`/inspect` 默认会从架构、可维护性、可扩展性、可靠性、安全、测试与工程化，以及 AI 项目特有风险等多个维度同时检查当前仓库，并要求主线程补查关键证据，而不是只转述子代理结论。

这个命令本身刻意保持很薄。真正的方法论、审查语气、降级规则、视觉标记和输出结构都由 `agent-inspect` skill 承载。

它的设计前提是运行在主上下文里，由命令本身负责协调子代理，而不是先把自己包装成子任务再去分派别人。

如果你想把“AI 写了很多代码之后，现在这个项目到底还能不能继续维护和扩展”变成一个可重复执行的命令，这个项目就是为这个目的准备的。

### 平台支持

| 平台 | 状态 | 安装路径 |
|------|------|----------|
| OpenCode | ✅ supported | `~/.config/opencode/commands/` + `~/.config/opencode/skills/` |
| Claude Code | ✅ supported | `~/.claude/commands/` + `~/.claude/skills/` |
| Codex | ✅ supported | `~/.codex/prompts/` + `~/.codex/skills/` |

skill 本体是平台无关的，不依赖 OpenCode 专有 API。差异主要在安装路径。

### 快速开始

同一版本的 command wrapper 和 skill 需要一起安装。按你的平台选择对应命令。

**OpenCode**

```bash
mkdir -p ~/.config/opencode/commands ~/.config/opencode/skills
cp commands/inspect.md ~/.config/opencode/commands/inspect.md
cp -R skills/agent-inspect ~/.config/opencode/skills/agent-inspect
```

**Claude Code**

```bash
mkdir -p ~/.claude/commands ~/.claude/skills
cp commands/inspect.md ~/.claude/commands/inspect.md
cp -R skills/agent-inspect ~/.claude/skills/agent-inspect
```

**Codex**

```bash
mkdir -p ~/.codex/prompts ~/.codex/skills
cp commands/inspect.md ~/.codex/prompts/inspect.md
cp -R skills/agent-inspect ~/.codex/skills/agent-inspect
```

如果你只安装 command 或 prompt wrapper，配置是不完整的，`/inspect` 无法拿到完整的方法论。

安装完成后，在 CLI 中运行：

```text
/inspect
```

### 特点

1. 单命令入口：`/inspect`
2. command 只是薄包装，`agent-inspect` skill 才是审计引擎
3. 多子代理并行，不是单线程泛泛 review
4. 输出偏审计报告风格，优先 findings、证据、风险和结论
5. 输出语言跟随当前用户对话
6. 子代理不可用时显式降级
7. 默认要求宿主平台原生子代理，而不是外部协作框架
8. 在主上下文里协调执行
9. 审查语气直接、技术化、证据优先，批评代码不攻击人
10. 使用 ✅/❌/⚠️/🟢/🟡/🔴 提升可扫描性
11. 默认覆盖 prompts、agents、tools、models、configs、evals 等 AI 项目风险

### `/inspect` 会产出什么

运行：

```text
/inspect
```

会生成一份结构化审计报告，包含：

1. Executive Summary
2. Top Findings
3. Strengths
4. Residual Risks
5. Verdict

如果运行环境里部分子代理排队、失败或没有返回，`/inspect` 不会假装“完整并行审计已经完成”。它会进入降级模式，并明确说明哪些结论主要来自主线程补查，以及当前有哪些证据覆盖限制。

### 适用场景

1. AI 辅助开发后的阶段性复评
2. 大量改动后重新评估可维护性
3. 识别结构债、测试债、异常处理债
4. 审查 AI 项目中的 prompt / agent / tool / model / config 风险

### 设计原则

1. 先证据，后结论
2. 先 findings，后概览
3. 默认只读，不擅自改代码
4. 多维度覆盖，不只盯单一子系统
5. 子代理失败或排队时必须显式披露证据覆盖限制
6. 直接指出问题，不用礼貌废话稀释严重性
7. emoji 只用于增强扫描效率，不能替代路径、行号和证据

### 维护与贡献

- 贡献说明：`CONTRIBUTING.md`
- 版本记录：`CHANGELOG.md`
- 详细安装说明：`docs/installation.md`

### 文件结构

```text
.
├── README.md
├── LICENSE
├── CHANGELOG.md
├── CONTRIBUTING.md
├── commands/
│   └── inspect.md
├── skills/
│   └── agent-inspect/
│       └── SKILL.md
├── docs/
│   └── installation.md
└── examples/
    └── sample-output.md
```

## License

MIT
