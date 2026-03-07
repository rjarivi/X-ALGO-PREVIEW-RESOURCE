# X Algorithm Architecture — Deep Dive

> All content in this document is sourced from [`xai-org/x-algorithm`](https://github.com/xai-org/x-algorithm). No claims are made beyond what is documented in that repository's README.

---

## Overview

The For You feed algorithm retrieves, ranks, and filters posts from two sources:

1. **In-Network (Thunder)** — Posts from accounts the requesting user follows
2. **Out-of-Network (Phoenix Retrieval)** — Posts discovered from a global corpus via ML similarity search

Both sources are combined and ranked together using **Phoenix**, a Grok-based transformer model that predicts engagement probabilities for each post. The final score is a weighted combination of these predictions.

> The system has eliminated every single hand-engineered feature and most heuristics. The Grok-based transformer does all the heavy lifting by understanding engagement history.

---

## System Architecture

```
FOR YOU FEED REQUEST
        │
        ▼
┌─────────────────┐
│   HOME MIXER    │  ← Orchestration layer (CandidatePipeline framework)
│                 │     exposes: ScoredPostsService (gRPC)
└────────┬────────┘
         │
         ▼
    QUERY HYDRATION
    ├── User Action Sequence (engagement history)
    └── User Features (following list, preferences)
         │
         ▼
   CANDIDATE SOURCES  (run in parallel)
   ├── THUNDER          ← In-network posts
   └── PHOENIX RETRIEVAL ← Out-of-network posts
         │
         ▼
     HYDRATION
     ├── Core post metadata
     ├── Author information
     ├── Video duration
     └── Subscription status
         │
         ▼
  PRE-SCORING FILTERS (10 filters)
         │
         ▼
       SCORING
       ├── Phoenix Scorer (Grok-based transformer)
       ├── Weighted Scorer  ← Final Score = Σ(weight_i × P(action_i))
       ├── Author Diversity Scorer
       └── OON Scorer
         │
         ▼
     SELECTION  ← Sort by score, top-K
         │
         ▼
POST-SELECTION FILTERS (2 filters)
         │
         ▼
  RANKED FEED RESPONSE
```

---

## Components

### Home Mixer

**Location in repo:** `home-mixer/`

The orchestration layer that assembles the For You feed using the `CandidatePipeline` framework. Exposes a **gRPC endpoint** (`ScoredPostsService`) that returns ranked posts for a given user.

Pipeline stages executed by Home Mixer:

| Stage | Description |
|---|---|
| Query Hydrators | Fetch user context (engagement history, following list) |
| Sources | Retrieve candidates from Thunder and Phoenix |
| Hydrators | Enrich candidates with additional data |
| Filters | Remove ineligible candidates |
| Scorers | Predict engagement and compute final scores |
| Selector | Sort by score and select top K |
| Post-Selection Filters | Final visibility and dedup checks |
| Side Effects | Cache request info for future use |

---

### Thunder

**Location in repo:** `thunder/`

An **in-memory post store and realtime ingestion pipeline** for in-network content.

**How it works:**
- Consumes post create/delete events from **Kafka**
- Maintains **per-user stores** for three post types:
  - Original posts
  - Replies and reposts
  - Video posts
- Serves in-network candidates from accounts the requesting user follows
- Automatically trims posts older than the retention period
- Enables **sub-millisecond lookups** without hitting an external database

---

### Phoenix

**Location in repo:** `phoenix/`

Phoenix has two distinct functions:

#### 1. Retrieval (Two-Tower Model)

Finds relevant **out-of-network** posts from the global corpus.

```
User Tower                    Candidate Tower
──────────────                ──────────────────
Engagement history     →      Post text
Following list         →      Media entities
Account preferences    →      Engagement signals
      │                              │
      └──────── dot product ─────────┘
                similarity
                    │
                top-K posts
```

Both towers use **hash-based embeddings** for lookup.

#### 2. Ranking (Grok-Based Transformer)

Predicts engagement probabilities for each candidate post.

**Key design decision — Candidate Isolation:**
> During transformer inference, candidates cannot attend to each other — only to the user context. This is enforced via special attention masking.

This means:
- A post's score is **independent of what other posts are in the batch**
- Scores are **consistent and cacheable**
- 1 post or 1,000 posts can be scored together with identical per-post results

**Prediction targets (15 action types):**

| Positive | Negative |
|---|---|
| P(favorite) | P(not_interested) |
| P(reply) | P(block_author) |
| P(repost) | P(mute_author) |
| P(quote) | P(report) |
| P(click) | |
| P(profile_click) | |
| P(video_view) | |
| P(photo_expand) | |
| P(share) | |
| P(dwell) | |
| P(follow_author) | |

---

### Candidate Pipeline Framework

**Location in repo:** `candidate-pipeline/`

A reusable Rust framework for building recommendation pipelines. Defines traits for:

| Trait | Purpose |
|---|---|
| `Source` | Fetch candidates from a data source |
| `Hydrator` | Enrich candidates with additional features |
| `Filter` | Remove candidates that shouldn't be shown |
| `Scorer` | Compute scores for ranking |
| `Selector` | Sort and select top candidates |
| `SideEffect` | Run async side effects (caching, logging) |

**Key property:** Runs sources and hydrators **in parallel** where possible, with configurable error handling and logging.

---

## Scoring in Detail

### Weighted Scorer

```
Final Score = Σ (weight_i × P(action_i))
```

- Positive actions (favorites, reposts, etc.) → **positive weights**
- Negative actions (not interested, blocks, reports) → **negative weights**
- **No hardcoded weight values are published in the repo** — magnitudes are learned by the transformer from engagement data

### Author Diversity Scorer

Runs after the Weighted Scorer to **attenuate repeated author scores**, ensuring no single account dominates the feed.

### OON Scorer

Adjusts scores for **out-of-network content** that came through Phoenix retrieval.

---

## Filtering in Detail

### Pre-Scoring Filters (run before Phoenix transformer)

These filters remove candidates cheaply before running the expensive ML model.

| Filter | Purpose |
|---|---|
| `DropDuplicatesFilter` | Remove duplicate post IDs |
| `CoreDataHydrationFilter` | Remove posts that failed to hydrate core metadata |
| `AgeFilter` | Remove posts older than retention threshold |
| `SelfpostFilter` | Remove viewer's own posts |
| `RepostDeduplicationFilter` | Deduplicate reposts of same content |
| `IneligibleSubscriptionFilter` | Remove paywalled content user can't access |
| `PreviouslySeenPostsFilter` | Remove posts user has already seen |
| `PreviouslyServedPostsFilter` | Remove posts already served in current session |
| `MutedKeywordFilter` | Remove posts matching user's muted keywords |
| `AuthorSocialgraphFilter` | Remove posts from blocked/muted authors |

### Post-Selection Filters (run after top-K selection)

| Filter | Purpose |
|---|---|
| `VFFilter` | Remove deleted/spam/violence/gore content |
| `DedupConversationFilter` | Deduplicate branches of the same conversation thread |

---

## Key Design Decisions

From the `xai-org/x-algorithm` README, five explicit design decisions are documented:

### 1. No Hand-Engineered Features
> "We have eliminated every single hand-engineered feature and most heuristics from the system."

The transformer learns relevance entirely from engagement sequences. No manual feature engineering for content relevance.

### 2. Candidate Isolation in Ranking
During transformer inference, candidates cannot attend to each other — only to user context. This makes scores consistent and cacheable.

### 3. Hash-Based Embeddings
Both retrieval and ranking use multiple hash functions for embedding lookup.

### 4. Multi-Action Prediction
Rather than predicting a single "relevance" score, the model predicts probabilities for 15 different action types.

### 5. Composable Pipeline Architecture
The `candidate-pipeline` crate provides a flexible framework with separation of pipeline execution from business logic, parallel execution of independent stages, and graceful error handling.

---

## What This Repo Does NOT Contain

For transparency, the `xai-org/x-algorithm` README does not specify:

- Specific numeric weight values for any action type
- Exact model parameter counts
- Candidate volume numbers (e.g., "500M → 1,500")
- In-network vs. out-of-network ratio splits
- Verification or subscription multipliers
- Specific retention window durations
- Inference probability outputs

Any tool claiming to know these specific values is either extrapolating or referencing the old `twitter/the-algorithm` (2023) codebase.

---

## Repository Languages

Per GitHub: **Rust (62.9%)** and **Python (37.1%)**

- Rust: Core pipeline framework, Thunder, Phoenix serving infrastructure
- Python: ML model training code (Phoenix Two-Tower and transformer)

---

*Last updated: March 2026 — based on the single commit in `xai-org/x-algorithm` (main branch)*
