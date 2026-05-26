# 调研工作流自审：Hermes Agent 案例

**日期**：2026-04-19
**触发**：用户要求复原调研过程的上下文载入路径，分析为什么没用到搜索工具

---

## 任务背景

用户要求调研 Hermes Agent（NousResearch/hermes-agent, 84k stars）的：
1. 飞书部署方法 + "飞书官方插件"说法核查
2. vs OpenClaw 在飞书上的优劣
3. 能否用 ChatGPT 订阅调 GPT-5.4
4. 克隆源码 + codex 子代理分析代码结构
5. 多智能体路由（GPT 部分任务 + Claude 部分任务）

## 调研结果（OK）

产出 6 份报告（~2000 行），结论准确：
- GPT-5.4 真实（2026-03-05 发布）
- Hermes 飞书实现逐字移植 OpenClaw
- 官方有 `hermes claw migrate` 迁移工具
- 4 层路由架构（default + smart routing + auxiliary + delegate）
- 飞书卡片 Bug #6893/#8764 未清
- 4GB VPS 不建议双系统并行

## 流程审计发现

### 上下文载入路径（实际 vs 应该）

| 步骤 | 应该 | 实际 |
|------|------|------|
| 读 manifest.md | ✅ CLAUDE.md 明确要求 | ❌ 跳过 |
| 加载 contexts/research.md | ✅ manifest 会路由到这里 | ❌ 跳过 |
| 分级 L1/L2/L3 | ✅ research.md 定义 | ❌ 凭直觉判断（事后确认是 L3/W3） |
| 读 workflows/W3.md | ✅ SKILL.md Step 6 | ❌ 跳过 |
| 派 Researcher 子代理 | ✅ | ✅（派了 4 个 + 2 个其他类型） |
| 子代理用 Tavily/Exa/Context7 | ✅ research.md 优先级表 | ❌ 全部用 WebSearch |
| 显示 [MODE: RESEARCH] | ✅ SKILL.md Step 5 | ❌ 没触发 skill 所以没显示 |
| 调 /research 命令 | ✅（当时叫 research-first） | ❌ 没调 |

### 根因分析

**根因 1：温规则经济学**
- 加载 contexts/research.md 的成本：具体（1 次 Read + 2k token）
- 收益：抽象（"可能有更好的流程"）
- 用户说"尽可能并行"→ 速度信号 → 温规则出局

**根因 2：Researcher 子代理工具约束**（这是最关键的发现）
- `~/.claude/agents/researcher.md` v2 的 `tools:` 只有 6 个基础工具
- Researcher 子代理**物理上不能调** MCP 工具
- 就算我加载了 research.md 并告诉子代理"用 Tavily"
- 子代理也只能**看到指令但调不了** → 静默退化为 WebSearch
- 这个矛盾在改 researcher.md v3 之前是**不可修复的**

**根因 3：速度信号 ≠ 跳过元步骤**
- "尽可能并行"被解读为"跳过准备直接干活"
- 实际上可以并行：(子代理 A: 加载规则) + (子代理 B: 开始工作)

### 搜索工具使用统计

| 工具 | 可用 | 实际使用 | 原因 |
|------|------|---------|------|
| WebSearch | ✅ | ✅（子代理默认） | 唯一在 Researcher tools 列表里的搜索工具 |
| WebFetch | ✅ | ✅（1次，主线程验证 URL） | — |
| Tavily | ✅（MCP 已注册） | ❌ | Researcher frontmatter 没列 |
| Exa | ✅（MCP 已注册） | ❌ | 同上 |
| Context7 | ✅（MCP 已注册） | ❌ | 同上 |
| Baidu | ✅（MCP 已注册） | ❌ | 同上 |
| Brave | ✅（MCP 已注册） | ❌ | 同上 |
| Jina | ✅（MCP 已注册） | ❌ | 同上 |
| Scrapling | ✅（MCP 已注册） | ❌ | 不需要反爬 |
| gh CLI | ✅ | ✅（2次，主线程验证 repo） | — |

### 修复措施（已执行）

1. `researcher.md` v2→v3：tools 扩展 +14 MCP + 搜索优先级表 + 项目级保存路径
2. `research-first` → `research`：支持 `/research` 确定性命令
3. CLAUDE.md 引用更新（+7 字）
4. 本项目建立（代码 + 设计文档 + git 版本管理）

---

## 关键教训

1. **子代理的能力由 frontmatter 决定，不由 prompt 决定** —— prompt 里写"用 Tavily"但 frontmatter 没列 = 看得见吃不到
2. **温规则在压力下趋向零** —— 任何需要主动加载的规则都不可靠，要么内联到热层，要么把执行者改成自带
3. **速度信号 ≠ 跳过验证** —— 并行和元步骤是正交的
4. **Researcher ≠ general-purpose** —— 两者的工具集、模型、system prompt 完全不同，选错类型决定了整个调研的上限
5. **沉默的退化是最危险的** —— 子代理用 WebSearch 替代 Tavily 不报错不警告，调研看起来"能用"但质量无法达到设计水平
