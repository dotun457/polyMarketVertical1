# V3 — Localization Engine

**Build Order:** Phase 4  
**Status:** 🔜 Planned  
**Priority:** Medium

---

## Purpose

V3 adds **regional intelligence** to prediction markets by:
- Tracking region-level beliefs and sentiment
- Detecting divergence between local and global predictions
- Mapping regional demand for specific markets
- Identifying regional information advantages

This vertical transforms global market data into **geo-contextual insights** that reveal how different regions predict differently.

---

## Core Features

### 1. Regional Belief Tracking
- Aggregate user predictions by region
- Compare regional vs global probabilities
- Track regional confidence levels
- Historical regional belief evolution

### 2. Divergence Detection
- Identify when regions significantly disagree
- Quantify divergence scores
- Alert on meaningful regional differences
- Correlation with regional news/events

### 3. Regional Demand Mapping
- "What would you like to see?" regional surveys
- Track requested market topics by region
- Regional interest heatmaps
- Cultural prediction preferences

### 4. Regional Heatmaps
- Visualize belief by geography
- Color-code conviction levels
- Interactive world/country maps
- Time-series regional changes

---

## Tech Stack

### Data Sources
- User region data (from V1 Sentiment Layer)
- IP geolocation (MaxMind, ipapi)
- Regional news feeds (from V2 Truth Feed)
- Market data from V0

### Storage
- PostgreSQL for regional aggregates
- Redis for real-time regional calculations
- GeoJSON for map visualizations

### Frontend
- Map libraries (Mapbox, Leaflet, D3)
- Regional breakdown charts
- Divergence alerts UI

---

## Dependencies

### Verticals
- **V0 (Analytics Core)** - Market foundation
- **V1 (Sentiment Layer)** - Regional sentiment votes
- **V2 (Truth Feed)** - Regional news impact

### Horizontals Required
- **H1** - API layer
- **H5** - Data aggregation pipelines
- **H4** - Frontend map components

---

## Key Features

### Regional Belief Comparison
Example:
```
Market: "Will Trump win 2024?"
- US users: 65%
- European users: 45%
- Asian users: 52%
- Global average: 58%
```

### Divergence Alerts
- **High Divergence**: US and Europe differ by > 20%
- **Regional Advantage**: Region with local information
- **Cultural Bias**: Systematic regional over/under-estimation

### Regional Demand
Track what markets users want to see:
- US users request: State-level election markets
- UK users request: Premier League outcome markets
- India users request: Cricket match markets

---

## Key Metrics

Success indicators for V3:
- ✅ Regional data available for X% of markets
- ✅ Divergence detected in interesting cases
- ✅ Regional demand mapped for top categories
- ✅ Regional heatmaps functional

---

## Example Use Cases

### 1. Information Advantage
- **Ukrainian users** predict Russia conflict outcomes better
- **Silicon Valley users** predict tech startup outcomes better
- **DC users** predict political outcomes better

### 2. Cultural Bias Detection
- **Home country bias**: Users over-estimate their country's success
- **Regional optimism/pessimism**: Systematic patterns
- **Language barriers**: English vs non-English market access

### 3. Market Opportunity Detection
- High regional demand but no markets → create new markets
- Regional price inefficiency → arbitrage opportunities
- Regional news first → trade before global awareness

---

## Data Model Preview

```sql
CREATE TABLE regional_beliefs (
  id              SERIAL PRIMARY KEY,
  market_id       TEXT NOT NULL,
  region          TEXT NOT NULL,      -- ISO country code
  yes_prob        NUMERIC,
  no_prob         NUMERIC,
  vote_count      INTEGER,
  avg_confidence  NUMERIC,
  last_updated    TIMESTAMP NOT NULL
);

CREATE TABLE regional_divergence (
  id              SERIAL PRIMARY KEY,
  market_id       TEXT NOT NULL,
  region_a        TEXT NOT NULL,
  region_b        TEXT NOT NULL,
  divergence      NUMERIC,            -- absolute difference
  significance    NUMERIC,            -- statistical significance
  detected_at     TIMESTAMP NOT NULL
);

CREATE TABLE regional_demand (
  id              SERIAL PRIMARY KEY,
  region          TEXT NOT NULL,
  topic           TEXT NOT NULL,
  request_count   INTEGER,
  first_requested TIMESTAMP NOT NULL,
  last_requested  TIMESTAMP NOT NULL
);
```

---

## Architecture Overview

```
┌─────────────────┐
│ User Votes      │
│ (with region)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Regional        │
│ Aggregation     │
└────────┬────────┘
         │
         ├──────▶ Regional Beliefs
         │
         ├──────▶ Divergence Detection
         │
         └──────▶ Demand Mapping
                  │
                  ▼
           ┌──────────────┐
           │  Heatmaps    │
           │  & Alerts    │
           └──────────────┘
```

---

## Next Steps

Once V3 is complete:
1. **V5 (Narratives)** - Integrate regional context into narrative graph
2. **Regional Market Creation** - Auto-generate region-specific markets
3. **Regional Rewards** - Incentivize regional prediction accuracy

---

## Related Documentation

- [Development Plan](./dev-plan.md)
- [Verticals Overview](../verticals-overview.md)
- [V1 Sentiment Layer](../v1-sentiment/overview.md)
- [V2 Truth Feed](../v2-truth-feed/overview.md)
