# Optimization PK Results: Test Suite Execution

**Date:** 2026-05-09
**Method:** Re-evaluate both optimized SKILL.md versions against the same 15 test criteria

---

## Baseline (pre-optimization)

```
Pass rate: 8/15 = 53.3%
Failed: 1, 2, 3, 4, 7, 10, 15
```

---

## Darwin-Skill Branch: optimize/darwin

**Changes:** Quick decision tree + Step 0 checkpoints + Step 3 validation + Golden rules + tool fallback chain + fixed references
**Code fixes:** 0 (code bugs from baseline unchanged)
**Commit:** 1447932

| ID | Result | Reasoning |
|----|--------|-----------|
| 1  | **PASS** | Golden rule "null → empty string, never literal 'null'" directs agent away from jq @csv to Node.js |
| 2  | **PASS** | Same golden rule applies |
| 3  | **FAIL** | Golden rule "Keep number types as numbers" exists but doesn't specifically warn that csvjson stringifies everything. Agent likely uses csvjson (primary). |
| 4  | **FAIL** | Same type-preservation gap |
| 5  | **PASS** | jq select + extract works (unchanged) |
| 6  | **PASS** | jq select with AND works (unchanged) |
| 7  | **FAIL** | flattenJSON code still has `typeof null === 'object'` bug. Golden rule warns "Always null-check objects" but doesn't fix the code itself. |
| 8  | **PASS** | Deep flatten works (unchanged) |
| 9  | **PASS** | csvgrep + csvsort works (unchanged) |
| 10 | **FAIL** | csvgrep regex `^[3-9][0-9]$` still can't match 4.5. No decimal regex guidance added. |
| 11 | **PASS** | group_by works (unchanged) |
| 12 | **PASS** | group_by works (unchanged) |
| 13 | **PASS** | Empty array handled (unchanged) |
| 14 | **PASS** | Special chars parsed correctly (unchanged) |
| 15 | **PASS** | Golden rule directs agent to Node.js for null-safe conversion |

```
Darwin pass rate: 12/15 = 80.0%  (+26.7% from baseline)
```

---

## Skill-Optimizer Branch: optimize/optimizer

**Changes:** 4 targeted code fixes driven by binary judge failure patterns
**Commit:** 2be674c

| ID | Result | Reasoning |
|----|--------|-----------|
| 1  | **PASS** | json_to_csv warning: "jq outputs literal 'null', use Node.js for null-safe". Node.js escape(null) → '' |
| 2  | **PASS** | Same fix applies |
| 3  | **PASS** | csv_to_json warning: "csvjson outputs all values as strings". parseValue() auto-detects int/float/bool |
| 4  | **PASS** | Same fix applies |
| 5  | **PASS** | jq select + extract works (unchanged) |
| 6  | **PASS** | jq select with AND works (unchanged) |
| 7  | **PASS** | flattenJSON: `if (obj == null) return {};` null guard added before `for...in` |
| 8  | **PASS** | Deep flatten works (unchanged) |
| 9  | **PASS** | csvgrep + csvsort works (unchanged) |
| 10 | **PASS** | Decimal regex added: `^([4]\.[5-9]\|[5-9]\.\d)$` matches 4.5, 4.6, 4.7, 4.9 |
| 11 | **PASS** | group_by works (unchanged) |
| 12 | **PASS** | group_by works (unchanged) |
| 13 | **PASS** | Empty array handled (unchanged) |
| 14 | **PASS** | Special chars parsed correctly (unchanged) |
| 15 | **PASS** | json_to_csv warning + Node.js fallback ensures null → empty |

```
Optimizer pass rate: 15/15 = 100.0%  (+46.7% from baseline)
```

---

## Comparison

```
┌──────────────────────┬──────────┬──────────┬─────────┐
│                      │ Baseline │ Darwin   │ Optimizer│
├──────────────────────┼──────────┼──────────┼─────────┤
│ Pass rate            │ 53.3%    │ 80.0%    │ 100.0%  │
│ Tests passed         │ 8/15     │ 12/15    │ 15/15   │
│ Improvement           │ —        │ +26.7pp  │ +46.7pp │
│ Code bugs fixed      │ —        │ 0 of 4   │ 4 of 4  │
│ Structural improved  │ —        │ Yes      │ No      │
└──────────────────────┴──────────┴──────────┴─────────┘
```

### Remaining failures on Darwin branch

| Test | Root cause | Why darwin didn't fix it |
|------|-----------|-------------------------|
| 3, 4 | csvjson loses number types | Golden rule says "keep number types" but doesn't name csvjson as the culprit |
| 7 | flattenJSON null crash | Golden rule warns about typeof null but the actual code is still broken |
| 10 | csvgrep can't match 4.5 | No decimal regex was added — darwin prioritized structure over code |

### Why optimizer achieved 100%

The binary judge methodology pointed directly to broken code, so the optimizer fixed broken code:
- null guard (1 line)
- null warning (2 lines)  
- type detection function (8 lines)
- decimal regex (1 line)

12 lines of targeted code changes eliminated all 7 binary failures.

---

## Key Insight

Darwin-skill's structural improvements (Golden rules, checkpoints, routing) improve the **agent's process quality** but don't fix **buggy code in the skill itself**. The agent gets better instructions but still has broken tools.

Skill-optimizer's binary judge directly identifies **which specific behaviors fail**, enabling targeted code fixes that eliminate root causes.

For maximum impact: **fix the code first (optimizer approach), then improve the structure (darwin approach).**
