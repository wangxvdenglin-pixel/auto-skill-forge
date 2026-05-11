# Experiment 2 PK Results — meeting-notes

**Subject:** meeting-notes (template/process skill, zero code)
**Test suite:** 8 prompts covering standard, messy, actions-only, decisions, client, executive summary, brainstorm, minimal input

---

## Baseline → Darwin Branch

**Changes:** Fixed frontmatter (empty→proper, 48→6 lines), template selection table, checkpoints, edge cases section (6 scenarios including minimal input)

```
Test  1: Standard notes       PASS (unchanged)
Test  2: Messy input          PASS (unchanged)
Test  3: Action items only    PASS (unchanged)  
Test  4: Decision tracking    PASS (unchanged)
Test  5: Client meeting       PASS (unchanged)
Test  6: Executive summary    PASS — Customization option covers this; edge cases guidance helps
Test  7: Brainstorm session   FAIL — No brainstorm template or guidance added
Test  8: Minimal input        PASS — Edge Cases explicitly handles "very short input"
                              ---
                              7/8 = 87.5%  (+25.0pp)
```

## Baseline → Optimizer Branch

**Changes:** Executive Summary template, Brainstorm template (with themes + votes), Handling Minimal Input section

```
Test  1: Standard notes       PASS (unchanged)
Test  2: Messy input          PASS (unchanged)
Test  3: Action items only    PASS (unchanged)
Test  4: Decision tracking    PASS (unchanged)
Test  5: Client meeting       PASS (unchanged)
Test  6: Executive summary    PASS — Dedicated template with specific format
Test  7: Brainstorm session   PASS — Dedicated template with theme grouping + vote table
Test  8: Minimal input        PASS — Explicit anti-fabrication rules + clarification prompts
                              ---
                              8/8 = 100.0%  (+37.5pp)
```

---

## Comparison

```
              Baseline    Darwin      Optimizer
Pass rate      62.5%  →   87.5%   →   100.0%
Tests passed   5/8         7/8          8/8
Improvement     —         +25.0pp      +37.5pp
```

### Why the gap is smaller this time

In experiment 1 (code skill), the gap was 20pp (80% vs 100%) because darwin's structural fixes fundamentally couldn't fix code bugs.

Here, the gap is 12.5pp (87.5% vs 100%) because both approaches are modifying text content — just from different angles:

- Darwin's edge cases section accidentally covered test 8 (minimal input) — a structural improvement that happened to fix a binary failure
- Optimizer's targeted template additions directly fixed the 3 binary failures

The remaining gap (test 7: brainstorm) exists because darwin's methodology prioritized structure over content gaps, while optimizer's binary judge directly pointed at the missing content.

### Key insight

For **pure content/template skills** (no code), the gap between the two approaches narrows significantly. Darwin's structural improvements can sometimes fix binary failures as a side effect, and optimizer's targeted content additions are the natural fix for missing templates. The difference becomes one of **prioritization strategy**, not fundamental capability difference.
