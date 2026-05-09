# Darwin-Skill Baseline Evaluation

**Skill:** json-and-csv-data-transformation
**Date:** 2026-05-09
**Evaluator:** Main agent (structural) + dry-run (effectiveness)
**Methodology:** Darwin-skill 8-dimension rubric (60 structural + 40 effectiveness)

---

## Structural Dimensions (60pts max)

### Dim 1: Frontmatter质量 (weight 8)

**Score: 7/10 → 5.6/8**

| Criteria | Status |
|----------|--------|
| name规范、描述性强 | PASS — `json-and-csv-data-transformation` 准确描述功能 |
| description含做什么+何时用+触发词 | PARTIAL — 有4个英文触发场景，但缺少中文触发词 |
| ≤1024字符 | PASS — description 约400字符 |

### Dim 2: 工作流清晰度 (weight 15)

**Score: 6/10 → 9.0/15**

| Criteria | Status |
|----------|--------|
| 步骤明确可执行 | PARTIAL — 6个功能各自有bash+Node.js代码，但缺少统一的线性工作流 |
| 有序号 | FAIL — 功能模块未编号，Agent prompt部分有编号但仅限该段 |
| 每步有明确输入/输出 | PARTIAL — 代码示例有输入输出，但无"用户说X → 执行步骤1,2,3"的映射 |

问题：用户说"帮我把JSON转CSV"时，agent 需要自己在6个功能模块中定位 `json_to_csv`，缺少顶层路由指引。

### Dim 3: 边界条件覆盖 (weight 10)

**Score: 5/10 → 5.0/10**

| Criteria | Status |
|----------|--------|
| 处理异常情况 | PARTIAL — Troubleshooting 覆盖了6种错误，但都是"出问题后诊断" |
| 有fallback路径 | FAIL — 无"如果jq不可用则用Node.js"之类的自动降级 |
| 错误恢复 | FAIL — 无"转换失败后如何恢复"的指引 |

缺失的关键边界：
- 输入文件不存在时的行为
- 输入不是合法JSON/CSV时的行为
- jq/csvkit未安装时的降级方案
- 空数组/空文件的处理

### Dim 4: 检查点设计 (weight 7)

**Score: 3/10 → 2.1/7**

| Criteria | Status |
|----------|--------|
| 关键决策前有用户确认 | FAIL — Agent prompt 中无任何确认步骤 |
| 防止自主失控 | FAIL — 无"覆盖前询问"或"大文件警告"机制 |

Agent prompt 直接告诉 agent "Always preserve data integrity" 但没有在覆盖文件前让用户确认。

### Dim 5: 指令具体性 (weight 15)

**Score: 7/10 → 10.5/15**

| Criteria | Status |
|----------|--------|
| 不模糊 | MOSTLY — bash命令具体，Node.js代码完整可运行 |
| 有具体参数/格式/示例 | PASS — 每个函数都有示例输入输出 |
| 可直接执行 | MOSTLY — 代码可直接复制运行 |

不足：Agent prompt 中的 "Handle edge cases"、"Validate output format" 等指令过于笼统，没有给出具体的验证步骤。

### Dim 6: 资源整合度 (weight 5)

**Score: 5/10 → 2.5/5**

| Criteria | Status |
|----------|--------|
| references/scripts/assets引用正确 | PARTIAL — 引用了jq和csvkit并给出安装方法 |
| 路径可达 | FAIL — "../database-query-and-export/SKILL.md" 等相对路径引用大概率不存在 |

---

## Effectiveness Dimensions (40pts max)

### Dim 7: 整体架构 (weight 15)

**Score: 6/10 → 9.0/15**

优点：
- 覆盖了JSON/CSV转换的主要场景
- 每个功能同时提供bash和Node.js方案
- 有Agent prompt告诉agent如何执行

问题：
- "Agent prompt" 段与6个功能模块内容重复（约200行冗余）
- 缺少TL;DR或快速决策树
- When to use → Required tools → Functions → Best practices → Agent prompt → Troubleshooting 的线性结构合理但不够紧凑

### Dim 8: 实测表现 (weight 25) — DRY_RUN

**Score: 6/10 → 15.0/25**

基于15个测试prompt的干跑推演：

| 测试场景 | 预测结果 | 说明 |
|----------|---------|------|
| JSON→CSV 基本转换 (test 1) | PASS | 有明确的jq命令可执行 |
| JSON→CSV 字段选择 (test 2) | PASS | jq字段提取语法清晰 |
| CSV→JSON 基本转换 (test 3) | PASS | csvjson直接可用 |
| CSV→JSON 类型保持 (test 4) | WARN | Node.js方案可保持类型，但bash方案(csvjson)默认全是字符串，skill未强调这点 |
| 过滤+提取嵌套字段 (test 5) | PASS | jq select + 嵌套提取有示例 |
| 多条件过滤 (test 6) | PASS | jq多条件语法有示例 |
| 嵌套展平(null处理) (test 7) | WARN | flattenJSON函数未处理null对象，Eve的null address会导致crash |
| 深层嵌套展平 (test 8) | WARN | 只提供了单用户展平，对复杂嵌套config可能需要多次递归 |
| CSV过滤+排序 (test 9) | PASS | csvgrep + csvsort明确可用 |
| CSV评分过滤 (test 10) | WARN | csvgrep正则 `^[4-9]` 不匹配4.5，需要更复杂的模式 |
| 分组聚合(region) (test 11) | PASS | jq group_by有明确示例 |
| 分组聚合(product) (test 12) | PASS | 同上 |
| 空数组 (test 13) | WARN | Node.js jsonToCSV返回''(空字符串)，但skill未明确说明空输入行为 |
| 特殊字符CSV (test 14) | PASS | Node.js CSV parser处理了引号转义 |
| null字段CSV (test 15) | WARN | Node.js escape()将null转为''，但bash jq方案未处理null |

预测通过率：9/15 = 60%

---

## 总分汇总

```
┌──────────────────────┬────────┬───────┬──────────────┐
│ Dimension            │ Weight │ Score │ 加权得分      │
├──────────────────────┼────────┼───────┼──────────────┤
│ 1. Frontmatter质量    │ 8      │ 7     │ 5.6          │
│ 2. 工作流清晰度        │ 15     │ 6     │ 9.0          │
│ 3. 边界条件覆盖        │ 10     │ 5     │ 5.0          │
│ 4. 检查点设计          │ 7      │ 3     │ 2.1          │
│ 5. 指令具体性          │ 15     │ 7     │ 10.5         │
│ 6. 资源整合度          │ 5      │ 5     │ 2.5          │
│ 7. 整体架构            │ 15     │ 6     │ 9.0          │
│ 8. 实测表现 (dry_run)  │ 25     │ 6     │ 15.0         │
├──────────────────────┼────────┼───────┼──────────────┤
│ TOTAL                 │ 100    │       │ 58.7         │
└──────────────────────┴────────┴───────┴──────────────┘
```

**Darwin-skill 基线总分: 58.7 / 100**

### 最弱维度（改进优先级）

1. **检查点设计** (2.1/7, 30%) — 几乎为零，无任何用户确认机制
2. **边界条件覆盖** (5.0/10, 50%) — 有事后诊断但无事前防护
3. **资源整合度** (2.5/5, 50%) — 跨skill引用大概率失效
