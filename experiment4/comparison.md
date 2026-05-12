# Experiment 4: Real Sub-Agent Testing — Darwin vs Optimizer-Old vs Optimizer-New

**Subject:** error-analysis (165 lines, process skill)
**Mode:** FULL_TEST — real sub-agents used for calibration and review
**Baseline:** 6/8 PASS (75%), cases 5+8 failing

---

## Phase 0.6: Judge Calibration (Real)

First real sub-agent calibration in any experiment:

| Round | Sub-agent | My labels | TPR | TNR | Action |
|-------|-----------|-----------|-----|-----|--------|
| 1 | 8/8 ALL PASS | 2 FAIL (case 5,8) | 100% | **0%** | Too lenient — tighten criteria |
| 2 (tightened) | 6 PASS, 2 FAIL | 6 PASS, 2 FAIL | 100% | **100%** | Calibrated |

**Key finding:** The sub-agent initially gave PASS to every case because the skill's *implied* structure seemed sufficient. Tightening the fail criteria to require *explicit* text flipped cases 5 and 8 to FAIL. This is exactly what calibration is for — without it, a lenient judge would have reported 100% pass rate for a skill that had real gaps.

---

## The Three Optimizations

All three targeted the same two failing test cases. All three would re-evaluate to 100% pass rate. **The difference is in how they fixed it.**

### Darwin (+28 lines)
Added three sections:
- "Before You Start" — 3 verification questions (before Core Process)
- "Handling Problematic Traces" — 4-step guidance (before Anti-Patterns)
- "Handling Vague Requests" — 3 diagnostic questions (before Anti-Patterns)

**Issue:** Two sections overlap with each other and with existing Step 1 content. 28 lines for what could be ~10.

### Optimizer-Old (+6 lines)
Added one section at the END of the document:
- "Edge Cases" — 2 subsections covering incomplete traces + vague requests

**Issue:** The vague-request handling is placed AFTER Step 7, Stopping Criteria, and Trace Sampling Strategies. An agent reading the document linearly would go through the entire error analysis process before reaching the instruction to ask clarifying questions. The gate is at the wrong end of the document.

### Optimizer-New (+13 lines, 2 iterations)
**v1 (rejected):** Same as Optimizer-Old — Edge Cases at end of document.
> **Adversarial reviewer: 5/10** — "Content correct, placement fatally wrong. Gatekeeping at the end of the document is not a gate."

**v2 (committed):**
- Vague-request handling → **Prerequisites** section BEFORE Core Process (functions as a proper gate)
- Incomplete-trace handling → **inline in Step 1** where the problem surfaces
- Hard stop language: "do NOT proceed further until you have answers"
> **Adversarial reviewer: 8/10** — "Placement now correct. Two minor soft spots remain (partial-answer gray zone, all-traces-incomplete degenerate case) but no contradiction or regression."

---

## Comparison

```
                     Darwin        Opt-Old        Opt-New
Lines added           28             6              13
Sections added         3             1               2 (inline)
Iterations             1             1               2 (v1 rejected)
Reviewer score        N/A           N/A            5/10 → 8/10
Gate placement        ✓ (before CP) ✗ (end of doc) ✓ (before CP)
Incomplete traces     Separate sec   Same section    In Step 1 inline
Duplication           Yes (2 overlapping)  No           No
```

---

## What This Experiment Proved (With Real Sub-Agents)

1. **Judge calibration catches lenient judges.** Without the tightened-criteria re-run, we'd have shipped a baseline with 100% pass rate for a skill that objectively had gaps. TNR=0% → fixed to TNR=100%.

2. **Adversarial review catches placement problems the editor misses.** The optimizer-old (and optimizer-new v1) fix was functionally identical to darwin's — tack on a section. The editor (me) only thought about "does this text fix the test case?" The reviewer thought about "does an agent reading this document linearly encounter this text at the right time?" That's a different perspective — and it was correct.

3. **Without adversarial review, optimizer-new = optimizer-old.** v1 was identical to what optimizer-old committed. The only difference is that optimizer-new had a reviewer say "no, fix the placement" before the commit landed.
