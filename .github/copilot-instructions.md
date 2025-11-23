# GitHub Copilot Instructions

## 1. Context for All Conversations

When helping with code or architecture in this repository, **always treat the docs in `docs/` as canonical**, especially:

- `README.md` – high-level project overview
- `docs/verticals/` – feature verticals (V0–V5) and dev plans
- `docs/horizontals/` – cross-cutting concerns
- `docs/backend/` – backend architecture and patterns
- `docs/dev-docs/` – roadmap, milestones, Polymarket dev references

This is a **prediction market analytics & narrative intelligence platform** built on Polymarket data and other sources.

The high-level idea is:

> Narrative Intelligence  
> powered by Prediction Data  
> feeding AI Agents.

The vertical build order is: **V0 → V2 → V1 → V3 → V5**.

---

## 2. Required Documentation Context

### 2.1 Project Overview & Architecture

Before making structural suggestions or larger changes, Copilot should read:

- `README.md`  
- `docs/verticals/verticals-overview.md`  
- `docs/horizontals/horizontals-overview.md`  
- `docs/dev-docs/development-roadmap.md`  
- `docs/dev-docs/milestones.md`

### 2.2 Vertical & Horizontal Design

When working on features for specific areas:

- For **V0 (Analytics Core)**, **V1 (Sentiment)**, **V2 (Truth Feed)**, **V3 (Localization)**, or **V5 (Narratives)**:
  - Check the corresponding folder in `docs/verticals/` (e.g. `v0-analytics/overview.md`, `dev-plan.md`, etc.)

- For cross-cutting concerns:
  - Read `docs/horizontals/horizontals-overview.md`
  - Then check the relevant horizontal (H0–H6)

### 2.3 Polymarket API Documentation

When discussing Polymarket APIs, always reference these files under:

`docs/dev-docs/polymarket-dev-docs/`

- **CLOB API**: `polymarket-clob.md` – orders, orderbook, trades, auth  
- **Gamma Markets**: `polymarket-gamma.md` – market search, metadata, discovery  
- **WebSocket (WSS)**: `polymarket-wss.md` – real-time price feeds  
- **RTDS**: `polymarket-rtds.md` – real-time data streams  
- **Subgraph**: `polymarket-subgraph.md` – GraphQL analytics  
- **Bridge**: `polymarket-bridge.md` – cross-chain deposits and swaps  
- **UMA Resolution**: `polymarket-resolution.md` – oracle and market resolution  
- **CTF**: `polymarket-ctf.md` – Conditional Token Framework  
- **Negative Risk**: `polymarket-neg-risk.md` – multi-outcome market system  
- **Rewards**: `polymarket-rewards.md` – liquidity rewards and incentives  
- **INDEX**: `INDEX.md` – navigation entry point

---

## 3. Code Generation Guidelines

When generating or modifying code:

1. **Consult docs first**
   - For Polymarket: check the appropriate `polymarket-*.md` file.
   - For architecture/flow: check relevant vertical/horizontal docs.

2. **Use TypeScript by default**
   - Prefer strong typing and shared types if present (e.g. `packages/shared` once added).

3. **Honor documented patterns**
   - Authentication flows from CLOB docs  
   - Recommended endpoints and pagination patterns  
   - Rate limits and batching behavior

4. **Error handling**
   - Include meaningful error handling and logging around external calls.
   - Respect network and rate-limit errors.

---

## 4. When Answering Repo Questions

When the user asks Copilot something in this repo:

- If the question is **Polymarket API-related**:
  - Explicitly reference the relevant `docs/dev-docs/polymarket-dev-docs/polymarket-*.md` file.
  - Prefer code patterns and examples that mirror the official docs.

- If the question is **product / system design-related**:
  - Reference:
    - `docs/verticals/verticals-overview.md`
    - The specific vertical’s `overview.md` / `dev-plan.md`
    - `docs/horizontals/horizontals-overview.md`
    - `docs/dev-docs/development-roadmap.md`

- If the question is **backend**:
  - Check `docs/backend/` for `api-contracts.md`, `db-schema.md`, `jobs.md` (when present).

---

## 5. Quick Reference Paths

```txt
README.md                     # Project overview
docs/
  dev-docs/
    development-roadmap.md    # Phase-by-phase plan
    milestones.md             # Milestone checklist
    polymarket-dev-docs/      # Polymarket API docs
  verticals/
    verticals-overview.md     # V0–V5 summary
    v0-analytics/             # Polysights clone
    v1-sentiment/
    v2-truth-feed/
    v3-localization/
    v5-narratives/
  horizontals/
    horizontals-overview.md   # H0–H6 summary
  backend/
    api-contracts.md          # API shapes
    db-schema.md              # DB tables and relations
    jobs.md                   # Ingest and cron jobs
