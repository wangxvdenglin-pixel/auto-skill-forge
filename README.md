# Skill Optimizer 详细介绍

**Skill Optimizer** 是一个自主的 Agent Skill 优化器，基于 darwin-skill（受 Karpathy autoresearch 启发）升级而来。它会评估 SKILL.md 文件的质量，针对低分维度做爬山式改进，通过 git 版本控制保留有效改动、回滚退步，最终生成可视化成果卡片。

---

## 目录

1. [评估体系](#1-评估体系)
2. [测试套件设计：Dimension × Tuple](#2-测试套件设计dimension--tuple)
3. [二进制 Judge 与 TPR/TNR 校准](#3-二进制-judge-与-tprtnr-校准)
4. [优化循环全流程](#4-优化循环全流程)
5. [数据文件格式](#5-数据文件格式)
6. [异常处理](#6-异常处理)
7. [反模式](#7-反模式)
8. [与 darwin-skill 的对比](#8-与-darwin-skill-的对比)
9. [实验验证结果](#9-实验验证结果)

---

## 1. 评估体系

总分 100，由两个组件构成：

### Component A：结构分析（75分）— 静态打分

主 agent 直接阅读 SKILL.md 打分，不需要运行任何测试。

| # | 维度 | 权重 | 检查什么 |
|---|------|------|---------|
| 1 | Frontmatter 质量 | 8 | `name` 用小写+连字符。`description` 包含"做什么"+"何时用"+"触发关键词"。≤1024 字符。 |
| 2 | 工作流清晰度 | 15 | 步骤编号且可执行。每步有明确的输入和输出。 |
| 3 | 边界条件覆盖 | 10 | 覆盖失败场景。有 fallback 路径。有错误恢复。 |
| 4 | 检查点设计 | 7 | 关键决策前有用户确认步骤，防止自主失控。 |
| 5 | 指令具体性 | 15 | 没有模糊指令。有具体参数、格式、示例。能直接执行。 |
| 6 | 资源完整性 | 5 | 引用的脚本、资源、文件路径真实可达。 |
| 7 | 整体架构 | 15 | 结构层次清晰，无冗余，无遗漏。 |

**计分公式：** 每个维度打 1-10 分，乘以权重后求和，除以 10。Component A 最高 75 分。

### Component B：效果评估（25分）— 二进制 Judge

这是 skill-optimizer 相比 darwin-skill 最大的创新。不再是主观打 1-10 分，而是用二进制测试套件做客观的 PASS/FAIL 判定。

| # | 维度 | 权重 | 检查什么 |
|---|------|------|---------|
| 8 | 实测表现 | 25 | 通过二进制 judge 执行测试套件。得分 = 通过率 × 25。 |

**效果分 =（PASS 数 / 总测试用例数）× 25。**

### 总分与棘轮规则

```
总分 = 结构分 + 效果分
满分 = 75 + 25 = 100

改进要求：新分 > 旧分（严格大于，不含等于）
保留 1 位小数
```

如果无法使用子 agent 执行测试，退化为干跑验证（dry-run），在 `results.tsv` 中标注 `eval_mode=dry_run`。

---

## 2. 测试套件设计：Dimension × Tuple

这是 skill-optimizer 的第二个核心创新——用维度交叉生成系统化的测试用例，而不是拍脑袋写几个 prompt。

### Step 1：定义维度

阅读 SKILL.md，识别 skill 最容易失败的地方。定义 3 个维度，每个 2-4 个值：

```
维度：任务复杂度 — 用户给了多少信息
  值：[输入极简, 输入详细, 输入模糊/混合]

维度：用户角色 — 谁在提问
  值：[新手（需要引导）, 有经验（要具体信息）, 管理者（要摘要）]

维度：边界条件 — 什么可能出错
  值：[正常运行, 数据缺失, 需求冲突]
```

**选择维度的原则：** 针对"最可能出问题"的方向，而不是随意选变量。不同 skill 类型选不同的维度——流程型 skill 选"用户角色 + 任务复杂度 + 边界条件"，构建型 skill 选"数据格式 + 展示复杂度 + 定制需求"。

### Step 2：生成元组

从 3 个维度的笛卡尔积中随机采样约 12 个组合：

```json
[
  {"任务复杂度": "输入详细", "用户角色": "新手",     "边界条件": "正常运行"},
  {"任务复杂度": "输入极简", "用户角色": "有经验",   "边界条件": "数据缺失"},
  {"任务复杂度": "输入模糊", "用户角色": "管理者",   "边界条件": "需求冲突"},
  ...
]
```

展示给用户。用户可以删除不现实的组合、补充遗漏的组合。迭代到确认为止。

### Step 3：展开为测试用例

每个元组生成一个测试用例，包含明确的二进制通过/失败标准：

```json
{
  "id": 1,
  "dimensions": {"任务复杂度": "输入详细", "用户角色": "新手", "边界条件": "正常运行"},
  "input": "反映这个场景的自然语言 prompt",
  "criteria": {
    "pass": [
      "具体的可观测条件 1",
      "具体的可观测条件 2"
    ],
    "fail": [
      "具体的失败模式 1",
      "具体的失败模式 2"
    ]
  }
}
```

**标准设计规则：**
- 每条必须可机械检查——不能有"感觉不错"这种主观判断
- 每个用例 2-5 条 pass 条件和 2-5 条 fail 条件
- 如果 skill 声称支持某功能，至少一个测试用例要验证这个功能

### Step 4：保存

写入 `{skill-dir}/test-suite.json`。如果已存在（之前跑过），询问用户复用/重写/追加。

**到此暂停。展示完整测试套件，用户确认后再进入下一步。**

---

## 3. 二进制 Judge 与 TPR/TNR 校准

这是 skill-optimizer 的第三个核心创新。darwin-skill 直接信任评估结果，但 skill-optimizer 先问一个问题：**judge 自己的判断靠谱吗？**

### 为什么需要校准

一个未经校准的二进制 judge 可能：
- **太严**：大量本该 PASS 的判 FAIL（TPR 低）
- **太松**：大量本该 FAIL 的判 PASS（TNR 低）
- **不一致**：相同场景两次判出不同结果

校准回答两个问题：
- **TPR（真阳性率）**：用户说 PASS 的用例，judge 也判 PASS 的比例 → judge 有没有漏杀？
- **TNR（真阴性率）**：用户说 FAIL 的用例，judge 也判 FAIL 的比例 → judge 有没有误杀？

**不用原始准确率。** 当 PASS/FAIL 类别不均衡时（比如 80% 的用例都 PASS），原始准确率有误导性——盲目判 PASS 就能拿 80% 准确率，但 TNR 是 0。

### 校准流程（Phase 0.6）

```
Step 1: 从测试套件中选 15-20 个用例作为校准集
        优先多样性：至少 3 个明确 PASS、3 个明确 FAIL、3 个边界

Step 2: 用户为这 ~20 个用例打上真实标签：PASS 或 FAIL

Step 3: 独立 judge agent 对同样的用例做判定
        给 judge: 测试套件 + 输入 + pass/fail 标准
        不给 judge: 用户的标签（独立评分）

Step 4: 计算混淆矩阵
                  用户:PASS    用户:FAIL
        Judge:PASS    TP          FP
        Judge:FAIL    FN          TN

        TPR = TP / (TP + FN)
        TNR = TN / (TN + FP)

Step 5: 迭代直到 judge 通过

        | 情况 | 含义 | 动作 |
        |------|------|------|
        | TPR>80% AND TNR>80% | Judge 校准完毕 | 进入 Step 6 |
        | TPR 低，TNR 正常 | Judge 太严 | 检视误判 FAIL 的用例，放宽 fail 标准或补充 pass 说明 |
        | TNR 低，TPR 正常 | Judge 太松 | 检视误判 PASS 的用例，收紧 fail 标准或补充边界示例 |
        | 两者都低 | 标准模糊 | 重写最混乱的测试用例标准，使其更机械、更具体 |
        | 迭代 3 次后停在 70-80% | 已达瓶颈 | 接受最好的一次，标注 `partial: true`，置信区间更宽 |

Step 6: 保存校准记录到 {skill-dir}/judge-calibration.json
```

### 什么是 Rogan-Gladen 校正

这是 validate-evaluator skill 中引入的技术，用于校正已知 judge 误差后的真实通过率估计：

```
θ̂ = (p_obs + TNR - 1) / (TPR + TNR - 1)

其中 p_obs 是 judge 观察到的原始通过率
```

举例：judge 观测通过率 70%，已知 TPR=0.85, TNR=0.80

```
θ̂ = (0.70 + 0.80 - 1) / (0.85 + 0.80 - 1) = 0.50 / 0.65 = 0.769
```

观测 70% 通过率，校正后实际估计是 76.9%——judge 的偏严倾向被修正了。

当前 skill-optimizer 的 SKILL.md 尚未在 Phase 2 决策步骤中显式应用此校正（这是已知待改进项），但 judge-calibration.json 中的 TPR/TNR 数据已为将来应用做好了准备。

---

## 4. 优化循环全流程

整个流程分 7 个阶段，系统在阶段内自主运行，在阶段间暂停等待人类确认。

### Phase 0：初始化

```
1. 确定优化范围：
   - 全部 skills → 扫描 .claude/skills/*/SKILL.md（排除 skill-optimizer 自身）
   - 指定 skills → 用户指定列表
2. 创建 git 分支 auto-optimize/YYYYMMDD-HHMM
3. 如果 results.tsv 不存在，创建并写入表头
4. 读取现有 results.tsv 了解历史记录
```

### Phase 0.5：测试套件设计（Dimension × Tuple）

对每个 skill 执行完整的 Dimension × Tuple 流程（见第 2 节）。产出 `test-suite.json`。

### Phase 0.6：Judge 校准

仅在 `judge-calibration.json` 不存在、或用户主动要求时执行。完整流程见第 3 节。产出 `judge-calibration.json`。

### Phase 1：基线评估

```
对每个 skill：
  1. 读取 SKILL.md 全文
  2. 对结构维度 1-7 打分，每维度附一行理由
  3. 效果评估：
     a. 检查 judge-calibration.json（存在且非 partial → 可用；否则重跑 Phase 0.6）
     b. 生成独立 judge agent
     c. 对所有测试用例执行判定
     d. 记录每个用例的 PASS/FAIL
     e. pass_rate = PASS 数 / 总数
     f. 效果分 = pass_rate × 25
  4. 总分 = 结构分 + 效果分
  5. 追加基线行到 results.tsv
```

评估完所有 skill 后展示评分卡，**暂停等用户确认。**

### Phase 2：优化循环（核心）

按基线总分从低到高排列，先优化最弱的 skill。每个 skill 最多 3 轮。

```
对每个 skill：
  round = 0
  while round < 3:
    round += 1

    Step 1 — 诊断
    找出最低分领域：
      - 结构：哪个维度得分最低（1-7）？
      - 效果：哪些测试用例 FAIL 了？
    选一个目标：
      - 如果 dim1-7 有 ≤5 分的 → 优先修该维度
      - 否则 → 修通过率最低的能力（把 FAIL 用例按能力聚类）

    Step 2 — 提出方案
    生成一个具体改动。说明：
      - 改 SKILL.md 的哪些行
      - 对应 rubric 的哪条标准或哪个 FAIL 用例
      - 预期对结构分或通过率的影响

    Step 3 — 执行
    编辑 SKILL.md
    git commit: "optimize {skill}: {改动摘要}"

    Step 4 — 重新评估
    结构：主 agent 重新打分
    效果：生成新的 judge agent（不能复用之前的评估上下文）
      - 重跑全部测试用例（不能跳过之前 PASS 的）
    计算新的总分

    Step 5 — 决策
    if 新分 > 旧分:
      status = "keep"，更新旧分
    else:
      status = "revert"
      git revert HEAD --no-edit
      追加失败记录到 results.tsv
      break  ← 该 skill 到瓶颈，跳到下一个

    Step 6 — 日志
    追加一行到 results.tsv

  # 人类检查点
  展示：
    - git diff（改前 vs 改后）
    - 每个结构维度的分数变化
    - 每个测试用例的通过率变化（哪些从 FAIL→PASS 或 PASS→FAIL）
    - 任何翻转的用例的 judge 判定理由
  等用户确认，再继续下一个 skill
  如果用户拒绝：回滚到优化前版本
```

**修复优先级表：**

| 优先级 | 触发条件 | 动作 |
|--------|---------|------|
| P0 | ≥2 个测试用例在同一能力上 FAIL | 重点修改该能力在 SKILL.md 中的指令 |
| P0 | FAIL 用例比不带 skill 的基线输出还差 | 简化过度约束的段落 |
| P1 | Frontmatter 缺少触发关键词 | 补充中英文触发词 |
| P1 | 工作流没有编号步骤 | 重组为顺序步骤，标注输入/输出 |
| P1 | 关键决策缺用户确认 | 插入确认步骤 |
| P2 | 步骤模糊（"处理图片"） | 替换为具体参数和格式 |
| P2 | 缺少输入/输出规格 | 补充格式、路径、具体示例 |
| P2 | 无错误处理 | 补充"如果 X 失败，则 Y"的 fallback |
| P3 | 段落过长 | 拆分并改用表格 |
| P3 | 内容重复 | 合并去重 |

### Phase 2.5：探索性重写（可选）

当连续 2 个 skill 都在第一轮就 break 时触发。需用户明确同意。

```
1. 选一个卡住的 skill
2. git stash 当前版本
3. 从头重写 SKILL.md（重组结构，不是微调）
4. 重跑 Phase 0.6（旧校准可能不再适用）
5. 用新 judge agent 重新评估
6. if 重写版 > stash 版 → 采用；else → git stash pop 还原
```

这解决爬山法的局部最优问题——有时候需要"先拆后建"才能突破瓶颈。

### Phase 3：汇总报告

展示总览：优化 skill 数、总实验次数、保留/回滚数、分数变化表、用例级别改进明细。然后生成视觉成果卡片（见后文）。

---

## 5. 数据文件格式

### results.tsv

位置：`.claude/skills/skill-optimizer/results.tsv`

```tsv
timestamp	commit	skill	old_score	new_score	status	dimension	note	eval_mode	pass_rate
2026-05-08T10:00	baseline	write-judge-prompt	-	68.6	baseline	-	初始评估；judge已校准	full_test	0.625
2026-05-08T10:05	a1b2c3d	write-judge-prompt	68.6	75.2	keep	边界条件	Case #3,#4 FAIL→PASS	full_test	0.750
2026-05-08T10:10	b2c3d4e	write-judge-prompt	75.2	72.8	revert	指令具体性	过度细化，Case #6 PASS→FAIL	dry_run	0.625
```

- 10 列，TSV 格式
- `eval_mode`：`full_test`（子 agent judge 实测）或 `dry_run`（模拟推演）
- `pass_rate`：0-1 之间的小数
- 每次评估/优化追加一行

### test-suite.json

位置：`{skill-dir}/test-suite.json`

```json
{
  "dimensions": {
    "任务复杂度": {
      "description": "用户请求提供的信息量",
      "values": ["输入详细", "输入极简", "输入模糊"]
    },
    "用户角色": {
      "description": "谁在使用这个 skill",
      "values": ["新手", "有经验", "管理者"]
    },
    "边界条件": {
      "description": "什么可能导致 skill 失败",
      "values": ["正常运行", "数据缺失", "需求冲突"]
    }
  },
  "test_cases": [
    {
      "id": 1,
      "dimensions": {"任务复杂度": "输入详细", "用户角色": "新手", "边界条件": "正常运行"},
      "input": "反映这个元组的自然语言 prompt",
      "criteria": {
        "pass": ["条件 1", "条件 2"],
        "fail": ["失败模式 1", "失败模式 2"]
      }
    }
  ]
}
```

### judge-calibration.json

位置：`{skill-dir}/judge-calibration.json`

```json
{
  "calibrated_at": "2026-05-08T10:00:00",
  "tpr": 0.90,
  "tnr": 0.85,
  "judge_model": "claude-sonnet-4-6",
  "test_cases_in_calibration": 20,
  "partial": false
}
```

- `partial: true` 表示 TPR/TNR 未达 80% 阈值，通过率估计有更宽的置信区间

### 成果卡片

每完成一个 skill 的优化后生成，全部完成后生成总览卡片。

模板：`templates/result-card.html`，3 种主题随机选一种：

| 主题 | Hash | 风格 |
|------|------|------|
| Warm Swiss | `#swiss` | 暖白底+赤陶橙，Inter 字体，干净网格 |
| Dark Terminal | `#terminal` | 近黑底+荧光绿，等宽字体，扫描线 |
| Newspaper | `#newspaper` | 暖白纸+深红，衬线字体，双栏编辑风 |

生成方式：复制模板 → 替换数据占位符 → `node scripts/screenshot.mjs` 截图（2x 高清）→ 自动打开 PNG。

---

## 6. 异常处理

所有异常先告知用户，再按规则处理。**绝不静默跳过。**

| 场景 | 触发条件 | 处理 |
|------|---------|------|
| 不在 git 仓库 | `git rev-parse` 失败 | 建议 `git init`；若拒绝，用文件备份代替 git revert |
| results.tsv 缺失 | 文件不存在 | 自动创建并写表头 |
| results.tsv 损坏 | 列数不对 | 备份为 `.bak` 后重建，告知用户 |
| 分支已存在 | `git checkout -b` 失败 | 分支名加 `-2`/`-3`，第 3 次失败切回现有分支并询问 |
| git revert 失败 | 冲突 | `git stash` 后重试；仍失败则从旧 commit 提取文件手动恢复 |
| MAX_ROUNDS 触顶 | 3 轮仍有短板 | 展示最弱维度和 FAIL 用例，询问"加一轮/探索重写/收工" |
| 文件超 150% | 新文件 > 原文件 × 1.5 | 拒绝提交，精简（删冗余/合并重复），再评 |
| test-suite.json 已存在 | 文件已存在 | 复用。询问复用/重写/追加 |
| SKILL.md 找不到 | 目录存在但无 SKILL.md | 终止该 skill，记录 `status=error`，继续下一个 |
| Judge 无法校准 | TPR/TNR 三次迭代后停在 70-80% | 标注 `partial: true` |
| 无子 agent 可用 | 无法生成独立 judge | 退化为 dry-run，主 agent 直接对照标准判定 |

---

## 7. 反模式

这些事绝对不能做：

1. **改变 skill 的核心功能和用途**——只优化"怎么写"和"怎么执行"，不改"做什么"
2. **添加新依赖**——不引入 skill 原本没有的脚本、引用或依赖
3. **一轮改多个不相关维度**——一次一个，确保改进可归因
4. **文件膨胀超 150%**——超过则拒绝提交
5. **用 `git reset --hard` 回滚**——必须用 `git revert`（保留历史）
6. **在同一 agent 上下文里改完自己评**——效果评估必须用独立 judge agent
7. **跳过 judge 校准直接信任原始分数**——没有 TPR/TNR 的 judge 分数不可靠
8. **基于随意变量设计测试维度**——维度必须是针对失败轴的预测，不能是随机排列
9. **用开发/测试数据做 judge 校准**——校准数据来自同一测试套件，但标签仅用于 TPR/TNR 计算，不用于 few-shot 示例

---

## 8. 与 darwin-skill 的对比

| 维度 | Darwin-Skill | Skill-Optimizer |
|------|-------------|-----------------|
| **效果评估方法** | 主观 1-10 打分 | 二进制 PASS/FAIL × 通过率 |
| **评估可复现性** | 低（依赖评估者） | 高（相同标准 → 相同结果） |
| **评估粒度** | 整个 skill 一个数字 | 逐用例 → 可归纳失效模式 |
| **评估可操作性** | "实测表现弱，6/10" | "jq null 输出违规，flattenJSON null crash" |
| **测试用例设计** | 临时写几个 prompt | Dimension × Tuple 系统化生成 |
| **Judge 可靠性** | 无校准机制 | TPR/TNR 校准 + Rogan-Gladen 校正 |
| **结构评估** | 7 维（1-10 × 权重 / 10） | 完全相同 |
| **棘轮机制** | `git revert` | `git revert`（相同） |
| **人在回路** | 每个 skill 后暂停 | 每个 skill 后暂停（相同） |
| **成果卡片** | 3 风格 PNG | 3 风格 + 通过率/校准数据（增强） |

**核心差异一句话：** darwin-skill 告诉你"这个 skill 大概 6/10 分"，skill-optimizer 告诉你"3 个用例因为 null 处理失败、2 个用例因为类型丢失失败——修这 4 个 bug，通过率从 53% 提到 93%。"

---

## 9. 实验验证结果

做了两场对照实验，比较 darwin-skill 和 skill-optimizer 的优化效果：

### 实验 1：json-and-csv-data-transformation（代码型 skill）

```
              基线      Darwin    Optimizer
通过率         53.3%  →  80.0%  →  100.0%
差距                    +26.7pp    +46.7pp
```

Darwin 修了结构（加检查点、Golden rules），但 3 个代码 bug 没修——`flattenJSON` 碰到 null 照样崩溃。Optimizer 4 个代码改动消灭全部 7 个失败。

### 实验 2：meeting-notes（模板型 skill，零代码）

```
              基线      Darwin    Optimizer
通过率         62.5%  →  87.5%  →  100.0%
差距                    +25.0pp    +37.5pp
```

纯文字内容 skill，差距从 20pp 缩到 12.5pp。两者都在改文字——darwin 的边缘情况处理碰巧覆盖了一个缺失场景，optimizer 直接补了缺失的模板。

### 结论

- **代码类 skill**：optimizer 显著优于 darwin（二进制 judge 精准定位代码 bug）
- **模板/流程类 skill**：两者差距缩小，优先级的差异 > 能力的差异
- **最佳实践**：先用 optimizer 修代码 bug，再用 darwin 提结构质量

---

## 核心哲学

> 像训练模型一样优化 Agent Skills。
> 每次只改一个 SKILL.md。
> 用结构性标准 + 实测效果双重评估。
> 只保留可测量的改进，其余全部回滚。
> 分数只升不降——一个只能向前转的棘轮。
> 人在回路——Skill 的好坏比 loss 更微妙，需要人的判断。
