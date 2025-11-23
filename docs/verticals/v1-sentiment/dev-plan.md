# V1 — Sentiment Layer Dev Plan

**Vertical:** 1  
**Build Order:** Phase 3  
**Goal:** Build a sentiment polling system with stake-weighted votes and regional tagging.

---

## 0. Tech Assumptions

- **Language:** TypeScript (Node.js + Next.js)
- **Database:** PostgreSQL
- **Cache:** Redis for sentiment aggregates
- **Optional:** ERC-20 token for staking (or in-app points)
- **Dependencies:**
  - V0 (Analytics Core) for market data
  - V2 (Truth Feed) for news context

---

## 1. High-Level Architecture

**Flow:**

1. **User Sentiment Submission**
   - User views a market
   - Casts a sentiment vote (YES/NO or probability estimate)
   - Optionally stakes tokens for conviction

2. **Sentiment Aggregation**
   - Aggregate votes per market
   - Calculate weighted sentiment (by stake)
   - Compare sentiment vs market price

3. **Regional Tagging**
   - Detect user region (IP geolocation or manual)
   - Store regional sentiment
   - Generate regional divergence insights

4. **Sentiment Feed**
   - Show markets with significant sentiment changes
   - Highlight sentiment-price divergence
   - Display regional sentiment differences

---

## 2. Data Model

### 2.1 `sentiment_votes`

```sql
CREATE TABLE sentiment_votes (
  id              SERIAL PRIMARY KEY,
  user_id         TEXT NOT NULL,
  market_id       TEXT NOT NULL REFERENCES markets(id),
  outcome         TEXT NOT NULL,      -- 'YES' or 'NO'
  probability     NUMERIC,            -- Optional: user's probability estimate
  stake_amount    NUMERIC DEFAULT 0,  -- Amount staked (0 if no stake)
  region          TEXT,               -- ISO country code or region
  confidence      INTEGER,            -- 1-5 confidence level
  reasoning       TEXT,               -- Optional: why they believe this
  created_at      TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMP NOT NULL DEFAULT NOW(),
  
  UNIQUE(user_id, market_id)
);

CREATE INDEX idx_sentiment_market ON sentiment_votes(market_id);
CREATE INDEX idx_sentiment_user ON sentiment_votes(user_id);
CREATE INDEX idx_sentiment_region ON sentiment_votes(region);
```

### 2.2 `sentiment_aggregates`

```sql
CREATE TABLE sentiment_aggregates (
  market_id         TEXT PRIMARY KEY REFERENCES markets(id),
  total_votes       INTEGER NOT NULL DEFAULT 0,
  yes_votes         INTEGER NOT NULL DEFAULT 0,
  no_votes          INTEGER NOT NULL DEFAULT 0,
  yes_stake         NUMERIC NOT NULL DEFAULT 0,
  no_stake          NUMERIC NOT NULL DEFAULT 0,
  avg_sentiment     NUMERIC,               -- Weighted average probability
  sentiment_price   NUMERIC,               -- Price derived from sentiment
  price_divergence  NUMERIC,               -- sentiment_price - market_price
  last_updated      TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_sentiment_divergence ON sentiment_aggregates(price_divergence DESC);
```

### 2.3 `regional_sentiment`

```sql
CREATE TABLE regional_sentiment (
  id                SERIAL PRIMARY KEY,
  market_id         TEXT NOT NULL REFERENCES markets(id),
  region            TEXT NOT NULL,
  yes_votes         INTEGER NOT NULL DEFAULT 0,
  no_votes          INTEGER NOT NULL DEFAULT 0,
  avg_sentiment     NUMERIC,
  last_updated      TIMESTAMP NOT NULL DEFAULT NOW(),
  
  UNIQUE(market_id, region)
);

CREATE INDEX idx_regional_market ON regional_sentiment(market_id);
CREATE INDEX idx_regional_region ON regional_sentiment(region);
```

### 2.4 `user_stakes` (optional token staking)

```sql
CREATE TABLE user_stakes (
  id              SERIAL PRIMARY KEY,
  user_id         TEXT NOT NULL,
  market_id       TEXT NOT NULL REFERENCES markets(id),
  amount          NUMERIC NOT NULL,
  outcome         TEXT NOT NULL,
  staked_at       TIMESTAMP NOT NULL DEFAULT NOW(),
  unstaked_at     TIMESTAMP,
  is_active       BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_stakes_user ON user_stakes(user_id);
CREATE INDEX idx_stakes_market ON user_stakes(market_id);
```

---

## 3. Epics & Tasks

### Epic 0 — Schema & Setup

**Goal:** Set up database schema and sentiment infrastructure.

**Tasks:**

- [ ] Add sentiment tables to database schema
- [ ] Create migration scripts
- [ ] Set up Redis for caching sentiment aggregates
- [ ] Add region detection library (MaxMind GeoIP or ipapi)

**Acceptance Criteria:**
- Schema created successfully
- Region detection works

---

### Epic 1 — Sentiment Voting UI

**Goal:** Users can cast sentiment votes on markets.

**Tasks:**

#### Vote Component
- [ ] Create sentiment voting widget:
  - Display current sentiment vs market price
  - Yes/No buttons or probability slider
  - Optional confidence level (1-5)
  - Optional reasoning text field

#### Vote API
- [ ] Create endpoints:
  - `POST /api/sentiment/vote` - Submit vote
  - `PUT /api/sentiment/vote` - Update vote
  - `GET /api/sentiment/:marketId` - Get market sentiment

#### Vote Logic
- [ ] Upsert user vote (one vote per user per market)
- [ ] Detect region from IP address
- [ ] Store timestamp for sentiment history

**Acceptance Criteria:**
- Users can vote on any market
- Votes are recorded with region

---

### Epic 2 — Stake-Weighted Voting (Optional)

**Goal:** Users can stake tokens to increase vote weight.

**Tasks:**

#### Stake UI
- [ ] Add "Stake tokens" option to voting widget
- [ ] Show user's available stake balance
- [ ] Display current stake on a market

#### Stake Logic
- [ ] Implement staking mechanism:
  - Deduct from user balance
  - Lock tokens to market + outcome
  - Allow unstaking (with cooldown?)

#### Weighted Aggregation
- [ ] Calculate weighted sentiment:
  - `weighted_sentiment = (yes_stake - no_stake) / (yes_stake + no_stake)`
- [ ] Show weighted vs unweighted sentiment

**Acceptance Criteria:**
- Users can stake tokens on their sentiment
- Weighted sentiment differs from simple vote count

---

### Epic 3 — Sentiment Aggregation Service

**Goal:** Continuously aggregate sentiment per market.

**Tasks:**

#### Aggregation Job
- [ ] Create `aggregate-sentiment` script:
  - For each market with votes:
    - Count yes/no votes
    - Sum yes/no stakes
    - Calculate average sentiment
    - Compute sentiment-derived price
  - Update `sentiment_aggregates` table
  - Cache in Redis

#### Divergence Detection
- [ ] Calculate `price_divergence`:
  - `divergence = sentiment_price - market_price`
- [ ] Flag markets with significant divergence (> 5%)

#### Regional Aggregation
- [ ] Aggregate sentiment by region:
  - Group votes by region
  - Calculate regional sentiment
  - Store in `regional_sentiment` table

**Acceptance Criteria:**
- Sentiment aggregates updated regularly (every 5 minutes)
- Divergence scores calculated correctly

---

### Epic 4 — Sentiment Visualization

**Goal:** Display sentiment data alongside price data.

**Tasks:**

#### Sentiment Chart
- [ ] Add sentiment overlay to price charts:
  - Price line (from V0)
  - Sentiment line (community belief)
  - Highlight divergence areas

#### Sentiment vs Price Widget
- [ ] Create comparison view:
  - Market price: X%
  - Community sentiment: Y%
  - Divergence: Z%
  - Color code (green if sentiment > price, red if < price)

#### Vote Distribution
- [ ] Show vote breakdown:
  - Yes: X votes (Y% weighted)
  - No: A votes (B% weighted)
  - Pie chart or bar chart

**Acceptance Criteria:**
- Sentiment visible on market detail pages
- Charts show sentiment vs price over time

---

### Epic 5 — Regional Sentiment Analysis

**Goal:** Display and compare regional sentiment patterns.

**Tasks:**

#### Regional Breakdown UI
- [ ] Add regional sentiment section to market page:
  - List regions with votes
  - Show sentiment per region
  - Highlight regions with divergent beliefs

#### Regional Heatmap
- [ ] Create world map visualization:
  - Color regions by sentiment (green = YES, red = NO)
  - Show regional confidence levels

#### Regional Divergence Alerts
- [ ] Detect when regions significantly disagree:
  - Example: US 80% YES, Europe 40% YES
  - Flag as "Regional Divergence Alert"

**Acceptance Criteria:**
- Regional sentiment visible for markets with diverse voters
- Heatmap shows geographical patterns

---

### Epic 6 — Sentiment Feed

**Goal:** A feed of interesting sentiment changes and divergences.

**Tasks:**

#### Feed Logic
- [ ] Detect sentiment events:
  - Sentiment changed > 10% in 1 hour
  - Divergence > 10% between sentiment and price
  - Regional divergence > 20%

#### Feed API
- [ ] Create `GET /api/sentiment/feed?limit=50`
- [ ] Return list of sentiment events with context

#### Feed UI
- [ ] Create sentiment feed page:
  - List recent sentiment events
  - Link to market detail
  - Show before/after sentiment

**Acceptance Criteria:**
- Feed shows interesting sentiment changes
- Users can browse sentiment events

---

### Epic 7 — Sentiment History & Trends

**Goal:** Track how sentiment evolves over time.

**Tasks:**

#### Sentiment History Table
- [ ] Create `sentiment_history` table:
  - Snapshot sentiment aggregates periodically
  - Store historical divergence

#### Sentiment Trend Charts
- [ ] Add sentiment-over-time chart:
  - X-axis: time
  - Y-axis: sentiment probability
  - Compare to price line

#### User Sentiment History
- [ ] Show user's past votes:
  - How accurate were they?
  - Conviction tracking (did they change vote?)

**Acceptance Criteria:**
- Historical sentiment data available
- Users can see sentiment trends

---

### Epic 8 — Observability & Polish

**Goal:** Production-ready sentiment system.

**Tasks:**

#### Logging
- [ ] Log vote submissions
- [ ] Log aggregation job runs
- [ ] Log divergence alerts

#### Metrics
- [ ] Track:
  - Total votes per day
  - Avg stake per vote
  - Divergence frequency

#### Abuse Prevention
- [ ] Rate limit vote changes (max 1 per hour?)
- [ ] Detect suspicious voting patterns
- [ ] Minimum account age for staking

**Acceptance Criteria:**
- System handles high vote volume
- Basic abuse prevention in place

---

## 4. Rough Implementation Order

1. **Epic 0** - Schema setup
2. **Epic 1** - Basic voting UI
3. **Epic 3** - Aggregation service
4. **Epic 4** - Visualization
5. **Epic 5** - Regional analysis
6. **Epic 6** - Sentiment feed
7. **Epic 2** - Stake-weighted voting (optional)
8. **Epic 7** - History & trends
9. **Epic 8** - Observability

---

## 5. Next Steps

Once V1 is complete:
1. **V3 (Localization)** - Deeper regional market insights
2. **V5 (Narratives)** - Integrate sentiment into narrative graph
3. **Rewards System** - Incentivize accurate sentiment predictions

---

**Last Updated:** November 23, 2025
