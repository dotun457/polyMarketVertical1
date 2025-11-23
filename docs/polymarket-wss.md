# 🔴 WebSocket (WSS) - Real-time Updates

Polymarket's WebSocket API provides real-time streaming of orderbook updates, trades, and market data.

---

## 📚 Official Documentation

- **WSS Overview**: https://docs.polymarket.com/developers/CLOB/websocket/wss-overview
- **WSS Quickstart**: https://docs.polymarket.com/quickstart/websocket/WSS-Quickstart
- **WSS Authentication**: https://docs.polymarket.com/developers/CLOB/websocket/wss-auth

---

## 🎯 Key Concepts

### What is WSS?

WebSocket Secure (WSS) enables:
- **Real-time orderbook updates**
- **Live price feeds**
- **User fill notifications**
- **Trade stream events**

### Connection Details

**WebSocket URL**: `wss://ws-subscriptions-clob.polymarket.com/ws/market`

**Protocol**: WebSocket Secure (wss://)

---

## 📡 Available Channels

### 1. Market Channel
Subscribes to orderbook updates for a specific market.

### 2. User Channel
Subscribes to user-specific events (fills, cancellations).

### 3. Trades Channel
Subscribes to trade execution events.

---

## 💻 TypeScript Implementation

### Basic WebSocket Connection

```typescript
import WebSocket from 'ws';

class PolymarketWebSocket {
  private ws: WebSocket | null = null;
  private readonly url = 'wss://ws-subscriptions-clob.polymarket.com/ws/market';

  connect() {
    this.ws = new WebSocket(this.url);

    this.ws.on('open', () => {
      console.log('✅ Connected to Polymarket WSS');
    });

    this.ws.on('message', (data: WebSocket.Data) => {
      const message = JSON.parse(data.toString());
      this.handleMessage(message);
    });

    this.ws.on('error', (error) => {
      console.error('❌ WebSocket error:', error);
    });

    this.ws.on('close', () => {
      console.log('🔌 Disconnected from Polymarket WSS');
      // Implement reconnection logic
      setTimeout(() => this.connect(), 5000);
    });
  }

  private handleMessage(message: any) {
    console.log('📨 Received:', message);
  }

  send(message: any) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(message));
    }
  }

  disconnect() {
    this.ws?.close();
  }
}
```

### Subscribe to Market Updates

```typescript
interface SubscribeMessage {
  type: 'subscribe';
  channel: 'market';
  market: string;
  assets_ids?: string[];
}

function subscribeToMarket(ws: PolymarketWebSocket, tokenId: string) {
  const subscribeMsg: SubscribeMessage = {
    type: 'subscribe',
    channel: 'market',
    market: tokenId,
  };

  ws.send(subscribeMsg);
  console.log(`📊 Subscribed to market: ${tokenId}`);
}
```

### Handle Orderbook Updates

```typescript
interface OrderbookUpdate {
  event_type: 'book';
  market: string;
  asset_id: string;
  timestamp: number;
  hash: string;
  bids: Array<{ price: string; size: string }>;
  asks: Array<{ price: string; size: string }>;
}

class OrderbookHandler {
  private orderbooks: Map<string, OrderbookUpdate> = new Map();

  handleUpdate(update: OrderbookUpdate) {
    this.orderbooks.set(update.asset_id, update);
    
    console.log(`📈 Orderbook Update for ${update.asset_id}`);
    console.log(`Best Bid: ${update.bids[0]?.price || 'N/A'}`);
    console.log(`Best Ask: ${update.asks[0]?.price || 'N/A'}`);
  }

  getOrderbook(assetId: string): OrderbookUpdate | undefined {
    return this.orderbooks.get(assetId);
  }

  getSpread(assetId: string): number | null {
    const book = this.orderbooks.get(assetId);
    if (!book || !book.bids[0] || !book.asks[0]) return null;
    
    return parseFloat(book.asks[0].price) - parseFloat(book.bids[0].price);
  }
}
```

### Subscribe to User Events

```typescript
interface UserSubscribeMessage {
  type: 'subscribe';
  channel: 'user';
  auth: {
    apiKey: string;
    secret: string;
    passphrase: string;
  };
}

function subscribeToUserChannel(ws: PolymarketWebSocket, credentials: any) {
  const subscribeMsg: UserSubscribeMessage = {
    type: 'subscribe',
    channel: 'user',
    auth: credentials,
  };

  ws.send(subscribeMsg);
  console.log('👤 Subscribed to user channel');
}
```

### Handle Trade Events

```typescript
interface TradeEvent {
  event_type: 'trade';
  market: string;
  asset_id: string;
  price: string;
  size: string;
  side: 'BUY' | 'SELL';
  timestamp: number;
  trade_id: string;
}

class TradeHandler {
  private trades: TradeEvent[] = [];

  handleTrade(trade: TradeEvent) {
    this.trades.push(trade);
    
    console.log(`💰 Trade Executed`);
    console.log(`  Price: ${trade.price}`);
    console.log(`  Size: ${trade.size}`);
    console.log(`  Side: ${trade.side}`);
  }

  getRecentTrades(assetId: string, limit: number = 10): TradeEvent[] {
    return this.trades
      .filter(t => t.asset_id === assetId)
      .slice(-limit);
  }

  getVolume(assetId: string): number {
    return this.trades
      .filter(t => t.asset_id === assetId)
      .reduce((sum, t) => sum + parseFloat(t.size), 0);
  }
}
```

---

## 🔐 Authentication

For user-specific channels, you need authentication:

### Generate API Credentials

```typescript
import { createHmac } from 'crypto';

function generateSignature(
  secret: string,
  timestamp: number,
  method: string,
  path: string
): string {
  const message = `${timestamp}${method}${path}`;
  return createHmac('sha256', secret).update(message).digest('base64');
}

interface WSAuthCredentials {
  apiKey: string;
  secret: string;
  passphrase: string;
}

async function authenticateWebSocket(credentials: WSAuthCredentials) {
  const timestamp = Date.now();
  const signature = generateSignature(
    credentials.secret,
    timestamp,
    'GET',
    '/ws/user'
  );

  return {
    apiKey: credentials.apiKey,
    signature,
    timestamp: timestamp.toString(),
    passphrase: credentials.passphrase,
  };
}
```

---

## 🐍 Python Implementation

### Basic WebSocket Connection

```python
import asyncio
import websockets
import json

class PolymarketWebSocket:
    def __init__(self):
        self.url = "wss://ws-subscriptions-clob.polymarket.com/ws/market"
        self.ws = None
    
    async def connect(self):
        self.ws = await websockets.connect(self.url)
        print("✅ Connected to Polymarket WSS")
        
    async def listen(self):
        async for message in self.ws:
            data = json.loads(message)
            await self.handle_message(data)
    
    async def handle_message(self, message):
        print(f"📨 Received: {message}")
    
    async def send(self, message):
        await self.ws.send(json.dumps(message))
    
    async def close(self):
        await self.ws.close()
```

### Subscribe to Market

```python
async def subscribe_to_market(ws: PolymarketWebSocket, token_id: str):
    subscribe_msg = {
        "type": "subscribe",
        "channel": "market",
        "market": token_id
    }
    
    await ws.send(subscribe_msg)
    print(f"📊 Subscribed to market: {token_id}")
```

### Complete Example

```python
import asyncio
import websockets
import json

async def polymarket_stream():
    url = "wss://ws-subscriptions-clob.polymarket.com/ws/market"
    
    async with websockets.connect(url) as ws:
        # Subscribe to a market
        subscribe_msg = {
            "type": "subscribe",
            "channel": "market",
            "market": "0x..." # Market address
        }
        
        await ws.send(json.dumps(subscribe_msg))
        
        # Listen for updates
        while True:
            message = await ws.recv()
            data = json.loads(message)
            
            if data.get("event_type") == "book":
                print(f"📈 Orderbook Update:")
                print(f"  Best Bid: {data['bids'][0]['price']}")
                print(f"  Best Ask: {data['asks'][0]['price']}")
            
            elif data.get("event_type") == "trade":
                print(f"💰 Trade: {data['price']} @ {data['size']}")

# Run the stream
asyncio.run(polymarket_stream())
```

---

## 📊 Message Types

### Orderbook Update
```json
{
  "event_type": "book",
  "market": "0x...",
  "asset_id": "21742633143463906290569050155826241533067272736897614950488156847949938836455",
  "timestamp": 1700000000,
  "hash": "0x...",
  "bids": [
    { "price": "0.52", "size": "100" }
  ],
  "asks": [
    { "price": "0.53", "size": "95" }
  ]
}
```

### Trade Event
```json
{
  "event_type": "trade",
  "market": "0x...",
  "asset_id": "21742633143463906290569050155826241533067272736897614950488156847949938836455",
  "price": "0.525",
  "size": "50",
  "side": "BUY",
  "timestamp": 1700000000,
  "trade_id": "0x..."
}
```

### User Fill Event
```json
{
  "event_type": "fill",
  "order_id": "0x...",
  "market": "0x...",
  "asset_id": "21742633143463906290569050155826241533067272736897614950488156847949938836455",
  "price": "0.52",
  "size": "10",
  "side": "BUY",
  "timestamp": 1700000000
}
```

---

## ⚡ Best Practices

### 1. Connection Management
```typescript
class ResilientWebSocket {
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;
  private reconnectDelay = 1000;

  async connect() {
    try {
      // Connection logic
      this.reconnectAttempts = 0;
    } catch (error) {
      this.reconnectAttempts++;
      if (this.reconnectAttempts < this.maxReconnectAttempts) {
        const delay = this.reconnectDelay * Math.pow(2, this.reconnectAttempts);
        setTimeout(() => this.connect(), delay);
      }
    }
  }
}
```

### 2. Heartbeat/Ping-Pong
```typescript
setInterval(() => {
  if (ws.readyState === WebSocket.OPEN) {
    ws.ping();
  }
}, 30000); // Every 30 seconds
```

### 3. Message Queue
```typescript
class MessageQueue {
  private queue: any[] = [];
  
  enqueue(message: any) {
    this.queue.push(message);
  }
  
  async processQueue(ws: WebSocket) {
    while (this.queue.length > 0) {
      const message = this.queue.shift();
      ws.send(JSON.stringify(message));
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }
}
```

---

## 🚨 Common Issues

### Issue: Connection Drops
**Solution**: Implement automatic reconnection with exponential backoff

### Issue: Missed Messages
**Solution**: Use message sequencing and request snapshots on reconnect

### Issue: Rate Limiting
**Solution**: Limit subscription requests and use connection pooling

---

## 🔗 Related Documentation

- [CLOB API](./polymarket-clob.md)
- [RTDS Real-time Data](./polymarket-rtds.md)
- [Gamma Markets](./polymarket-gamma.md)

---

## 📝 Additional Resources

- **WebSocket Spec**: https://docs.polymarket.com/api-reference/websocket
- **Examples**: https://github.com/Polymarket/examples
