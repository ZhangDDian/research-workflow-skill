# Changelog

## v1.2.1 — 2026-05-26

### 公开分享准备

- 重写 README，面向朋友/外部使用者解释调研工作流引擎的定位、安装方式和触发示例
- 明确澄清当前路由是 L1/L2/L3 调研深度 + W1-W6 调研场景，不是 L1-L6
- 补充与早期 GitHub 仓库 `research-first-workflow`、`research-first-workflow-v2` 的关系说明
- 补充 Claude Code、Codex / 其他 Agent 的安装说明

---

## v1.1.0 — 2026-04-19

### 触发门修复 + 资产合并

#### 触发词补缺（根因修复）
- SKILL.md description：补了"信息查询（新版本/新功能/现状调研）"触发词
  - 根因：之前"调研 Claude Code 最新版本"这类信息查询任务不在触发词里，skill 自动识别不会命中

#### 命名与输出规范
- contexts/research.md 命名规则：补了中英文约定（通用产物英文，主题型中文）
- SKILL.md Step 8 输出模板：`{文件路径}` → `{绝对路径}`

#### 资产合并
- `docs/user-guide.md` ← 从 `~/AI工作流/docs/调研工作流/README.md` 迁入（用户说明文档）
- `docs/origins/`（7 篇）← 从 `~/AI工作流/docs/调研报告/调研方法/` 迁入（方法论奠基文档，2026-03-08 ~ 03-11）
- `archive/`（3 份旧 skill）← 从 `.agents/`、`.qoderwork/`、`Claude Code Skills/` 归档
- `src/` 同步为最新活跃版本（含触发词修复 + 命名规则修复）

#### 已知的外部调研资产（不迁入，仅记录）
- `~/.claude/memory-bank/research/`：63 个旧 Researcher 子代理报告（v2 时代保存路径）
- `~/AI工作流/docs/research/`：~50 个具体调研报告（工作流产物，不是工作流定义）
- `~/AI工作流/docs/调研报告/`：分类调研报告（上下文与记忆/多智能体/工具与平台/Harness）

---

## v1.0.0 — 2026-04-19

### 起因
Hermes Agent 调研任务的流程自审，发现三个结构性缺陷：温规则不可靠、Researcher 子代理无 MCP 工具、速度信号导致跳过元步骤。

### 变更

#### researcher.md v2 → v3
- **tools 列表**：6 → 20 个（新增 14 个 MCP 搜索工具：tavily×2, exa×2, context7×2, baidu×1, jina×1, brave×2, gitmcp×2, scrapling×1, hotnews×1）
- **搜索优先级表**：新增 9 行场景→首选/次选/兜底映射
- **保存路径**：`.claude/memory-bank/research/` → 项目级 `docs/research/{板块}/`（修复与 CLAUDE.md 的冲突）
- **输出模板**：新增 `[调研子代理 | tools: xxx×N | saved: path]` 模式标记

#### research-first skill → research skill
- **目录重命名**：`~/.claude/skills/research-first/` → `~/.claude/skills/research/`
- **name 字段**：`research-first` → `research`
- **description**：追加"用户可通过 `/research` 命令确定性激活"
- **内部路径**：所有 `research-first` 引用更新为 `research`

#### CLAUDE.md
- 第 14 行：`research-first skill` → `research skill（用户可用 /research 命令确定性激活）`（+7 字，零段落新增）

### 已验证的 MCP 工具（来自 ~/.claude.json 注册）
tavily-search, exa, context7, baidu-search, brave-search, jina, gitmcp, scrapling, hotnews

### 未变更
- `contexts/research.md`（分级规则不变）
- `workflows/W1-W6.md`（场景模板不变）
- SKILL.md 的 Step 1-8 主控流程（不变，只改路径引用）
