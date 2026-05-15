# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

**Auto-Skill-Forge** — an autonomous Agent Skill lifecycle engine. It creates new Claude Code skills from user requirements OR optimizes existing SKILL.md files. The core mechanism: evaluate → improve → judge → keep only improvements that move the score up (ratchet). Directly inspired by Karpathy's [autoresearch](https://github.com/karpathy/autoresearch).

The skill itself lives at `.claude/skills/auto-skill-forge/SKILL-v4.md`. Supporting scripts are in `scripts/`.

## Project layout

```
.claude/skills/auto-skill-forge/   ← the forge skill itself (SKILL-v4.md is canonical)
  scripts/                          ← packaging tools (package_skill.py, quick_validate.py)
.claude/skills/*/                   ← other skills (targets for optimization, or reference)
experiment/ → experiment6/          ← historical experiments (each tests one skill pipeline run)
改进思路.md                          ← full engineering retrospective (design evolution)
新灵感.txt                           ← external inspiration / design notes
```

## Evaluation system (100 points total)

**Component A — Structural (75 pts):** 7 dimensions scored 1-10, weighted, summed, divided by 10.
Dimensions: Frontmatter quality (8), Workflow clarity (15), Edge case coverage (10), Checkpoint design (7), Instruction specificity (15), Resource integrity (5), Overall architecture (15).

**Component B — Effectiveness (25 pts):** 3-6 yes/no checklist questions × 8-12 test inputs, judged by independent sub-agent. `pass_rate × 25`.

**Total = structural + effectiveness.** Improvement requires strict `>` (not `≥`). Round to 1 decimal.

## Core workflow (6 phases)

```
Phase 0  Intent      → determine create vs optimize mode
Phase 1  Design      → extract claims → propose checklist → user approves
Phase 2  Calibration → verify checklist mechanical clarity (TPR/TNR ≥ 80%)
Phase 3  Baseline    → initial scoring (create mode: dual anchor vs bare Claude)
Phase 4  Optimize    → autonomous hill-climbing: Diagnose→Edit→Review+Re-score→Keep/Revert (max 3 rounds)
Phase 5  Rewrite     → exploratory rewrite when stuck (user consent required)
Phase 6  Report      → summary table + package .skill
```

## Critical design rules (do NOT violate)

1. **The agent that edits must never score.** In Phase 4 Step 3, spawn a fresh independent sub-agent for adversarial review + re-scoring. Give it only: original SKILL.md, optimized SKILL.md, git diff, target problem, and test-checklist.json. Do NOT give it your diagnosis or reasoning.

2. **Same-judge scoring.** In each Phase 4 round, the SAME review agent scores BOTH baseline and optimized versions in one call. This eliminates between-instance judge variance — the delta is real improvement, not "a different judge was more lenient."

3. **Ratchet.** `new_total > old_total` → keep. `new_total ≤ old_total` → `git revert HEAD --no-edit`, log failure, break (ceiling reached).

4. **One change per round.** Edit only the single weakest dimension or capability. Multiple unrelated changes make attribution impossible.

5. **Intent Expansion in create mode (Phase 0 Step A4).** Before generating a checklist from a draft, the agent MUST proactively challenge the draft's boundaries and ask the user to confirm which edge cases to include. Without this, the agent grades its own homework and the draft scores artificially high.

6. **Phase 1 checklist questions must be mechanically yes/no.** "Is the output good?" is banned. "Does the output contain column headers?" is correct. 3-6 questions max.

## Data files

| File | Location | Purpose |
|------|----------|---------|
| `results.tsv` | `.claude/skills/auto-skill-forge/` | Optimization history (timestamp, commit, skill, old/new scores, pass_rate) |
| `test-checklist.json` | `{skill-dir}/` | Checklist questions + test inputs for a skill |
| `changelog.md` | `{skill-dir}/` | Per-round diagnosis → change → review → decision log |

## Git conventions

- Create branch `auto-optimize/YYYYMMDD-HHMM` for each optimization run
- Commit per edit with message: `"optimize {skill}: {brief summary}"`
- Never use `git reset --hard` as a revert mechanism — use `git revert`
- File size guard: reject commits that grow SKILL.md beyond 150% of original size

## Experiment directories

Each `experiment{N}/` is a self-contained pipeline run on a target skill. Typical structure:
```
experiment6/
  paper-weekly/              ← the target skill
    SKILL.md                 ← the skill under test
    test-checklist.json      ← generated in Phase 1
    changelog.md             ← optimization round history
    trigger-eval.json        ← (deprecated, from removed Phase 6)
  v3-results/ or v2-results/ ← judge outputs, baseline/optimization data
  comparison.md              ← cross-version comparison report
```

## Scripts

Only 2 scripts remain (Phase 6 was removed in v4):
- `package_skill.py` — creates `.skill` distributable (requires PyYAML). Run from skill directory: `python -m scripts.package_skill <path>`
- `quick_validate.py` — validates SKILL.md format (dependency of package_skill.py)

Both live at `.claude/skills/auto-skill-forge/scripts/` and must be run from that directory.
