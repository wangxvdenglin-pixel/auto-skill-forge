---
name: skill-optimizer
description: "Autonomous skill optimizer. Evaluates SKILL.md files using structural analysis (7 dimensions, 75pts) + binary test suite pass rate (25pts). Runs hill-climbing improvement with adversarial review, git version control, and generates visual result cards. Use when user mentions \"优化skill\", \"skill评分\", \"自动优化\", \"auto optimize\", \"skill质量检查\", \"skill review\", \"skill打分\", \"提升skill质量\", \"优化所有skills\", \"优化某个skill\", \"评估所有skills\", \"skill优化历史\"."
---

# Skill Optimizer

Evaluate and improve SKILL.md files. Two-part scoring: structural rubric (75pts) + test suite pass rate (25pts). Each round: diagnose the weakest point → edit → adversarial review + re-score → keep or revert. Generate visual result cards.

## Prerequisites

- Git repo at project root
- `results.tsv` in skill-optimizer directory (auto-created on first run if missing)

## Evaluation Rubric (100 points)

### Component A: Structural (75 points)

Score each dimension 1-10. Multiply by weight. Sum all, divide by 10.

| # | Dimension | Wt | Check |
|---|-----------|----|-------|
| 1 | Frontmatter quality | 8 | `name` is lowercase-hyphenated. `description` says what it does, when to use, and lists trigger keywords. ≤1024 chars. |
| 2 | Workflow clarity | 15 | Numbered, executable steps. Each step has explicit input and output. |
| 3 | Edge case coverage | 10 | Covers failure scenarios. Has fallback paths. |
| 4 | Checkpoint design | 7 | User confirmation before critical/irreversible actions. |
| 5 | Instruction specificity | 15 | No vague directives. Concrete parameters, formats, examples. |
| 6 | Resource integrity | 5 | Referenced files, scripts, paths actually exist. |
| 7 | Overall architecture | 15 | Clear structure. No redundancy. No gaps. |

### Component B: Effectiveness (25 points)

Design 8-15 test prompts for the skill. Each prompt simulates a real user request. Each has explicit PASS/FAIL criteria.

Spawn an independent agent (the "judge"). Give it the SKILL.md and all test prompts. It executes each prompt and checks the output against the criteria.

```
pass_rate = PASS count / total cases
effectiveness_score = pass_rate × 25
```

If spawning a sub-agent is not possible, use dry-run simulation and note `eval_mode=dry_run` in results.

### Total Score

```
total = structural_score + effectiveness_score   (max 100)
```

Improvement requires strict `>` (not `≥`). Round to 1 decimal.

## Core Loop

```
Phase 0   Init          → 确定范围，建分支
Phase 0.5 Test Design   → Dimension × Tuple，12道考题，用户确认
Phase 0.6 Calibration   → 抽5题人工标注，校准判卷agent
Phase 1   Baseline      → 摸底考试，出评分卡
Phase 2   Optimize       → 每个skill改→审→判，最多3轮
Phase 3   Report        → 汇总 + 成果卡片
```

---

### Phase 0: Initialize

1. Determine scope: all skills (scan `.claude/skills/*/SKILL.md`, skip optimizer itself) or user-specified list.
2. Create branch `auto-optimize/YYYYMMDD-HHMM`.
3. If `results.tsv` missing, create with header: `timestamp	commit	skill	old_score	new_score	status	dimension	note	eval_mode	pass_rate`.
4. Read existing `results.tsv`.

---

### Phase 0.5: Test Design (Dimension × Tuple)

For each skill. This produces `{skill-dir}/test-prompts.json`.

#### Step 1: Define Dimensions

Read SKILL.md. Identify where the skill is most likely to fail. Define 3 dimensions that target those failure regions. Each dimension: a name, a one-line description, and 2-4 values.

```
Dimension: {Name} — {What it captures, one sentence}
  Values: [{value_a}, {value_b}, {value_c}]
```

Choose dimensions based on anticipated failure axes, not random variation. Examples:

For a process skill (e.g. meeting-notes, error-analysis):
```
Dimension: Input quality — how much detail the user provides
  Values: [minimal, detailed, ambiguous]

Dimension: User need — what kind of output the user wants
  Values: [full detail, summary only, action items only]

Dimension: Edge conditions — what could go wrong
  Values: [normal, missing data, conflicting info]
```

For a builder skill (e.g. code generation, data transformation):
```
Dimension: Input format — what the input looks like
  Values: [clean/valid, malformed, mixed types, empty]

Dimension: Complexity — how hard the task is
  Values: [simple/1-step, moderate/3-step, complex/multi-file]

Dimension: Constraints — what limits apply
  Values: [no constraints, strict format required, performance-sensitive]
```

#### Step 2: Generate Tuples

Create ~12 tuples by sampling from the cross-product of dimension values. Cover diversity: include at least 3 edge combinations and 3 common combinations.

Show tuples to user. User removes unrealistic ones, adds missing ones. Iterate until confirmed.

#### Step 3: Expand to Test Cases

For each confirmed tuple, write a test case:

```json
{
  "id": 1,
  "dimensions": {"Input quality": "minimal", "User need": "summary only", "Edge conditions": "normal"},
  "prompt": "natural language a real user would say, reflecting this combination",
  "pass": [
    "observable condition — mechanically checkable",
    "observable condition"
  ],
  "fail": [
    "failure mode — what would be wrong",
    "failure mode"
  ]
}
```

Rules:
- 2-5 pass conditions, 2-5 fail conditions per case.
- Every condition must be mechanically checkable — no "feels right" or subjective language.
- If the skill claims to handle X, at least one test case must verify X.
- Total: 8-15 test cases (aim for ~12).

#### Step 4: Save

Write to `{skill-dir}/test-prompts.json`. If the file already exists, ask: reuse / rewrite / append.

Pause. Display the full test suite. Do not proceed until user confirms.

---

### Phase 0.6: Criteria Calibration

A sub-agent cannot be "the same person" across calls — each spawn is a fresh instance with its own judgment thresholds. What you CAN calibrate is the test criteria: make them mechanically checkable so different judges produce the same results.

Only needs 5 test cases.

#### Step 1: Select 5 Cases

Pick 5 test cases from the suite that represent diverse difficulty:
- 2 cases that should clearly PASS
- 2 cases that should clearly FAIL
- 1 borderline case

#### Step 2: User Labels

Present each of the 5 cases to the user (show the prompt + pass/fail criteria). User labels each: PASS or FAIL. This takes ~2 minutes.

#### Step 3: Judge Scoring

Spawn an independent judge agent. Give it the SKILL.md and the 5 test cases (with criteria, WITHOUT user labels). Judge returns PASS or FAIL per case with a one-line justification.

#### Step 4: Check Criteria Clarity

Compare user labels vs judge results:

```
             User: PASS    User: FAIL
Judge: PASS      TP             FP
Judge: FAIL      FN             TN

TPR = TP / (TP + FN)   — criteria clear enough that judge agrees with user on PASS cases?
TNR = TN / (TN + FP)   — criteria clear enough that judge agrees with user on FAIL cases?
```

Low TPR or TNR means the test criteria are ambiguous, NOT that the judge is "bad." Fix the criteria, not the judge.

#### Step 5: Act on Results

| Condition | What it means | Action |
|-----------|--------------|--------|
| TPR ≥ 80% AND TNR ≥ 80% | Criteria are mechanically clear. Proceed. |
| TPR < 80% | PASS criteria too vague — judge doesn't know what "good" looks like | Make pass conditions more specific. Add concrete, observable must-haves. Re-test. |
| TNR < 80% | FAIL criteria too vague — judge doesn't know what "bad" looks like | Make fail conditions more specific. Remove "feels wrong" language. Re-test. |
| Both < 80% after 2 retries | These cases are inherently subjective | Accept best result. Annotate `criteria_partial: true` for these specific cases. |

#### Step 6: Record

Save calibration result alongside results.tsv:

```
criteria_calibrated: true (or partial)
tpr: 0.XX
tnr: 0.XX
```

This is NOT a score of the judge's ability. It's a score of how mechanically clear the test criteria are — can two independent evaluators (you and a fresh sub-agent) reach the same conclusion from the same criteria?

---

### Phase 1: Baseline Assessment

For each skill:

1. Read SKILL.md. Score structural dimensions 1-7. One-line reason per dimension.
2. Spawn an independent judge agent. Run ALL test cases. Record PASS/FAIL per case.
3. Compute `pass_rate` and `effectiveness_score`.
4. `total = structural + effectiveness`.
5. Append baseline row to `results.tsv`.

Log the scorecard and proceed directly to Phase 2. No pause needed — the test suite and judge were already confirmed in Phase 0.5-0.6.

---

### Phase 2: Optimization

Process skills from lowest score to highest. Max 3 rounds per skill. **No human intervention — runs autonomously like auto-research.**

Each round has 4 steps:

| Step | What | Who |
|------|------|-----|
| 1. Diagnose | Find the ONE weakest point | Main agent |
| 2. Edit | Fix it. git commit. | Main agent |
| 3. Review+Score | Critique the change + re-run all tests | **Independent review agent** |
| 4. Decide | Keep if score improved, revert if not | Main agent |

No pauses between rounds or between skills. The loop runs until all skills hit their ceiling (no improvement after 1 round) or reach 3 rounds, whichever comes first. Log every round to results.tsv.

#### Step 1: Diagnose

Pick ONE target:

- Any structural dimension scored ≤ 5? → Target that dimension.
- Multiple test cases FAIL on the same capability? → Target that capability.
- Otherwise: target the capability with the lowest pass rate.

#### Step 2: Edit

- State: which lines, which target, expected impact.
- Edit SKILL.md.
- Git commit: `"optimize {skill}: {brief summary}"`.

#### Step 3: Adversarial Review + Same-Judge Re-score

**Spawn ONE independent agent.** Give it ALL of the following:
- The ORIGINAL SKILL.md (baseline, before any edits)
- The OPTIMIZED SKILL.md (after your edit)
- The git diff of your change
- The target problem you were trying to fix
- The full test suite (test-prompts.json)

Do NOT give it your reasoning or diagnosis.

The review agent has a dual role — it is both a **critic** and a **judge**:

**As critic, it answers:**
1. Did this change actually fix the target problem? (Yes / Partially / No)
2. Did it introduce new problems? (list specific issues, or "None")
3. Is there a simpler way to achieve the same fix? (suggest, or "No")
4. Overall quality of this change: /10

**As judge, it scores BOTH versions against the SAME test suite — same judge, same context, same standards:**

5. Baseline score: PASS/FAIL per test case (one-line justification each) + baseline pass_rate
6. Optimized score: PASS/FAIL per test case (one-line justification each) + new pass_rate
7. Delta: new pass_rate − baseline pass_rate

**Why both versions in the same call?** Each sub-agent spawn is a fresh instance with its own judgment thresholds. Different judges give different verdicts on the same skill (as proven in experiment 4). Having the SAME judge score BOTH versions eliminates judge variance — the delta is real improvement, not "a different judge was more lenient."

If the review score is < 6/10 or the answer to question 1 is "No": go back to Step 1 (re-diagnose).
If 6-7/10: address the specific issues raised, re-submit to review (do not count as a new round).
If ≥ 8/10: proceed to Step 4.

Critical rule: **the main agent that edited the skill must not score it.** The review agent has no memory of the edit process and only sees the diff + both skill versions + test suite. This isolation is what makes the review trustworthy.

#### Step 4: Decide

```
new_total > old_total → keep (update baseline)
new_total ≤ old_total → git revert HEAD --no-edit, log failure, break (this skill is at its ceiling)
```

Append row to `results.tsv` regardless.

#### Fix Priority

| Priority | Trigger | Action |
|----------|---------|--------|
| P0 | ≥2 test cases FAIL on the same capability | Fix that capability's instructions |
| P1 | Structural dimension ≤ 5 | Fix that structural weakness |
| P1 | Frontmatter missing trigger keywords | Add Chinese + English triggers |
| P2 | Step is vague | Replace with specific parameters and format |
| P2 | Missing error handling | Add "if X fails, then Y" |
| P3 | Paragraph too long or repeated | Split / merge |

---

### Phase 2.5: Exploratory Rewrite

Trigger: a skill hits its ceiling after 1-2 rounds with no improvement, AND structural score is still < 50.

Requires explicit user consent.

```
1. git stash current version.
2. Rewrite SKILL.md from scratch (restructure, not micro-edit).
3. Re-run baseline assessment with fresh judge agent.
4. If rewrite > stashed: adopt. Else: git stash pop.
```

---

### Phase 3: Summary

Display overview table:

```
┌──────────────────────┬────────┬────────┬────────┬──────────────┐
│ Skill                │ Before │ After  │ Δ      │ Pass Rate Δ   │
├──────────────────────┼────────┼────────┼────────┼───────────────┤
│ {name}               │ 63.6   │ 78.2   │ +14.6  │ 62% → 88%     │
└──────────────────────┴────────┴────────┴────────┴───────────────┘
```

Generate visual result card. Template: `templates/result-card.html`. Three themes (random): Warm Swiss / Dark Terminal / Newspaper. Replace data-field placeholders, screenshot at 2x, auto-open PNG.

## Data Files

### results.tsv

Location: `.claude/skills/skill-optimizer/results.tsv`

```tsv
timestamp	commit	skill	old_score	new_score	status	dimension	note	eval_mode	pass_rate
2026-05-09T10:00	baseline	meeting-notes	-	62.5	baseline	-	Initial assessment	dry_run	0.625
2026-05-09T10:15	a1b2c3d	meeting-notes	62.5	78.2	keep	Edge cases	Case #6,#8 FAIL→PASS	dry_run	0.875
```

### test-prompts.json

Location: `{skill-dir}/test-prompts.json`

```json
[
  {
    "id": 1,
    "prompt": "what a real user would say",
    "pass": ["observable condition 1", "observable condition 2"],
    "fail": ["failure mode 1"]
  }
]
```

## Exception Handling

Always notify user before applying a fallback. Never silently skip.

| Condition | Action |
|-----------|--------|
| Not a git repo | Ask user to `git init`. If declined: backup files as `.bak.YYYYMMDD-HHMM` instead of git revert. |
| results.tsv missing | Create with header row. |
| Branch name collision | Append `-2`/`-3`. After 3 failures, switch to existing branch and ask. |
| git revert fails | `git stash`, retry. Still failing: extract file from previous commit manually. |
| MAX_ROUNDS reached (3) | Show remaining gaps. Ask: add one more round / exploratory rewrite / stop. |
| File > 150% original size | Reject commit. Trim redundancy. Re-score. |
| test-prompts.json already exists | Reuse. Ask: reuse / rewrite / append. |
| SKILL.md not found | Terminate that skill. Write `status=error`. Continue to next. |
| No sub-agent for judge | Fall back to dry-run. Main agent evaluates tests directly against criteria. Note `eval_mode=dry_run`. |

## Anti-Patterns

- Changing the skill's core purpose. Only improve how it's written and executed.
- Adding new dependencies or scripts the skill didn't already have.
- Editing multiple unrelated dimensions in one round. One change at a time.
- Growing SKILL.md beyond 150% of original size.
- Using `git reset --hard` instead of `git revert`.
- Re-using the same agent context for editing and scoring. The review agent must be independent.
- Designing test prompts that only cover happy path. Include edge cases and failure modes.
