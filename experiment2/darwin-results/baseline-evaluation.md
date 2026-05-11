# Darwin-Skill Baseline — meeting-notes

**Date:** 2026-05-09 | **Mode:** dry_run

## Structural (60pts)

### Dim 1: Frontmatter质量 (w=8) — 3/10 → 2.4
- `name`: "meeting-notes" ✓
- `description`: ">" — **空描述！**严重违规
- frontmatter 过度膨胀（48行），充斥非标准字段（models, mcp, capabilities, tags...）

### Dim 2: 工作流清晰度 (w=15) — 7/10 → 10.5
- How to Use 有 4 步编号 ✓
- 4 种模板各有结构 ✓
- 缺决策树：用户说"帮我整理会议纪要"，agent 不知道该选哪个模板

### Dim 3: 边界条件覆盖 (w=10) — 5/10 → 5.0
- Limitations 4 条 ✓
- 缺：无 owner 怎么处理？语言不匹配？输入矛盾？

### Dim 4: 检查点设计 (w=7) — 3/10 → 2.1
- 无用户确认步骤。"review before distributing"说得太轻

### Dim 5: 指令具体性 (w=15) — 7/10 → 10.5
- 模板具体 ✓，有 input/output 示例 ✓
- 部分指令模糊："May need clarification"

### Dim 6: 资源整合度 (w=5) — 4/10 → 2.0
- MCP create_docx 在 frontmatter 提及但流程中未使用
- 无其他引用

### Dim 7: 整体架构 (w=15) — 6/10 → 9.0
- 流程合理但有冗余：Output Format 和 Standard Template 重复

## Effectiveness (40pts)

### Dim 7 整体架构: 已计入结构分（实际 dim7=9.0）
### Dim 8 实测表现 (w=25) — 6/10 → 15.0

| ID | 场景 | 评估 |
|----|------|------|
| 1 | 标准笔记→结构化 | ✓ 模板匹配好 |
| 2 | 脏输入 | ✓ 技能声明能处理 |
| 3 | 仅行动项 | ✓ 定制选项支持 |
| 4 | 决策记录 | ✓ 决策识别指南 |
| 5 | 客户会议 | ✓ 专属模板 |
| 6 | 高管摘要 | △ 选项提到但无模板 |
| 7 | 头脑风暴 | △ 无专属指导 |
| 8 | 极简输入 | ✗ 可能过度加工 |

## 总分: **56.5/100**
