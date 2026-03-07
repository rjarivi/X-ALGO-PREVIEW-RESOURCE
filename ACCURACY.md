# Accuracy Report

**Source verified against:** [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm)  
**Last audited:** March 2026  
**Score:** 38/38 core architectural claims confirmed ✅

---

## What This Document Is

Every factual claim in X-Algo Preview Resource has been verified word-for-word against the `xai-org/x-algorithm` README. This document is the transparency record of that process — what was confirmed, what was removed, and why.

---

## ✅ Confirmed Accurate Claims (38/38)

### Thunder
- Consumes post create/delete events from **Kafka** ✅
- Maintains **per-user stores** for: original posts, replies/reposts, and video posts ✅
- Serves **in-network** candidates from accounts the requesting user follows ✅
- Enables **sub-millisecond lookups** without hitting an external database ✅
- Automatically trims posts older than the **retention period** ✅

### Phoenix — Retrieval
- Uses a **Two-Tower model** for out-of-network retrieval ✅
- **User Tower** encodes user features and engagement history ✅
- **Candidate Tower** encodes all posts into embeddings ✅
- Top-K retrieved via **dot product similarity** ✅

### Phoenix — Ranking
- Uses a **Grok-based transformer** ported from [Grok-1](https://github.com/xai-org/grok-1) ✅
- **Candidate isolation**: candidates cannot attend to each other during inference ✅
- Special **attention masking** enforces candidate isolation ✅
- Scores are consistent and **cacheable** regardless of batch composition ✅

### Scoring
- Formula: `Final Score = Σ (weight_i × P(action_i))` ✅
- All **15 prediction targets** listed correctly (see below) ✅
- **Positive actions** (favorites, reposts, replies, quotes, clicks, video_view, photo_expand, shares, dwell, follow_author) have positive weights ✅
- **Negative actions** (not_interested, block_author, mute_author, report) have negative weights ✅
- **Author Diversity Scorer** attenuates repeated author scores ✅
- **OON Scorer** adjusts scores for out-of-network content ✅

### 15 Phoenix Prediction Targets (all confirmed in README)
```
P(favorite)         P(reply)           P(repost)
P(quote)            P(click)           P(profile_click)
P(video_view)       P(photo_expand)    P(share)
P(dwell)            P(follow_author)   P(not_interested)
P(block_author)     P(mute_author)     P(report)
```

### Pre-Scoring Filters (all 10, exact names from repo)
```
DropDuplicatesFilter            CoreDataHydrationFilter
AgeFilter                       SelfpostFilter
RepostDeduplicationFilter       IneligibleSubscriptionFilter
PreviouslySeenPostsFilter       PreviouslyServedPostsFilter
MutedKeywordFilter              AuthorSocialgraphFilter
```

### Post-Selection Filters (both confirmed)
```
VFFilter                        DedupConversationFilter
```

### Architecture
- **Home Mixer** is the orchestration layer ✅
- **ScoredPostsService** is a gRPC endpoint ✅
- **CandidatePipeline** framework defines traits: Source, Hydrator, Filter, Scorer, Selector, SideEffect ✅
- **Hash-based embeddings** used for both retrieval and ranking ✅
- Source correctly labeled as **xai-org/x-algorithm** ✅
- Algorithm sourced from **Apache 2.0** licensed repository ✅

### Key Design Decisions (all 5 from README confirmed)
1. No hand-engineered features — transformer learns from engagement sequences ✅
2. Candidate isolation in ranking ✅
3. Hash-based embeddings ✅
4. Multi-action prediction (not a single relevance score) ✅
5. Composable pipeline architecture ✅

---

## ⚠️ Simulator Estimates (not from repo, clearly labeled in UI)

These are **extrapolations used in the Score Calculator simulator only**. They are clearly labeled with `*` or disclaimer text in the UI. They do not represent factual claims about the algorithm.

| Element | Why it's an estimate |
|---|---|
| Probability values (e.g. P(favorite) = 0.3) | Repo publishes no inference outputs or probability values |
| Media type bonus values | No media weights published in repo |
| P(video_view) triggered by video selection | Action type confirmed; probability value is estimated |
| Relative engagement signal weights | Repo confirms positive/negative direction only, not magnitudes |

---

## ❌ What Was Removed — Old Repo Data

The following claims were present in earlier versions of this tool and have been removed because they originated from the **old 2023 `twitter/the-algorithm` repo**, not the current `xai-org/x-algorithm` repo.

### Removed: Fabricated Numeric Weights
| Claim | Status | Source |
|---|---|---|
| "Favorites = 30x more than Replies" | ❌ Removed | Old `twitter/the-algorithm` only |
| "Reports = −369x penalty" | ❌ Removed | Old `twitter/the-algorithm` only |
| "Not Interested = −74x" | ❌ Removed | Old `twitter/the-algorithm` only |
| "Block Author = −36x" | ❌ Removed | Old `twitter/the-algorithm` only |
| "48M parameters" (model size) | ❌ Removed | Old repo / not in new README |
| "500M posts → 1,500 candidates" | ❌ Removed | Old repo / not in new README |
| "50%/50% in-network/out-of-network split" | ❌ Removed | Not stated in new repo |
| "1.5x Blue Check multiplier" | ❌ Removed | Not in new repo |
| "0.7x OON penalty" | ❌ Removed | Not in new repo |
| "−40% link penalty" | ❌ Removed | Not in new repo |
| "0.85^n diversity decay" | ❌ Removed | Not in new repo |
| "15% Global Discovery" | ❌ Removed | Not in new repo |

### Removed: Stale Terminology
| Old Term | Correct Term |
|---|---|
| Heavy Ranker | Phoenix / Grok-based transformer |
| tweet | post |
| retweet | repost |
| params.rs | *(file does not exist in new repo)* |
| weighted_scorer.rs | *(file does not exist in new repo)* |
| TweepCred | *(concept removed from new repo)* |
| Social Graph filters | AuthorSocialgraphFilter |

### Removed: Concepts Not in New Repo
| Concept | Reason |
|---|---|
| Ads / promoted content mixing stage | Not described in `xai-org/x-algorithm` |
| "Who to follow" blending | Not described in `xai-org/x-algorithm` |
| Subscription Boost / verification multiplier | Not described in `xai-org/x-algorithm` |
| Freshness decay curves | Not described in `xai-org/x-algorithm` |
| "Mixing" as a 4th pipeline stage | New repo ends at post-selection filters, no mixing stage |

---

## Accuracy Maintenance

If you find a claim in this tool that is not verifiable in the current `xai-org/x-algorithm` README:

1. Open an issue on GitHub with the label `accuracy`
2. Include: the claim text, where it appears in the UI, and what the repo actually says (or that it's absent)
3. PRs with corrections are welcome — see [`CONTRIBUTING.md`](CONTRIBUTING.md)
