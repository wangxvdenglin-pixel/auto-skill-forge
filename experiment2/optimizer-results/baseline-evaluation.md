# Skill-Optimizer Baseline — meeting-notes

**Date:** 2026-05-09 | **Mode:** dry_run

## Component A: Structural (75pts)

| # | Dimension | Score | Weighted |
|---|-----------|-------|----------|
| 1 | Frontmatter | 3/10 | 2.4 |
| 2 | Workflow | 7/10 | 10.5 |
| 3 | Edge cases | 5/10 | 5.0 |
| 4 | Checkpoints | 3/10 | 2.1 |
| 5 | Specificity | 7/10 | 10.5 |
| 6 | Resources | 4/10 | 2.0 |
| 7 | Architecture | 6/10 | 9.0 |
| **Total** | | | **41.5/75** |

## Component B: Binary Judge (25pts)

For a template skill, criteria focus on: Does the SKILL.md provide sufficient guidance to produce structurally-correct output for each test case?

| ID | Scenario | PASS/FAIL | Reasoning |
|----|----------|-----------|-----------|
| 1 | Standard notes | PASS | Standard template + How to Use covers this |
| 2 | Messy input | PASS | Claims to handle "messy handwritten notes" |
| 3 | Action items only | PASS | Customization: "Action items only" |
| 4 | Decision tracking | PASS | Decision identification guidelines exist |
| 5 | Client meeting | PASS | Client Meeting Notes template |
| 6 | Executive summary | FAIL | Customization mentions it but no template or specific guidance |
| 7 | Brainstorm session | FAIL | No brainstorm guidance or template |
| 8 | Minimal input | FAIL | No guidance for minimal input; may over-elaborate |

**pass_rate = 5/8 = 62.5%**
**Effectiveness = 0.625 × 25 = 15.6/25**

## Total: 41.5 + 15.6 = **57.1/100**
