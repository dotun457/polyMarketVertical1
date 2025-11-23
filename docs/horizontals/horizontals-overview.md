# Horizontals Overview

Horizontals are platform capabilities that support **multiple verticals**.

---

## **H0 – Core Infrastructure**
- Monorepo structure  
- Env configuration  
- Logging  
- Error monitoring  
- Basic health checks  

Used: V0 → V5

---

## **H1 – Data Normalization**
- Market schema  
- Canonical mapping rules  
- Price normalization  
- Volume normalization  
- Divergence calculations  

Used: V0, V2, V1, V3, V5

---

## **H2 – Authentication & User Profiles**
- Supabase/Auth provider  
- Region preference  
- Watchlist + portfolio  
- Truth Token balance  

Used: V0, V1, V3, V5

---

## **H3 – Notifications & Alerts**
- Alert creation API  
- Email/push notifications  
- Market movement triggers  

Used: V0 → V5

---

## **H4 – Frontend Component Library**
Core shared UI components:
- Market Card  
- Narrative Card  
- Truth Poll Widget  
- Headline Chip  
- Region Badge  
- Divergence Badge  
- Chart Components  

Used everywhere.

---

## **H5 – Jobs & Ingest Pipelines**
- Polymarket ingest  
- News ingest  
- Price ticks  
- Poll aggregation  
- Narrative update job  

Introduced in V2, expanded in V5.

---

## **H6 – Agent Layer (Tools + Prompts)**
- Agent tool APIs  
- System prompts  
- Narrative persona engines  
- Prediction Market Copilot  

Activated during V5.

---

## Horizontal Introduction Table

| Vertical | H0 | H1 | H2 | H3 | H4 | H5 | H6 |
|---------|----|----|----|----|----|----|----|
| V0      | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |    |    |
| V2      | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |    |
| V1      | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |    |
| V3      | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |    |
| V5      | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |

