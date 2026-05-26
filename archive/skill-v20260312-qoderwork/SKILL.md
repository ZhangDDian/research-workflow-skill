---
name: research-first
version: 1.0.0
description: Research workflow engine for technical investigation. Use when the task involves technology selection, solution comparison, feasibility evaluation, finding existing tools, or best practices research. Automatically determines research level (L1/L2/L3) and executes appropriate workflow. Note: QoderWork executes all research in single thread (no sub-agents).
description_zh: 调研工作流引擎。当任务涉及技术选型、方案对比、可行性评估、寻找现有工具或最佳实践时触发。自动判断调研级别（L1/L2/L3）并按工作流执行。注意：QoderWork 单线程执行（无子代理）。
---

# Research-First 调研工作流引擎

> **QoderWork 适配版说明**：本 Skill 针对 QoderWork 单线程环境调整。所有调研任务在当前对话中串行执行，通过阶段性汇报模拟"子代理"效果。

## When to Use

Use this skill when the user needs:
- Technology selection and comparison
- Solution evaluation and feasibility analysis
- Finding existing tools or libraries
- Best practices research
- Competitive analysis
- Industry trends investigation

## Research Levels

### L1 - Quick Check
- **Condition**: Confirm usage, check configuration, find files, ≤3 tool calls
- **Execution**: Direct answer, no report
- **Examples**: "How to use X", "What's the config for Y"

### L2 - Standard Research
- **Condition**: Technology selection, solution comparison, GitHub search, 5-15 tool calls
- **Execution**: Multi-phase execution with progress updates
- **Examples**: "Compare React vs Vue", "Find best PDF library"

### L3 - Deep Research
- **Condition**: Multi-solution comparison, cross-domain, network + code analysis, >15 tool calls
- **Execution**: Structured multi-phase with intermediate reports
- **Examples**: "Evaluate microservices architecture", "Research AI deployment options"

## Workflow Selection

| User Intent | Workflow |
|-------------|----------|
| "Is there a tool for..." / "Find libraries" | W1: Solution Discovery |
| "What are users complaining about" | W2: User Insights |
| "Can we implement..." / "Technical feasibility" | W3: Technical Evaluation |
| "How do others do it" / "Best practices" | W4: Best Practices |
| "What's new" / "Latest techniques" | W5: Knowledge Expansion |
| "What does academic research say" | W6: Literature Research |

## Execution Steps

### Step 1: Deduplication Check (L2/L3 only)

Before starting L2/L3 research:

1. Read project `docs/research/INDEX.md` if exists
2. Read global `~/.qoderwork/memory-bank/research/INDEX.md`
3. Grep keywords to find existing research
4. If found, ask user: **Reuse** / **Refresh** / **Ignore**

### Step 2: Level Determination

Based on task complexity:
- User says "check" / "confirm" → L1
- Involves "selection" / "comparison" / "GitHub" → L2
- Involves "unknown domain" / "multi-dimension" / "high-risk" → L3
- Default to L2 if uncertain

### Step 3: Scenario Identification

Select W1-W6 workflow based on user intent (see table above).

### Step 4: Requirements Clarification (if needed)

If uncertainties exist, use AskUserQuestion (max 2-3 questions).
Skip for L1 and when requirements are clear.

### Step 5: Execution Declaration

Before execution, show configuration:

```markdown
[MODE: RESEARCH] Research Initiated
- Level: L{1|2|3}
- Workflow: W{N} {Workflow Name}
- Validation Depth: {🔴|🟡|🟢}
- Timeliness: {🔥|📅|📚}
- Tools: {List of main tools to use}
- Execution: Single-thread with progress updates
```

### Step 6: Execute (QoderWork Single-Thread Mode)

#### L1: Quick Check
- Use Read/Grep/Glob → Direct answer
- No report, no PRD

#### L2: Multi-Phase Research

**Phase 1: Code Analysis**
- Read existing code if applicable
- Identify patterns and conventions
- Report: "Phase 1 complete: Found X relevant files..."

**Phase 2: External Research**
- WebSearch/WebFetch for documentation
- Bash(gh search repos) for GitHub search
- Report: "Phase 2 complete: Found Y potential solutions..."

**Phase 3: Analysis & Report**
- Analyze findings
- Write report to `~/.qoderwork/memory-bank/research/{topic}-{YYYYMMDD}.md`
- Return summary to user

#### L3: Structured Deep Research

For complex multi-domain research, use TodoWrite to track progress:

```javascript
TodoWrite({
  todos: [
    {content: "Domain A: Solution discovery", status: "in_progress"},
    {content: "Domain B: Technical evaluation", status: "pending"},
    {content: "Domain C: Best practices", status: "pending"},
    {content: "Synthesis & report", status: "pending"}
  ]
})
```

Execute each domain sequentially with intermediate updates.

### Step 7: Persist Results

Confirm:
- Report saved to correct path
- INDEX.md updated
- Storage level correct (project vs global)

### Step 8: Output + Handoff

#### L1 Output
```markdown
## Research Summary
### Findings
- [Key finding 1-3]
### Recommendations
[Brief recommendation]
```

#### L2/L3 Output
```markdown
## Research Summary

**Report Location**: `{file path}`

### Key Findings (≤10 points)
1. [Finding 1]
2. [Finding 2]
...

### Reuse Analysis
| Solution/Project | Strategy | Reason |
|------------------|----------|--------|
| {Name} | 🟢 Use Directly / 🟡 Adapt / 🔵 Reference / ⚪ Build Own | {Brief} |

### Recommended Solution
[Summary]

### PRD Framework Draft
**Recommended**: [From research]
**Core Features**: [From key findings]
**Technical Dependencies**: [From reuse analysis]
**Risk Items**: [From validation]
**To Clarify**: [From uncertainties]

### Next Steps
[Clear recommendation for next action]
```

## Tool Priority

1. **GitHub**: Use `gh search repos`, `gh repo view` commands
2. **Documentation**: WebFetch for official docs
3. **Search**: WebSearch (general), add site: for specific sources
4. **Academic**: WebSearch with "site:arxiv.org"
5. **Chinese**: WebSearch with 知乎/掘金/V2EX

## Quality Requirements

- Cross-validate key conclusions with multiple sources
- Distinguish "confirmed facts" from "speculation"
- Include source URL for each key claim
- Dig deeper than first-page results
- Update user on progress for L2/L3 (don't go silent)

## Research→Action Handoff

- After research, give clear "next step recommendations"
- If coding needed → suggest switching to development mode
- If decision needed → present "options + recommendation" format

## QoderWork vs Claude Code Differences

| Feature | Claude Code | QoderWork (This Skill) |
|---------|-------------|------------------------|
| Execution | Sub-agents (parallel) | Single thread (serial) |
| Progress | Background execution | Real-time updates |
| Report | Agent writes to file | I write directly |
| Parallel | Multiple agents | Sequential phases |
