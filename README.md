# agent-inspect

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

## 中文

`agent-inspect` 是一个全局 slash command（`/inspect`），支持 **OpenCode**、**Claude Code** 和 **Codex**。

它的目标很简单：

- 在任意项目里触发一次**多子代理并行**的代码质量审计
- 输出一份**结构化、证据导向**的审计报告
- 特别适合 AI 辅助开发项目的阶段性复评

和普通 code review prompt 不同，`/inspect` 默认会从架构、可维护性、可扩展性、可靠性、安全、测试与工程化，以及 AI 项目特有风险等多个维度同时检查当前仓库，并要求主线程补查关键证据，而不是只转述子代理结论。

它的设计前提是运行在主上下文里，由命令本身负责协调子代理，而不是先把自己包装成子任务再去分派别人。

如果你想把“AI 写了很多代码之后，现在这个项目到底还能不能继续维护和扩展”变成一个可重复执行的 OpenCode 命令，这个项目就是为这个目的准备的。

## Platform Support

| Platform | Status | Install paths |
|----------|--------|---------------|
| OpenCode | ✅ supported | `~/.config/opencode/commands/` + `~/.config/opencode/skills/` |
| Claude Code | ✅ supported | `~/.claude/commands/` + `~/.claude/skills/` |
| Codex | ✅ supported | `~/.codex/prompts/` + `~/.codex/skills/` |

The skill body is platform-neutral: it does not depend on any OpenCode-specific API. Only the install paths differ.

## Quick Start

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

## Features

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

## 特点

1. 单命令聚合型：只需要 `/inspect`
2. command 只是入口，`agent-inspect` skill 承载审计方法论
3. 多子代理并行：不是单线程泛泛而谈
4. 审计报告型输出：优先给 findings、证据、风险、结论
5. 输出语言跟随用户当前对话语言
6. 子代理不可用时显式降级，不伪装成完整并行审计
7. 默认使用宿主平台原生子代理，而不是外部协作框架
8. 运行在主上下文中，由命令自身协调子代理
9. 默认使用 blunt but technical 的审查语气：直接、证据优先、批评代码不攻击人
10. 使用 ✅/❌/⚠️/🟢/🟡/🔴 等轻量视觉标记提升可读性
11. 默认覆盖 AI 项目更关心的维度：
   - 可维护性
   - 可扩展性
   - 可靠性
   - 安全
   - 测试与工程化
   - 架构边界
   - AI 特有风险

## 命令效果

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

如果运行环境里部分子代理排队、失败或没有返回，`/inspect` 也不会假装“已经完成完整并行审计”。它会进入降级模式，并在报告里明确说明当前有哪些结论主要来自主线程补查与抽样证据。

## 安装

选择对应的平台复制 command wrapper 和 skill：

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

任一平台装完后，在对应 CLI 中输入：

```text
/inspect
```

详细安装说明见：`docs/installation.md`

## 维护与贡献

- 贡献说明：`CONTRIBUTING.md`
- 版本记录：`CHANGELOG.md`

## 适用场景

1. AI 辅助开发后做阶段性代码复评
2. 大量改动后评估是否仍然可维护
3. 识别结构债、测试债、异常处理债
4. 审查 AI 项目中的 prompt / agent / tool / model / config 风险

## 设计原则

1. 先证据，后结论
2. 先 findings，后概览
3. 只读审计，不擅自改代码
4. 多维度覆盖，不只盯一个子系统
5. 子代理失败或排队时，必须显式披露证据覆盖限制
6. 直接指出问题，不用礼貌废话稀释严重性
7. emoji 只用于增强扫描效率，不能替代证据、路径和行号

## 文件结构

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
