# 🧱 CLOB (Central Limit Order Book)

The CLOB is Polymarket's order matching engine where users can place limit and market orders.

---

## 📚 Official Documentation

- **CLOB Introduction**: https://docs.polymarket.com/developers/CLOB/introduction
- **CLOB Clients**: https://docs.polymarket.com/developers/CLOB/clients
- **Orders Overview**: https://docs.polymarket.com/developers/CLOB/orders/orders
- **Trades Overview**: https://docs.polymarket.com/developers/CLOB/trades/trades-overview

---

## 🎯 Key Concepts

### What is the CLOB?

The Central Limit Order Book (CLOB) is an off-chain matching engine that:
- Matches buy and sell orders
- Provides liquidity aggregation
- Supports limit and market orders
- Settles trades on-chain via Polygon

### Order Types

1. **Limit Orders** - Orders at a specific price
2. **Market Orders** - Orders at best available price
3. **Post-Only Orders** - Orders that add liquidity (maker only)

---

## 🔌 API Endpoints

### Base URL
```
https://clob.polymarket.com
```

### Get Order Book

**Endpoint**: `GET /book`

**Parameters**:
- `token_id` (required) - The token ID for the market

**Example Request**:
```bash
curl "https://clob.polymarket.com/book?token_id=21742633143463906290569050155826241533067272736897614950488156847949938836455"
```

**Example Response**:
```json
{
  "market": "0x...",
  "asset_id": "21742633143463906290569050155826241533067272736897614950488156847949938836455",
  "bids": [
    {
      "price": "0.52",
      "size": "100.5"
    }
  ],
  "asks": [
    {
      "price": "0.53",
      "size": "95.2"
    }
  ]
}
```

### Get Bid/Ask Spreads

**Endpoint**: `GET /spreads`

**Parameters**:
- `token_id` (optional) - Filter by token ID

**Example Request**:
```bash
curl "https://clob.polymarket.com/spreads?token_id=21742633143463906290569050155826241533067272736897614950488156847949938836455"
```

---

## 💻 TypeScript SDK Usage

### Installation

```bash
npm install @polymarket/clob-client
```

### Basic Setup

```typescript
import { ClobClient } from '@polymarket/clob-client';

const client = new ClobClient({
  host: 'https://clob.polymarket.com',
  chainId: 137, // Polygon mainnet
});
```

### Get Order Book

```typescript
async function getOrderBook(tokenId: string) {
  const orderBook = await client.getOrderBook(tokenId);
  
  console.log('Bids:', orderBook.bids);
  console.log('Asks:', orderBook.asks);
  
  return orderBook;
}
```

### Place a Limit Order

```typescript
import { ethers } from 'ethers';

async function placeLimitOrder() {
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!);
  
  const order = {
    tokenId: '21742633143463906290569050155826241533067272736897614950488156847949938836455',
    price: 0.52,
    size: 10,
    side: 'BUY', // or 'SELL'
    feeRateBps: 0,
    nonce: Date.now(),
    expiration: Math.floor(Date.now() / 1000) + 3600, // 1 hour
  };
  
  const signedOrder = await client.createOrder(order, wallet);
  const result = await client.postOrder(signedOrder);
  
  console.log('Order placed:', result.orderID);
  return result;
}
```

### Cancel an Order

```typescript
async function cancelOrder(orderId: string) {
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!);
  
  const result = await client.cancelOrder(orderId, wallet);
  console.log('Order cancelled:', result);
  
  return result;
}
```

### Get User Orders

```typescript
async function getUserOrders(address: string) {
  const orders = await client.getOrders({
    maker: address,
  });
  
  console.log('User orders:', orders);
  return orders;
}
```

---

## 🐍 Python SDK Usage

### Installation

```bash
pip install py-clob-client
```

### Basic Setup

```python
from py_clob_client.client import ClobClient

client = ClobClient(
    host="https://clob.polymarket.com",
    chain_id=137
)
```

### Get Order Book

```python
def get_order_book(token_id: str):
    order_book = client.get_order_book(token_id)
    
    print("Bids:", order_book["bids"])
    print("Asks:", order_book["asks"])
    
    return order_book
```

### Place a Limit Order

```python
from py_clob_client.client import ClobClient
from py_clob_client.order_builder.constants import BUY

def place_limit_order():
    order = client.create_order(
        token_id="21742633143463906290569050155826241533067272736897614950488156847949938836455",
        price=0.52,
        size=10,
        side=BUY,
        fee_rate_bps=0,
    )
    
    signed_order = client.sign_order(order)
    result = client.post_order(signed_order)
    
    print("Order placed:", result["orderID"])
    return result
```

---

## 🔑 Authentication

For placing/canceling orders, you need:

1. **Wallet Private Key** - To sign orders
2. **API Key** (optional) - For rate limit increases
3. **Signature** - EIP-712 signed order data

### Creating API Keys

```typescript
import { ClobClient } from '@polymarket/clob-client';
import { ethers } from 'ethers';

async function createApiKey() {
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!);
  const client = new ClobClient({ host: 'https://clob.polymarket.com' });
  
  const apiKey = await client.createApiKey(wallet);
  console.log('API Key:', apiKey);
  
  return apiKey;
}
```

---

## ⚡ Best Practices

### 1. Order Management
- Always set reasonable expiration times
- Cancel stale orders to free up margin
- Monitor fill rates

### 2. Price Optimization
- Check spreads before placing orders
- Use limit orders for better execution
- Consider slippage on market orders

### 3. Gas Efficiency
- Batch operations when possible
- Use post-only orders to save on fees
- Monitor polygon gas prices

### 4. Error Handling
```typescript
async function safeOrderPlacement(order: Order) {
  try {
    const result = await client.postOrder(order);
    return { success: true, data: result };
  } catch (error) {
    console.error('Order placement failed:', error);
    return { success: false, error };
  }
}
```

---

## 📊 Market Data Endpoints

### Get Recent Trades

```typescript
async function getRecentTrades(tokenId: string) {
  const trades = await client.getTrades({
    asset_id: tokenId,
    limit: 100,
  });
  
  return trades;
}
```

### Get Market Prices

```typescript
async function getMarketPrices(tokenIds: string[]) {
  const prices = await client.getPrices(tokenIds);
  return prices;
}
```

---

## 🚨 Common Issues

### Issue: Order Rejected
**Solution**: Check margin requirements, order size limits, and signature validity

### Issue: Order Not Filling
**Solution**: Check price competitiveness, market liquidity, and order type

### Issue: Rate Limiting
**Solution**: Implement exponential backoff, request API key for higher limits

---

## 🔗 Related Documentation

- [WebSocket for Real-time Updates](./polymarket-wss.md)
- [Gamma Markets API](./polymarket-gamma.md)
- [Conditional Token Framework](./polymarket-ctf.md)

---

## 📝 Additional Resources

- **CLOB API Specification**: https://docs.polymarket.com/api-reference/clob
- **GitHub Examples**: https://github.com/Polymarket/clob-client
- **Discord Support**: https://discord.gg/polymarket
