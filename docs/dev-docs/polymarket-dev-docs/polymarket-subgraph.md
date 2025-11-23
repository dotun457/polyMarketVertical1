# 🟨 Subgraph (GraphQL Analytics Layer)

Polymarket's Subgraph provides a GraphQL interface for querying market data, positions, and historical analytics.

---

## 📚 Official Documentation

- **Subgraph Overview**: https://docs.polymarket.com/developers/subgraph/overview

---

## 🎯 Key Concepts

### What is the Subgraph?

The Subgraph is a GraphQL API that provides:
- **Market positions** and balances
- **Volume and liquidity** data
- **Trade history** and execution
- **User portfolio** tracking
- **Market analytics** and metrics

**Powered by**: Goldsky (public hosted service)

### GraphQL Endpoint

```
https://api.goldsky.com/api/public/project_clhk16b61ay9t49vm6ntn4mkz/subgraphs/polymarket/prod/gn
```

---

## 💻 TypeScript Implementation

### Setup GraphQL Client

```bash
npm install @apollo/client graphql
```

### Create Subgraph Client

```typescript
import { ApolloClient, InMemoryCache, gql, HttpLink } from '@apollo/client';

const SUBGRAPH_URL = 'https://api.goldsky.com/api/public/project_clhk16b61ay9t49vm6ntn4mkz/subgraphs/polymarket/prod/gn';

const client = new ApolloClient({
  link: new HttpLink({ uri: SUBGRAPH_URL }),
  cache: new InMemoryCache(),
});

class PolymarketSubgraph {
  private client: ApolloClient<any>;

  constructor() {
    this.client = client;
  }

  async query<T = any>(query: string, variables?: any): Promise<T> {
    const result = await this.client.query({
      query: gql(query),
      variables,
    });
    return result.data;
  }
}

export const subgraph = new PolymarketSubgraph();
```

---

## 📊 Common Queries

### Get Market Data

```typescript
const GET_MARKET_QUERY = gql`
  query GetMarket($id: ID!) {
    market(id: $id) {
      id
      question
      outcomes
      outcomePrices
      volume
      liquidity
      creationTimestamp
      endTimestamp
      resolved
      payoutReported
      positions {
        id
        user {
          id
          address
        }
        outcome
        quantityBought
        quantitySold
        netQuantity
      }
    }
  }
`;

async function getMarket(marketId: string) {
  const data = await subgraph.query(GET_MARKET_QUERY, { id: marketId });
  
  console.log('📊 Market:', data.market.question);
  console.log('💰 Volume:', data.market.volume);
  console.log('💧 Liquidity:', data.market.liquidity);
  
  return data.market;
}
```

### Get User Positions

```typescript
const GET_USER_POSITIONS_QUERY = gql`
  query GetUserPositions($userAddress: String!) {
    user(id: $userAddress) {
      id
      address
      positions {
        id
        market {
          id
          question
        }
        outcome
        quantityBought
        quantitySold
        netQuantity
        realizedProfit
        unrealizedProfit
      }
    }
  }
`;

async function getUserPositions(userAddress: string) {
  const data = await subgraph.query(GET_USER_POSITIONS_QUERY, {
    userAddress: userAddress.toLowerCase(),
  });

  console.log(`👤 Positions for ${userAddress}:`);
  
  data.user.positions.forEach((position: any) => {
    console.log(`\n📈 ${position.market.question}`);
    console.log(`  Outcome: ${position.outcome}`);
    console.log(`  Net Quantity: ${position.netQuantity}`);
    console.log(`  Realized P&L: ${position.realizedProfit}`);
  });

  return data.user.positions;
}
```

### Get Trade History

```typescript
const GET_TRADES_QUERY = gql`
  query GetTrades($marketId: ID!, $first: Int = 100) {
    trades(
      where: { market: $marketId }
      first: $first
      orderBy: timestamp
      orderDirection: desc
    ) {
      id
      user {
        id
        address
      }
      market {
        id
        question
      }
      outcome
      type
      price
      amount
      timestamp
      transactionHash
    }
  }
`;

async function getMarketTrades(marketId: string, limit: number = 100) {
  const data = await subgraph.query(GET_TRADES_QUERY, {
    marketId,
    first: limit,
  });

  console.log(`📜 Recent trades for market ${marketId}:`);
  
  data.trades.forEach((trade: any) => {
    const date = new Date(parseInt(trade.timestamp) * 1000);
    console.log(`\n${date.toLocaleString()}`);
    console.log(`  ${trade.type}: ${trade.amount} @ ${trade.price}`);
    console.log(`  User: ${trade.user.address}`);
  });

  return data.trades;
}
```

### Get Market Statistics

```typescript
const GET_MARKET_STATS_QUERY = gql`
  query GetMarketStats($marketId: ID!) {
    market(id: $marketId) {
      id
      question
      volume
      liquidity
      tradesCount
      uniqueTraders
      outcomes
      outcomePrices
      creationTimestamp
      endTimestamp
    }
  }
`;

async function getMarketStatistics(marketId: string) {
  const data = await subgraph.query(GET_MARKET_STATS_QUERY, { id: marketId });
  const market = data.market;

  const stats = {
    question: market.question,
    volume: parseFloat(market.volume),
    liquidity: parseFloat(market.liquidity),
    tradesCount: market.tradesCount,
    uniqueTraders: market.uniqueTraders,
    avgTradeSize: parseFloat(market.volume) / market.tradesCount,
    marketAge: Date.now() / 1000 - parseInt(market.creationTimestamp),
    outcomes: market.outcomes,
    prices: market.outcomePrices,
  };

  console.log('📊 Market Statistics:');
  console.log(`  Volume: $${stats.volume.toLocaleString()}`);
  console.log(`  Liquidity: $${stats.liquidity.toLocaleString()}`);
  console.log(`  Trades: ${stats.tradesCount}`);
  console.log(`  Unique Traders: ${stats.uniqueTraders}`);
  console.log(`  Avg Trade Size: $${stats.avgTradeSize.toFixed(2)}`);

  return stats;
}
```

### Get Top Markets by Volume

```typescript
const GET_TOP_MARKETS_QUERY = gql`
  query GetTopMarkets($first: Int = 10) {
    markets(
      first: $first
      orderBy: volume
      orderDirection: desc
      where: { resolved: false }
    ) {
      id
      question
      volume
      liquidity
      outcomePrices
      tradesCount
    }
  }
`;

async function getTopMarkets(limit: number = 10) {
  const data = await subgraph.query(GET_TOP_MARKETS_QUERY, { first: limit });

  console.log(`🏆 Top ${limit} Markets by Volume:`);
  
  data.markets.forEach((market: any, index: number) => {
    console.log(`\n${index + 1}. ${market.question}`);
    console.log(`   Volume: $${parseFloat(market.volume).toLocaleString()}`);
    console.log(`   Liquidity: $${parseFloat(market.liquidity).toLocaleString()}`);
    console.log(`   Trades: ${market.tradesCount}`);
  });

  return data.markets;
}
```

---

## 🐍 Python Implementation

### Setup

```bash
pip install gql aiohttp
```

### Create Subgraph Client

```python
from gql import gql, Client
from gql.transport.aiohttp import AIOHTTPTransport

SUBGRAPH_URL = "https://api.goldsky.com/api/public/project_clhk16b61ay9t49vm6ntn4mkz/subgraphs/polymarket/prod/gn"

transport = AIOHTTPTransport(url=SUBGRAPH_URL)
client = Client(transport=transport, fetch_schema_from_transport=True)

class PolymarketSubgraph:
    def __init__(self):
        self.client = client
    
    def query(self, query_string: str, variables: dict = None):
        query = gql(query_string)
        result = self.client.execute(query, variable_values=variables)
        return result

subgraph = PolymarketSubgraph()
```

### Get User Positions

```python
def get_user_positions(user_address: str):
    query_string = """
    query GetUserPositions($userAddress: String!) {
        user(id: $userAddress) {
            id
            address
            positions {
                id
                market {
                    id
                    question
                }
                outcome
                netQuantity
                realizedProfit
            }
        }
    }
    """
    
    result = subgraph.query(query_string, {"userAddress": user_address.lower()})
    
    print(f"👤 Positions for {user_address}:")
    for position in result["user"]["positions"]:
        print(f"\n📈 {position['market']['question']}")
        print(f"  Net Quantity: {position['netQuantity']}")
        print(f"  Realized P&L: {position['realizedProfit']}")
    
    return result["user"]["positions"]
```

### Get Market Data

```python
def get_market(market_id: str):
    query_string = """
    query GetMarket($id: ID!) {
        market(id: $id) {
            id
            question
            volume
            liquidity
            outcomePrices
            tradesCount
        }
    }
    """
    
    result = subgraph.query(query_string, {"id": market_id})
    market = result["market"]
    
    print(f"📊 Market: {market['question']}")
    print(f"💰 Volume: ${float(market['volume']):,.2f}")
    print(f"💧 Liquidity: ${float(market['liquidity']):,.2f}")
    print(f"📈 Trades: {market['tradesCount']}")
    
    return market
```

---

## 📈 Advanced Analytics

### Calculate Portfolio Value

```typescript
interface PortfolioValue {
  totalValue: number;
  unrealizedPnL: number;
  realizedPnL: number;
  positions: number;
}

async function calculatePortfolioValue(userAddress: string): Promise<PortfolioValue> {
  const GET_PORTFOLIO_QUERY = gql`
    query GetPortfolio($userAddress: String!) {
      user(id: $userAddress) {
        positions {
          netQuantity
          market {
            outcomePrices
          }
          outcome
          realizedProfit
          unrealizedProfit
        }
      }
    }
  `;

  const data = await subgraph.query(GET_PORTFOLIO_QUERY, {
    userAddress: userAddress.toLowerCase(),
  });

  let totalValue = 0;
  let unrealizedPnL = 0;
  let realizedPnL = 0;

  data.user.positions.forEach((position: any) => {
    const outcomeIndex = parseInt(position.outcome);
    const currentPrice = parseFloat(position.market.outcomePrices[outcomeIndex]);
    const quantity = parseFloat(position.netQuantity);

    totalValue += quantity * currentPrice;
    unrealizedPnL += parseFloat(position.unrealizedProfit);
    realizedPnL += parseFloat(position.realizedProfit);
  });

  const portfolio: PortfolioValue = {
    totalValue,
    unrealizedPnL,
    realizedPnL,
    positions: data.user.positions.length,
  };

  console.log('💼 Portfolio Summary:');
  console.log(`  Total Value: $${totalValue.toLocaleString()}`);
  console.log(`  Unrealized P&L: $${unrealizedPnL.toLocaleString()}`);
  console.log(`  Realized P&L: $${realizedPnL.toLocaleString()}`);
  console.log(`  Active Positions: ${portfolio.positions}`);

  return portfolio;
}
```

### Market Leaderboard

```typescript
const GET_LEADERBOARD_QUERY = gql`
  query GetLeaderboard($first: Int = 100) {
    users(
      first: $first
      orderBy: totalProfit
      orderDirection: desc
    ) {
      id
      address
      totalProfit
      totalVolume
      tradesCount
    }
  }
`;

async function getLeaderboard(limit: number = 100) {
  const data = await subgraph.query(GET_LEADERBOARD_QUERY, { first: limit });

  console.log(`🏆 Top ${limit} Traders:`);
  
  data.users.forEach((user: any, index: number) => {
    console.log(`\n${index + 1}. ${user.address.slice(0, 10)}...`);
    console.log(`   Total P&L: $${parseFloat(user.totalProfit).toLocaleString()}`);
    console.log(`   Volume: $${parseFloat(user.totalVolume).toLocaleString()}`);
    console.log(`   Trades: ${user.tradesCount}`);
  });

  return data.users;
}
```

### Time Series Data

```typescript
interface VolumeDataPoint {
  timestamp: number;
  volume: number;
  date: string;
}

async function getVolumeTimeSeries(
  marketId: string,
  interval: number = 3600 // 1 hour
): Promise<VolumeDataPoint[]> {
  const GET_TRADES_TIMESERIES_QUERY = gql`
    query GetTradesTimeSeries($marketId: ID!) {
      trades(
        where: { market: $marketId }
        orderBy: timestamp
        orderDirection: asc
        first: 1000
      ) {
        timestamp
        amount
      }
    }
  `;

  const data = await subgraph.query(GET_TRADES_TIMESERIES_QUERY, { marketId });

  const buckets: Map<number, number> = new Map();

  data.trades.forEach((trade: any) => {
    const timestamp = parseInt(trade.timestamp);
    const bucket = Math.floor(timestamp / interval) * interval;
    const currentVolume = buckets.get(bucket) || 0;
    buckets.set(bucket, currentVolume + parseFloat(trade.amount));
  });

  const timeSeries: VolumeDataPoint[] = Array.from(buckets.entries())
    .map(([timestamp, volume]) => ({
      timestamp,
      volume,
      date: new Date(timestamp * 1000).toISOString(),
    }))
    .sort((a, b) => a.timestamp - b.timestamp);

  console.log('📊 Volume Time Series:');
  timeSeries.slice(-10).forEach(point => {
    console.log(`  ${point.date}: $${point.volume.toFixed(2)}`);
  });

  return timeSeries;
}
```

---

## ⚡ Best Practices

### 1. Pagination

```typescript
async function getAllMarkets() {
  const markets: any[] = [];
  let skip = 0;
  const first = 1000;
  let hasMore = true;

  while (hasMore) {
    const QUERY = gql`
      query GetMarkets($first: Int!, $skip: Int!) {
        markets(first: $first, skip: $skip) {
          id
          question
        }
      }
    `;

    const data = await subgraph.query(QUERY, { first, skip });
    markets.push(...data.markets);
    
    hasMore = data.markets.length === first;
    skip += first;

    console.log(`Fetched ${markets.length} markets...`);
  }

  return markets;
}
```

### 2. Caching

```typescript
class CachedSubgraph extends PolymarketSubgraph {
  private cache: Map<string, { data: any; timestamp: number }> = new Map();
  private cacheDuration = 30000; // 30 seconds

  async query<T = any>(query: string, variables?: any): Promise<T> {
    const cacheKey = JSON.stringify({ query, variables });
    const cached = this.cache.get(cacheKey);

    if (cached && Date.now() - cached.timestamp < this.cacheDuration) {
      return cached.data;
    }

    const data = await super.query<T>(query, variables);
    this.cache.set(cacheKey, { data, timestamp: Date.now() });

    return data;
  }
}
```

---

## 🔗 Related Documentation

- [Gamma API](./polymarket-gamma.md)
- [CLOB API](./polymarket-clob.md)
- [WebSocket](./polymarket-wss.md)

---

## 📝 Additional Resources

- **Subgraph Playground**: https://api.goldsky.com/api/public/project_clhk16b61ay9t49vm6ntn4mkz/subgraphs/polymarket/prod/gn
- **GraphQL Docs**: https://graphql.org/learn/
