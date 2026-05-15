# Auto-Skill-Forge

**Create and optimize Agent Skills the way you train models.**

Inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch). Autonomous experiment loops, applied to the full skill lifecycle. Create → Evaluate → Improve → Judge → Keep only what works. A ratchet that only turns forward.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Agent Skill](https://img.shields.io/badge/Agent%20Skill-Compatible-blueviolet)](https://skills.sh)
[![Skills](https://img.shields.io/badge/skills.sh-Compatible-green)](https://skills.sh)

---

## Quick Start

```bash
npx skills add <your-username>/auto-skill-forge
```

Then just tell Claude Code what you want.

**Create a new skill:**
> "Create a skill that organizes my weekly paper reading notes into a structured digest"

**Optimize an existing skill:**
> "Optimize the paper-weekly skill"

The forge handles the rest — interviews you, designs a custom checklist, measures baseline, and hill-climbs autonomously. You only participate at key checkpoints to approve the test design.

---

## Why This Exists

Agent skill ecosystems are expanding fast. Claude Code, Codex, OpenClaw, Trae, and more all support the SKILL.md format. 10 skills you can maintain by hand. 60+ skills need a system. And it's not just about improving existing ones — **generating a high-quality skill from a one-sentence user request** demands automation too.

Traditional review is purely structural: does the frontmatter look right? Are steps numbered? Do file paths exist? But a perfectly formatted skill can still produce terrible output.

Auto-Skill-Forge covers **create→evaluate→optimize→package** — testing effectiveness with a checklist, enforcing improvement with a ratchet, catching trigger blind spots with semantic adversarial review.

---

## Core Loop

```
Phase 0   Intent      → Determine mode (create/optimize), interview, draft, intent expansion
Phase 1   Design      → Extract claims → propose checklist → select test inputs → user approves
Phase 2   Calibration → Verify checklist mechanical clarity (TPR/TNR ≥ 80%)
Phase 3   Baseline    → Initial assessment (create mode: dual anchor vs bare Claude)
Phase 4   Optimize    → Autonomous hill-climb: Diagnose→Edit→Review+Re-score→Keep/Revert (max 3)
Phase 5   Rewrite     → Exploratory rewrite when stuck (user consent required)
Phase 6   Report      → Summary + package .skill
```

---

## Six Core Principles

| # | Principle | Details |
|:---|:---|:---|
| 01 | **Single editable asset** | One SKILL.md per experiment. One change, one measurement, one decision |
| 02 | **Dual evaluation** | Structure scoring (7 dimensions, 75 pts) + checklist testing (25 pts) |
| 03 | **Ratchet mechanism** | Score can only go up. Regressions are auto-reverted |
| 04 | **Independent scoring** | The agent that edits is never the agent that scores |
| 05 | **Human in the loop** | Phases 0-2: user defines boundaries and standards. Phases 3-6: fully autonomous |
| 06 | **Creation is optimization** | Creating a new skill = hill-climbing from scratch. Same pipeline, same ratchet |

---

## 7-Dimension Evaluation Rubric

Total: 100 points. Structure (75) + Effectiveness (25).

| # | Dimension | Weight | What it checks |
|---|-----------|--------|----------------|
| 1 | Frontmatter quality | 8 | Name/description correct, trigger keywords complete |
| 2 | Workflow clarity | 15 | Numbered executable steps with explicit input/output |
| 3 | Edge case coverage | 10 | Failure scenarios, fallback paths |
| 4 | Checkpoint design | 7 | User confirmation before critical actions |
| 5 | Instruction specificity | 15 | No vague directives. Concrete parameters, formats, examples |
| 6 | Resource integrity | 5 | Referenced files, scripts, paths actually exist |
| 7 | Overall architecture | 15 | Clear hierarchy, no redundancy, no gaps |

Effectiveness: 3-6 yes/no checklist questions × 8-12 test inputs. Independent sub-agent judges every item. pass_rate × 25.

---

## The Optimization Cycle

**Phase 4 is the heart:**

1. Find the weakest dimension or checklist question (including frontmatter trigger blind spots)
2. Generate one targeted improvement
3. Edit SKILL.md, git commit
4. Independent sub-agent plays two roles — critic (reviews change + semantic trigger blind-spot check) and judge (scores both versions with same checklist)
5. Score up → keep. Score down → git revert
6. Loop until ceiling or 3 rounds max

---

## Checklist: Lowering the Barrier

You don't need to understand "dimension sampling" or "tuple generation."

The agent reads the skill, extracts every claimed capability, then proactively proposes 3-6 yes/no checklist questions. Your job: nod, shake your head, or add one. Like grading essays with a checklist — each item is a clear yes or no, not a fuzzy 1-10 scale.

---

## Creation Mode

Give the agent a one-sentence requirement (e.g., "make a skill for weekly paper reading digests"). The agent interviews → writes a draft → **Intent Expansion** (proactively asks "what if the user only has a DOI? what about encoding errors?") — preventing self-grading bias. Then the same pipeline takes over.

After creation, a bare-Claude baseline quantifies the skill's absolute value: "with this skill, success rate went from 50% → 85%."

## The Ratchet

Scores can only go up. Failed experiments are cleanly reverted. No regressions accumulate.

```
Round 0:  Baseline 72
Round 1:  78 → Keep ✅
Round 2:  75 → Revert ❌ (below 78, auto-rollback)
Round 3:  84 → Keep ✅
```

---

## Design Lineage

Auto-Skill-Forge builds on ideas from three projects, each contributing a different piece to the whole.

**From [autoresearch](https://github.com/karpathy/autoresearch):** The ratchet — run an experiment, measure the result, keep it only if the metric improved. We adopted this loop as the backbone of Phase 4, but replaced "code + loss function" with "skill edit + checklist pass rate" as the measurable unit of progress.

**From [Darwin-Skill](https://github.com/alchaincyf/darwin-skill):** The original vision of applying evolutionary pressure to skill improvement. We inherited the baseline→optimize→rewrite skeleton, then rebuilt the evaluation system from subjective 1-10 scoring into a mechanical binary checklist judged by independent agents.

What makes Auto-Skill-Forge distinct: the combination of checklist-based evaluation (not assertions, not subjective scores), same-judge scoring to eliminate between-instance variance, adversarial review integrated directly into the optimization step, and Intent Expansion to prevent self-grading bias in creation mode.

---

## License

MIT

