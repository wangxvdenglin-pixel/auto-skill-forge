---
name: skill-optimizer
description: "Autonomous skill optimizer. Evaluates SKILL.md files using structural analysis (7 dimensions, 75pts) + binary judge on dimension×tuple test suites (1 dimension, 25pts). Runs hill-climbing improvement with git version control, validates changes through calibrated binary judges, and generates visual result cards. Use when user mentions \"优化skill\", \"skill评分\", \"自动优化\", \"auto optimize\", \"skill质量检查\", \"skill review\", \"skill打分\", \"提升skill质量\", \"优化所有skills\", \"优化某个skill\", \"评估所有skills\", \"skill优化历史\"."
---

# Skill Optimizer

Evaluate SKILL.md files against structural rubric + binary judge test suites. Improve low-scoring dimensions. Keep only changes that increase total score. Generate visual result cards.

## Prerequisites

- Git repository (`git rev-parse --git-dir` succeeds)
- `results.tsv` in skill-optimizer directory (auto-created on first run)

## Evaluation Rubric

2 components, 100 points total.

### Component A: Structural Analysis (75 points) — Static, Scored by Main Agent

| # | Dimension | Weight | What to check |
|---|-----------|--------|---------------|
| 1 | Frontmatter quality | 8 | `name` is lowercase-hyphenated. `description` includes what the skill does, when to use it, and trigger keywords. ≤1024 characters. |
| 2 | Workflow clarity | 15 | Steps are numbered and executable. Each step has explicit inputs and outputs. |
| 3 | Edge case coverage | 10 | Covers failure scenarios. Has fallback paths. Handles error recovery. |
| 4 | Checkpoint design | 7 | Critical decisions are preceded by user confirmation. |
| 5 | Instruction specificity | 15 | No vague directives. Supplies concrete parameters, formats, and examples. Executable as written. |
| 6 | Resource integrity | 5 | References to scripts, assets, and files resolve to paths that exist. |
| 7 | Overall architecture | 15 | Structure is clear, no redundancy, no gaps. |

Score each dimension 1-10. Multiply by weight. Sum all weighted scores, divide by 10. **Structural score ≤ 75.**

### Component B: Effectiveness (25 points) — Binary Judge, Requires Execution

| # | Dimension | Weight | What to check |
|---|-----------|--------|---------------|
| 8 | Observed performance | 25 | Execute the test suite through a binary judge. Score = pass_rate × 25. |

**Effectiveness score = (PASS count / total test cases) × 25.**

### Total Score

```
total = structural_score + effectiveness_score
```

Max = 100. Improvement requires strict `>` (not `≥`). Round to 1 decimal place. If sub-agent execution is unavailable, use dry-run simulation and annotate `eval_mode=dry_run` in results.tsv.

## Core Instructions

### Phase 0: Initialization

1. Determine scope:
   - All skills → scan `.claude/skills/*/SKILL.md`. Exclude the optimizer itself.
   - Specific skills → use the user-supplied list.
2. Create git branch `auto-optimize/YYYYMMDD-HHMM`.
3. If `results.tsv` is missing, create it with header: `timestamp	commit	skill	old_score	new_score	status	dimension	note	eval_mode	pass_rate`.
4. Read existing `results.tsv` for prior records.

### Phase 0.5: Test Suite Design (Dimension × Tuple)

For each skill in scope. This phase produces a `test-suite.json` that replaces the legacy `test-prompts.json`.

#### Step 1: Define Dimensions

Read SKILL.md. Identify where the skill is most likely to fail. Define 3 dimensions that target those failure regions.

```
Dimension: {Name} — {What it captures, one sentence}
  Values: [{value_a}, {value_b}, {value_c}, ...]
```

Choose dimensions based on anticipated failure axes. Each dimension: 2-4 values. Examples for different skill types:

For a skill that helps users through a process (e.g. error-analysis):
```
Dimension: Task complexity — how much the user gives to work with
  Values: [minimal input, detailed input, ambiguous/mixed input]

Dimension: User role — who is doing the asking
  Values: [novice (needs guidance), experienced (wants specifics), manager (wants summary)]

Dimension: Edge conditions — what could go wrong
  Values: [normal operation, missing data, conflicting requirements]
```

For a skill that builds artifacts (e.g. build-review-interface):
```
Dimension: Data format — what the input data looks like
  Values: [clean JSON, malformed JSON, CSV with mixed types]

Dimension: Display complexity — what needs to be shown
  Values: [simple text, nested objects, markdown with code blocks]

Dimension: Customization — what the user wants to change
  Values: [default layout, custom styling, domain-specific columns]
```

#### Step 2: Generate Tuples

Generate ~12 tuples by random sampling from the cross-product of dimension values. Each tuple is one combination:

```json
[
  {"任务复杂度": "输入详细",   "用户角色": "新手",     "边界条件": "正常运行"},
  {"任务复杂度": "输入最少",   "用户角色": "有经验",   "边界条件": "数据缺失"},
  {"任务复杂度": "输入模糊",   "用户角色": "管理者",   "边界条件": "需求冲突"},
  ...
]
```

Present all tuples to the user. User marks unrealistic combinations for removal and may suggest additions. Iterate until confirmed.

#### Step 3: Expand Tuples to Test Cases

For each confirmed tuple, generate a test case with explicit binary criteria:

```json
{
  "id": 1,
  "dimensions": {"任务复杂度": "输入详细", "用户角色": "新手", "边界条件": "正常运行"},
  "input": "natural language prompt that a user would actually say, reflecting this tuple",
  "criteria": {
    "pass": [
      "specific observable condition 1",
      "specific observable condition 2",
      "specific observable condition 3"
    ],
    "fail": [
      "specific failure mode 1",
      "specific failure mode 2"
    ]
  }
}
```

Rules for criteria:
- Each pass/fail condition must be mechanically checkable — no "feels right" judgments.
- 2-5 pass conditions and 2-5 fail conditions per case.
- Conditions come from the skill's stated capabilities. If the skill claims to handle X, at least one test case must verify X.

#### Step 4: Save

Write all test cases to `{skill-dir}/test-suite.json`.

If `test-suite.json` already exists (from a prior run): reuse it. Ask user: reuse / rewrite / append.

Pause. Display the full test suite. Do not proceed until user confirms.

### Phase 0.6: Judge Calibration

Only run this phase if `{skill-dir}/judge-calibration.json` does not exist, or if the user requests re-calibration.

#### Step 1: Select Calibration Set

Select 15-20 test cases from the suite. Prioritize diversity: include at least 3 clear-PASS cases, 3 clear-FAIL cases, and 3 borderline cases.

#### Step 2: Collect Ground Truth Labels

Present each of the ~20 test cases to the user. User labels each: PASS or FAIL.

#### Step 3: Judge Scoring

Spawn an independent judge agent. Give the judge:
- The skill's test-suite.json
- The same 15-20 test cases (input + pass/fail criteria only, NOT the user's labels)

The judge executes each test case: runs the skill on the input, checks the output against the pass/fail criteria, and returns `PASS` or `FAIL` with a short justification.

#### Step 4: Compute TPR and TNR

```
             User: PASS    User: FAIL
Judge: PASS      TP             FP
Judge: FAIL      FN             TN

TPR = TP / (TP + FN)   — "Of what user says PASS, what fraction does judge agree?"
TNR = TN / (TN + FP)   — "Of what user says FAIL, what fraction does judge agree?"
```

Do not use raw accuracy (TP + TN) / total. With imbalanced classes it is misleading.

#### Step 5: Iterate Until Judge Passes

| Condition | Action |
|-----------|--------|
| TPR > 80% AND TNR > 80% | Judge is calibrated. Proceed to Step 6. |
| TPR low, TNR ok | Judge is too strict. Examine false-FAIL cases. Loosen fail criteria or add clarification to pass criteria. Go to Step 3. |
| TNR low, TPR ok | Judge is too lenient. Examine false-PASS cases. Tighten fail criteria or add edge-case examples. Go to Step 3. |
| Both low | Judge criteria are ambiguous. Rewrite the most confused test case criteria with more concrete, mechanical conditions. Go to Step 3. |
| Both plateau 70-80% after 3 iterations | Accept the best iteration. Annotate calibration record with `partial: true`. The pass_rate from this judge carries a wider confidence interval. |

#### Step 6: Save Calibration Record

Write `{skill-dir}/judge-calibration.json`:

```json
{
  "calibrated_at": "ISO timestamp",
  "tpr": 0.90,
  "tnr": 0.85,
  "judge_model": "model used for judging",
  "test_cases_in_calibration": 20,
  "partial": false
}
```

### Phase 1: Baseline Assessment

For each skill in scope:

```
1. Read SKILL.md in full.

2. Score structural dimensions 1-7. Attach a one-line reason per dimension.

3. Effectiveness scoring via binary judge:
   a. Check {skill-dir}/judge-calibration.json:
      - Exists and not partial → use calibrated judge
      - Missing or partial → re-run Phase 0.6 first
   b. Spawn an independent judge agent.
   c. Run ALL test cases in test-suite.json through the judge.
   d. Record PASS/FAIL per case.
   e. pass_rate = PASS count / total cases.
   f. effectiveness_score = pass_rate × 25.

4. Compute total = structural_score + effectiveness_score.

5. Append baseline row to results.tsv.
```

After scoring all skills, display:

```
┌──────────────────────────┬──────────────┬──────────────────┬──────────────────┬───────────┐
│ Skill                    │ Structural   │ Pass Rate        │ Score (Total)     │ Weakest   │
├──────────────────────────┼──────────────┼──────────────────┼──────────────────┼───────────┤
│ {name}                   │ 48/75        │ 5/8 (62.5%)      │ 48+15.6=63.6      │ Case #3,4 │
├──────────────────────────┼──────────────┼──────────────────┼──────────────────┼───────────┤
│ Average                  │ {N}          │ {X%}             │ {N}              │           │
└──────────────────────────┴──────────────┴──────────────────┴──────────────────┴───────────┘
```

Pause. Wait for user confirmation.

### Phase 2: Optimization Loop

Process skills sorted by baseline total score (lowest first). Default max 3 rounds per skill.

```
for each skill:
  round = 0
  while round < 3:
    round += 1

    # Step 1: Diagnose
    Identify the lowest-performing area:
      - Structural: which dimension scored lowest (1-7)?
      - Effectiveness: which test cases FAILED?

    Pick ONE target:
      - If any dimension 1-7 scored ≤ 5: target that dimension.
      - Else: target the capability with the lowest pass rate (cluster FAIL cases by their shared capability).

    # Step 2: Propose fix
    Generate ONE specific change. State:
      - Which lines to edit in SKILL.md
      - Which rubric dimension or FAIL case it addresses
      - Expected impact on structural score or pass_rate

    # Step 3: Apply fix
    Edit SKILL.md.
    Commit: "optimize {skill}: {change summary}"

    # Step 4: Re-score
    Structural: main agent re-scores dimensions 1-7.
    Effectiveness: spawn NEW judge agent (do not reuse prior scoring context).
      - Re-run ALL test cases. Do not skip previously-PASS cases.
    Compute new total.

    # Step 5: Decide
    if new_total > old_total:
      status = "keep". Update old_total reference.
    else:
      status = "revert"
      git revert HEAD --no-edit
      Append failure to results.tsv.
      break

    # Step 6: Log
    Append row to results.tsv.

  # Human checkpoint
  Display:
    - git diff (before vs after)
    - Score delta per structural dimension
    - Pass rate delta per test case (which cases changed from FAIL→PASS or PASS→FAIL)
    - Judge PASS/FAIL justifications for any cases that flipped
  Wait for user confirmation before next skill.
  If user rejects: revert to pre-optimization version.
```

**Fix priority per round:**

| Priority | Trigger | Action |
|----------|---------|--------|
| P0 | ≥2 test cases FAIL on the same capability | Target that capability's instructions in SKILL.md |
| P0 | FAIL cases show WORSE output than without the skill | Simplify over-constrained sections |
| P1 | Frontmatter missing trigger keywords | Add Chinese + English trigger words |
| P1 | Workflow has no numbered steps | Restructure into sequential steps with I/O |
| P1 | Critical decision lacks user checkpoint | Insert confirmation step |
| P2 | Step is vague ("process the image") | Replace with specific parameters and format |
| P2 | Missing input/output specification | Add format, path, and concrete example |
| P2 | No error handling | Add "if X fails, then Y" fallback |
| P3 | Paragraph too long | Split and use a table |
| P3 | Repeated content | Merge duplicates |

### Phase 2.5: Exploratory Rewrite (Optional)

When 2 consecutive skills break at round 1, propose this phase. Requires explicit user consent.

```
1. Choose a plateaued skill.
2. git stash the current version.
3. Rewrite SKILL.md from scratch (restructure, not micro-edits).
4. Re-run Phase 0.6 (judge calibration) — the old calibration may not apply.
5. Re-score with fresh judge agent.
6. If rewrite > stashed: adopt rewrite. Else: git stash pop.
```

### Phase 3: Summary Report

```
## Optimization Report

### Overview
- Skills optimized: N
- Total experiments: M
- Improvements retained: X (Y%)
- Reverts: Z
- Evaluation mode: A judge_calibrated / B dry_run

### Score Changes
┌──────────────────────────┬────────┬────────┬────────┬──────────────┐
│ Skill                    │ Before │ After  │ Δ      │ Pass Rate Δ   │
├──────────────────────────┼────────┼────────┼────────┼───────────────┤
│ {name}                   │ {N}    │ {N}    │ {±N}   │ 50%→75%       │
├──────────────────────────┼────────┼────────┼────────┼───────────────┤
│ Average                  │ {N}    │ {N}    │ {±N}   │ {X%→Y%}       │
└──────────────────────────┴────────┴────────┴────────┴───────────────┘

### Test Case Improvements
1. [{skill}] Case #3 "{capability}" — FAIL→PASS: added output template instruction
2. [{skill}] Case #5 "{capability}" — FAIL→PASS: clarified edge case handling
```

## Result Card Generation

Generate a visual result card after each skill optimization and again after the full summary.

### Template

File: `templates/result-card.html`. Three themes selected randomly by URL hash:

| Theme | Hash | Visual |
|-------|------|--------|
| Warm Swiss | `#swiss` | Warm white background, terracotta orange, Inter font, clean grid |
| Dark Terminal | `#terminal` | Near-black background, neon green, monospace font, scan lines |
| Newspaper | `#newspaper` | Warm paper background, deep red, serif font, two-column layout |

### Generation Steps

```
1. Copy templates/result-card.html to a temp working file.
2. Replace placeholder data via data-field attributes:
   - data-field="skill-name" → actual skill name
   - data-field="score-before/after/delta" → actual scores
   - 8 dimension bar widths (dim-bar-before/after) → actual percentages
   - data-field="improvement-1/2/3" → actual improvement summaries
   - data-field="date" → current date
   - data-field="pass-rate-before/after" → pass rates
   - data-field="judge-tpr/tnr" → calibration values
3. Set hash to swiss, terminal, or newspaper (randomly chosen).
4. Screenshot at 2x resolution, element-only (.card):
   node scripts/screenshot.mjs {path/to/card.html} {path/to/output.png}
   Fallback:
   npx playwright screenshot "file:///{path/to/card.html}#[theme]" output.png --viewport-size=960,1280 --wait-for-timeout=2000
5. Open the output PNG.
```

### Resource Paths

| Path | Purpose |
|------|---------|
| `templates/result-card.html` | 3-theme main template (swiss/terminal/newspaper via hash) |
| `templates/result-card-dark.html` | Single-theme dark alternative |
| `templates/result-card-white.html` | Single-theme light alternative |
| `scripts/screenshot.mjs` | 2x screenshot, element-only capture, auto-open |
| `results.tsv` | Optimization history (10 columns including pass_rate) |
| `{skill-dir}/test-suite.json` | Per-skill binary-judge test suite (dimension×tuple) |
| `{skill-dir}/judge-calibration.json` | Per-skill judge calibration record (TPR, TNR) |

## Data File Formats

### results.tsv

```tsv
timestamp	commit	skill	old_score	new_score	status	dimension	note	eval_mode	pass_rate
2026-05-08T10:00	baseline	write-judge-prompt	-	68.6	baseline	-	Initial; judge calibrated TPR=0.90 TNR=0.85	full_test	0.625
2026-05-08T10:05	a1b2c3d	write-judge-prompt	68.6	75.2	keep	Edge cases	Case #3,#4 FAIL→PASS: added fallback for missing input	full_test	0.750
2026-05-08T10:10	b2c3d4e	write-judge-prompt	75.2	72.8	revert	Instruction specificity	Over-refined — Case #6 PASS→FAIL	dry_run	0.625
```

File location: `.claude/skills/skill-optimizer/results.tsv`.

`eval_mode`: `full_test` (sub-agent judge used) or `dry_run` (simulated, judge unavailable).

### test-suite.json

Location: `{skill-dir}/test-suite.json`.

```json
{
  "dimensions": {
    "任务复杂度": {
      "description": "How much detail the user provides in their request",
      "values": ["输入详细", "输入最少", "输入模糊"]
    },
    "用户角色": {
      "description": "Who is using the skill",
      "values": ["新手", "有经验", "管理者"]
    },
    "边界条件": {
      "description": "What could cause the skill to fail",
      "values": ["正常运行", "数据缺失", "需求冲突"]
    }
  },
  "test_cases": [
    {
      "id": 1,
      "dimensions": {"任务复杂度": "输入详细", "用户角色": "新手", "边界条件": "正常运行"},
      "input": "natural language prompt reflecting this tuple",
      "criteria": {
        "pass": ["condition 1", "condition 2"],
        "fail": ["failure mode 1", "failure mode 2"]
      }
    }
  ]
}
```

### judge-calibration.json

Location: `{skill-dir}/judge-calibration.json`.

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

## Exception Handling

Notify user before applying any fallback. Do not silently skip or continue.

| Condition | Trigger | Action |
|-----------|---------|--------|
| Not a git repo | `git rev-parse --git-dir` fails | Prompt user to `git init`. If declined: backup as `SKILL.md.bak.YYYYMMDD-HHMM` instead of git revert. |
| results.tsv missing | File does not exist | Create with header row. |
| results.tsv corrupted | Wrong column count or non-TSV | Backup as `.bak.YYYYMMDD-HHMM`. Recreate. Notify user. |
| Branch exists | `git checkout -b` fails | Append `-2` / `-3` to branch name. After 3 failures: switch to existing branch and ask continue / restart. |
| git revert fails | Conflict or dirty working tree | `git stash`, retry. If still failing: extract SKILL.md from previous commit and overwrite file. |
| MAX_ROUNDS reached | round = 3, gaps remain | Display weakest dimension and FAIL cases. Ask: "add round / exploratory rewrite (Phase 2.5) / stop." |
| File > 150% original | New size > old × 1.5 | Reject commit. Simplify (remove redundancy, merge repeats). Re-score. |
| test-suite.json exists | File already in skill directory | Reuse it. Ask: reuse / rewrite / append. |
| judge-calibration.json exists | File already in skill directory | Reuse it. Skip Phase 0.6 unless user requests re-calibration. |
| SKILL.md not found | Directory exists, no SKILL.md | Terminate that skill. Write `status=error` to results.tsv. Continue to next. |
| Judge cannot be calibrated | TPR/TNR plateau 70-80% after 3 iterations | Annotate `partial: true`. Flag pass_rate as approximate in reports. |
| No sub-agent available | Cannot spawn independent judge | Fall back to dry-run: main agent evaluates each test case against criteria directly. Annotate `eval_mode=dry_run`. |

## Anti-Patterns

- Changing what the skill does (its core purpose and capabilities). Only change how it is written and executed.
- Adding new scripts, references, or dependencies the skill did not already have.
- Editing multiple unrelated dimensions in one round.
- Growing SKILL.md beyond 150% of its original size.
- Using `git reset --hard` for rollback. Use `git revert`.
- Scoring effectiveness in the same agent context that performed the edit.
- Skipping judge calibration and trusting raw judge scores without TPR/TNR.
- Designing test dimensions around arbitrary variation instead of anticipated failure axes.
- Reusing dev/test case data for judge calibration. Calibration cases must come from the same test-suite but their labels are used only for TPR/TNR calculation, not for few-shot examples.
