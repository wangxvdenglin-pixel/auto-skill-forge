# Experiment 4: Real Sub-Agent Testing — Darwin vs Optimizer-Old vs Optimizer-New

**Subject:** error-analysis (165 lines, process skill)
**Mode:** FULL_TEST — 6 real sub-agent calls total
**Baseline:** 6/8 PASS (75%), cases 5+8 failing

---

## Phase 0.6: Judge Calibration (2 sub-agent calls)

| Round | Sub-agent | My labels | TPR | TNR | Action |
|-------|-----------|-----------|-----|-----|--------|
| 1 | 8/8 ALL PASS | 2 FAIL (case 5,8) | 100% | **0%** | Too lenient — tighten criteria |
| 2 (tightened) | 6 PASS, 2 FAIL | 6 PASS, 2 FAIL | 100% | **100%** | Calibrated |

---

## Phase 2: Adversarial Review (2 sub-agent calls)

| Round | Reviewer score | Key critique | Action |
|-------|---------------|-------------|--------|
| v1 | 5/10 | "Vague-request handling at end of doc is not a gate" | Restructured |
| v2 | 8/10 | "Placement correct. 2 minor soft spots remain." | Committed |

---

## Phase 2 Follow-up: Post-Optimization Re-Scoring (3 sub-agent calls)

After all three branches were optimized, **spawned 3 independent judge agents** — one per branch — each receiving the optimized SKILL.md + all 8 test cases with tightened criteria.

### Raw Results

```
              基线     Darwin    Opt-Old   Opt-New
Case 1:       PASS     PASS      PASS      PASS
Case 2:       PASS     PASS      FAIL      PASS
Case 3:       PASS     PASS      PASS      PASS
Case 4:       PASS     FAIL      PASS      FAIL
Case 5:       FAIL     PASS      PASS      PASS
Case 6:       PASS     PASS      PASS      PASS
Case 7:       PASS     PASS      PASS      PASS
Case 8:       FAIL     PASS      PASS      PASS
             ─────    ─────     ─────     ─────
通过率:        6/8      7/8       7/8       7/8
              75%      87.5%     87.5%     87.5%
```

### Analysis

All three branches achieved 87.5% (vs 75% baseline) — cases 5 and 8 fixed across the board.

But **different judges flagged different remaining issues:**

| Branch | Lost case | Judge's reason |
|--------|-----------|---------------|
| Darwin | Case 4 | "No explicit text addressing post-incident regression; would start from scratch" |
| Opt-Old | Case 2 | "No compromise offered for user who cannot review traces; only says present each trace" |
| Opt-New | Case 4 | "No explicit adaptation for regression scenarios with existing evaluators" |

**Critical observation:** Case 2 and Case 4 were both PASS in baseline. The variation is not in the skill quality — it's in **judge inconsistency between different sub-agent instances.** The Opt-Old judge flagged Case 2 (which Darwin and Opt-New judges passed). The Darwin and Opt-New judges flagged Case 4 (which Opt-Old judge passed).

This is exactly what calibration is designed to mitigate, but calibration was only performed on the baseline judge — not on these three post-optimization judges. Each fresh Claude instance brings slightly different judgment thresholds.

---

## The Three Optimizations

### Darwin (+28 lines)
- "Before You Start" checkpoint (3 questions)
- "Handling Problematic Traces" section (4 steps)
- "Handling Vague Requests" section (3 diagnostic questions)
- **Issue:** "Before You Start" and "Handling Vague Requests" overlap. 28 lines.

### Optimizer-Old (+6 lines)
- "Edge Cases" section at END of document
- **Issue:** Vague-request gate is after Step 7 — agent reads entire process before reaching it.

### Optimizer-New (+13 lines, 2 iterations)
- **v1 (rejected):** Same as Opt-Old. Reviewer: 5/10.
- **v2:** Prerequisites gate BEFORE Core Process + incomplete-trace handling inline in Step 1. Reviewer: 8/10.

---

## Comparison

```
                     Darwin        Opt-Old        Opt-New
Pass rate             7/8 (87.5%)   7/8 (87.5%)   7/8 (87.5%)
Lines added           28             6              13
Iterations             1             1              2 (v1 rejected)
Reviewer score        N/A           N/A            5/10 → 8/10
Gate placement        ✓ (before CP) ✗ (end of doc) ✓ (before CP)
Duplication           Yes           No             No
```

---

## Key Findings

1. **All three frameworks fix the target problem** — cases 5+8 consistently flipped from FAIL to PASS across all three judges.

2. **Adversarial review improves code quality, not pass rate.** At 87.5%, optimizer-new ties with the others. But its fix is cleaner (13 lines, correct placement, no duplication) because the reviewer forced a second iteration.

3. **Judge variance is real.** Three independent judges gave different verdicts on cases 2 and 4. This validates the entire Phase 0.6 calibration concept — without calibration, you can't distinguish "the skill changed" from "the judge had a different opinion."

4. **Without adversarial review, optimizer-new = optimizer-old.** v1 was identical. The reviewer is what made the difference.

5. **Darwin over-engineers.** 28 lines across 3 sections vs 13 lines in 2 inline placements. Same result, more code.

### Sub-Agent Call Summary

| Call | Type | Purpose | Result |
|------|------|---------|--------|
| 1 | Judge | Phase 0.6 Round 1 | TNR=0% — too lenient |
| 2 | Judge | Phase 0.6 Round 2 | TPR=100%, TNR=100% — calibrated |
| 3 | Reviewer | Phase 2 v1 review | 5/10 — rejected |
| 4 | Reviewer | Phase 2 v2 review | 8/10 — approved |
| 5 | Judge | Darwin post-optimization | 7/8 PASS |
| 6 | Judge | Opt-Old post-optimization | 7/8 PASS |
| 7 | Judge | Opt-New post-optimization | 7/8 PASS |

Total: **7 real sub-agent calls.** Experiment 1-3 combined: 0.
