# agent-inspect

`agent-inspect` 是一个给 OpenCode 用的全局 slash command：`/inspect`。

它的目标很简单：

- 在任意项目里触发一次**多子代理并行**的代码质量审计
- 输出一份**结构化、证据导向**的审计报告
- 特别适合 AI 辅助开发项目的阶段性复评

和普通 code review prompt 不同，`/inspect` 默认会从架构、可维护性、可扩展性、可靠性、安全、测试与工程化，以及 AI 项目特有风险等多个维度同时检查当前仓库，并要求主线程补查关键证据，而不是只转述子代理结论。

如果你想把“AI 写了很多代码之后，现在这个项目到底还能不能继续维护和扩展”变成一个可重复执行的 OpenCode 命令，这个项目就是为这个目的准备的。

## Quick Start

```bash
mkdir -p ~/.config/opencode/commands
cp commands/inspect.md ~/.config/opencode/commands/inspect.md
```

然后在 OpenCode 中输入：

```text
/inspect
```

## 特点

1. 单命令聚合型：只需要 `/inspect`
2. 多子代理并行：不是单线程泛泛而谈
3. 审计报告型输出：优先给 findings、证据、风险、结论
4. 输出语言跟随用户当前对话语言
5. 默认覆盖 AI 项目更关心的维度：
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

## 安装

把 `commands/inspect.md` 复制到：

```text
~/.config/opencode/commands/inspect.md
```

然后在 OpenCode 中输入：

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

## 文件结构

```text
.
├── README.md
├── LICENSE
├── commands/
│   └── inspect.md
├── docs/
│   └── installation.md
└── examples/
    └── sample-output.md
```

## License

MIT
