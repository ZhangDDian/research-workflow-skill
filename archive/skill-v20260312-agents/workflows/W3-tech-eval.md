# W3: 技术评估（"能不能做 + 怎么复用"）

**目标**：评估技术可行性 + 制定复用策略
**典型耗时**：L2 = 30-60min, L3 = 2-4h

## 步骤

### SCOPE
定义技术约束：栈/性能/部署环境/团队能力。

### ORIENT（广度扫描技术方案空间）
- GitHub: `gh search repos`（最佳实践框架）
- 文档: Context7/GitMCP（关键库的 API 能力）
- 社区: "how to implement X" / "X architecture"

### SHORTLIST
筛选 2-4 个可行候选方案。

### SPIKE（可选）
针对最大技术风险点的快速验证：
- 写最小可行代码测试关键集成点
- 跑 benchmark 验证性能假设

### REUSE ANALYSIS
每个候选的复用评估：

| 组件 | 🟢 直接用 | 🟡 借鉴 | 🔵 参考思路 | ⚪ 自己写 | 原因 |
|------|----------|--------|-----------|---------|------|

### ADR DRAFT
技术决策记录草案：
- **Context**: 为什么需要决策
- **Options**: 考虑了什么
- **Decision**: 选了什么
- **Consequences**: 代价和收益

### PERSIST
按持久化规则写报告 + 更新 INDEX.md。
