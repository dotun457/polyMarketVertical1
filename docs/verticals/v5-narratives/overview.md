# V5 — Narrative Intelligence

**Build Order:** Phase 5 (Final)  
**Status:** 🔮 Future  
**Priority:** High (Strategic)

---

## Purpose

V5 is the **culmination** of all verticals — a **Narrative Graph** that:
- Groups related markets into canonical narratives
- Computes composite probabilities across market sets
- Tracks belief evolution over time (Narrative Bonds)
- Powers AI agents that reason about complex, multi-market predictions
- Generates a Narrative Feed that tells the story behind the numbers

This vertical transforms isolated market predictions into **structured intelligence** that AI agents can query, explain, and reason about.

---

## Core Concept

**A Narrative is:**
- A collection of related markets (e.g., "2024 US Election")
- Composite probability (volume-weighted across markets)
- Sentiment layer (from V1)
- News timeline (from V2)
- Regional context (from V3)
- Historical belief trajectory (Narrative Bond)

**Example Narrative:**
```json
{
  "narrative_id": "2024-us-election",
  "title": "2024 US Presidential Election",
  "markets": [
    "Will Trump win 2024?",
    "Will Biden win 2024?",
    "Will DeSantis win 2024?",
    "Will Democrats win 2024?"
  ],
  "composite_prob": {
    "Trump": 0.45,
    "Biden": 0.38,
    "Other": 0.17
  },
  "sentiment_divergence": 0.12,
  "regional_context": {
    "US": 0.48,
    "Europe": 0.35
  },
  "recent_news": [...],
  "belief_bond": [
    {"date": "2024-01-01", "trump_prob": 0.35},
    {"date": "2024-06-01", "trump_prob": 0.42},
    {"date": "2024-11-01", "trump_prob": 0.45}
  ]
}
```

---

## Core Features

### 1. Canonical Market Grouping
- Automatically group related markets
- Manual curation of narrative sets
- Hierarchical narratives (parent/child)
- Cross-narrative connections

### 2. Composite Probability
- Volume-weighted probability across markets
- Handle contradictions (market inefficiencies)
- Confidence scores
- Arbitrage detection

### 3. Narrative Bond (Belief History)
- Track composite probability over time
- Identify inflection points
- Measure volatility
- Predict future trajectories

### 4. Narrative Feed
- "Top Narratives Moving Today"
- "Narratives with High Divergence"
- "Emerging Narratives"
- "Resolved Narratives"

### 5. Narrative Agents
AI agents that can:
- Answer "What's the current belief about X?"
- Explain "Why did probability change?"
- Compare "How does US election compare to UK election?"
- Predict "What happens if Trump wins primary?"

---

## Tech Stack

### Data Sources
- All verticals (V0-V3)
- Market relationships (manual + ML)
- External knowledge graphs (Wikipedia, Wikidata)

### Storage
- PostgreSQL for narrative definitions
- Graph database (Neo4j optional) for relationships
- Vector DB (Pinecone, Weaviate) for semantic search
- TimescaleDB for belief history

### AI/ML
- OpenAI/Anthropic for agent reasoning
- Sentence transformers for semantic grouping
- Time-series analysis for bonds

---

## Dependencies

### Verticals
- **V0 (Analytics Core)** - Market data
- **V1 (Sentiment Layer)** - Community wisdom
- **V2 (Truth Feed)** - News context
- **V3 (Localization)** - Regional insights

### Horizontals Required
- **H1** - API layer
- **H5** - Data aggregation pipelines
- **H6** - AI/ML inference layer
- **H4** - Advanced frontend components

---

## Key Features

### Canonical Grouping
Example groupings:
- **"2024 US Election"** → All 2024 US election markets
- **"Russia-Ukraine Conflict"** → All war-related markets
- **"AI Safety"** → All AI risk/regulation markets
- **"Crypto Markets"** → BTC, ETH, regulatory markets

### Composite Probability
When markets overlap:
- "Will Trump win 2024?" = 45%
- "Will Republican win 2024?" = 52%
- **Inconsistency detected** → arbitrage opportunity or market inefficiency

### Narrative Bond
Track belief over time:
```
Jan 2024: Trump 35%
Mar 2024: Trump 38% (+3% after primary wins)
Jun 2024: Trump 42% (+4% after debate)
Nov 2024: Trump 45% (+3% pre-election)
```

### Narrative Agents
Agent capabilities:
- `@agent What's the latest on the US election?`
- `@agent Why did Trump's odds increase today?`
- `@agent Compare US election to French election narratives`
- `@agent Show me narratives with high regional divergence`

---

## Key Metrics

Success indicators for V5:
- ✅ X canonical narratives defined
- ✅ Composite probabilities calculated
- ✅ Narrative agents answer questions accurately
- ✅ Narrative feed shows meaningful updates

---

## Example Use Cases

### 1. Multi-Market Reasoning
**Question:** "What's the current belief about AI regulation?"

**Agent Response:**
"Based on 12 related markets, there's a 68% probability of significant AI regulation by 2026. This narrative has moved +15% in the last month due to:
- EU AI Act passage (linked news)
- OpenAI safety concerns (sentiment shift)
- Regional divergence (US: 55%, EU: 82%)"

### 2. Narrative Comparison
**Question:** "Compare the US 2024 election narrative to the 2020 narrative at the same point in time."

**Agent Response:**
"At this point in 2020, Biden had 62% vs Trump's 38%. Currently in 2024, Trump has 45% vs Biden's 38% — a significant reversal. Key differences:
- Higher volatility in 2024 (+12% vs +8% in 2020)
- More regional divergence (US vs Europe)
- Negative sentiment shift on Biden due to age concerns"

### 3. Emerging Narratives
**System Alert:** "New narrative detected: 'AI Energy Crisis' (3 markets, rapid growth, high sentiment divergence)"

---

## Data Model Preview

```sql
CREATE TABLE narratives (
  id                TEXT PRIMARY KEY,
  title             TEXT NOT NULL,
  description       TEXT,
  category          TEXT,
  parent_narrative  TEXT,            -- Hierarchical structure
  is_canonical      BOOLEAN DEFAULT TRUE,
  created_at        TIMESTAMP NOT NULL,
  updated_at        TIMESTAMP NOT NULL
);

CREATE TABLE narrative_markets (
  narrative_id      TEXT REFERENCES narratives(id),
  market_id         TEXT REFERENCES markets(id),
  weight            NUMERIC DEFAULT 1.0,
  is_primary        BOOLEAN DEFAULT FALSE,
  added_at          TIMESTAMP NOT NULL,
  PRIMARY KEY (narrative_id, market_id)
);

CREATE TABLE narrative_bonds (
  id                SERIAL PRIMARY KEY,
  narrative_id      TEXT REFERENCES narratives(id),
  timestamp         TIMESTAMP NOT NULL,
  composite_prob    JSONB NOT NULL,      -- {"outcome_a": 0.45, ...}
  volume_usd        NUMERIC,
  sentiment_avg     NUMERIC,
  news_count        INTEGER,
  UNIQUE(narrative_id, timestamp)
);

CREATE TABLE narrative_events (
  id                SERIAL PRIMARY KEY,
  narrative_id      TEXT REFERENCES narratives(id),
  event_type        TEXT NOT NULL,       -- 'prob_shift', 'news', 'divergence'
  magnitude         NUMERIC,
  description       TEXT,
  occurred_at       TIMESTAMP NOT NULL
);
```

---

## Architecture Overview

```
┌──────────────┐
│  V0: Markets │
│  V1: Sentiment│
│  V2: News    │
│  V3: Regional│
└──────┬───────┘
       │
       ▼
┌──────────────────┐
│ Narrative Engine │
│ - Grouping       │
│ - Composite Prob │
│ - Bond Tracking  │
└──────┬───────────┘
       │
       ├────▶ Narrative Graph
       │
       ├────▶ Narrative Feed
       │
       └────▶ Narrative Agents
                │
                ▼
         ┌─────────────┐
         │ AI Reasoning│
         │ Interface   │
         └─────────────┘
```

---

## Next Steps

Once V5 is complete:
1. **Narrative Marketplace** - Users can create/curate narratives
2. **Narrative Predictions** - Predict narrative trajectories
3. **Narrative Trading** - Trade on narrative composites
4. **Advanced Agents** - Multi-agent reasoning systems

---

## Related Documentation

- [Development Plan](./dev-plan.md)
- [Verticals Overview](../verticals-overview.md)
- [V0 Analytics Core](../v0-analytics/overview.md)
- [V1 Sentiment Layer](../v1-sentiment/overview.md)
- [V2 Truth Feed](../v2-truth-feed/overview.md)
- [V3 Localization Engine](../v3-localization/overview.md)
