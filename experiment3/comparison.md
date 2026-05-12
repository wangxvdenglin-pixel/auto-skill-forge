# Experiment 3: Darwin vs Optimizer-Old vs Optimizer-New

**Subject:** write-judge-prompt (145 lines, process skill)
**Baseline pass rate:** 7/8 (87.5%) — only test 8 (minimal input) failing
**Optimization target:** test 8 — handling "make me a judge" with no details

---

## What Each Framework Did

| | Darwin | Optimizer-Old | Optimizer-New |
|---|---|---|---|
| **Diagnosis** | dim4 (checkpoints, 2.8/7) weakest | test 8 FAIL → direct target | test 8 FAIL → direct target |
| **Fix approach** | Add checkpoints + edge case sections | Add minimal-input handling section | Integrate into Prerequisites |
| **Adversarial review** | No | No | **Yes — caught duplication, forced v2** |
| **Lines changed** | +18 | +12 | +7 |
| **Iterations** | 1 | 1 | **2 (v1 rejected by reviewer)** |

---

## Side-by-Side: What Each Branch Added

### Darwin (+18 lines)
Added TWO sections:
1. "Before You Start" — 4 checkpoint questions in the workflow
2. "Handling Insufficient Input" — 4-step handling for vague requests

**Problem:** These two sections overlap. Both mention asking about failure mode, both mention labeled data. Maintenance risk: update one, forget the other.

### Optimizer-Old (+12 lines)
Added ONE section:
- "Handling Minimal or Vague Requests" — 4-step diagnostic + anti-fabrication

**Problem:** Standalone section, not integrated with Prerequisites. The labeled-data requirement now appears in two unconnected places.

### Optimizer-New (+7 lines)
Two iterations:

**v1 (rejected):** Same standalone section as Optimizer-Old.
> Reviewer: 7/10 — "Fixes test 8 but duplicates Prerequisites content. Integrate instead of adding a new section."

**v2 (committed):** Removed standalone section. Embedded 3 diagnostic questions + anti-fabrication rule + template directly into Prerequisites. The gatekeeping is now at the earliest possible point in the skill — before the agent even starts.

---

## Final Scores

All three branches pass 8/8 (100%). The difference is in the **quality of the fix**, not the pass rate:

```
                Baseline    Darwin    Opt-Old   Opt-New
Pass rate        87.5%       100%      100%      100%
Lines added       —          +18       +12       +7
Duplicate content —          Yes       Minor     No
Iterations        —          1         1         2 (reviewer forced v2)
```

| Quality metric | Darwin | Opt-Old | Opt-New |
|---------------|--------|---------|---------|
| Fixes the test? | ✓ | ✓ | ✓ |
| No new duplicates? | ✗ (two overlapping sections) | △ (standalone, not integrated) | ✓ |
| Minimal diff? | ✗ (18 lines) | △ (12 lines) | ✓ (7 lines) |
| Gatekeeping at right point? | △ (mid-flow) | △ (end of doc) | ✓ (prerequisites, earliest point) |

---

## What the Adversarial Review Caught

The optimizer-new v1 diff was identical to optimizer-old. Without adversarial review it would have been committed as-is — a 12-line standalone section with implicit duplication.

The reviewer spotted:
1. **Duplication with Prerequisites** — "labeled data" and "code-based check" appear in both places
2. **Wrong placement** — gatekeeping should happen upfront, not at the end of the document
3. **Suggested integration** — embed into Prerequisites instead of adding a new section

This is exactly what ARIS's auto-review-loop is designed to catch: the editor (focused on "fixing test 8") doesn't notice the architectural problem (duplication + wrong placement). The reviewer (focused on "is this change clean?") catches it.

**Without adversarial review, you get optimizer-old quality. With it, you get optimizer-new quality.**

---

## Conclusion

In this experiment, all three reached the same pass rate (100%), but the **quality of the solution** differed significantly:

- **Darwin** over-engineered (two overlapping sections, 18 lines)
- **Optimizer-Old** was functional but not clean (standalone section, 12 lines)
- **Optimizer-New** produced the best result (integrated, 7 lines, correct placement) because the adversarial reviewer forced a second iteration that eliminated duplication and improved structure

The adversarial review didn't change the score — it changed the **code quality**. A less measurable but equally important outcome.
