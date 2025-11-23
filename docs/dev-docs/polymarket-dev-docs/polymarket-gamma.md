# 🟦 Gamma Markets API

Gamma is Polymarket's market discovery and metadata API, providing comprehensive market information, search, filtering, and categorization.

---

## 📚 Official Documentation

- **Gamma Overview**: https://docs.polymarket.com/developers/gamma-markets-api/overview
- **Gamma Structure**: https://docs.polymarket.com/developers/gamma-markets-api/gamma-structure
- **Fetching Markets Guide**: https://docs.polymarket.com/developers/gamma-markets-api/fetch-markets-guide
- **List Teams (Sports)**: https://docs.polymarket.com/api-reference/sports/list-teams
- **List Series**: https://docs.polymarket.com/api-reference/series/list-series

---

## 🎯 Key Concepts

### What is Gamma?

Gamma powers:
- **Market search and discovery**
- **Market metadata (title, description, tags)**
- **Event structures and relationships**
- **Sports teams and series data**
- **Market filtering and categorization**
- **Resolution source tracking**

### Base URL

```
https://gamma-api.polymarket.com
```

---

## 📡 Core Endpoints

### 1. Get All Markets

**Endpoint**: `GET /markets`

**Parameters**:
- `limit` (optional) - Number of results (default: 100)
- `offset` (optional) - Pagination offset
- `active` (optional) - Filter by active status
- `closed` (optional) - Filter by closed status
- `tag` (optional) - Filter by tag

**Example Request**:
```bash
curl "https://gamma-api.polymarket.com/markets?limit=10&active=true"
```

### 2. Get Market by ID

**Endpoint**: `GET /markets/{condition_id}`

**Example Request**:
```bash
curl "https://gamma-api.polymarket.com/markets/0x..."
```

### 3. Search Markets

**Endpoint**: `GET /search`

**Parameters**:
- `q` (required) - Search query
- `limit` (optional) - Number of results
- `offset` (optional) - Pagination offset

**Example Request**:
```bash
curl "https://gamma-api.polymarket.com/search?q=bitcoin&limit=20"
```

### 4. Get Events

**Endpoint**: `GET /events`

**Parameters**:
- `limit` (optional)
- `offset` (optional)
- `archived` (optional)

**Example Request**:
```bash
curl "https://gamma-api.polymarket.com/events?limit=50"
```

---

## 💻 TypeScript SDK Usage

### Installation

```bash
npm install axios
```

### Create Gamma Client

```typescript
import axios, { AxiosInstance } from 'axios';

interface Market {
  condition_id: string;
  question: string;
  description: string;
  market_slug: string;
  end_date_iso: string;
  outcomes: string[];
  outcomePrices: string[];
  volume: string;
  liquidity: string;
  active: boolean;
  closed: boolean;
  tags: string[];
  competitive: number;
  icon?: string;
}

interface Event {
  id: string;
  slug: string;
  title: string;
  description: string;
  markets: Market[];
  startDate: string;
  endDate: string;
  creationDate: string;
  archived: boolean;
}

class GammaClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: 'https://gamma-api.polymarket.com',
      timeout: 10000,
    });
  }

  async getAllMarkets(params?: {
    limit?: number;
    offset?: number;
    active?: boolean;
    closed?: boolean;
    tag?: string;
  }): Promise<Market[]> {
    const response = await this.client.get('/markets', { params });
    return response.data;
  }

  async getMarket(conditionId: string): Promise<Market> {
    const response = await this.client.get(`/markets/${conditionId}`);
    return response.data;
  }

  async searchMarkets(query: string, limit: number = 20): Promise<Market[]> {
    const response = await this.client.get('/search', {
      params: { q: query, limit },
    });
    return response.data;
  }

  async getEvents(params?: {
    limit?: number;
    offset?: number;
    archived?: boolean;
  }): Promise<Event[]> {
    const response = await this.client.get('/events', { params });
    return response.data;
  }

  async getEvent(eventSlug: string): Promise<Event> {
    const response = await this.client.get(`/events/${eventSlug}`);
    return response.data;
  }
}

// Usage
const gamma = new GammaClient();
```

### Example: Get Active Markets

```typescript
async function getActiveMarkets() {
  const gamma = new GammaClient();
  
  const markets = await gamma.getAllMarkets({
    active: true,
    limit: 50,
  });

  console.log(`📊 Found ${markets.length} active markets`);
  
  markets.forEach(market => {
    console.log(`\n${market.question}`);
    console.log(`  Volume: $${parseFloat(market.volume).toLocaleString()}`);
    console.log(`  Outcomes: ${market.outcomes.join(' vs ')}`);
    console.log(`  Prices: ${market.outcomePrices.join(' / ')}`);
  });

  return markets;
}
```

### Example: Search for Specific Markets

```typescript
async function searchCryptoMarkets() {
  const gamma = new GammaClient();
  
  const markets = await gamma.searchMarkets('bitcoin', 20);

  console.log(`🔍 Found ${markets.length} Bitcoin-related markets`);

  markets.forEach(market => {
    console.log(`\n📈 ${market.question}`);
    console.log(`   Ends: ${new Date(market.end_date_iso).toLocaleDateString()}`);
    console.log(`   Tags: ${market.tags.join(', ')}`);
  });

  return markets;
}
```

### Example: Get Market Details

```typescript
async function getMarketDetails(conditionId: string) {
  const gamma = new GammaClient();
  
  const market = await gamma.getMarket(conditionId);

  console.log('📊 Market Details');
  console.log('═'.repeat(50));
  console.log(`Question: ${market.question}`);
  console.log(`Description: ${market.description}`);
  console.log(`Slug: ${market.market_slug}`);
  console.log(`\nOutcomes:`);
  market.outcomes.forEach((outcome, i) => {
    console.log(`  ${outcome}: ${market.outcomePrices[i]}`);
  });
  console.log(`\nVolume: $${parseFloat(market.volume).toLocaleString()}`);
  console.log(`Liquidity: $${parseFloat(market.liquidity).toLocaleString()}`);
  console.log(`Active: ${market.active ? '✅' : '❌'}`);
  console.log(`Closed: ${market.closed ? '✅' : '❌'}`);

  return market;
}
```

### Example: Filter Markets by Tag

```typescript
async function getMarketsByTag(tag: string) {
  const gamma = new GammaClient();
  
  const markets = await gamma.getAllMarkets({
    tag: tag,
    active: true,
    limit: 100,
  });

  console.log(`🏷️ Markets tagged with "${tag}": ${markets.length}`);

  // Group by volume
  const sorted = markets.sort((a, b) => 
    parseFloat(b.volume) - parseFloat(a.volume)
  );

  sorted.slice(0, 10).forEach((market, i) => {
    console.log(`${i + 1}. ${market.question}`);
    console.log(`   Volume: $${parseFloat(market.volume).toLocaleString()}`);
  });

  return sorted;
}

// Usage
getMarketsByTag('crypto');
getMarketsByTag('politics');
getMarketsByTag('sports');
```

---

## 🏆 Sports Markets

### Get Sports Teams

**Endpoint**: `GET /sports/teams`

```typescript
interface Team {
  id: string;
  name: string;
  abbreviation: string;
  logo?: string;
  league: string;
}

async function getSportsTeams(league?: string): Promise<Team[]> {
  const gamma = new GammaClient();
  
  const params = league ? { league } : {};
  const response = await gamma.client.get('/sports/teams', { params });
  
  return response.data;
}

// Get NBA teams
const nbaTeams = await getSportsTeams('NBA');
console.log(`🏀 NBA Teams: ${nbaTeams.length}`);
```

### Get Series

**Endpoint**: `GET /series`

```typescript
interface Series {
  id: string;
  title: string;
  slug: string;
  description: string;
  markets: Market[];
  startDate: string;
  endDate: string;
}

async function getSeries(limit: number = 50): Promise<Series[]> {
  const gamma = new GammaClient();
  
  const response = await gamma.client.get('/series', {
    params: { limit },
  });
  
  return response.data;
}

// Get all series
const series = await getSeries();
console.log(`📚 Series: ${series.length}`);
```

---

## 🐍 Python Implementation

### Create Gamma Client

```python
import requests
from typing import List, Dict, Optional

class GammaClient:
    def __init__(self):
        self.base_url = "https://gamma-api.polymarket.com"
        self.session = requests.Session()
    
    def get_all_markets(
        self,
        limit: int = 100,
        offset: int = 0,
        active: Optional[bool] = None,
        closed: Optional[bool] = None,
        tag: Optional[str] = None
    ) -> List[Dict]:
        params = {
            "limit": limit,
            "offset": offset,
        }
        if active is not None:
            params["active"] = active
        if closed is not None:
            params["closed"] = closed
        if tag:
            params["tag"] = tag
        
        response = self.session.get(f"{self.base_url}/markets", params=params)
        response.raise_for_status()
        return response.json()
    
    def get_market(self, condition_id: str) -> Dict:
        response = self.session.get(f"{self.base_url}/markets/{condition_id}")
        response.raise_for_status()
        return response.json()
    
    def search_markets(self, query: str, limit: int = 20) -> List[Dict]:
        params = {"q": query, "limit": limit}
        response = self.session.get(f"{self.base_url}/search", params=params)
        response.raise_for_status()
        return response.json()
    
    def get_events(
        self,
        limit: int = 50,
        offset: int = 0,
        archived: Optional[bool] = None
    ) -> List[Dict]:
        params = {"limit": limit, "offset": offset}
        if archived is not None:
            params["archived"] = archived
        
        response = self.session.get(f"{self.base_url}/events", params=params)
        response.raise_for_status()
        return response.json()

# Usage
gamma = GammaClient()

# Get active markets
markets = gamma.get_all_markets(active=True, limit=50)
print(f"📊 Found {len(markets)} active markets")

# Search for Bitcoin markets
btc_markets = gamma.search_markets("bitcoin", limit=20)
print(f"🔍 Found {len(btc_markets)} Bitcoin markets")
```

### Example: Market Discovery Bot

```python
class MarketDiscoveryBot:
    def __init__(self):
        self.gamma = GammaClient()
    
    def find_high_volume_markets(self, min_volume: float = 10000) -> List[Dict]:
        markets = self.gamma.get_all_markets(active=True, limit=500)
        
        high_volume = [
            m for m in markets
            if float(m.get("volume", 0)) >= min_volume
        ]
        
        # Sort by volume
        high_volume.sort(key=lambda m: float(m["volume"]), reverse=True)
        
        print(f"💰 Found {len(high_volume)} markets with volume >= ${min_volume:,.0f}")
        
        for market in high_volume[:10]:
            print(f"\n{market['question']}")
            print(f"  Volume: ${float(market['volume']):,.2f}")
            print(f"  Liquidity: ${float(market['liquidity']):,.2f}")
        
        return high_volume
    
    def find_trending_topics(self) -> Dict[str, int]:
        markets = self.gamma.get_all_markets(active=True, limit=500)
        
        tag_counts = {}
        for market in markets:
            for tag in market.get("tags", []):
                tag_counts[tag] = tag_counts.get(tag, 0) + 1
        
        # Sort by count
        sorted_tags = sorted(tag_counts.items(), key=lambda x: x[1], reverse=True)
        
        print("🔥 Trending Topics:")
        for tag, count in sorted_tags[:10]:
            print(f"  {tag}: {count} markets")
        
        return dict(sorted_tags)

# Usage
bot = MarketDiscoveryBot()
bot.find_high_volume_markets(min_volume=50000)
bot.find_trending_topics()
```

---

## 🔍 Advanced Filtering

### Multi-Tag Filter

```typescript
async function getMarketsByMultipleTags(tags: string[]) {
  const gamma = new GammaClient();
  
  const allMarkets = await gamma.getAllMarkets({
    active: true,
    limit: 500,
  });

  const filtered = allMarkets.filter(market =>
    tags.every(tag => market.tags.includes(tag))
  );

  console.log(`🔍 Markets with tags [${tags.join(', ')}]: ${filtered.length}`);
  return filtered;
}

// Find markets that are both "crypto" and "price"
const cryptoPriceMarkets = await getMarketsByMultipleTags(['crypto', 'price']);
```

### Volume-Based Ranking

```typescript
interface MarketRanking {
  market: Market;
  volumeRank: number;
  liquidityRank: number;
  competitiveScore: number;
}

async function rankMarkets(): Promise<MarketRanking[]> {
  const gamma = new GammaClient();
  
  const markets = await gamma.getAllMarkets({
    active: true,
    limit: 500,
  });

  // Sort by volume
  const byVolume = [...markets].sort((a, b) =>
    parseFloat(b.volume) - parseFloat(a.volume)
  );

  // Sort by liquidity
  const byLiquidity = [...markets].sort((a, b) =>
    parseFloat(b.liquidity) - parseFloat(a.liquidity)
  );

  const rankings: MarketRanking[] = markets.map(market => {
    const volumeRank = byVolume.findIndex(m => m.condition_id === market.condition_id) + 1;
    const liquidityRank = byLiquidity.findIndex(m => m.condition_id === market.condition_id) + 1;
    const competitiveScore = market.competitive || 0;

    return {
      market,
      volumeRank,
      liquidityRank,
      competitiveScore,
    };
  });

  // Sort by combined score
  rankings.sort((a, b) => {
    const scoreA = (1000 - a.volumeRank) + (1000 - a.liquidityRank) + a.competitiveScore * 100;
    const scoreB = (1000 - b.volumeRank) + (1000 - b.liquidityRank) + b.competitiveScore * 100;
    return scoreB - scoreA;
  });

  console.log('🏆 Top 10 Markets:');
  rankings.slice(0, 10).forEach((ranking, i) => {
    console.log(`${i + 1}. ${ranking.market.question}`);
    console.log(`   Volume Rank: #${ranking.volumeRank}`);
    console.log(`   Liquidity Rank: #${ranking.liquidityRank}`);
  });

  return rankings;
}
```

---

## 📊 Market Analytics

### Calculate Market Statistics

```typescript
interface MarketStats {
  totalVolume: number;
  totalLiquidity: number;
  averageVolume: number;
  averageLiquidity: number;
  activeMarkets: number;
  closedMarkets: number;
  tagDistribution: Record<string, number>;
}

async function getMarketStatistics(): Promise<MarketStats> {
  const gamma = new GammaClient();
  
  const markets = await gamma.getAllMarkets({ limit: 1000 });

  const stats: MarketStats = {
    totalVolume: 0,
    totalLiquidity: 0,
    averageVolume: 0,
    averageLiquidity: 0,
    activeMarkets: 0,
    closedMarkets: 0,
    tagDistribution: {},
  };

  markets.forEach(market => {
    stats.totalVolume += parseFloat(market.volume);
    stats.totalLiquidity += parseFloat(market.liquidity);
    
    if (market.active) stats.activeMarkets++;
    if (market.closed) stats.closedMarkets++;

    market.tags.forEach(tag => {
      stats.tagDistribution[tag] = (stats.tagDistribution[tag] || 0) + 1;
    });
  });

  stats.averageVolume = stats.totalVolume / markets.length;
  stats.averageLiquidity = stats.totalLiquidity / markets.length;

  console.log('📊 Market Statistics:');
  console.log(`Total Volume: $${stats.totalVolume.toLocaleString()}`);
  console.log(`Total Liquidity: $${stats.totalLiquidity.toLocaleString()}`);
  console.log(`Active Markets: ${stats.activeMarkets}`);
  console.log(`Closed Markets: ${stats.closedMarkets}`);

  return stats;
}
```

---

## ⚡ Best Practices

### 1. Caching
```typescript
class CachedGammaClient extends GammaClient {
  private cache: Map<string, { data: any; timestamp: number }> = new Map();
  private cacheDuration = 60000; // 1 minute

  async getMarket(conditionId: string): Promise<Market> {
    const cached = this.cache.get(conditionId);
    
    if (cached && Date.now() - cached.timestamp < this.cacheDuration) {
      return cached.data;
    }

    const market = await super.getMarket(conditionId);
    this.cache.set(conditionId, { data: market, timestamp: Date.now() });
    
    return market;
  }
}
```

### 2. Pagination
```typescript
async function getAllMarketsWithPagination(): Promise<Market[]> {
  const gamma = new GammaClient();
  const allMarkets: Market[] = [];
  const pageSize = 100;
  let offset = 0;
  let hasMore = true;

  while (hasMore) {
    const markets = await gamma.getAllMarkets({
      limit: pageSize,
      offset: offset,
    });

    allMarkets.push(...markets);
    offset += pageSize;
    hasMore = markets.length === pageSize;

    console.log(`Fetched ${allMarkets.length} markets...`);
  }

  return allMarkets;
}
```

---

## 🔗 Related Documentation

- [CLOB API](./polymarket-clob.md)
- [WebSocket](./polymarket-wss.md)
- [Subgraph](./polymarket-subgraph.md)

---

## 📝 Additional Resources

- **Gamma API Reference**: https://docs.polymarket.com/api-reference/gamma
- **Examples**: https://github.com/Polymarket/gamma-examples
