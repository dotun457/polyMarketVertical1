# V2 — Truth Feed (News ↔ Markets)

**Build Order:** Phase 2  
**Status:** 🚧 In Progress  
**Priority:** Critical

---
## Purpose

V2 creates a [Nevua-markets style product](https://docs.nevua.markets/introduction/what-is-nevua-markets) — a **news-to-markets intelligence engine** that:
- Fetches headlines from news providers continuously
- Maps headlines to relevant Polymarket markets
- Computes impact scores based on price movements
- Exposes a Truth Feed showing which news moved which markets

This vertical transforms prediction markets from isolated price data into **narrative-driven market intelligence**.

---

## Core Features

### 1. News Ingestion Pipeline
- Continuous headline fetching (NewsAPI, GDELT, AP, RSS)
- Multi-source aggregation
- De-duplication and normalization
- Source quality ranking

### 2. News-to-Market Mapping
- Keyword-based matching (v1)
- Semantic matching via embeddings (v2)
- Entity extraction and linking
- Relevance scoring

### 3. Impact Analysis
- Price movement detection around headline timestamps
- Impact score calculation (before/after analysis)
- Volume and volatility correlation
- Statistical significance testing

### 4. Truth Feed Interface
- Timeline of impactful headlines
- Market-specific news feeds
- Global feed sorted by impact
- Real-time updates

---

## Tech Stack

### Data Sources
- **News Providers:**
  - NewsAPI, GDELT, Mediastack
  - RSS feeds (Reuters, AP, Bloomberg)
- **Polymarket APIs:**
  - Gamma for market metadata → [`docs/polymarket-gamma.md`](../../polymarket-gamma.md)
  - CLOB for price history → [`docs/polymarket-clob.md`](../../polymarket-clob.md)
  - WebSocket for real-time prices → [`docs/polymarket-wss.md`](../../polymarket-wss.md)

### Storage
- PostgreSQL for headlines and mappings
- Redis for real-time processing
- Optional: Elasticsearch for semantic search

### Processing
- Keyword extraction (NLP libraries)
- Optional: OpenAI/Anthropic for semantic matching
- Background jobs for continuous ingestion

---

## Dependencies

### Verticals
- **V0 (Analytics Core)** - Market data foundation
- **V1 (Sentiment)** - User reactions to news (future integration)

### Horizontals Required
- **H1** - API client layer (Gamma, CLOB)
- **H5** - News ingestion pipeline
- **H6** - Background job scheduler

---

## Key Metrics

Success indicators for V2:
- ✅ Headlines ingested continuously (> 100/hour)
- ✅ 80%+ of major markets have news mappings
- ✅ Impact scores correlate with actual price movements
- ✅ Truth Feed shows live, meaningful updates

---

## Example Use Cases

### 1. Market Impact Detection
- Headline: "Biden withdraws from 2024 election"
- Market: "Will Biden win 2024 election?"
- Price before: 45%
- Price after: 2%
- Impact: -43% (MASSIVE)

### 2. Cross-Market Impact
- Headline: "Fed announces rate cut"
- Affected markets:
  - "Will Fed cut rates in Q4?" → +85%
  - "S&P 500 to hit 5000?" → +12%
  - "Recession in 2024?" → -8%

### 3. Predictive Signals
- News mentions "Trump indictment" at 2pm
- Market starts moving at 2:05pm
- Alert users to news-driven moves

---

## Data Model Preview

```sql
CREATE TABLE headlines (
  id              SERIAL PRIMARY KEY,
  title           TEXT NOT NULL,
  description     TEXT,
  source_name     TEXT,
  url             TEXT,
  published_at    TIMESTAMP,
  fetched_at      TIMESTAMP NOT NULL
);

CREATE TABLE headline_market_links (
  id              SERIAL PRIMARY KEY,
  headline_id     INTEGER REFERENCES headlines(id),
  market_id       TEXT REFERENCES markets(id),
  match_method    TEXT,
  keyword_score   NUMERIC,
  semantic_score  NUMERIC,
  impact_abs      NUMERIC,
  impact_signed   NUMERIC,
  created_at      TIMESTAMP NOT NULL
);
```

---

## Architecture Overview

```
┌─────────────┐
│ News Sources│
│ (API/RSS)   │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌──────────────┐
│   Ingest    │────▶│  Headlines   │
│   Job       │     │  Table       │
└─────────────┘     └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Mapping    │
                    │   Engine     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Impact     │
                    │   Engine     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Truth Feed  │
                    │  Interface   │
                    └──────────────┘
```

---

## Next Steps

Once V2 is complete:
1. **V1 (Sentiment)** - Add user reactions to news
2. **V3 (Localization)** - Regional news impact analysis
3. **V5 (Narratives)** - Integrate news into narrative graph

---

## Related Documentation

- [Development Plan](./dev-plan.md)
- [Verticals Overview](../verticals-overview.md)
- [V0 Analytics Core](../v0-analytics/overview.md)
- [Polymarket Gamma API](../../polymarket-gamma.md)
- [Polymarket CLOB API](../../polymarket-clob.md)
