# CLAUDE.md — 调研工作流引擎

## 这是什么

Claude Code 调研系统的源码、设计文档和变更历史。活跃文件在 `~/.claude/` 下，这里是版本管理镜像。

## 目录结构

```
src/                              # 活跃文件镜像（改配置后同步到这里 + commit）
├── agents/researcher.md          #   Researcher 子代理定义（20 工具 + 优先级表 + 输出模板）
├── skills/research/SKILL.md      #   /research 命令主控流程（8 步）
├── skills/research/workflows/    #   W1-W6 场景工作流模板
└── contexts/research.md          #   L1/L2/L3 分级 + W1-W6 场景映射

docs/                             # 设计与历史
├── user-guide.md                 #   用户说明文档（同事可读版，解释"什么是调研工作流"）
├── self-audit-2026-04-19.md      #   Hermes Agent 调研复盘（结构性缺陷发现 + 修复）
└── origins/                      #   方法论奠基文档（2026-03-08 ~ 03-11，7 篇）
    ├── 调研方法-最佳实践综述-20260308.md
    ├── 调研方法-第一性原理-20260308.md
    ├── 调研方法-调研复用工作流-20260308.md
    ├── 调研方法-AI调研最佳实践-20260311.md
    ├── 调研方法-MCP工具对比-20260311.md
    ├── 调研方法-领域调研工作流-20260311.md
    └── 调研方法-领域调研策略-20260311.md

archive/                          # 旧版本快照（记录 skill 演化）
├── skill-v20260312-agents/       #   最早版本（~/.agents/，2026-03-12）
├── skill-v20260312-qoderwork/    #   Qoder 版本（2026-03-12）
└── skill-v20260320-ccskills/     #   Claude Code Skills 备份（2026-03-20）

DESIGN.md                         # 设计决策记录（三个缺陷 + 五个设计原则）
CHANGELOG.md                      # 变更历史
```

## 外部调研资产（不在本仓库，仅记录位置）

| 位置 | 内容 | 说明 |
|------|------|------|
| `~/.claude/memory-bank/research/` | 63 个旧调研报告 | Researcher v2 时代产物，v3 已改为项目级保存 |
| `~/AI工作流/docs/research/` | ~50 个调研报告 | 工作流产物，不是工作流定义 |
| `~/AI工作流/docs/调研报告/` | 分类报告（调研方法/上下文/多智能体/工具/Harness） | 其中"调研方法"子目录已迁入 docs/origins/ |

## 活跃文件路径映射

| 这里 | 活跃位置（Claude Code 读取） |
|------|---------------------------|
| `src/agents/researcher.md` | `~/.claude/agents/researcher.md` |
| `src/skills/research/SKILL.md` | `~/.claude/skills/research/SKILL.md` |
| `src/skills/research/workflows/*.md` | `~/.claude/skills/research/workflows/*.md` |
| `src/contexts/research.md` | `~/.claude/contexts/research.md` |

## 修改流程

1. 改活跃文件（`~/.claude/` 下的）
2. 复制到 `src/` 对应路径
3. `git add -A && git commit -m "描述"`
4. 如需更新 CHANGELOG.md

## 核心教训（快速回顾用）

1. **子代理能力 = frontmatter tools 列表**，不是 prompt 里写什么
2. **温规则不可靠** → 规则写在执行者身上，不写在指挥者身上
3. **Researcher 和 general-purpose 完全不同** → 研究任务用 Researcher（带专用 prompt + 工具），不用 general-purpose
4. **WebSearch 是兜底不是默认** → Tavily/Exa/Context7/Baidu 是首选
5. **速度信号 ≠ 跳过元步骤** → 并行也可以包含元步骤
