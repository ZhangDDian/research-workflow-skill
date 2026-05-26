# 设计决策记录

## 起源：2026-04-19 Hermes Agent 调研复盘

这个项目源于一次 Hermes Agent 调研任务的自审。调研结果本身是好的（6 份报告、523 行代码索引），但流程审计发现了三个结构性缺陷。本项目是修复这些缺陷的产物。

---

## 缺陷 1：温规则在执行压力下不可靠

### 问题

Claude Code 的规则分三层温度：
- **热**（自动加载）：CLAUDE.md、MEMORY.md → 每次会话都在
- **温**（被热规则指向，需主动 Read）：contexts/*.md、skills/* → 需要 1 次工具调用
- **冷**（存在但未被指向）：deferred MCP 工具 → 需要 ToolSearch

CLAUDE.md 说"收到新任务时，读取 manifest.md 确认任务类型"。这是一条温规则指针。在执行压力下（用户说"尽可能并行"），温规则被跳过 —— 因为加载成本具体（1 次 Read + 2k token），收益抽象（"可能有更好的工作流"）。

### 决策

**不通过 CLAUDE.md 修复**。规则写在 CLAUDE.md 但还是靠自觉。改为：
- `/research` 命令 = **确定性触发器**，用户可手动激活
- Skill description 里的关键词 = 自动触发，覆盖"用户没打命令"的场景
- 两条路径都直接加载 SKILL.md → 读 contexts/research.md → 执行，绕过 manifest.md

**温规则不可靠 → 解法不是"让它更热" → 而是"让入口更确定"。**

---

## 缺陷 2：Researcher 子代理物理上不能调 MCP 工具

### 问题

`~/.claude/agents/researcher.md` 的 `tools:` frontmatter 只列了 6 个基础工具。Researcher 子代理**物理上不能调用** Tavily、Exa、Context7、Baidu 等 MCP 搜索工具。

同时，`contexts/research.md` 和 `SKILL.md` Step 6 都规定了"工具优先级：Exa > Tavily > Brave > WebSearch"。但这条规则写在主 Claude 的 prompt 里 → 传给子代理的 prompt 里 → 子代理看到了但调不了 → **静默退化成 WebSearch**。

这个矛盾是沉默的：不报错、不警告、不记录。每次派 Researcher 都会重现。

### 决策

**修 researcher.md 的 frontmatter** —— 把 14 个 MCP 搜索工具加入 `tools:` 列表。这一手同时解决：
- 子代理能力边界（工具可用性）
- 子代理行为规范（在 body 里加搜索优先级表）

**规则要写在执行者的大脑里，不在指挥者的大脑里。** 主 Claude 忘了没关系 —— 子代理自己知道该用什么工具。

### 为什么不用 general-purpose 子代理

general-purpose 有 `tools: *`（所有工具），理论上能调 MCP。但：
- 没有专用 system prompt → 不知道搜索优先级、输出格式、保存路径
- 模型继承父进程（Opus 4.6 [1m]）→ 成本高，调研任务 Sonnet 够用
- 没有报告模板 → 输出格式不稳定

Researcher + 扩展工具 = **专用能力 + 广泛工具**，比 general-purpose 的"什么都能干但什么都不专"更好。

---

## 缺陷 3：速度信号和跳过元步骤的耦合

### 问题

用户说"尽可能并行" → 主 Claude 把这解读为"跳过准备工作，直接干活"。但实际上**并行和元步骤是正交的** —— 完全可以同时：
- 派一个子代理去加载规则
- 派另一个子代理去开始识别工作

### 决策

这是行为层面的问题，不是配置层面。记录在 feedback memory 中：

> "用户说'尽可能并行'意味着**在并行轨道上也并行跑元步骤**，不是跳过元步骤。"

---

## 设计原则总结

1. **规则住在执行者的大脑里** —— 子代理的 system prompt 里有工具优先级表，而不是靠主 Claude 记得给它说
2. **确定性入口 + 自动触发双轨** —— `/research` 命令保证激活，关键词匹配覆盖遗忘
3. **模式标记 = 可观测性** —— `[MODE: RESEARCH]` + `[tools: tavily×3...]` 同时服务 UX 和 debug
4. **热规则不膨胀** —— CLAUDE.md 只加了一个括号（7 个字），不塞新段落
5. **镜像 + git = 版本化** —— 活跃文件在 `~/.claude/`，镜像在这里，git 追踪变更历史
