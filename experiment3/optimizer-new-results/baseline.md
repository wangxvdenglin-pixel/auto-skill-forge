# Skill-Optimizer NEW Baseline — write-judge-prompt

**Date:** 2026-05-12 | **Mode:** dry_run, calibration skipped

## Component A: Structural (75pts)

| # | Dimension | Score | Weighted |
|---|-----------|-------|----------|
| 1 | Frontmatter | 8/10 | 6.4 |
| 2 | Workflow | 8/10 | 12.0 |
| 3 | Edge cases | 6/10 | 6.0 |
| 4 | Checkpoints | 4/10 | 2.8 |
| 5 | Specificity | 8/10 | 12.0 |
| 6 | Resources | 7/10 | 3.5 |
| 7 | Architecture | 7/10 | 10.5 |

**Structural: 53.2/75**

## Component B: Binary Judge (25pts)

| ID | Result | Reasoning |
|----|--------|-----------|
| 1 | PASS | All 4 components present for VIP tone |
| 2 | PASS | Anti-pattern #1 blocks vague criteria |
| 3 | PASS | Prerequisites enforce labeled data requirement |
| 4 | PASS | Prerequisites: code-based check first |
| 5 | PASS | Components cover multi-failure-mode with borderline |
| 6 | PASS | Anti-pattern explicitly bans Likert scales |
| 7 | PASS | Data leakage warning in few-shot rules |
| 8 | FAIL | No explicit minimal-input handling |

**Pass rate: 7/8 = 87.5%**
**Effectiveness: 0.875 × 25 = 21.9/25**

## Total: 53.2 + 21.9 = **75.1/100**
