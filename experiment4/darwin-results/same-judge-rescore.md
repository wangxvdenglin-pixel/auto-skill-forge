# Darwin Branch — Same-Judge Re-Score

**Method:** Same judge agent scores BOTH baseline and optimized in one call

## Results

```
              基线    优化 (Darwin)
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
- Case 5: Baseline says "Capture the full trace" (aspirational) but has zero procedural text for handling incomplete or messy trace data.
- Case 8: Baseline jumps straight to "Step 1: Collect Traces," assuming user already has data. No text for handling vague/underspecified requests.

**Both fixed in Darwin** via "Handling Problematic Traces" + "Handling Vague Requests" + "Before You Start" sections.
