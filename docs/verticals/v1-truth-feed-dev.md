# V1 — Truth Feed Dev Plan (News ↔ Markets Engine)

**Vertical:** 1  
**Build Order:** Phase 1  
**Goal:** A working pipeline that:

1. Fetches news headlines continuously
2. Maps them to Polymarket markets (via Gamma)
3. Computes basic impact scores from price moves (via CLOB / price endpoints)
4. Exposes a simple “Truth Feed” interface (CLI / API / docs) that looks alive even with 0 users

---

## 0. Tech Assumptions (for this repo)

These are *implementation suggestions*, not hard constraints:

- **Language:** TypeScript (Node) or Python
- **Storage (for dev):** SQLite / Postgres (any RDBMS is fine, schema below)
- **Polymarket APIs:**
  - **Gamma Markets API** for markets & metadata → [`docs/polymarket-gamma.md`](../polymarket-gamma.md)
  - **CLOB/API (book / prices / trades)** for orderbook + prices + history → [`docs/polymarket-clob.md`](../polymarket-clob.md)  
- **News Provider:** any of:
  - NewsAPI, GDELT, Mediastack, or a simple RSS aggregator

You can refine these choices as you implement, but the **epics + schema** assume:

- one relational DB
- a small job scheduler (cron / GitHub Action / local script)

---

## 1. High-Level Architecture

**Flow:**

1. **Market Crawler**
   - Fetch market list + metadata from Gamma
   - Cache locally (ids, titles, categories, tags, active status)

2. **News Ingestion**
   - Periodically pull headlines from news provider
   - Store normalized `headlines` records

3. **Mapping Engine**
   - Convert market metadata → search queries
   - For each headline:
     - compute keyword & (later) semantic scores vs markets
     - store `headline_market_links`

4. **Impact Engine**
   - For each `headline_market_link`:
     - fetch market prices around headline time
     - compute simple `impact_score`

5. **Feed Interface**
   - CLI / simple API / markdown feed:
     - “Latest impactful headlines for market X”
     - “Global Truth Feed (sorted by impact desc, time desc)”

---

## 2. Data Model (Dev Schema)

You can adapt field names and types; this is a **starting point**.

### 2.1 `markets` (local cache from Gamma)

Pulled from Gamma `/markets`. :contentReference[oaicite:2]{index=2}

```sql
CREATE TABLE markets (
  id              TEXT PRIMARY KEY,         -- Gamma market.id
  question        TEXT NOT NULL,            -- market question / title
  event_id        TEXT,                     -- Gamma event id (if needed)
  category        TEXT,                     -- e.g. Politics, Crypto
  subcategory     TEXT,                     -- if available
  tags            TEXT,                     -- JSON array of tags/labels
  status          TEXT,                     -- open / closed / resolved etc.
  last_price_yes  REAL,                     -- cached implied prob
  last_price_no   REAL,                     -- alternatively 1 - yes
  last_synced_at  TIMESTAMP NOT NULL
);
2.2 headlines (news ingestion)
sql
Copy code
CREATE TABLE headlines (
  id              INTEGER PRIMARY KEY AUTOINCREMENT,
  external_id     TEXT,             -- id from news API, if any
  title           TEXT NOT NULL,
  description     TEXT,
  source_name     TEXT,
  source_domain   TEXT,
  url             TEXT,
  published_at    TIMESTAMP,        -- from provider
  fetched_at      TIMESTAMP NOT NULL,
  raw_json        TEXT              -- original provider response (optional)
);
CREATE INDEX idx_headlines_published_at ON headlines(published_at);
2.3 headline_market_links (mapping + impact)
sql
Copy code
CREATE TABLE headline_market_links (
  id                    INTEGER PRIMARY KEY AUTOINCREMENT,
  headline_id           INTEGER NOT NULL REFERENCES headlines(id),
  market_id             TEXT NOT NULL REFERENCES markets(id),

  -- Matching metadata
  match_method          TEXT NOT NULL,  -- 'keyword_v1', 'semantic_v1', etc.
  keyword_score         REAL,           -- simple overlap score
  semantic_score        REAL,           -- later, embedding-based
  combined_score        REAL,           -- final relevance score
  is_primary            BOOLEAN DEFAULT 0,  -- whether this is the "main" market

  -- Impact calculation
  prob_before           REAL,           -- YES prob before headline
  prob_after            REAL,           -- YES prob after headline
  impact_abs            REAL,           -- |after - before|
  impact_signed         REAL,           -- after - before
  price_window_before   INTERVAL,       -- config used, e.g. '30m'
  price_window_after    INTERVAL,       -- config used, e.g. '60m'

  created_at            TIMESTAMP NOT NULL,
  updated_at            TIMESTAMP NOT NULL
);

CREATE INDEX idx_links_market ON headline_market_links(market_id);
CREATE INDEX idx_links_headline ON headline_market_links(headline_id);
CREATE INDEX idx_links_score ON headline_market_links(combined_score DESC);
CREATE INDEX idx_links_impact ON headline_market_links(impact_abs DESC);
2.4 market_price_history (optional, but useful)
If you don’t want to hit CLOB / price endpoints every time, cache prices:

sql
Copy code
CREATE TABLE market_price_history (
  id              INTEGER PRIMARY KEY AUTOINCREMENT,
  market_id       TEXT NOT NULL REFERENCES markets(id),
  ts              TIMESTAMP NOT NULL,
  prob_yes        REAL NOT NULL,   -- from midprice / orderbook etc.
  prob_no         REAL NOT NULL,
  source          TEXT NOT NULL,   -- 'CLOB_book', 'Gamma_price', etc.

  UNIQUE(market_id, ts, source)
);
CREATE INDEX idx_prices_market_ts ON market_price_history(market_id, ts);
3. Epics & Tasks
Epic 0 — Repo & Config Setup
Goal: Make it trivial to run the Truth Feed pipeline locally.

Tasks:

 Add docs/verticals/v1-truth-feed-overview.md (done)

 Create this dev doc at docs/verticals/v1-truth-feed-dev.md

 Add /config/ or .env structure:

NEWS_API_KEY

NEWS_PROVIDER (e.g. newsapi, gdelt)

DB_URL

 Add simple Makefile or npm scripts:

bootstrap-db

run-news-job

run-mapping-job

print-feed

Acceptance Criteria:

New dev can clone repo, run one command, and be ready to run jobs.

Epic 1 — Gamma & Market Crawler
**Goal:** Have a local cache of Polymarket markets with enough metadata to generate search queries and map headlines.

Gamma provides markets & events with categories/tags. See [`docs/polymarket-gamma.md`](../polymarket-gamma.md)
Polymarket Documentation
+2
Polymarket Documentation
+2

Tasks:

Gamma Client

 Implement simple Gamma client:

GET https://gamma-api.polymarket.com/markets

Optional filters: active, category, etc.

 Add pagination support.

DB Integration

 Implement upsertMarketFromGamma(marketJson):

Insert new markets

Update existing ones (question, tags, status, last_price)

 Store tags/categories where available.

Market Sync Job

 CLI / script: sync-markets

 Strategy:

Full sync daily

Incremental sync more frequently (only open/active markets)

 Log counts and new markets detected.

Generate “Search Profile” per Market

 For each market, derive a small JSON:

json
Copy code
{
  "market_id": "123",
  "primary_query": "Will Biden win the 2028 US election?",
  "keywords": ["Biden", "US election", "2028", "president"],
  "entities": ["Joe Biden", "US Politics"]
}
 Store in DB (markets table extra columns or a market_search_profile column as JSON).

Acceptance Criteria:

Can run sync-markets and see markets table populated.

For any market_id you care about, you can inspect a derived search profile.

Epic 2 — News Fetch Engine
Goal: Continuously ingest headlines & store them in headlines.

Tasks:

Select Provider

 Decide: NEWS_PROVIDER = newsapi | gdelt | other.

 Document quota / pricing / rate limits in docs/verticals/v1-truth-feed-dev.md.

News Client

 Implement fetchHeadlines(params):

By keyword / topic

Or general top headlines

 Normalize to our internal structure (title, description, url, published_at, source).

News Ingestion Job

 ingest-headlines script:

Accepts optional --query or --category

Fetches headlines

De-duplicates by (source_domain, title, published_at) or external_id

Inserts into headlines.

Scheduling

 Decide how often to run:

Example: every 5–15 minutes in dev.

 For now, simple cron or manual invocation is fine.

Acceptance Criteria:

headlines table populated with real data.

Duplicate headlines are not spammed.

Epic 3 — Headline → Market Mapping (v1: Keywords)
Goal: A working, simple mapping engine connecting headlines to markets via keyword overlap, using the “search profile” from Epic 1.

Tasks:

Keyword Extraction

 For markets:

Use question text + tags

Basic text cleaning (lowercase, remove stopwords, etc.)

 For headlines:

Use title + description

Same text cleaning

 Implement helper: extractKeywords(text) -> [tokens].

Matching Algorithm v1

 For each new headline, compute a keyword overlap score vs a subset of markets:

naive version: intersect headline keywords with each market’s keywords

keyword_score = overlap_count / sqrt(len(headline_kw) * len(market_kw))

 Allow filtering by category:

e.g., for sports-related headlines, only compare vs sports markets.

Storing Links

 For each (headline, market) pair with keyword_score >= threshold:

Create headline_market_links row:

match_method = 'keyword_v1'

keyword_score = ...

combined_score = keyword_score (for now)

 Mark the highest scoring market per headline as is_primary = true.

CLI Tool

 map-headlines script:

Option 1: run for “unmapped” headlines only

Option 2: run for specific headline id

 Logging: print top matches per headline.

Acceptance Criteria:

For a known headline (e.g., “Biden says X about election”), you see it linked to the correct election-related market with a reasonable keyword score.

headline_market_links table contains mappings with keyword_v1.

Epic 4 — Impact Engine (Price Movement)
**Goal:** For each headline-market link, compute a basic impact score using prices from Polymarket APIs.

Polymarket CLOB API exposes orderbook + midprice endpoints; these can be used to derive implied probabilities. See [`docs/polymarket-clob.md`](../polymarket-clob.md)
Polymarket Documentation
+3
Polymarket Documentation
+3
Polymarket Documentation
+3

Tasks:

Price Fetch Helper

 Implement getMarketPriceAt(market_id, ts):

Option A (simple): sample a recent midprice around ts using CLOB book / price endpoints.

Option B (more robust): use a small local market_price_history filled by a separate job.

 For now, use:

“Nearest price in ±X minutes” heuristic if exact timestamp is not present.

Price History Job (Optional but Recommended)

 sync-market-prices script:

For all open markets:

Query CLOB /book or /price periodically.

Store implied prob_yes / prob_no in market_price_history.

- [ ] Respect CLOB rate limits for `/book`, `/prices`, etc. See [`docs/polymarket-clob.md`](../polymarket-clob.md)

Impact Calculation

For each headline_market_links row:

 Compute:

prob_before from time window (published_at - window_before)

prob_after from (published_at + window_after)

impact_abs = |prob_after - prob_before|

impact_signed = prob_after - prob_before

 Update row fields.

Default windows (tunable constants):

window_before = 30 minutes

window_after = 60 minutes

Impact Job

 compute-impact script:

Finds headline_market_links rows with impact_abs IS NULL

Runs impact calculation

Logs top movers.

Acceptance Criteria:

For a headline clearly coinciding with a big move, impact_abs is meaningfully > 0.

For random low-news markets, many links have small or near-zero impact_abs.

Epic 5 — Truth Feed Interface (Dev-Mode UI)
Goal: Expose a simple way to see the Truth Feed, even before a full front-end is built.

Tasks:

CLI Feed

 print-feed --limit 20:

Prints latest 20 links sorted by published_at DESC or impact_abs DESC.

Example output line:

text
Copy code
[2025-11-22T13:10Z] (impact +0.12) "Biden announces X" -> Market: "Will Biden win 2028 US election?"
URL: https://...
 Options:

--market-id

--min-impact

--since 2025-11-01

Markdown Export

 Script: export-feed-md:

Writes docs/feed/latest-truth-feed.md with a simple list/table:

Date

Headline (linked)

Market

Impact

(Optional) Minimal JSON API

If you want to serve this from a dev server:

 GET /api/truth-feed?limit=50&minImpact=0.05

JSON with list of {headline, market, impact} objects.

Acceptance Criteria:

A human (or your future agent) can glance at CLI or markdown and see the feed as a timeline of news → market reactions.

Epic 6 — MCP / Agent Integration Hooks
Goal: Make Truth Feed directly usable by AI agents (Copilot / Cursor / ChatGPT with MCP).

Polymarket already provides an open-source agents repo with standardized connectors (e.g., GammaMarketClient) that you can mirror or adapt. 
GitHub

Tasks:

Define Tools

 Add MCP tool definitions for:

list_markets(query?)

list_headlines(limit?, since?)

list_headlines_for_market(market_id, limit?)

get_truth_feed(limit?, min_impact?)

Wire to Implementation

 For each MCP tool, connect to:

DB queries (markets, headlines, links)

Simple filtering logic

Prompt Examples

 In ReadME.md add a section:

“Using Truth Feed with your AI agent”

 Prompt examples:

“List the most impactful headlines connected to the US election markets in the last 24 hours.”

“Explain why the probability moved for market X yesterday based on Truth Feed.”

Acceptance Criteria:

From VS Code / Copilot Agent, you can type:

“@agent show me the top headlines impacting ETH-related markets today”

and get back Truth Feed data.

Epic 7 — Observability & Guardrails
Goal: Make the pipeline debuggable and safe-ish from day one.

Tasks:

Logging

 Standardized logging pattern:

job name

counts (fetched, inserted, mapped)

error summaries

Metrics (even if just logs)

 Track:

headlines per day

mapped headlines per day

avg links per headline

avg impact_abs per link

#### Rate Limit Awareness
- [ ] Ensure Gamma & CLOB calls respect documented rate limits. See [`docs/polymarket-clob.md`](../polymarket-clob.md)
- [ ] Add backoff / retry behavior.

Config & Secrets

 .env.example with fake values.

 Clear docs for how to set up NEWS_API_KEY and DB_URL.

Acceptance Criteria:

You can run any job and see understandable logs.

You don’t accidentally nuke your rate limits.

4. Rough Implementation Order (Inside V1)
Within Vertical 1, a realistic dev flow:

Epic 0 + 1

Repo config + Gamma market sync

Epic 2

News ingestion (no mapping yet)

Epic 3

Keyword-based mapping engine

Epic 4

Impact engine & price caching

Epic 5

CLI / markdown Truth Feed

Epic 6

MCP tools so agents can use it

Epic 7

Logging / metrics / polish

At the end of this, you have a real, tangible V1 where:

Markets are pulled from Polymarket

Headlines stream in

Links show which markets each headline touches

Impact scores quantify how much markets reacted

A feed view makes the app feel alive

This is the foundation for **V2 (Sentiment Layer)**, **V3 (Localization)**, and eventually **V5/V6 (Narrative Graph & Agents)**.

---

## 5. Polymarket API References

Quick links to the comprehensive docs in this repo:

| API/System | Purpose | Doc Link |
|------------|---------|----------|
| **Gamma Markets** | Search, metadata, market discovery | [`docs/polymarket-gamma.md`](../polymarket-gamma.md) |
| **CLOB** | Orders, orderbook, trades, prices | [`docs/polymarket-clob.md`](../polymarket-clob.md) |
| **WebSocket** | Real-time price feeds | [`docs/polymarket-wss.md`](../polymarket-wss.md) |
| **RTDS** | Real-time data streams | [`docs/polymarket-rtds.md`](../polymarket-rtds.md) |
| **Subgraph** | GraphQL analytics layer | [`docs/polymarket-subgraph.md`](../polymarket-subgraph.md) |
| **CTF** | Conditional tokens framework | [`docs/polymarket-ctf.md`](../polymarket-ctf.md) |

---

## 6. Next Steps

Once V1 is complete:

1. **Test with real data** — Run for 24-48 hours, validate mappings and impact scores
2. **Refine thresholds** — Tune `keyword_score` thresholds and impact windows
3. **Add sentiment** — Move to V2 (Sentiment Layer)
4. **Build UI** — Create a simple web dashboard for the Truth Feed
5. **Deploy** — Host the pipeline on a server with scheduled jobs

---

**Last Updated:** November 22, 2025
