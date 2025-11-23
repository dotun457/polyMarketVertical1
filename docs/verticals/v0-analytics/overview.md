# V0 — Analytics Core

**Build Order:** Phase 1 (Foundation)  
**Status:** 🚧 In Progress  
**Priority:** Critical

---

## Purpose

V0 is the **foundation layer** — a Polysights-style market analytics dashboard that provides:
- Market discovery and search
- Real-time price charts
- Order book visualization
- Watchlists and alerts
- Cross-venue price comparison

This vertical establishes the core data infrastructure that all other verticals build upon.

---

## Core Features

### 1. Market Explorer
- Browse all Polymarket markets by category
- Search by keywords, tags, or topics
- Filter by status (open/closed/resolved)
- View market metadata (volume, liquidity, expiry)

### 2. Price Charts & History
- Real-time price updates via WebSocket
- Historical price data visualization
- Probability-over-time graphs
- Volume and trade history

### 3. Order Book
- Live bid/ask spreads
- Depth visualization
- Best prices and liquidity levels
- Trade execution insights

### 4. Watchlist
- User-curated market tracking
- Custom alerts (price thresholds, volume spikes)
- Portfolio overview
- Position tracking

### 5. Cross-Venue Comparison
- Compare Polymarket prices with other platforms
- Arbitrage opportunity detection
- Liquidity comparison

---

## Tech Stack

### Data Sources
- **Gamma Markets API** ([`docs/polymarket-gamma.md`](../../polymarket-gamma.md)) - Market metadata and search
- **CLOB API** ([`docs/polymarket-clob.md`](../../polymarket-clob.md)) - Order book, trades, prices
- **WebSocket** ([`docs/polymarket-wss.md`](../../polymarket-wss.md)) - Real-time price feeds
- **Subgraph** ([`docs/polymarket-subgraph.md`](../../polymarket-subgraph.md)) - Historical analytics

### Storage
- PostgreSQL for market cache
- Redis for real-time data
- TimescaleDB for price history (optional)

### Frontend
- Next.js / React
- TailwindCSS
- Chart libraries (Recharts, Lightweight Charts)

---

## Dependencies

### Horizontals Required
- **H0** - Database schema & models
- **H1** - API client layer (Gamma, CLOB, WSS)
- **H2** - Authentication & user management
- **H3** - Caching strategy
- **H4** - Frontend components library

---

## Key Metrics

Success indicators for V0:
- ✅ All active markets synced and searchable
- ✅ Real-time price updates < 1s latency
- ✅ Order book refresh rate < 500ms
- ✅ User can create watchlists and receive alerts
- ✅ Historical data available for charts

---

## Next Steps

Once V0 is complete:
1. **V2 (Truth Feed)** - Add news-to-market mapping
2. **V1 (Sentiment)** - Layer on user sentiment polling
3. **V3 (Localization)** - Regional market insights
4. **V5 (Narratives)** - Narrative graph intelligence

---

## Related Documentation

- [Development Plan](./dev-plan.md)
- [Verticals Overview](../verticals-overview.md)
- [Polymarket API Index](../../INDEX.md)
