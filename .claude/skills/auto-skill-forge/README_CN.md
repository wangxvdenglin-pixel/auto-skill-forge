# Auto-Skill-Forge

**像训练模型一样，从零创建并自主优化你的 Agent Skills。**

受 [Karpathy 的 autoresearch](https://github.com/karpathy/autoresearch) 启发，将自主实验循环从模型训练搬到 Skill 全生命周期。创建 → 评估 → 改进 → 判卷 → 只保留真正有效的改动。一个只能向前转的棘轮。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Agent Skill](https://img.shields.io/badge/Agent%20Skill-Compatible-blueviolet)](https://skills.sh)
[![Skills](https://img.shields.io/badge/skills.sh-Compatible-green)](https://skills.sh)

---

## 快速开始

```bash
npx skills add wangxvdenglin-pixel/auto-skill-forge
```

然后直接告诉 Claude Code 你想要什么。

**创建新 skill：**
> "帮我创建一个整理论文阅读笔记的 skill"

**优化已有 skill：**
> "帮我优化 paper-weekly 这个 skill"

锻造工坊接管后续——访谈需求、设计专属 checklist、摸底盘查、自主爬山优化。你只需在关键检查点审批测试设计。

---

## 为什么做这个

Agent Skill 生态在快速扩张。Claude Code、Codex、OpenClaw、Trae 等工具都支持 SKILL.md 格式。10 个 Skill 可以手动维护，60+ 个 Skill 则需要一个系统。而且不只是优化已有的——根据用户一句话需求**从零生成高质量 Skill**，同样需要自动化。

传统审查只看**结构**：格式对不对、步骤有没有编号、路径能不能访问。但一个格式完美的 Skill，跑出来效果可能很差。

Auto-Skill-Forge 覆盖**创建→评估→优化→打包**全生命周期——用 checklist 实测效果，用棘轮确保改进，用语义对抗堵触发盲区。

---

## 核心循环

```
Phase 0   Intent      → 判断模式（创建/优化），访谈需求，写草稿，边界扩展
Phase 1   Design      → 提炼能力 → 提 checklist → 选测试输入 → 用户审批
Phase 2   Calibration → 验证 checklist 机械清晰度（TPR/TNR ≥ 80%）
Phase 3   Baseline    → 摸底考试（创建模式：双锚点对比裸 Claude）
Phase 4   Optimize    → 自主爬山：诊断→改→审稿+判卷→保留/回滚（最多 3 轮）
Phase 5   Rewrite     → 触顶时从零重写（需用户同意）
Phase 6   Report      → 汇总报告 + 打包 .skill
```

---

## 六条核心原则

| # | 原则 | 说明 |
|:---|:---|:---|
| 01 | **单一可编辑资产** | 每次只改一个 SKILL.md，变量可控，改进可归因 |
| 02 | **双重评估** | 结构评分（静态分析，7 维度 75 分）+ 效果验证（checklist 实测，25 分） |
| 03 | **棘轮机制** | 只保留改进，自动回滚退步，分数只升不降 |
| 04 | **独立评分** | 编辑和评分用不同的 agent 上下文，避免"自己改自己评" |
| 05 | **人在回路** | Phase 0-2 用户定义边界和标准，Phase 3-6 全自动闭环 |
| 06 | **创建即优化** | 创建新 skill = 从零开始爬山，同一条流水线，同一个棘轮 |

---

## 7 维度评估体系

总分 100。结构分靠静态分析（75 分），效果分靠 checklist 实测（25 分）。

| # | 维度 | 权重 | 考察内容 |
|---|------|------|---------|
| 1 | Frontmatter 质量 | 8 | name/description 格式正确，触发关键词完整 |
| 2 | 工作流清晰度 | 15 | 步骤有编号、可执行，每步有明确的输入/输出 |
| 3 | 边界条件覆盖 | 10 | 覆盖失败场景，有降级路径 |
| 4 | 检查点设计 | 7 | 关键操作前需用户确认 |
| 5 | 指令具体性 | 15 | 无模糊措辞，具体的参数、格式、示例 |
| 6 | 资源完整性 | 5 | 引用的文件、脚本、路径真实存在 |
| 7 | 整体架构 | 15 | 层次清晰，无冗余，无遗漏 |

效果分：3-6 个 yes/no checklist 问题 × 8-12 个测试输入，子 agent 独立判卷。pass_rate × 25。

---

## 优化循环

**Phase 4 是核心：**

1. 找出得分最低的维度或 checklist 问题（含 frontmatter 触发词盲区）
2. 生成 1 个针对性改进方案
3. 编辑 SKILL.md，git commit
4. 独立子 agent 同时做两件事——审稿（批评改动 + 语义触发盲区检测）+ 判卷（同一标准打两版）
5. 新分 > 旧分 → 保留；否则 → git revert
6. 循环直到触顶或满 3 轮

---

## Checklist：降低用户门槛

你不需理解"维度采样"或"tuple 组合"这些概念。

Agent 读一遍 skill，列出它声称能做哪些事，然后主动提议 3-6 个 yes/no 检查问题。你要做的只是点头、摇头、或加一条。像一个老师用 checklist 批改作文——每一项都是清晰的是或否。

---

## 创建模式

用户只需一句话描述需求（如"做一个论文阅读周报整理 skill"）。Agent 访谈 → 写草稿 → **Intent Expansion（边界扩展）**——主动反问"如果用户只给 DOI 怎么办？编码异常怎么办？"——防止自证陷阱。然后和优化模式走同一条流水线。

创建完成后用裸 Claude 做对比基线，证明 skill 的绝对价值："封装后成功率从 50% → 85%"。

## 棘轮机制

分数只能上升。每轮要么改进 Skill，要么干净地回滚。不会随时间积累局部退化。

```
轮次 0:  基线 72 分
轮次 1:  78 分 → 保留 ✅
轮次 2:  75 分 → 回滚 ❌（低于 78，自动撤销）
轮次 3:  84 分 → 保留 ✅
```

---

## 设计血统

Auto-Skill-Forge 建立在三个项目的思想之上，各自贡献了不同的一块拼图。

**来自 [autoresearch](https://github.com/karpathy/autoresearch)：** 棘轮机制——跑实验、测结果、只保留指标变好的那次。我们把这个循环作为 Phase 4 的骨架，但把"代码 + loss 函数"替换成了"skill 编辑 + checklist 通过率"作为可度量的进步单位。

**来自 [Darwin-Skill](https://github.com/alchaincyf/darwin-skill)：** 用进化压力推动 skill 改进的原始愿景。我们继承了基线→优化→重写的骨架，然后将评估体系从主观 1-10 打分重建为独立 agent 执行的机械二进制 checklist 判卷。

让 Auto-Skill-Forge 与众不同的，是 checklist 评估（非断言、非主观打分）、same-judge 消除实例方差、审稿直接嵌入优化步骤、Intent Expansion 防创建自证陷阱。

---

## 许可证

MIT
