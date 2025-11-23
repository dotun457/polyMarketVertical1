# 📁 Documentation Index

Quick reference guide to all Polymarket documentation files in this repository.

---

## 🗂️ File Structure

```
/
├── README.md                           # Main index and overview
└── docs/
    ├── polymarket-clob.md             # Central Limit Order Book API
    ├── polymarket-wss.md              # WebSocket real-time updates
    ├── polymarket-rtds.md             # Real-Time Data Stream
    ├── polymarket-gamma.md            # Gamma Markets API
    ├── polymarket-bridge.md           # Bridge & Swaps
    ├── polymarket-resolution.md       # UMA Resolution System
    ├── polymarket-subgraph.md         # GraphQL Analytics
    ├── polymarket-rewards.md          # Rewards System
    ├── polymarket-ctf.md              # Conditional Token Framework
    └── polymarket-neg-risk.md         # Negative Risk System
```

---

## 📚 Quick Links

### Trading & Markets
- **[CLOB](./polymarket-clob.md)** - Place orders, get orderbooks, manage trades
- **[Gamma](./polymarket-gamma.md)** - Search markets, get metadata, discover opportunities
- **[WebSocket](./polymarket-wss.md)** - Real-time price feeds and orderbook updates
- **[RTDS](./polymarket-rtds.md)** - Crypto prices and comment streams

### Smart Contracts
- **[CTF](./polymarket-ctf.md)** - Split, merge, and redeem outcome tokens
- **[Negative Risk](./polymarket-neg-risk.md)** - Efficient multi-outcome markets
- **[Resolution](./polymarket-resolution.md)** - UMA oracle integration

### Analytics & Data
- **[Subgraph](./polymarket-subgraph.md)** - GraphQL queries for positions and history
- **[Rewards](./polymarket-rewards.md)** - Track and claim liquidity rewards

### Infrastructure
- **[Bridge](./polymarket-bridge.md)** - Cross-chain deposits and swaps

---

## 🚀 Getting Started

### For AI Agents
When working with Polymarket APIs, reference these docs:

```
"Using docs/polymarket-clob.md, create a function to place a limit order"
"Using docs/polymarket-gamma.md, search for markets about Bitcoin"
"Using docs/polymarket-wss.md, subscribe to real-time price updates"
```

### For Developers

1. **Browse Markets**: Start with [Gamma](./polymarket-gamma.md)
2. **Place Orders**: Use [CLOB](./polymarket-clob.md)
3. **Real-time Data**: Connect via [WebSocket](./polymarket-wss.md)
4. **Analytics**: Query [Subgraph](./polymarket-subgraph.md)

---

## 📖 Documentation by Use Case

### Building a Trading Bot
1. [Gamma](./polymarket-gamma.md) - Find markets
2. [CLOB](./polymarket-clob.md) - Execute trades
3. [WebSocket](./polymarket-wss.md) - Monitor prices
4. [Subgraph](./polymarket-subgraph.md) - Track performance

### Market Making
1. [CLOB](./polymarket-clob.md) - Provide liquidity
2. [Rewards](./polymarket-rewards.md) - Earn incentives
3. [CTF](./polymarket-ctf.md) - Manage positions
4. [WebSocket](./polymarket-wss.md) - Stay updated

### Analytics Dashboard
1. [Gamma](./polymarket-gamma.md) - Market metadata
2. [Subgraph](./polymarket-subgraph.md) - Historical data
3. [RTDS](./polymarket-rtds.md) - Live sentiment
4. [Resolution](./polymarket-resolution.md) - Track outcomes

### Multi-Outcome Betting
1. [Negative Risk](./polymarket-neg-risk.md) - Efficient collateral
2. [CTF](./polymarket-ctf.md) - Token mechanics
3. [CLOB](./polymarket-clob.md) - Execute bets
4. [Bridge](./polymarket-bridge.md) - Fund account

---

## 🎯 Code Examples

Each documentation file contains:
- ✅ TypeScript implementations
- ✅ Python implementations
- ✅ REST API examples
- ✅ WebSocket patterns
- ✅ Best practices
- ✅ Common pitfalls

---

## 🔗 Official Resources

- **Polymarket Docs**: https://docs.polymarket.com/
- **API Reference**: https://docs.polymarket.com/api-reference
- **Discord**: https://discord.gg/polymarket
- **GitHub**: https://github.com/Polymarket

---

## 📝 Contributing

To add or update documentation:

1. Follow the existing structure
2. Include both TypeScript and Python examples
3. Add practical use cases
4. Link to related documentation
5. Update this index file

---

## ⚡ Quick Command Reference

### TypeScript Setup
```bash
npm install @polymarket/clob-client @apollo/client ethers
```

### Python Setup
```bash
pip install py-clob-client gql web3 requests
```

### Environment Variables
```bash
PRIVATE_KEY=your_private_key
RPC_URL=https://polygon-rpc.com
API_KEY=your_polymarket_api_key
```

---

**Last Updated**: November 22, 2025
