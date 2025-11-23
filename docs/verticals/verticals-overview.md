# Verticals Overview

Prediction Router is built from **bottom to top**, with each vertical providing a new data layer or intelligence layer.

## Vertical Order (Critical)
**We build in this sequence:**

1. **V0 – Analytics Core (Polysights Clone)**  
   Foundation: market ingestion, charts, watchlists, alerts.

2. **V2 – Truth Feed (News ↔ Market)**  
   Map headlines to markets, compute impact.

3. **V1 – Sentiment Layer**  
   Truth Token polls + stake-intent + region tagging.

4. **V3 – Localization Engine**  
   Regional divergence, region heatmaps, regional demand.

5. **V5 – Narrative Intelligence (Narrative Graph)**  
   Canonical market grouping → composite probability → belief history → narrative bonds → narrative feed → narrative agents.

6. **V4 – Exchange Layer (Future)**  
   Local markets when licensing is acquired.

---

## The Core Idea
**Narrative Intelligence  
powered by Prediction Data  
feeding AI Agents.**

We transform:
- markets  
- news  
- sentiment  
- regional signals  

…into **structured narratives** that AI agents can reason about, explain, and compare.

---

## Summary of Verticals

### **V0 – Analytics Core**
- Polysights-style market explorer  
- Price charts  
- Price ticks  
- Watchlist  
- Alerts  
- Cross-venue compare  

### **V2 – Truth Feed**
- Fetch news (AP/NewsAPI/GDELT)  
- Map headlines → markets  
- Compute Δ-probability impact  
- Truth Feed UI  

### **V1 – Sentiment Layer**
- Truth Token polls  
- User sentiment  
- Regional sentiment  
- Stake-intent as soft-weight  

### **V3 – Localization Engine**
- Region-level belief  
- Divergence between local/global  
- “What would you like to see?” demand mapping  
- Region hot topics  

### **V5 – Narrative Intelligence**
- Canonical grouping of related markets  
- Composite probability (volume-weighted)  
- Narrative object (markets + sentiment + news + regions)  
- Narrative Bond (belief over time)  
- Narrative Feed  
- Narrative Agents  

### **V4 – Exchange Layer (Later)**
- Future regulated local markets.  
- Out of scope for early builds.

---

## Vertical Dependencies → Horizontals
Each vertical introduces new horizontals:

| Vertical | Needed Horizontals |
|---------|---------------------|
| V0 | H0, H1, H2, H3, H4 |
| V2 | H5 (news ingest), H1 |
| V1 | H2, H3, H4 |
| V3 | H1, H5 |
| V5 | H1, H5, H6 |
| V4 | (optional future) |

See `../horizontals/horizontals-overview.md` for definitions.

