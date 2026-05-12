# Darwin-Skill Baseline — write-judge-prompt

**Date:** 2026-05-12 | **Mode:** dry_run

## Structural (dim1-6: 60pts)

| # | Dimension | Score | Weighted | Reasoning |
|---|-----------|-------|----------|-----------|
| 1 | Frontmatter | 8/10 | 6.4/8 | name hyphenated. description covers what+when+when NOT to use. Missing Chinese triggers. |
| 2 | Workflow | 8/10 | 12.0/15 | 4 components numbered. Each has template + rules. Prerequisites section clear. No decision tree for different user scenarios. |
| 3 | Edge cases | 6/10 | 6.0/10 | Anti-patterns cover 7 failure modes. But no explicit "if X, then Y" fallback paths. No guidance for minimal/vague input. |
| 4 | Checkpoints | 4/10 | 2.8/7 | References validate-evaluator as next step. Asks for labeled data upfront. No explicit "confirm before X" checkpoints in the flow. |
| 5 | Specificity | 8/10 | 12.0/15 | Very concrete: exact templates, specific examples (luxury buyer, first-time homebuyer), explicit anti-patterns. |
| 6 | Resources | 7/10 | 3.5/5 | References validate-evaluator skill. References Instructor/Outlines libraries. No broken paths. |

**Structural subtotal: 42.7/60**

## Effectiveness (dim7-8: 40pts)

### Dim 7: Architecture (w=15) — 7/10 → 10.5/15
Clear Prerequisites → Components → Model Selection → Anti-Patterns flow. Minor: "Choosing What to Pass" section feels tacked on rather than integrated.

### Dim 8: Observed Performance (w=25, dry_run) — 7/10 → 17.5/25

| ID | Scenario | Result | Basis |
|----|----------|--------|-------|
| 1 | Standard VIP tone | PASS | All 4 components directly match skill structure |
| 2 | Vague "helpful" | PASS | Anti-pattern #1 explicitly rejects vague criteria |
| 3 | No labels | PASS | Prerequisites state 20 pass + 20 fail required upfront |
| 4 | Code-checkable | PASS | Prerequisites: "exhaust code-based options first" with example |
| 5 | Legal document | PASS | 4 components + borderline examples cover this |
| 6 | Likert scale | PASS | Anti-pattern explicitly rejects Likert, suggests binary judges |
| 7 | Data leakage | PASS | Few-shot rules warn about training split vs dev/test |
| 8 | Minimal input | FAIL | No explicit guidance for handling "basically no input" requests |

**Pass rate: 7/8 = 87.5%**

## Total: 42.7 + 10.5 + 17.5 = **70.7/100**
