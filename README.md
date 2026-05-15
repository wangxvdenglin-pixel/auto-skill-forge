# Auto-Skill-Forge

**Create and optimize Agent Skills the way you train models.**

Inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch). Autonomous experiment loops, applied to the full skill lifecycle. Create → Evaluate → Improve → Judge → Keep only what works. A ratchet that only turns forward.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Agent Skill](https://img.shields.io/badge/Agent%20Skill-Compatible-blueviolet)](https://skills.sh)
[![Skills](https://img.shields.io/badge/skills.sh-Compatible-green)](https://skills.sh)

## Quick Start

```bash
npx skills add <your-username>/auto-skill-forge
```

Then just tell Claude Code what you want:

- **Create a new skill:** "Create a skill that organizes my weekly paper reading notes"
- **Optimize an existing skill:** "Optimize the paper-weekly skill"

## How It Works

Auto-Skill-Forge runs a 6-phase autonomous pipeline:

```
Phase 0  Intent      → Determine mode (create/optimize), interview, draft
Phase 1  Design      → Extract claims → propose checklist → user approves
Phase 2  Calibration → Verify checklist mechanical clarity (TPR/TNR >= 80%)
Phase 3  Baseline    → Initial assessment (create mode: dual anchor vs bare Claude)
Phase 4  Optimize    → Autonomous hill-climb: Diagnose→Edit→Review+Re-score→Keep/Revert
Phase 5  Rewrite     → Exploratory rewrite when stuck (user consent required)
Phase 6  Report      → Summary + package .skill
```

## Evaluation (100 points)

- **Structural (75 pts):** 7 dimensions — Frontmatter, Workflow clarity, Edge cases, Checkpoints, Specificity, Resource integrity, Architecture
- **Effectiveness (25 pts):** 3-6 yes/no checklist questions x 8-12 test inputs, scored by independent sub-agent

## Six Core Principles

| # | Principle |
|:---|:---|
| 01 | **Single editable asset** — One SKILL.md per experiment |
| 02 | **Dual evaluation** — Structure scoring + checklist testing |
| 03 | **Ratchet mechanism** — Score can only go up |
| 04 | **Independent scoring** — Editor never scores |
| 05 | **Human in the loop** — Phases 0-2 user-defined, 3-6 autonomous |
| 06 | **Creation is optimization** — Creating = hill-climbing from scratch |

## License

MIT
