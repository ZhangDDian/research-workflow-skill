# 调研工作流引擎

一个面向 Claude Code / Codex / Agent Skills 的结构化调研工作流。

它把“上网查一下”拆成可复用的路由系统：先判断调研深度 L1/L2/L3，再选择调研场景 W1-W6，最后按工具优先级、来源质量和持久化规则执行。目标不是让 Agent 更会搜索，而是让 Agent 在设计、选型和实现前先系统确认：有没有现成方案、别人怎么做、风险在哪里、证据从哪里来。

> 说明：当前设计是 **L1/L2/L3 三档调研深度 + W1-W6 六类调研场景**，不是 L1-L6。

## 解决什么问题

日常用 AI Agent 做开发或研究时，常见问题是：

- 太快进入实现，没有先确认有没有成熟方案。
- 技术选型只看一两个搜索结果，缺少对比和反面证据。
- 调研结果散落在对话里，后续无法复用。
- 复杂调研没有分工，主线程上下文被大量搜索结果淹没。
- 不同类型的问题都用同一种搜索方式，效率和质量不稳定。

这个工作流的处理方式：

- 简单问题快速查，直接回答。
- 标准调研交给一个 Researcher 子代理。
- 深度调研拆成多个方向，并行子代理执行。
- 关键声明附来源，区分事实和推测。
- 调研报告保存到项目或全局 research memory，并维护索引。

## 快速使用

确定性入口：

```text
/research 调研 XXX 的 YYY
```

自然语言触发：

```text
帮我调研一下 XXX
评估一下要不要用 YYY
找找有没有现成开源方案
```

触发后，Agent 会先声明调研配置：

```markdown
[MODE: RESEARCH] 调研启动
- 级别: L2
- 工作流: W3 技术评估
- 验证深度: 🟡
- 时效性: 📅
- 工具: gh CLI, Tavily, Exa, Context7
- 执行方式: 单个子代理
```

## 调研分级

| 级别 | 适用场景 | 执行方式 |
|---|---|---|
| L1 快查 | 确认用法、查配置、找文件、少量信息查询 | 主线程直接完成 |
| L2 标准调研 | 技术选型、方案对比、GitHub 搜索、最佳实践调研 | 单个 Researcher 子代理 |
| L3 深度调研 | 多方案对比、跨领域问题、高风险决策、需要网络和代码分析 | 多个 Researcher 子代理并行 |

默认判断规则：

- “查一下”“确认一下”通常是 L1。
- “选型”“比较”“找 GitHub 项目”通常是 L2。
- “完全未知领域”“多维度对比”“高风险决策”通常是 L3。
- 不确定时默认 L2，执行中发现范围扩大可以升级到 L3。

## W1-W6 工作流

| 工作流 | 名称 | 适合的问题 | 模板 |
|---|---|---|---|
| W1 | 方案发现 | 有没有现成工具、开源项目、竞品或替代方案 | [W1-discovery.md](src/skills/research/workflows/W1-discovery.md) |
| W2 | 用户洞察 | 用户在抱怨什么、真实痛点是什么 | [W2-user-insight.md](src/skills/research/workflows/W2-user-insight.md) |
| W3 | 技术评估 | 能不能实现、怎么复用、有哪些技术路线 | [W3-tech-eval.md](src/skills/research/workflows/W3-tech-eval.md) |
| W4 | 最佳实践 | 别人怎么做、行业内有哪些成熟做法 | [W4-best-practice.md](src/skills/research/workflows/W4-best-practice.md) |
| W5 | 知识扩展 | 最新技术、最新技巧、近期动态 | [W5-knowledge.md](src/skills/research/workflows/W5-knowledge.md) |
| W6 | 文献研究 | 学术上怎么看、理论依据和证据质量如何 | [W6-literature.md](src/skills/research/workflows/W6-literature.md) |

复杂问题可以组合多个工作流。例如：

- “这个功能该不该自研，技术上可不可行”通常组合 W1 方案发现 + W3 技术评估。
- “评估一个产品方向有没有机会”通常组合 W1 方案发现 + W2 用户洞察 + W4 最佳实践。
- “系统研究某个学术问题”通常使用 W6 文献研究，并按 L3 深度执行。

## 仓库结构

```text
.
├── src/
│   ├── agents/
│   │   └── researcher.md
│   ├── contexts/
│   │   └── research.md
│   └── skills/
│       └── research/
│           ├── SKILL.md
│           └── workflows/
│               ├── W1-discovery.md
│               ├── W2-user-insight.md
│               ├── W3-tech-eval.md
│               ├── W4-best-practice.md
│               ├── W5-knowledge.md
│               └── W6-literature.md
├── docs/
│   ├── user-guide.md
│   ├── self-audit-2026-04-19.md
│   └── origins/
├── archive/
├── DESIGN.md
├── CHANGELOG.md
└── README.md
```

核心文件：

| 文件 | 作用 |
|---|---|
| `src/skills/research/SKILL.md` | 主控流程：去重、分级、场景识别、派发子代理、输出 |
| `src/skills/research/workflows/*.md` | W1-W6 六种调研场景模板 |
| `src/contexts/research.md` | L1/L2/L3 分级、W1-W6 映射、工具路由、质量要求 |
| `src/agents/researcher.md` | Researcher 子代理能力边界、工具优先级和输出规范 |

## 安装

### Claude Code

复制 Skill：

```bash
mkdir -p ~/.claude/skills
cp -R src/skills/research ~/.claude/skills/research
```

复制调研上下文：

```bash
mkdir -p ~/.claude/contexts
cp src/contexts/research.md ~/.claude/contexts/research.md
```

复制 Researcher 子代理：

```bash
mkdir -p ~/.claude/agents
cp src/agents/researcher.md ~/.claude/agents/researcher.md
```

然后在你的 `~/.claude/CLAUDE.md` 或项目 `CLAUDE.md` 里加入触发说明，例如：

```markdown
当任务涉及技术调研、方案对比、开源项目评估、竞品分析、行业趋势、工具选型、可行性分析、学术文献或最佳实践调研时，使用 research skill。
```

### Codex / 其他 Agent

如果你的 Agent 使用 `~/.agents/skills`：

```bash
mkdir -p ~/.agents/skills
cp -R src/skills/research ~/.agents/skills/research
```

如果你的运行时不支持 `~/.claude/contexts/`，可以把 `src/contexts/research.md` 里的规则合入自己的 `AGENTS.md`、`CLAUDE.md` 或全局配置。关键是保留这些内容：

- L1/L2/L3 调研分级。
- W1-W6 工作流选择。
- 调研前检查已有索引。
- L2/L3 使用子代理。
- 关键结论附来源。
- 调研结果持久化。

## 推荐依赖

这个仓库本身是一套工作流规则，不强绑定某个搜索服务。但为了效果更好，推荐准备以下能力：

- Agent Skills 支持。
- 子代理能力，用于 L2/L3 调研。
- `gh` CLI，用于 GitHub 项目搜索和仓库元数据查询。
- 搜索工具，例如 Tavily、Exa、Brave Search、Baidu MCP。
- 官方文档工具，例如 Context7、GitMCP、WebFetch。
- 浏览器或网页抓取工具，例如 Playwright、Scrapling、camoufox。
- 学术搜索能力，例如 arXiv、Semantic Scholar、Connected Papers。

## 典型触发示例

```text
帮我查一下有没有现成的开源方案可以做 AI 会议纪要。
```

通常触发：L2 + W1 方案发现。

```text
评估一下我们要不要用 LangGraph 做这个 Agent 工作流。
```

通常触发：L2 + W3 技术评估。

```text
调研一下别人是怎么做用户反馈收集和归因分析的。
```

通常触发：L2 + W4 最佳实践。

```text
我想了解最近半年 RAG 评估有什么新方法。
```

通常触发：L2 + W5 知识扩展。

```text
系统研究一下 AI pair programming 对开发效率的学术证据。
```

通常触发：L3 + W6 文献研究。

## 输出形态

L1 快查通常直接输出简短结论：

```markdown
## 调研摘要

### 发现
- ...

### 建议
...
```

L2/L3 通常输出报告路径和精炼摘要：

```markdown
## 调研摘要

**报告位置**: `docs/research/YYYY-MM-DD-topic.md`

### 关键发现
1. ...

### 复用分析
| 方案/项目 | 策略 | 理由 |
|---|---|---|

### 推荐方案
...

### 待澄清
...
```

W1 方案发现和 W3 技术评估还可以补一版 PRD 框架草案，方便从调研自然衔接到研发。

## 持久化约定

推荐规则：

- 项目相关调研：保存到 `docs/research/`。
- 跨项目通用调研：保存到 `~/.claude/memory-bank/research/`。
- 临时数据：保存到 `/tmp/`。
- 报告命名：`YYYY-MM-DD-{主题}.md`。
- 索引文件：`INDEX.md`。

调研前优先查索引：

```text
docs/research/INDEX.md
~/.claude/memory-bank/research/INDEX.md
~/.codex/memory-bank/research/INDEX.md
```

如果找到已有相关报告，推荐让用户选择：

- 复用：直接引用旧报告。
- 刷新：基于旧报告更新。
- 忽略：重新调研。

## 与旧仓库的关系

这个版本是当前的源码形态。早期公开仓库保留了旧思路，但不是当前实现：

- [`research-first-workflow`](https://github.com/wangjialiang678/research-first-workflow)：2026-01 的 v1，主打“调研优先”。
- [`research-first-workflow-v2`](https://github.com/wangjialiang678/research-first-workflow-v2)：2026-01 的 v2，加入 Sub Agent 和 Memory Bank 思路。
- 当前仓库：后续演进出的调研工作流引擎，包含 L1/L2/L3 分级、W1-W6 场景模板、Researcher 子代理、工具路由和质量要求。

## 设计原则

- 先调研，再设计，再实现。
- 不重复造轮子。
- 关键声明必须能追溯来源。
- 高风险决策需要反面证据。
- 调研阶段不直接进入编码实现。
- 调研输出要能被下次复用。

## License

目前未声明 License。公开分享前可按需要补充 MIT、Apache-2.0 或其他许可证。
