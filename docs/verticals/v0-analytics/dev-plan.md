# V0 — Analytics Core Dev Plan

**Vertical:** 0  
**Build Order:** Phase 1 (Foundation)  
**Goal:** A working Polysights-style market analytics dashboard with real-time data, charts, watchlists, and alerts.

---

## 0. Tech Assumptions

- **Language:** TypeScript (Node.js + Next.js)
- **Database:** PostgreSQL (or SQLite for dev)
- **Cache:** Redis (optional, but recommended for real-time data)
- **Polymarket APIs:**
  - **Gamma Markets API** for market metadata → [`docs/polymarket-gamma.md`](../../polymarket-gamma.md)
  - **CLOB API** for order books and trades → [`docs/polymarket-clob.md`](../../polymarket-clob.md)
  - **WebSocket** for real-time prices → [`docs/polymarket-wss.md`](../../polymarket-wss.md)
  - **Subgraph** for historical analytics → [`docs/polymarket-subgraph.md`](../../polymarket-subgraph.md)

---

## 1. High-Level Architecture

**Flow:**

1. **Market Sync Job**
   - Fetch markets from Gamma API
   - Cache in PostgreSQL
   - Update metadata periodically

2. **Real-Time Price Feed**
   - Connect to Polymarket WebSocket
   - Subscribe to market price updates
   - Cache in Redis, broadcast to frontend

3. **Order Book Service**
   - Fetch order books from CLOB API
   - Cache and refresh on interval
   - Provide REST API for frontend

4. **User Watchlist & Alerts**
   - User-specific market tracking
   - Alert triggers (price, volume)
   - Notification service (email, push, in-app)

5. **Frontend Dashboard**
   - Market explorer UI
   - Price charts (real-time + historical)
   - Order book visualization
   - Watchlist management

---

## 2. Data Model (Dev Schema)

### 2.1 `markets` (local cache from Gamma)

```sql
CREATE TABLE markets (
  id              TEXT PRIMARY KEY,
  question        TEXT NOT NULL,
  event_id        TEXT,
  category        TEXT,
  subcategory     TEXT,
  tags            TEXT,              -- JSON array
  status          TEXT,              -- open / closed / resolved
  outcomes        TEXT,              -- JSON array of outcomes
  volume_usd      NUMERIC,
  liquidity_usd   NUMERIC,
  last_price_yes  NUMERIC,
  last_price_no   NUMERIC,
  end_date        TIMESTAMP,
  created_at      TIMESTAMP NOT NULL,
  last_synced_at  TIMESTAMP NOT NULL
);

CREATE INDEX idx_markets_category ON markets(category);
CREATE INDEX idx_markets_status ON markets(status);
CREATE INDEX idx_markets_volume ON markets(volume_usd DESC);
```

### 2.2 `market_prices` (price history)

```sql
CREATE TABLE market_prices (
  id              SERIAL PRIMARY KEY,
  market_id       TEXT NOT NULL REFERENCES markets(id),
  timestamp       TIMESTAMP NOT NULL,
  price_yes       NUMERIC NOT NULL,
  price_no        NUMERIC NOT NULL,
  source          TEXT NOT NULL,     -- 'CLOB', 'WSS', 'Subgraph'
  
  UNIQUE(market_id, timestamp, source)
);

CREATE INDEX idx_prices_market_ts ON market_prices(market_id, timestamp DESC);
```

### 2.3 `user_watchlists`

```sql
CREATE TABLE user_watchlists (
  id              SERIAL PRIMARY KEY,
  user_id         TEXT NOT NULL,
  market_id       TEXT NOT NULL REFERENCES markets(id),
  added_at        TIMESTAMP NOT NULL DEFAULT NOW(),
  notes           TEXT,
  
  UNIQUE(user_id, market_id)
);

CREATE INDEX idx_watchlist_user ON user_watchlists(user_id);
```

### 2.4 `user_alerts`

```sql
CREATE TABLE user_alerts (
  id              SERIAL PRIMARY KEY,
  user_id         TEXT NOT NULL,
  market_id       TEXT NOT NULL REFERENCES markets(id),
  alert_type      TEXT NOT NULL,     -- 'price_above', 'price_below', 'volume_spike'
  threshold       NUMERIC,
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMP NOT NULL DEFAULT NOW(),
  triggered_at    TIMESTAMP
);

CREATE INDEX idx_alerts_user_active ON user_alerts(user_id, is_active);
```

---

## 3. Epics & Tasks

### Epic 0 — Repo & Config Setup

**Goal:** Bootstrap the V0 project with proper configuration.

**Tasks:**

- [ ] Initialize Next.js project with TypeScript
- [ ] Set up PostgreSQL database
- [ ] Add environment variables:
  - `DATABASE_URL`
  - `REDIS_URL` (optional)
  - `POLYMARKET_API_KEY` (if needed)
- [ ] Add database migration tool (Prisma, Drizzle, or raw SQL)
- [ ] Create initial schema (markets, prices, watchlists, alerts)

**Acceptance Criteria:**
- Can run `npm run dev` and see empty dashboard
- Database connects and schema is created

---

### Epic 1 — Market Sync Service

**Goal:** Continuously sync markets from Gamma API into local database.

See [`docs/polymarket-gamma.md`](../../polymarket-gamma.md) for API details.

**Tasks:**

#### Gamma Client
- [ ] Implement Gamma API client:
  - `GET /markets` with pagination
  - Filter by status, category
- [ ] Add retry logic and rate limit handling

#### Sync Job
- [ ] Create `sync-markets` script:
  - Fetch all markets (or incremental updates)
  - Upsert into `markets` table
  - Log sync statistics
- [ ] Schedule job (cron or Next.js API route)

#### Market Search
- [ ] Implement search endpoint:
  - `/api/markets?q=election&category=politics`
- [ ] Add full-text search (PostgreSQL `tsvector` or Algolia)

**Acceptance Criteria:**
- Markets table populated with current Polymarket data
- Search returns relevant results

---

### Epic 2 — Real-Time Price Feed (WebSocket)

**Goal:** Stream live price updates from Polymarket WebSocket to frontend.

See [`docs/polymarket-wss.md`](../../polymarket-wss.md) for WebSocket details.

**Tasks:**

#### WebSocket Client
- [ ] Connect to Polymarket WebSocket endpoint
- [ ] Subscribe to market price updates
- [ ] Handle reconnection logic

#### Price Cache
- [ ] Store latest prices in Redis (or in-memory)
- [ ] Update `market_prices` table periodically for history

#### Frontend Integration
- [ ] Create WebSocket hook for Next.js:
  - Subscribe to specific market updates
  - Display live price changes
- [ ] Show price change indicators (+/- %)

**Acceptance Criteria:**
- Frontend displays real-time prices
- Price updates reflect within 1 second
- Historical prices stored for charts

---

### Epic 3 — Order Book Visualization

**Goal:** Fetch and display order books from CLOB API.

See [`docs/polymarket-clob.md`](../../polymarket-clob.md) for CLOB details.

**Tasks:**

#### CLOB Client
- [ ] Implement CLOB API client:
  - `GET /book?market_id=...` for order book
  - Handle rate limits (100 req/min)

#### Order Book Service
- [ ] Create `/api/orderbook/:marketId` endpoint
- [ ] Cache order books (Redis or in-memory, 5-10s TTL)

#### Frontend Order Book Component
- [ ] Build order book table:
  - Bids (buy orders) on left
  - Asks (sell orders) on right
  - Show price, size, total
- [ ] Add depth chart visualization

**Acceptance Criteria:**
- Order book displays correctly for any market
- Refreshes automatically every 5-10 seconds

---

### Epic 4 — Price Charts & History

**Goal:** Display historical price charts and trends.

**Tasks:**

#### Historical Data Fetching
- [ ] Use Subgraph to fetch historical prices ([`docs/polymarket-subgraph.md`](../../polymarket-subgraph.md))
- [ ] Store in `market_prices` table

#### Chart Component
- [ ] Add charting library (Recharts, Lightweight Charts, or TradingView)
- [ ] Create price-over-time chart:
  - X-axis: time
  - Y-axis: probability (0-1)
- [ ] Add time range selector (1H, 1D, 1W, 1M, ALL)

#### Chart API Endpoint
- [ ] Create `/api/chart/:marketId?range=1d`
- [ ] Return price data in chart-friendly format

**Acceptance Criteria:**
- Charts display smoothly for markets with price history
- User can switch time ranges

---

### Epic 5 — Watchlist & Alerts

**Goal:** Users can track favorite markets and set alerts.

**Tasks:**

#### Watchlist API
- [ ] Create endpoints:
  - `POST /api/watchlist` - Add market
  - `DELETE /api/watchlist/:id` - Remove market
  - `GET /api/watchlist` - List user's watchlist

#### Watchlist UI
- [ ] Add "Add to Watchlist" button on market pages
- [ ] Create watchlist page:
  - Show all tracked markets
  - Quick price overview
  - Jump to market detail

#### Alerts System
- [ ] Create alert endpoints:
  - `POST /api/alerts` - Create alert
  - `GET /api/alerts` - List user alerts
  - `DELETE /api/alerts/:id` - Remove alert
- [ ] Implement alert checker job:
  - Check price thresholds every minute
  - Trigger notifications when conditions met

#### Notifications
- [ ] In-app notification system (toast/banner)
- [ ] Optional: Email notifications
- [ ] Optional: Push notifications (web push API)

**Acceptance Criteria:**
- Users can add markets to watchlist
- Alerts trigger correctly based on price conditions
- Notifications appear in UI

---

### Epic 6 — Market Detail Page

**Goal:** Comprehensive view of a single market.

**Tasks:**

#### Market Detail UI
- [ ] Create `/market/[id]` page with:
  - Market question and metadata
  - Current price and 24h change
  - Price chart
  - Order book
  - Recent trades (if available)
  - Add to watchlist button
  - Set alert button

#### Related Markets
- [ ] Show similar or related markets (by tags/category)

#### Social Features (Optional)
- [ ] Comment section
- [ ] Share buttons (Twitter, copy link)

**Acceptance Criteria:**
- Market detail page shows all relevant data
- User can navigate to related markets

---

### Epic 7 — Cross-Venue Comparison (Optional)

**Goal:** Compare Polymarket prices with other prediction platforms.

**Tasks:**

#### External API Integration
- [ ] Research other platforms with public APIs (if any)
- [ ] Implement fetching logic

#### Comparison UI
- [ ] Show side-by-side price comparison
- [ ] Highlight arbitrage opportunities

**Acceptance Criteria:**
- (If implemented) Prices from multiple venues displayed

---

### Epic 8 — Observability & Polish

**Goal:** Production-ready monitoring and UX polish.

**Tasks:**

#### Logging
- [ ] Add structured logging (Winston, Pino)
- [ ] Log API errors, sync job results, alert triggers

#### Metrics
- [ ] Track:
  - Markets synced per hour
  - WebSocket connection uptime
  - Alert trigger rate

#### Error Handling
- [ ] Graceful error messages in UI
- [ ] Retry logic for API failures
- [ ] Rate limit awareness (especially CLOB API)

#### Performance
- [ ] Optimize database queries
- [ ] Add caching where appropriate
- [ ] Lazy load charts and order books

**Acceptance Criteria:**
- Application handles API failures gracefully
- Performance is acceptable (< 2s page loads)

---

## 4. Rough Implementation Order

1. **Epic 0** - Repo setup
2. **Epic 1** - Market sync
3. **Epic 2** - Real-time prices (WebSocket)
4. **Epic 4** - Charts (can do in parallel with Epic 3)
5. **Epic 3** - Order book
6. **Epic 6** - Market detail page
7. **Epic 5** - Watchlist & alerts
8. **Epic 8** - Observability & polish
9. **Epic 7** - Cross-venue comparison (optional)

---

## 5. Polymarket API References

| API/System | Purpose | Doc Link |
|------------|---------|----------|
| **Gamma Markets** | Market search, metadata | [`docs/polymarket-gamma.md`](../../polymarket-gamma.md) |
| **CLOB** | Order book, trades, prices | [`docs/polymarket-clob.md`](../../polymarket-clob.md) |
| **WebSocket** | Real-time price feeds | [`docs/polymarket-wss.md`](../../polymarket-wss.md) |
| **Subgraph** | Historical analytics | [`docs/polymarket-subgraph.md`](../../polymarket-subgraph.md) |

---

## 6. Next Steps

Once V0 is complete:
1. **V2 (Truth Feed)** - Add news-to-market correlation
2. **V1 (Sentiment)** - User sentiment polling layer
3. **V3 (Localization)** - Regional market insights
4. **V5 (Narratives)** - Narrative graph intelligence

---

**Last Updated:** November 23, 2025
