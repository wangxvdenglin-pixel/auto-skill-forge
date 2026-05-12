# Optimizer-New Branch — Same-Judge Re-Score

**Method:** Same judge agent scores BOTH baseline and optimized in one call

## Results

```
              基线    优化 (Opt-New)
Case 1:       PASS    PASS
Case 2:       PASS    PASS
Case 3:       PASS    PASS
Case 4:       PASS    PASS
Case 5:       FAIL    PASS  ✓ fixed
Case 6:       PASS    PASS
Case 7:       PASS    PASS
Case 8:       FAIL    PASS  ✓ fixed
             ─────   ─────
通过率:        6/8     8/8
             75%     100%

Delta: +2
```

## Judge's Reasoning

**Baseline failures (cases 5+8):**
- Case 5: Baseline treats "Capture the full trace" as aspirational goal with no procedural handling for incomplete data.
- Case 8: Baseline has no prerequisite gate or vagueness handling. Agent would give generic advice or assume user is ready for Step 1.

**Both fixed in Opt-New:**
- Prerequisites section (BEFORE Core Process) handles vague requests with 3 diagnostic questions + hard stop ("do NOT proceed further until you have answers")
- Step 1 inline text addresses incomplete/messy trace data with concrete warnings and data subset guidance

**Placement matters:** Unlike Opt-Old (Edge Cases at document end), Opt-New's prerequisites gate at document beginning + inline Step 1 handling produced clean +2 with no regressions. The adversarial reviewer (8/10) specifically caught and fixed the placement issue that may have caused Opt-Old's case 7 regression.
