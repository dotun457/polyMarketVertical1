# V1 — Sentiment Layer

**Build Order:** Phase 3  
**Status:** 🔜 Planned  
**Priority:** High

---

## Purpose

V1 adds a **sentiment intelligence layer** on top of market prices, enabling users to express opinions through Truth Token polls while tracking stake-based conviction and regional sentiment patterns.

This vertical transforms passive price observation into active community wisdom aggregation.

---

## Core Features

### 1. Truth Token Polls
- User polls on market outcomes (beyond just trading)
- Stake-weighted voting (optional token staking for conviction)
- Anonymous sentiment capture
- Poll results vs actual market prices

### 2. Sentiment Analytics
- Aggregate community sentiment per market
- Sentiment vs price divergence detection
- Conviction tracking (stake amount as signal)
- Sentiment trends over time

### 3. Regional Sentiment
- Tag sentiment by user region
- Compare regional vs global sentiment
- Identify regional belief divergence
- Cultural/geographical prediction patterns

### 4. Sentiment Feed
- "Sentiment changed significantly on Market X"
- Highlight where sentiment differs from price
- User-generated prediction reasoning
- Community consensus tracking

---

## Tech Stack

### Data Sources
- User sentiment submissions
- Optional token staking (ERC-20)
- Regional IP/location tagging
- Market data from V0 (Analytics Core)

### Storage
- PostgreSQL for sentiment records
- Cache sentiment aggregates in Redis
- Historical sentiment trends

### Frontend
- Sentiment polling UI
- Sentiment charts alongside price charts
- Regional heatmaps
- Stake management interface

---

## Dependencies

### Verticals
- **V0 (Analytics Core)** - Market data foundation
- **V2 (Truth Feed)** - News context for sentiment

### Horizontals Required
- **H2** - User authentication & profiles
- **H3** - Caching for sentiment aggregates
- **H4** - Frontend components for polls and charts

---

## Key Features

### Truth Token Polls
Users can:
- Vote on market outcomes without trading
- Optionally stake tokens to show conviction
- Change votes over time (tracked as sentiment evolution)
- See how their prediction compares to market price

### Stake-Weighted Sentiment
- Higher stakes = stronger signal
- Prevents spam/manipulation
- Creates "skin in the game" for predictions
- Rewards for accurate sentiment (future)

### Regional Tagging
- Detect user region (IP or manual selection)
- Compare local vs global beliefs
- Identify regional prediction advantages
- Cultural prediction bias analysis

---

## Key Metrics

Success indicators for V1:
- ✅ X% of markets have sentiment data
- ✅ Sentiment diverges from price in interesting ways
- ✅ Regional differences observable
- ✅ Stake-weighted predictions show value

---

## Example Use Cases

### 1. Sentiment vs Price Divergence
- Market price: 65% Biden wins
- Community sentiment: 75% Biden wins
- → Potential buying opportunity if sentiment is correct

### 2. Regional Insight
- US users: 70% believe X will happen
- European users: 45% believe X will happen
- → Regional information advantage

### 3. Conviction Tracking
- User stakes 100 tokens on "Trump wins"
- Market moves against them
- Do they unstake (lose conviction) or hold?
- → Measure conviction over time

---

## Data Model Preview

```sql
CREATE TABLE sentiment_votes (
  id              SERIAL PRIMARY KEY,
  user_id         TEXT NOT NULL,
  market_id       TEXT NOT NULL,
  outcome         TEXT NOT NULL,
  stake_amount    NUMERIC DEFAULT 0,
  region          TEXT,
  created_at      TIMESTAMP NOT NULL,
  updated_at      TIMESTAMP NOT NULL
);

CREATE TABLE sentiment_aggregates (
  market_id       TEXT PRIMARY KEY,
  yes_votes       INTEGER,
  no_votes        INTEGER,
  yes_stake       NUMERIC,
  no_stake        NUMERIC,
  avg_sentiment   NUMERIC,
  last_updated    TIMESTAMP NOT NULL
);
```

---

## Next Steps

Once V1 is complete:
1. **V3 (Localization)** - Deeper regional analysis
2. **V5 (Narratives)** - Combine sentiment with narrative graph
3. **Rewards** - Incentivize accurate sentiment predictions

---

## Related Documentation

- [Development Plan](./dev-plan.md)
- [Verticals Overview](../verticals-overview.md)
- [V0 Analytics Core](../v0-analytics/overview.md)
- [V2 Truth Feed](../v2-truth-feed/overview.md)
