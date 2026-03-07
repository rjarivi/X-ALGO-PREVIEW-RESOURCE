# X-Algo Preview Resource

> An open-source, browser-based dashboard for decoding, learning, and understanding the X (formerly Twitter) For You feed algorithm — sourced entirely from the official [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm) repository.

[![License: MIT](https://img.shields.io/badge/License-MIT-00e5ff.svg)](LICENSE)
[![Source](https://img.shields.io/badge/Algorithm%20Source-xai--org%2Fx--algorithm-7c3aed)](https://github.com/xai-org/x-algorithm)
[![No dependencies](https://img.shields.io/badge/Dependencies-None-10b981)](index.html)
[![Single file](https://img.shields.io/badge/Build-Single%20HTML%20File-f59e0b)](index.html)

---

## What Is This?

X-Algo Preview Resource is a zero-dependency, single-file web app that translates the complex Rust/Python logic of the X open-source algorithm into an interactive, educational dashboard.

Every architectural claim in this tool is verified against the [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm) README. No fabricated numbers. No data from the old 2023 `twitter/the-algorithm` repo.

**Live demo:** [https://github.com/rjarivi/X-ALGO-PREVIEW-RESOURCE](https://github.com/rjarivi/X-ALGO-PREVIEW-RESOURCE)

---

## Features

### Score Calculator
A real-time post simulator that estimates how Phoenix (the Grok-based transformer) would evaluate a post based on the repo's documented Weighted Scorer formula:

```
Final Score = Σ (weight_i × P(action_i))
```

- Toggle engagement signals: favorites, reposts, replies, quote posts
- Toggle negative signals: not interested, mute, block, report
- Select media type (video, image, GIF, none)
- Switch between Thunder (in-network) and Phoenix OON pipeline paths
- Live score ring, signal breakdown bars, and Phoenix prediction panel
- All values clearly labeled as simulator estimates — no fake hardcoded weights

### Algo Decoder
An interactive clickable pipeline map of every stage in the For You feed, based directly on the architecture diagram in the `xai-org/x-algorithm` README:

| Stage | Component |
|---|---|
| 1 | Query Hydration (User Action Sequence + User Features) |
| 2 | Thunder — in-memory Kafka-backed in-network retrieval |
| 3 | Phoenix Retrieval — Two-Tower dot-product similarity search |
| 4 | Candidate Hydration (post metadata, author info, video duration) |
| 5 | Pre-Scoring Filters (10 filters, exact names from repo) |
| 6 | Scoring — Phoenix Transformer + Weighted Scorer + Author Diversity Scorer |
| 7 | Selection — sort by score, top-K |
| 8 | Post-Selection Filters (VFFilter, DedupConversationFilter) |

### Strategy Lab
Six what-if scenarios exploring how different post decisions interact with the pipeline:

- Thread → Single Video Post
- Links: No Penalty (myth-busting the old 2023 claim)
- Reply → Original Post
- Text Only → Add Image
- Verification Boost — Myth Check (no multiplier exists in the repo)
- Thunder Cache Timing

Plus a Trend Alignment section covering Phoenix similarity search optimization, in-network vs. out-of-network content strategy, and filter avoidance.

### Learn (Gamified)
9 progressive lessons with XP, level progression, streaks, and quizzes — all content sourced directly from the `xai-org/x-algorithm` README:

| # | Lesson | XP |
|---|---|---|
| L1 | What is the For You Feed? | 15 |
| L2 | Thunder — In-Network Retrieval | 20 |
| L3 | Phoenix — Two-Tower Retrieval | 25 |
| L4 | Candidate Isolation in Ranking | 30 |
| L5 | The Weighted Scorer | 25 |
| L6 | Filters: Pre-Scoring & Post-Selection | 20 |
| L7 | No Hand-Engineered Features | 35 |
| L8 | Composable Pipeline Architecture | 25 |
| L9 | Old Repo vs New Repo — What Changed | 30 |

---

## Accuracy Policy

This project follows a strict accuracy policy:

- ✅ **Only** claims verifiable in [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm) are presented as facts
- ✅ All 10 pre-scoring filter names match exactly from the repo
- ✅ All 15 Phoenix prediction targets listed correctly
- ✅ Score formula, architecture stages, and component names match the README
- ⚠️ Simulator probability values are **estimates only** — the repo publishes no inference outputs
- ⚠️ No hardcoded numeric weights (e.g. "30x likes") — those are from the old 2023 repo
- ⚠️ Fields marked `*` in the UI are simulator extrapolations, not repo facts

See [`ACCURACY.md`](ACCURACY.md) for the full verification report and what was removed.

---

## Getting Started

No build step. No npm install. No server required.

```bash
git clone https://github.com/rjarivi/X-ALGO-PREVIEW-RESOURCE.git
cd X-ALGO-PREVIEW-RESOURCE
open index.html
```

Or just open `index.html` directly in any modern browser.

---

## File Structure

```
X-ALGO-PREVIEW-RESOURCE/
├── index.html          # The entire application (single file)
├── README.md           # This file
├── ACCURACY.md         # Fact-check report against xai-org/x-algorithm
├── CONTRIBUTING.md     # How to contribute
├── LICENSE             # MIT License
└── docs/
    └── ARCHITECTURE.md # Deep-dive on the X algorithm pipeline
```

---

## Open Source

This project is fully open source under the [MIT License](LICENSE).

**You are free to:**
- ✅ Use it for any purpose, commercial or personal
- ✅ Modify and adapt it however you like
- ✅ Redistribute it with or without changes
- ✅ Fork it and build your own version
- ✅ Embed it in other projects

**The only requirement:** Keep the accuracy policy intact — don't re-introduce fabricated weights or stale data from the old 2023 Twitter algorithm.

---

## Contributing

Contributions are welcome! See [`CONTRIBUTING.md`](CONTRIBUTING.md) for guidelines.

The most valuable contributions are:
1. **Accuracy fixes** — if you spot a claim not in the current `xai-org/x-algorithm` repo, open an issue
2. **New learn lessons** — additional lessons covering parts of the repo not yet covered
3. **New scenarios** — additional Strategy Lab what-if scenarios
4. **UI improvements** — accessibility, mobile responsiveness, dark/light mode

---

## Algorithm Source

This tool is based on the official X algorithm open-source release:

**Repository:** [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm)  
**License:** Apache 2.0  
**Components covered:** Home Mixer, Thunder, Phoenix, Candidate Pipeline

> **Note:** This tool is NOT based on the older [`twitter/the-algorithm`](https://github.com/twitter/the-algorithm) repository (2023). That codebase had hardcoded weights, a "Heavy Ranker" model, and different terminology. See [`ACCURACY.md`](ACCURACY.md) for a full comparison.

---

## Disclaimer

X-Algo Preview Resource is an independent educational tool. It is not affiliated with, endorsed by, or connected to X Corp or xAI. All algorithm information is sourced from the public open-source repository linked above.
