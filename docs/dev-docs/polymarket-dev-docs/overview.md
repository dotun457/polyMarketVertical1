# 📘 Polymarket Developer Reference# Polymarket Dev Playground



This repository contains a curated, structured reference of Polymarket Developer Documentation, designed for:This repo is a sandbox for building agents and services on top of the **Polymarket API**.



- **GitHub Copilot Agent**The goal:  

- **VS Code OpenAI MCP agents**Use VS Code's AI agent + MCP tools to explore, trade (in dev), and prototype “bets-as-a-service” and Polymarket-based agents.

- **LLM-driven development**

- **Bots and agents that interact with Polymarket**---

- **Your future WhatsApp → Polymarket trader agent**

## 1. Key Links

This README acts as the top-level index for all major Polymarket components:

CLOB, Gamma, RTDS, WebSocket, Bridge, Subgraph, Rewards, CTF, and more.- **Polymarket Docs Home**: https://docs.polymarket.com/

Use this file as your main AI context anchor inside VS Code.- **Developer Quickstart**: https://docs.polymarket.com/developer-quickstart

- **CLOB (Central Limit Order Book)**: https://docs.polymarket.com/clob/introduction

---- **WebSocket / RTDS**: https://docs.polymarket.com/developers/CLOB/websocket/wss-overview

- **Gamma / Markets**: https://docs.polymarket.com/gamma/overview

## 🚀 1. Overview- **Conditional Tokens**: https://docs.polymarket.com/conditional-token-frameworks/overview

- **Builders Program**: https://docs.polymarket.com/polymarket-builders-program/builder-program-introduction

Polymarket provides an extensive set of APIs and systems to:

---

- Fetch markets

- Stream real-time prices## 2. Local Knowledge for the AI Agent

- Submit orders to the CLOB

- Monitor trades and liquidityThe AI agent (Copilot, Cursor, etc.) should read these files first:

- Query market metadata via Gamma

- Fetch sports and series data- `docs/polymarket-index.md` – overview of all Polymarket doc sections.

- Resolve markets via UMA- `docs/polymarket-orders.md` – placing/cancelling orders, reward scoring.

- Access analytics via Subgraph- `docs/polymarket-wss.md` – WebSocket channels, RTDS, auth.

- Manage conditional tokens- `docs/polymarket-gamma.md` – market structure, endpoints, fetching markets.

- Bridge funds from other chains- `docs/polymarket-conditional-tokens.md` – splitting, merging, redeeming.



This reference organizes all the major documentation endpoints so your AI tooling has a stable foundation.When asking the agent Polymarket questions, prefer prompts like:



---> “Using the docs in `docs/polymarket-orders.md`, generate a TypeScript helper to place and cancel orders via the Polymarket CLOB.”



## 🧱 2. CLOB (Central Limit Order Book)---



### 2.1 Core Concepts## 3. MCP Tools



- **CLOB Introduction**  This workspace is intended to use the following MCPs:

  https://docs.polymarket.com/developers/CLOB/introduction

1. **polymarket-mcp**

- **CLOB Clients**     - Repo: https://github.com/berlinbra/polymarket-mcp

  https://docs.polymarket.com/developers/CLOB/clients   - Capabilities:

     - Fetch markets, events, trades, orderbooks

### 2.2 Orderbook     - (Optionally) place orders in dev/test

   - Usage examples (for the agent):

- **Get Order Book**       - “Use polymarket-mcp to list markets on [topic].”

  https://docs.polymarket.com/api-reference/orderbook/get-order-book-     - “Get the orderbook for market X and suggest a limit order.”



- **Get Bid/Ask Spreads**  2. **web-search MCP**

  https://docs.polymarket.com/api-reference/spreads/get-bid-ask-spreads   - Generic web search to hit `docs.polymarket.com` and related resources.

   - Usage:

### 2.3 Orders     - “Search docs.polymarket.com for ‘RTDS Crypto Prices’ and summarize.”

     - “Find the Polymarket docs section on ‘Negative Risk’ and explain it.”

- **Orders Overview**  

  https://docs.polymarket.com/developers/CLOB/orders/orders---



- **Trades Overview**  ## 4. Environment Setup

  https://docs.polymarket.com/developers/CLOB/trades/trades-overview

Basic TypeScript SDK usage:

📄 **Detailed Reference**: [`docs/polymarket-clob.md`](docs/polymarket-clob.md)

```bash

---npm install @polymarket/sdk


## 🔴 3. WebSocket (WSS)

### 3.1 Core WebSocket Docs

- **WSS Overview**  
  https://docs.polymarket.com/developers/CLOB/websocket/wss-overview

- **WSS Quickstart**  
  https://docs.polymarket.com/quickstart/websocket/WSS-Quickstart

- **WSS Authentication**  
  https://docs.polymarket.com/developers/CLOB/websocket/wss-auth

**Use WSS for:**
- Realtime orderbook updates
- Market price updates
- User fills
- Trade stream events

📄 **Detailed Reference**: [`docs/polymarket-wss.md`](docs/polymarket-wss.md)

---

## 🟩 4. Real-Time Data Stream (RTDS)

### 4.1 RTDS Systems

- **RTDS Overview**  
  https://docs.polymarket.com/developers/RTDS/RTDS-overview

- **RTDS Crypto Prices**  
  https://docs.polymarket.com/developers/RTDS/RTDS-crypto-prices

- **RTDS Comments**  
  https://docs.polymarket.com/developers/RTDS/RTDS-comments

📄 **Detailed Reference**: [`docs/polymarket-rtds.md`](docs/polymarket-rtds.md)

---

## 🟦 5. Gamma Markets API (Search, Metadata, Market Discovery)

Gamma powers:
- Market search
- Market metadata
- Event structures
- Tags, series, teams
- Filtering
- Categorization

### 5.1 Gamma Docs

- **Gamma Overview**  
  https://docs.polymarket.com/developers/gamma-markets-api/overview

- **Gamma Structure**  
  https://docs.polymarket.com/developers/gamma-markets-api/gamma-structure

- **Fetching Markets Guide**  
  https://docs.polymarket.com/developers/gamma-markets-api/fetch-markets-guide

### 5.2 Example API References

- **List Teams (Sports)**  
  https://docs.polymarket.com/api-reference/sports/list-teams

- **List Series**  
  https://docs.polymarket.com/api-reference/series/list-series?playground=open

📄 **Detailed Reference**: [`docs/polymarket-gamma.md`](docs/polymarket-gamma.md)

---

## 🟫 6. Bridge & Swaps

Used for deposits and cross-chain transfers.

- **Bridge API Overview**  
  https://docs.polymarket.com/developers/misc-endpoints/bridge-overview

Can be repurposed for payment-like flows or onboarding UX.

📄 **Detailed Reference**: [`docs/polymarket-bridge.md`](docs/polymarket-bridge.md)

---

## 🟥 7. Resolution (Oracle Layer via UMA)

Polymarket relies on UMA's Optimistic Oracle (OO) for market resolution.

- **UMA Resolution System**  
  https://docs.polymarket.com/developers/resolution/UMA

**Covers:**
- Price requests
- Disputes
- DVM voting
- Polymarket's UmaCtfAdapter

📄 **Detailed Reference**: [`docs/polymarket-resolution.md`](docs/polymarket-resolution.md)

---

## 🟨 8. Subgraph (GraphQL Analytics Layer)

- **Subgraph Overview**  
  https://docs.polymarket.com/developers/subgraph/overview

**Provides:**
- Market positions
- Volume data
- Liquidity
- Market history
- Real-time updates

Powered by Goldsky (public).

📄 **Detailed Reference**: [`docs/polymarket-subgraph.md`](docs/polymarket-subgraph.md)

---

## 🟧 9. Rewards

- **Rewards Overview**  
  https://docs.polymarket.com/developers/rewards/overview

**Includes:**
- Liquidity rewards
- Market maker incentives

📄 **Detailed Reference**: [`docs/polymarket-rewards.md`](docs/polymarket-rewards.md)

---

## 🟪 10. Conditional Token Framework (CTF)

- **CTF Overview**  
  https://docs.polymarket.com/developers/CTF/overview

**CTF covers:**
- Splitting
- Merging
- Redeeming
- Outcome tokens
- Market resolution logic

📄 **Detailed Reference**: [`docs/polymarket-ctf.md`](docs/polymarket-ctf.md)

---

## 🟩 11. Proxy Wallet System

- **Proxy Wallet Overview**  
  https://docs.polymarket.com/developers/proxy-wallet

**Allows:**
- Non-custodial trading abstraction
- Session wallet experience

---

## 🟦 12. Negative Risk

- **Negative Risk Overview**  
  https://docs.polymarket.com/developers/neg-risk/overview

**Covers:**
- Augmented negative risk
- Placeholder outcomes
- Original outcomes
- Explicit other

📄 **Detailed Reference**: [`docs/polymarket-neg-risk.md`](docs/polymarket-neg-risk.md)

---

## ⚙️ 13. Repository Structure

```
/docs
  polymarket-clob.md
  polymarket-wss.md
  polymarket-rtds.md
  polymarket-gamma.md
  polymarket-bridge.md
  polymarket-resolution.md
  polymarket-subgraph.md
  polymarket-rewards.md
  polymarket-ctf.md
  polymarket-neg-risk.md

README.md  <-- (this file)
```

Ideal for VS Code AI indexing, MCP agents, and Copilot Agent navigation.

---

## 🧠 14. Usage With VS Code / Copilot Agent

In VS Code, you can now say:

- _"Using the Gamma Guide, build a search function to list markets by team."_
- _"Using the WSS Overview, generate a TS client that subscribes to real-time price updates."_
- _"Using the CLOB Orders docs, create a function to place a limit order."_
- _"Using the Subgraph docs, generate a query to fetch all active positions for a user."_

Because the README exposes canonical links, your AI agent has a reliable base context.

---

## 🔧 15. Environment Setup

Basic TypeScript SDK usage:

```bash
npm install @polymarket/sdk
```

---

## ✔️ Next Steps

This README provides the foundation. The detailed documentation files in `/docs` contain:
- Code examples (TypeScript/Python)
- API endpoints
- WebSocket channel details
- GraphQL queries
- Best practices

Start exploring with your AI agent using the references above!


Narrative Intelligence
powered by Prediction Data
feeding AI Agents.

Markets → News → Sentiment → Regions → Narrative Graph → Agents