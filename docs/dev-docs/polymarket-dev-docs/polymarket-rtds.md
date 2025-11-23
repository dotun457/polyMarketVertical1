# 🟩 RTDS (Real-Time Data Stream)

Polymarket's Real-Time Data Stream provides live feeds for crypto prices, comments, and market activity.

---

## 📚 Official Documentation

- **RTDS Overview**: https://docs.polymarket.com/developers/RTDS/RTDS-overview
- **RTDS Crypto Prices**: https://docs.polymarket.com/developers/RTDS/RTDS-crypto-prices
- **RTDS Comments**: https://docs.polymarket.com/developers/RTDS/RTDS-comments

---

## 🎯 Key Concepts

### What is RTDS?

Real-Time Data Stream is a specialized system for:
- **Live crypto price feeds**
- **Real-time comment streams**
- **Market sentiment analysis**
- **Social trading signals**

RTDS complements the WSS (WebSocket) system but focuses on auxiliary data rather than trading data.

---

## 💰 RTDS Crypto Prices

### Overview

The Crypto Prices stream provides real-time price updates for major cryptocurrencies, used in Polymarket's crypto prediction markets.

### WebSocket Connection

**URL**: `wss://rtds.polymarket.com/prices`

### Supported Assets

- Bitcoin (BTC)
- Ethereum (ETH)
- Solana (SOL)
- Polygon (MATIC)
- And more...

---

## 💻 TypeScript Implementation

### Connect to Crypto Prices Stream

```typescript
import WebSocket from 'ws';

interface CryptoPriceUpdate {
  symbol: string;
  price: number;
  timestamp: number;
  source: string;
  volume_24h?: number;
  change_24h?: number;
}

class CryptoPriceStream {
  private ws: WebSocket | null = null;
  private readonly url = 'wss://rtds.polymarket.com/prices';
  private prices: Map<string, CryptoPriceUpdate> = new Map();

  connect() {
    this.ws = new WebSocket(this.url);

    this.ws.on('open', () => {
      console.log('✅ Connected to RTDS Crypto Prices');
      this.subscribeToAssets(['BTC', 'ETH', 'SOL']);
    });

    this.ws.on('message', (data: WebSocket.Data) => {
      const update: CryptoPriceUpdate = JSON.parse(data.toString());
      this.handlePriceUpdate(update);
    });

    this.ws.on('error', (error) => {
      console.error('❌ RTDS error:', error);
    });

    this.ws.on('close', () => {
      console.log('🔌 Disconnected from RTDS');
      setTimeout(() => this.connect(), 5000);
    });
  }

  private subscribeToAssets(symbols: string[]) {
    const message = {
      type: 'subscribe',
      symbols: symbols,
    };
    this.ws?.send(JSON.stringify(message));
    console.log(`📊 Subscribed to: ${symbols.join(', ')}`);
  }

  private handlePriceUpdate(update: CryptoPriceUpdate) {
    this.prices.set(update.symbol, update);
    
    console.log(`💰 ${update.symbol}: $${update.price.toFixed(2)}`);
    if (update.change_24h) {
      const emoji = update.change_24h >= 0 ? '📈' : '📉';
      console.log(`  ${emoji} 24h: ${update.change_24h.toFixed(2)}%`);
    }
  }

  getPrice(symbol: string): number | undefined {
    return this.prices.get(symbol)?.price;
  }

  getAllPrices(): Map<string, CryptoPriceUpdate> {
    return this.prices;
  }

  disconnect() {
    this.ws?.close();
  }
}

// Usage
const priceStream = new CryptoPriceStream();
priceStream.connect();
```

### Price Alert System

```typescript
interface PriceAlert {
  symbol: string;
  targetPrice: number;
  condition: 'above' | 'below';
  callback: (price: number) => void;
}

class PriceAlertSystem {
  private alerts: PriceAlert[] = [];
  private priceStream: CryptoPriceStream;

  constructor() {
    this.priceStream = new CryptoPriceStream();
    this.priceStream.connect();
  }

  addAlert(alert: PriceAlert) {
    this.alerts.push(alert);
    console.log(`🔔 Alert set: ${alert.symbol} ${alert.condition} $${alert.targetPrice}`);
  }

  checkAlerts(symbol: string, currentPrice: number) {
    this.alerts
      .filter(alert => alert.symbol === symbol)
      .forEach(alert => {
        const triggered =
          (alert.condition === 'above' && currentPrice > alert.targetPrice) ||
          (alert.condition === 'below' && currentPrice < alert.targetPrice);

        if (triggered) {
          alert.callback(currentPrice);
          // Remove triggered alert
          this.alerts = this.alerts.filter(a => a !== alert);
        }
      });
  }
}

// Usage
const alertSystem = new PriceAlertSystem();

alertSystem.addAlert({
  symbol: 'BTC',
  targetPrice: 45000,
  condition: 'above',
  callback: (price) => {
    console.log(`🚨 ALERT: BTC crossed $45,000! Current: $${price}`);
  },
});
```

---

## 💬 RTDS Comments Stream

### Overview

The Comments stream provides real-time commentary and social sentiment from Polymarket users.

### WebSocket Connection

**URL**: `wss://rtds.polymarket.com/comments`

### TypeScript Implementation

```typescript
interface Comment {
  id: string;
  market_id: string;
  user_id: string;
  username: string;
  text: string;
  timestamp: number;
  likes: number;
  replies: number;
}

class CommentsStream {
  private ws: WebSocket | null = null;
  private readonly url = 'wss://rtds.polymarket.com/comments';
  private comments: Map<string, Comment[]> = new Map();

  connect() {
    this.ws = new WebSocket(this.url);

    this.ws.on('open', () => {
      console.log('✅ Connected to RTDS Comments');
    });

    this.ws.on('message', (data: WebSocket.Data) => {
      const comment: Comment = JSON.parse(data.toString());
      this.handleComment(comment);
    });

    this.ws.on('error', (error) => {
      console.error('❌ Comments stream error:', error);
    });
  }

  subscribeToMarket(marketId: string) {
    const message = {
      type: 'subscribe',
      channel: 'comments',
      market_id: marketId,
    };
    this.ws?.send(JSON.stringify(message));
  }

  private handleComment(comment: Comment) {
    const marketComments = this.comments.get(comment.market_id) || [];
    marketComments.push(comment);
    this.comments.set(comment.market_id, marketComments);

    console.log(`💬 New comment on ${comment.market_id}:`);
    console.log(`   @${comment.username}: ${comment.text}`);
  }

  getComments(marketId: string, limit: number = 50): Comment[] {
    return (this.comments.get(marketId) || []).slice(-limit);
  }

  disconnect() {
    this.ws?.close();
  }
}
```

### Sentiment Analysis

```typescript
class SentimentAnalyzer {
  private positiveWords = ['bullish', 'up', 'win', 'yes', 'great', 'moon'];
  private negativeWords = ['bearish', 'down', 'lose', 'no', 'bad', 'crash'];

  analyzeSentiment(text: string): 'positive' | 'negative' | 'neutral' {
    const lowercaseText = text.toLowerCase();
    
    const positiveCount = this.positiveWords.filter(word =>
      lowercaseText.includes(word)
    ).length;
    
    const negativeCount = this.negativeWords.filter(word =>
      lowercaseText.includes(word)
    ).length;

    if (positiveCount > negativeCount) return 'positive';
    if (negativeCount > positiveCount) return 'negative';
    return 'neutral';
  }

  getMarketSentiment(comments: Comment[]): {
    positive: number;
    negative: number;
    neutral: number;
  } {
    const sentiments = {
      positive: 0,
      negative: 0,
      neutral: 0,
    };

    comments.forEach(comment => {
      const sentiment = this.analyzeSentiment(comment.text);
      sentiments[sentiment]++;
    });

    return sentiments;
  }
}

// Usage
const analyzer = new SentimentAnalyzer();
const commentsStream = new CommentsStream();
commentsStream.connect();

// Get sentiment for a market
setTimeout(() => {
  const marketId = '0x...';
  const comments = commentsStream.getComments(marketId);
  const sentiment = analyzer.getMarketSentiment(comments);
  
  console.log('📊 Market Sentiment:');
  console.log(`  Positive: ${sentiment.positive}`);
  console.log(`  Negative: ${sentiment.negative}`);
  console.log(`  Neutral: ${sentiment.neutral}`);
}, 60000); // After 1 minute
```

---

## 🐍 Python Implementation

### Crypto Prices Stream

```python
import asyncio
import websockets
import json

class CryptoPriceStream:
    def __init__(self):
        self.url = "wss://rtds.polymarket.com/prices"
        self.prices = {}
    
    async def connect(self):
        async with websockets.connect(self.url) as ws:
            # Subscribe to assets
            subscribe_msg = {
                "type": "subscribe",
                "symbols": ["BTC", "ETH", "SOL"]
            }
            await ws.send(json.dumps(subscribe_msg))
            print("✅ Connected to RTDS Crypto Prices")
            
            # Listen for updates
            async for message in ws:
                update = json.loads(message)
                await self.handle_update(update)
    
    async def handle_update(self, update):
        symbol = update["symbol"]
        price = update["price"]
        self.prices[symbol] = price
        
        print(f"💰 {symbol}: ${price:.2f}")
        
        if "change_24h" in update:
            change = update["change_24h"]
            emoji = "📈" if change >= 0 else "📉"
            print(f"  {emoji} 24h: {change:.2f}%")

# Run the stream
async def main():
    stream = CryptoPriceStream()
    await stream.connect()

asyncio.run(main())
```

### Comments Stream

```python
import asyncio
import websockets
import json

class CommentsStream:
    def __init__(self):
        self.url = "wss://rtds.polymarket.com/comments"
        self.comments = {}
    
    async def connect(self, market_id: str):
        async with websockets.connect(self.url) as ws:
            # Subscribe to market
            subscribe_msg = {
                "type": "subscribe",
                "channel": "comments",
                "market_id": market_id
            }
            await ws.send(json.dumps(subscribe_msg))
            print(f"✅ Subscribed to comments for {market_id}")
            
            # Listen for comments
            async for message in ws:
                comment = json.loads(message)
                await self.handle_comment(comment)
    
    async def handle_comment(self, comment):
        market_id = comment["market_id"]
        username = comment["username"]
        text = comment["text"]
        
        print(f"💬 @{username}: {text}")
        
        # Store comment
        if market_id not in self.comments:
            self.comments[market_id] = []
        self.comments[market_id].append(comment)

# Run the stream
async def main():
    stream = CommentsStream()
    await stream.connect("0x...")  # Market ID

asyncio.run(main())
```

---

## 📊 Use Cases

### 1. Trading Bot with Price Triggers

```typescript
class TradingBot {
  private priceStream: CryptoPriceStream;
  
  constructor() {
    this.priceStream = new CryptoPriceStream();
    this.priceStream.connect();
  }
  
  async monitorPriceAndTrade(symbol: string, threshold: number) {
    setInterval(() => {
      const price = this.priceStream.getPrice(symbol);
      if (price && price > threshold) {
        console.log(`🤖 Executing trade: ${symbol} @ ${price}`);
        // Place trade via CLOB API
      }
    }, 1000);
  }
}
```

### 2. Social Sentiment Dashboard

```typescript
class SentimentDashboard {
  private commentsStream: CommentsStream;
  private analyzer: SentimentAnalyzer;
  
  constructor() {
    this.commentsStream = new CommentsStream();
    this.analyzer = new SentimentAnalyzer();
    this.commentsStream.connect();
  }
  
  async displaySentiment(marketId: string) {
    setInterval(() => {
      const comments = this.commentsStream.getComments(marketId);
      const sentiment = this.analyzer.getMarketSentiment(comments);
      
      console.clear();
      console.log('📊 Market Sentiment Dashboard');
      console.log('═'.repeat(40));
      console.log(`Positive: ${'█'.repeat(sentiment.positive)} ${sentiment.positive}`);
      console.log(`Negative: ${'█'.repeat(sentiment.negative)} ${sentiment.negative}`);
      console.log(`Neutral:  ${'█'.repeat(sentiment.neutral)} ${sentiment.neutral}`);
    }, 5000);
  }
}
```

---

## ⚡ Best Practices

### 1. Data Buffering
```typescript
class DataBuffer<T> {
  private buffer: T[] = [];
  private maxSize: number;
  
  constructor(maxSize: number = 1000) {
    this.maxSize = maxSize;
  }
  
  add(item: T) {
    this.buffer.push(item);
    if (this.buffer.length > this.maxSize) {
      this.buffer.shift();
    }
  }
  
  getRecent(count: number): T[] {
    return this.buffer.slice(-count);
  }
}
```

### 2. Rate Limiting
```typescript
class RateLimiter {
  private timestamps: number[] = [];
  private limit: number;
  private window: number;
  
  constructor(limit: number, windowMs: number) {
    this.limit = limit;
    this.window = windowMs;
  }
  
  canMakeRequest(): boolean {
    const now = Date.now();
    this.timestamps = this.timestamps.filter(t => now - t < this.window);
    return this.timestamps.length < this.limit;
  }
  
  recordRequest() {
    this.timestamps.push(Date.now());
  }
}
```

---

## 🔗 Related Documentation

- [WebSocket API](./polymarket-wss.md)
- [CLOB API](./polymarket-clob.md)
- [Gamma Markets](./polymarket-gamma.md)

---

## 📝 Additional Resources

- **RTDS API Docs**: https://docs.polymarket.com/api-reference/rtds
- **Examples**: https://github.com/Polymarket/rtds-examples
