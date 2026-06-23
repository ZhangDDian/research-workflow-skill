# AI 自主代理技术调研：claude-avatar 演进方向

> 调研日期：2026-05-27
> 调研范围：2025-2026 自主代理开源方案与技术趋势
> 目标：评估 claude-avatar 项目的下一步演进方向

---

## 一、调研过程（认知演化记录）

### 第 1 轮：框架全景扫描

**搜索方向**：2025-2026 自主代理开源框架全景

**关键发现**：
- 7 大主流框架：LangGraph（图状态机，企业采用最广）、CrewAI（角色化，简单）、AutoGen/AG2（微软，已进入维护模式）、OpenAI Agents SDK、Google ADK、Mastra（TypeScript，4 层记忆）、Dify（可视化低代码）
- Claude Agent SDK 2025 年 9 月发布，把 Claude Code 暴露为 Python/TypeScript 库
- Mastra 的 4 层记忆架构（消息历史/工作记忆/语义召回/RAG）是框架层面最完整的
- AutoGen 进入维护模式，已不推荐新项目使用

**假设更新**：
- 初始假设 H2（多代理是下一步）部分确认——但多数生产系统仍以单代理+多工具为主
- 新假设：Claude Agent SDK 可能是最自然的演进路径，因为 claude-avatar 已经在用 Claude Code CLI

**意外**：Anthropic 2026 年 2 月发布了"测量代理自主性"研究，追踪到 Claude Code 代理自主工作时长已达 45 分钟以上。

---

### 第 2 轮：Claude Agent SDK 深挖

**搜索方向**：基于第 1 轮发现，深入 Claude Agent SDK 架构

**关键发现**：
- SDK 核心抽象：`query()` 函数返回异步消息流，内置工具执行，无需手写 tool loop
- **Session 机制**：可恢复、可分叉、可持久化到外部存储。状态存为 JSONL 文件
- **Subagent 支持**：通过 `Agent` tool 定义专用子代理，带独立指令和工具集
- **Hooks 生命周期**：PreToolUse / PostToolUse / Stop / SessionStart / SessionEnd 等 7 个钩子
- **MCP 原生集成**：直接在 options 中声明 MCP server
- `claude -p` 和 SDK 功能等价，SDK 是库模式运行（in-process）
- 2026 年 6 月 15 日起 Agent SDK 独立计费

**假设更新**：
- H1（记忆瓶颈）重大更新——SDK 的 session 机制天然解决跨轮上下文问题
- H4（文件即接口是优势）增强——SDK 自己也用 JSONL 文件存状态

**认知转折**：claude-avatar 从 `claude -p` subprocess 迁移到 Agent SDK 可以获得 session 恢复、hooks 观测、subagent 编排等能力，同时保持同一套工具集。这不是重写，是升级。

---

### 第 3 轮：记忆 SOTA + 推理模式

**搜索方向**：基于前两轮，聚焦长期记忆和推理模式的最新方案

**关键发现**——记忆框架：

| 框架 | 架构 | 适用场景 | 独特能力 |
|------|------|----------|----------|
| **Letta (MemGPT)** | OS 式三层：核心(RAM)/归档(向量)/回忆(历史) | 长期运行代理首选 | 代理主动管理自己的记忆 |
| **Zep/Graphiti** | 时序知识图谱 | 需要追踪事实演变 | LongMemEval 63.8%（Mem0 仅 49%） |
| **Mem0** | 混合存储（向量+图+KV） | 通用，社区最大(48K stars) | 有 MCP server，可直接集成 |
| **LangMem** | 可插拔后端 | LangChain 生态内 | 独有程序性记忆——代理可改写自己的系统指令 |
| **Supermemory** | 统一记忆 API | 编码代理 | MCP 原生，专为 Claude Code 设计 |

**关键发现**——推理模式：

| 模式 | 机制 | 优势 | 适用 |
|------|------|------|------|
| **ReAct** | 思考→行动→观察循环 | 简单、自适应、可调试 | 探索性任务 |
| **Plan-and-Execute** | 强模型规划→弱模型执行 | LLM 调用少、可审核 | 结构化多步任务 |
| **ReWOO** | 2 次 LLM 调用，并行工具 | 5x token 效率 | 标准化批量操作 |
| **Reflexion** | 执行→评估→自我反思→重试 | GPT-4 pass rate 80%→91% | 有明确验证标准的任务 |

生产环境最常见的混合模式：**ReAct + Reflexion**。

**假设更新**：
- H1（记忆瓶颈）强确认——但 Letta 的"代理主动管理记忆"与 claude-avatar 的哲学最契合
- H5（自我反思优于重试计数）强确认——Reflexion 模式远优于简单计数
- 新发现：Mem0 和 Supermemory 都有 MCP server，可以**零代码改造**增强 claude-avatar 的记忆

**意外**：LangMem 的"程序性记忆"——代理改写自己的系统指令——正是 claude-avatar 人格文件自更新的方向。

---

### 第 4 轮：数字孪生生态 + 协议标准

**搜索方向**：类似 claude-avatar 的极简代理项目 + MCP/A2A 协议生态

**关键发现**——Second Me 数字孪生：
- 开源项目，3 层记忆架构：L0 原始数据 → L1 自然语言摘要 → L2 AI 原生（知识烘焙到模型权重）
- **矛盾保留机制**：不消除人格矛盾，而是计算"叙事张力"分数来丰富人格表现力
- 动态集体记忆图(DCM)，带生物启发的遗忘机制
- 大五人格稳定性跨千次交互保持一致

**关键发现**——协议生态：

| 层级 | 协议 | 功能 | 状态 |
|------|------|------|------|
| 工具接入 | **MCP** | 代理→工具 | 97M 下载量，事实标准 |
| 代理协调 | **A2A** | 代理→代理 | Google 主导，50+ 合作伙伴 |
| 商业交易 | **ACP** | 代理间商务 | IBM/Linux Foundation，早期 |
| 商业交易 | **UCP** | Google 商务 | Google 专属 |

A2A 的 **Agent Cards**（JSON 能力描述文件）可用于多分身动态发现和协作。

**假设更新**：
- 记忆进化方向不是走 Second Me 的极端（微调模型权重），而是通过 MCP 集成 Mem0/Supermemory
- 协议接口优于框架锁定——在 MCP 基础上扩展是正确策略

**意外**：Second Me 的矛盾保留机制与 claude-avatar `persona.md` 中"内在矛盾"章节异曲同工——都认为矛盾是人格复杂性的来源而非缺陷。

---

### 第 5 轮：长期运行代理工程 + 自我改进

**搜索方向**：Anthropic 官方 harness 设计模式 + 类似项目对比 + 自我改进机制

**关键发现**——Nightcrawler（claude-avatar 直接对标项目）：

| 维度 | claude-avatar | Nightcrawler |
|------|---------------|--------------|
| 语言 | Python ~600 LOC | TypeScript ~550 LOC |
| 进程管理 | 手动启动，fcntl 锁 | launchd 原生，崩溃恢复 |
| 任务追踪 | goals.md 手动勾选 | **不可变 tasks.json**（代理只能标记完成，不能修改描述） |
| 验证机制 | 无 | **git diff 交叉验证**（对比 handoff 声称 vs 实际提交） |
| 终止条件 | STOP 文件 + idle 退避 | **8 种终止条件**（含预算/时间/递减回报检测） |
| 记忆 | memory/ 全量注入 | STATE.json + HANDOFF.md + checkpoints |
| 人格 | persona.md 注入 | 无人格机制 |
| 创造力 | sparks.md 灵感池 | 无 |
| 沟通 | inbox/outbox 双向 | STOP 文件单向 |

**claude-avatar 独有优势**：人格注入、灵感积累、双向沟通协议、非阻塞失败机制（blocked_topics）、记忆自主管理（代理决定记什么）。

**Nightcrawler 值得借鉴**：不可变任务追踪、git diff 验证、递减回报检测、launchd 进程管理、结构化 episode 边界。

**关键发现**——Anthropic 官方 harness 模式：
1. **两代理模式**：initializer agent（环境准备+任务分解）→ coding agent（逐任务执行+测试+提交）
2. **Brain/Hands/Session 解耦**：推理循环 / 沙箱执行环境 / 持久事件日志 三者独立可替换
3. **核心原则**："代理本身失忆，但文件系统不会"——与 claude-avatar 的设计哲学完全一致
4. **测试棘轮**：禁止删除或修改测试来通过验证——防止代理改写成功标准
5. **完成条件前置**：开始工作前写好可测试的完成标准——"长期运行中最高杠杆的单一动作"
6. **生成器与评估器分离**：产出工作的实体不应是质量的唯一裁判

**关键发现**——Claude Code 架构：
- 98.4% 是确定性基础设施，1.6% 是 AI 决策逻辑——**验证了 claude-avatar "调度器尽可能笨"的哲学**
- 5 层上下文压缩管道（预算削减→裁剪→微压缩→上下文折叠→自动摘要）
- 记忆完全基于文件（无向量数据库）——用 LLM 扫描记忆文件标题，选最多 5 个相关文件
- 子代理侧链记录保护父上下文：只有摘要返回父级

**关键发现**——自我改进机制：
- METR 指标：代理自主完成任务的时间跨度每 7→4 个月翻倍（R²=0.98）
- SICA：代理改写自己的脚本，SWE-bench 从 17%→53%
- Reflexion + ExpeL：自然语言自我反思存入持久记忆，无需参数更新
- 核心限制：自我改进**仅在有客观可验证结果的领域可靠**（代码编译、数学证明、算法速度）
- Goodhart 风险：代理可能"游戏评估而非真正改进"

---

## 二、开源方案全景图

### 2.1 框架层

```
重量级 ──────────────────────────────────────── 轻量级
LangGraph    CrewAI    Google ADK    OpenAI SDK    claude -p
(图状态机)   (角色化)   (Google 生态)  (极简抽象)   (零框架)
                                                    ↑
                                           claude-avatar 在这里
```

claude-avatar 处于**最轻量级的位置**——零外部依赖，标准库即全部。这不是劣势，而是设计选择。Anthropic 官方的 harness 推荐也倾向于轻量级编排+重量级大脑。

### 2.2 记忆层

```
参数化(重) ──────────────────────────────────── 文件化(轻)
Second Me    Letta    Zep     Mem0    Supermemory    .md 文件
(微调权重)  (OS式)  (时序图)  (混合)   (MCP原生)      ↑
                                                claude-avatar
```

### 2.3 协议层

```
MCP (工具接入) → A2A (代理协调) → ACP/UCP (商业交易)
      ↑ claude-avatar 可以立即利用这一层
```

---

## 三、claude-avatar 演进建议

### 路线图：三个阶段

#### Phase 1：工程加固（1-2 周）

改动小、收益大、完全向后兼容的改进：

**1. 不可变任务追踪**（借鉴 Nightcrawler）
```python
# 启动时从 goals.md 生成 tasks.json，锁定任务描述
# 代理只能 flip passes: false → true，不能改描述
tasks = extract_tasks(GOALS_FILE)
save_immutable(TASKS_FILE, tasks)
```
防止代理改写成功标准假装完成。

**2. 结构化 handoff**
```python
# 每轮结束写 HANDOFF.md（替代 next_plan 字段）
# 下轮开始注入 HANDOFF.md + git diff
handoff = {
    "completed": "...",
    "in_progress": "...",
    "blockers": "...",
    "context_for_next": "..."
}
```
比单行 `next_plan` 字符串更丰富的跨轮上下文传递。

**3. 递减回报检测**
```python
# 最近 N 轮任务完成速度 < 阈值时自动报警
recent = state["history"][-5:]
tasks_done = sum(1 for a in recent if a["result"] == "success")
if tasks_done / len(recent) < 0.3:
    notify("递减回报", "最近 5 轮仅完成 {tasks_done} 个任务")
```

**4. 完成条件前置**
```markdown
<!-- goals.md 新格式 -->
- [ ] 重构 auth 模块
  - done-when: `pytest tests/auth/ -q` 全绿 + coverage > 80%
```
可测试的完成标准，防止主观判断"我做完了"。

**5. launchd 集成**（macOS）
```xml
<!-- com.claude-avatar.agent.plist -->
<dict>
    <key>KeepAlive</key><true/>
    <key>ThrottleInterval</key><integer>30</integer>
    <key>TimeOut</key><integer>43200</integer>
</dict>
```
崩溃恢复、开机自启、系统级进程管理。

#### Phase 2：能力升级（2-4 周）

**6. 迁移到 Claude Agent SDK**

从 subprocess 调用 `claude -p` 迁移到 SDK 库调用：

```python
# Before: subprocess
proc = subprocess.Popen(["claude", "-p", prompt, ...])
stdout, stderr = proc.communicate(timeout=CYCLE_TIMEOUT)

# After: Agent SDK
from claude_agent_sdk import query, ClaudeAgentOptions

async for message in query(
    prompt=prompt,
    options=ClaudeAgentOptions(
        allowed_tools=allowed_tools,
        hooks={"PostToolUse": [audit_logger]},
    ),
):
    handle_message(message)
```

获得的能力：
- Session 恢复/分叉（不再每轮完全失忆）
- Hooks 观测（审计日志、安全拦截）
- Streaming（实时看到代理在做什么）
- Subagent（内置子代理支持）
- 结构化输出（不再需要 regex 解析 JSON）

**7. MCP 记忆增强**

通过 MCP 集成 Mem0 或 Supermemory，零代码修改代理逻辑：

```python
options = ClaudeAgentOptions(
    mcp_servers={
        "memory": {
            "command": "npx",
            "args": ["@mem0/mcp-server"]
        }
    }
)
```

代理自动获得结构化记忆存取能力，保留 `memory/` 目录作为人类可读的备份。

**8. Reflexion 自我反思**

替代简单的 retry_count：

```python
# 失败时生成自然语言反思，存入记忆
if result == "failed":
    reflection_prompt = f"""
    任务：{did}
    失败详情：{detail}
    之前的反思：{load_reflections()}

    分析失败原因，提出不同的方案。
    """
    reflection = run_reflection(reflection_prompt)
    append_memory("reflections.md", reflection)
```

**9. 生成/评估分离**

每 N 轮引入独立评估循环：

```python
if state["cycle"] % 5 == 0:
    evaluator_prompt = f"""
    审查最近 5 轮的工作：
    {recent_history}
    {recent_changes}  # git diff

    评估：
    1. 声称的进展与实际代码变更是否一致？
    2. 有没有被跳过或降级的任务？
    3. 记忆中是否有过时或矛盾的条目？
    """
```

#### Phase 3：生态扩展（4-8 周）

**10. 多分身协作（A2A Agent Cards）**

```json
// agent-card.json
{
    "name": "research-avatar",
    "capabilities": ["web-search", "paper-analysis", "experiment-design"],
    "inputs": {"topic": "string", "depth": "enum:survey|deep-dive"},
    "outputs": {"report": "markdown", "findings": "json"}
}
```

通过 A2A 协议实现多个 claude-avatar 实例的动态发现和任务委派。

**11. 记忆衰减与遗忘**

借鉴 Second Me 的 DCM 图和 Letta 的分层机制：

```python
# 记忆条目带权重，根据使用频率和时间衰减
for entry in memory_entries:
    entry.weight *= decay_factor(entry.last_accessed)
    if entry.weight < FORGET_THRESHOLD:
        archive_memory(entry)  # 移入 memory/archive/
```

替代现有的每 10 轮手动整理机制。

**12. 人格演化追踪**

```python
# 每 50 轮自动对比 persona.md 的变化
# 保留人格版本历史
# 借鉴 LangMem 的程序性记忆：代理基于反馈改写自己的运行规则
persona_versions/
├── persona-v1.md      # 初始
├── persona-v2.md      # 第 50 轮自动演化
├── persona-v3.md      # 第 100 轮
└── evolution-log.md   # 每次变更的原因记录
```

---

## 四、优先级矩阵

| 改进项 | 难度 | 收益 | 优先级 |
|--------|------|------|--------|
| 不可变任务追踪 | 低 | 高 | P0 |
| 结构化 handoff | 低 | 高 | P0 |
| 递减回报检测 | 低 | 中 | P0 |
| 完成条件前置 | 低 | 高 | P0 |
| launchd 集成 | 低 | 中 | P1 |
| Agent SDK 迁移 | 中 | 极高 | P1 |
| MCP 记忆增强 | 中 | 高 | P1 |
| Reflexion 自我反思 | 中 | 高 | P1 |
| 生成/评估分离 | 中 | 高 | P2 |
| 多分身协作 | 高 | 中 | P2 |
| 记忆衰减 | 中 | 中 | P2 |
| 人格演化追踪 | 中 | 低 | P3 |

---

## 五、意外发现

### 5.1 "代理本身失忆，但文件系统不会"

Anthropic 官方 harness 设计的核心原则与 claude-avatar 的设计哲学**完全一致**。claude-avatar 的"文件即接口"不是简陋，而是与 Anthropic 推荐的最佳实践不谋而合。连 Claude Code 自身的记忆系统也是文件化的（无向量数据库），用 LLM 扫描 .md 文件标题选取相关记忆。

### 5.2 98.4/1.6 法则

Claude Code 内部 98.4% 的代码是确定性基础设施，仅 1.6% 是 AI 决策逻辑。这验证了 claude-avatar "调度器尽可能笨"的设计直觉——**所有智能都应该在模型里，harness 只做可靠的管道**。

### 5.3 矛盾是资产不是缺陷

Second Me 的研究发现，保留人格矛盾（而非消除）可以产生更丰富的代理行为。计算"叙事张力"分数让矛盾成为人格深度的来源。claude-avatar `persona.md` 中的"内在矛盾"章节是这个方向的先驱。

### 5.4 Nightcrawler 的不可变任务机制

防止代理改写成功标准来宣称完成——这是自主代理安全中被严重低估的问题。claude-avatar 的 `goals.md` 允许代理自由勾选，但也允许代理改写任务描述，存在隐性风险。

### 5.5 METR 加速曲线

代理自主完成任务的时间跨度倍增周期从 7 个月缩短到 4 个月（R²=0.98）。这意味着 claude-avatar 这类项目的能力天花板在快速抬升——今天的架构瓶颈可能在 6-12 个月内被模型能力本身突破。

### 5.6 MCP 已成事实标准

97M 下载量，所有主要厂商采用。claude-avatar 应该把 MCP 作为唯一的工具扩展方式——不要自己写 tool wrapper，而是接入 MCP server。Mem0 和 Supermemory 都已有 MCP server，意味着记忆增强可以零代码完成。

---

## 六、核心判断

claude-avatar 的设计直觉是正确的：

1. **"调度器尽可能笨"** = Anthropic 的 98.4/1.6 法则
2. **"文件即接口"** = "代理本身失忆，但文件系统不会"
3. **"记忆是涌现的"** = Letta 的"代理主动管理记忆"
4. **"失败是信号不是终点"** = 非阻塞失败 + Reflexion 的自我反思方向

差距不在架构哲学，在工程成熟度：

- 缺少验证机制（git diff 交叉验证、测试棘轮、不可变任务）
- 缺少自适应终止（递减回报检测、预算控制）
- 缺少结构化 handoff（HANDOFF.md > 单行 next_plan）
- 未接入 Agent SDK（失去 session/hooks/subagent 能力）
- 未利用 MCP 生态（记忆、搜索等能力的即插即用）

最高 ROI 的演进路径：**先做 Phase 1 加固（P0 项，1-2 周），再迁移 Agent SDK（P1 项，2-4 周）**。Phase 1 的四项改动加起来不到 200 行代码，但能显著提升代理的可靠性和可审计性。

---

## 七、参考来源

- [Firecrawl: Best Open Source Agent Frameworks 2026](https://www.firecrawl.dev/blog/best-open-source-agent-frameworks)
- [Claude Agent SDK Documentation](https://code.claude.com/docs/en/agent-sdk/overview)
- [Atlan: Best AI Agent Memory Frameworks 2026](https://atlan.com/know/best-ai-agent-memory-frameworks-2026/)
- [The AI Engineer: 4 Single Agent Patterns](https://theaiengineer.substack.com/p/the-4-single-agent-patterns)
- [Nightcrawler: Autonomous Overnight Agent Loop](https://github.com/thebasedcapital/nightcrawler)
- [Second Me: AI Digital Twin](https://www.emergentmind.com/topics/second-me)
- [AI Agent Protocol Ecosystem Map 2026](https://www.digitalapplied.com/blog/ai-agent-protocol-ecosystem-map-2026-mcp-a2a-acp-ucp)
- [O-mega: Self-Improving AI Agents Guide 2026](https://o-mega.ai/articles/self-improving-ai-agents-the-2026-guide)
- [VILA-Lab: Dive into Claude Code](https://github.com/VILA-Lab/Dive-into-Claude-Code)
- [Addy Osmani: Long-running Agents](https://addyo.substack.com/p/long-running-agents)
- [Anthropic: Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Measuring Agent Autonomy](https://www.anthropic.com/news/measuring-agent-autonomy)

---

[执行模式: 递归]
[搜索轮次: 5]
