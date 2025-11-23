# V1 — Truth Feed (News ↔ Markets Engine)

**Vertical:** 1  
**Build Order:** Phase 1  
**Purpose:** Make the app feel alive — powered by world events, not user activity.

---

# 🎯 Mission

Truth Feed maps **real-world news → prediction markets**.

It is the first vertical because it:
- continuously updates markets with context,
- requires ZERO user activity to function,
- turns the app into a living, breathing system on day one.

This solves the cold-start problem for the entire product.

---

# 🧩 What Truth Feed Does

## 1. Fetch News
- Choose a provider (NewsAPI, GDELT, Mediastack)
- Store headlines with timestamps, URLs, sources
- De-duplicate and normalize

## 2. Map Headlines → Markets
- keyword matching
- semantic embeddings
- entity extraction (“Biden”, “Ethereum”, “OpenAI”)
- store many-to-many links

## 3. Compute Impact Scores
Compare market probability before and after a headline:


Later versions will use:
- regression-adjusted impact  
- multi-headline reinforcement  
- narrative activation thresholds  

---

# 🔥 Why V1 Comes First

- Markets look reactive  
- News drives visible price movement  
- App has a heartbeat  
- Users immediately see something valuable  
- Sets up all later verticals

Truth Feed = foundation for:
- V2: Sentiment Layer  
- V3: Localization  
- V5: Narrative Graph  
- V6: Narrative Agents  

---

# 📦 Components

## Milestone 1 — News Fetch Engine
- Decide provider  
- Create `headlines` table  
- Fetch + store  
- Market-specific search queries  
- Deduplication  

## Milestone 2 — Headline-Market Mapping
- keyword matching v1  
- semantic scoring v2  
- `headline_market_links` table  
- impact scores  

## Milestone 3 — Truth Feed UI
- Feed page  
- Market detail → “News” tab  
- Headline chips  
- Filtering by impact, recency, category  

---

# 🟢 Definition of Done

Truth Feed is DONE when:

1. Headlines fetch correctly  
2. Mappings appear under each market  
3. Impact scores are visible  
4. Feed UI updates live  
5. App looks alive **even with 0 users**

---

# 🔮 Future Connection to V5 & V6

Truth Feed is the backbone of:
- **Narrative Graph** (V5)  
- **Narrative Agents** (V6)

Headlines become narrative nodes.
Market reactions become narrative movement.
Agents use these signals as reasoning tools.

Truth Feed is the **heartbeat** of the entire ecosystem.

