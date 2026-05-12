# Skill-Optimizer OLD Baseline — write-judge-prompt

**Date:** 2026-05-12 | **Mode:** dry_run

## Component A: Structural (75pts)

Same 7 dimensions, same scoring as darwin (criteria identical):

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
| 1 | PASS | 4 components, specific VIP tone definitions |
| 2 | PASS | Skill explicitly rejects vague criteria |
| 3 | PASS | Prerequisites block unvalidated judges |
| 4 | PASS | Prerequisites: code-based first |
| 5 | PASS | Covers multi-issue failure modes |
| 6 | PASS | Anti-pattern explicitly bans Likert |
| 7 | PASS | Few-shot rules explicitly address data leakage |
| 8 | FAIL | No guidance for handling minimal/vague input |

**Pass rate: 7/8 = 87.5%**
**Effectiveness: 0.875 × 25 = 21.9/25**

## Total: 53.2 + 21.9 = **75.1/100**
