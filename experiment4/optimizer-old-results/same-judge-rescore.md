# Optimizer-Old Branch — Same-Judge Re-Score

**Method:** Same judge agent scores BOTH baseline and optimized in one call

## Results

```
              基线    优化 (Opt-Old)
Case 1:       PASS    PASS
Case 2:       PASS    PASS
Case 3:       PASS    PASS
Case 4:       PASS    PASS
Case 5:       FAIL    PASS  ✓ fixed
Case 6:       PASS    PASS
Case 7:       PASS    FAIL  ✗ regression (judge inconsistency)
Case 8:       FAIL    PASS  ✓ fixed
             ─────   ─────
通过率:        6/8     7/8
             75%     87.5%

Delta: +1
```

## Judge's Reasoning

**Case 5+8 fixed:** "Edge Cases" section addresses incomplete traces and vague requests, flipping both from FAIL to PASS.

**Case 7 regression:** Judge flagged case 7 as FAIL in optimized version despite case 7 being PASS in baseline. The Opt-Old optimization only added an "Edge Cases" section about incomplete traces and vague requests — nothing touched case 7's domain (stopping criteria, failure prioritization). This is likely an instance of intra-call judge inconsistency, not a real regression. However, the fact that this judge found the document structure confusing (Edge Cases placed at end of document, after Stopping Criteria and Trace Sampling) may have contributed — the gatekeeping text at the wrong location disrupted the judge's evaluation flow.
