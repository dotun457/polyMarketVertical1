# 🟫 Bridge & Swaps API

Polymarket's Bridge API enables cross-chain deposits and swaps for seamless onboarding.

---

## 📚 Official Documentation

- **Bridge API Overview**: https://docs.polymarket.com/developers/misc-endpoints/bridge-overview

---

## 🎯 Key Concepts

### What is the Bridge?

The Bridge API provides:
- **Cross-chain deposits** (Ethereum → Polygon)
- **Token swaps** (USDC, ETH, etc.)
- **Fast onboarding** for new users
- **Gas-optimized transactions**

### Supported Chains

- Ethereum (Mainnet)
- Polygon (Mainnet)
- Arbitrum (coming soon)

---

## 🔌 API Endpoints

### Base URL
```
https://api.polymarket.com/bridge
```

### Get Supported Tokens

**Endpoint**: `GET /tokens`

**Example Request**:
```bash
curl "https://api.polymarket.com/bridge/tokens"
```

**Example Response**:
```json
{
  "tokens": [
    {
      "symbol": "USDC",
      "address": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
      "decimals": 6,
      "chainId": 1
    },
    {
      "symbol": "USDC",
      "address": "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174",
      "decimals": 6,
      "chainId": 137
    }
  ]
}
```

### Get Bridge Quote

**Endpoint**: `POST /quote`

**Body**:
```json
{
  "fromChainId": 1,
  "toChainId": 137,
  "fromToken": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "toToken": "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174",
  "amount": "1000000000"
}
```

**Example Request**:
```bash
curl -X POST "https://api.polymarket.com/bridge/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "fromChainId": 1,
    "toChainId": 137,
    "fromToken": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
    "toToken": "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174",
    "amount": "1000000000"
  }'
```

**Example Response**:
```json
{
  "quote": {
    "fromAmount": "1000000000",
    "toAmount": "999500000",
    "estimatedGas": "150000",
    "bridgeFee": "500000",
    "exchangeRate": "0.9995",
    "estimatedTime": 600
  }
}
```

### Execute Bridge Transaction

**Endpoint**: `POST /bridge`

**Body**:
```json
{
  "fromChainId": 1,
  "toChainId": 137,
  "fromToken": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "toToken": "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174",
  "amount": "1000000000",
  "recipient": "0x...",
  "signature": "0x..."
}
```

---

## 💻 TypeScript Implementation

### Create Bridge Client

```typescript
import axios, { AxiosInstance } from 'axios';
import { ethers } from 'ethers';

interface Token {
  symbol: string;
  address: string;
  decimals: number;
  chainId: number;
}

interface BridgeQuote {
  fromAmount: string;
  toAmount: string;
  estimatedGas: string;
  bridgeFee: string;
  exchangeRate: string;
  estimatedTime: number;
}

interface BridgeTransaction {
  txHash: string;
  status: 'pending' | 'confirmed' | 'failed';
  fromChainId: number;
  toChainId: number;
  amount: string;
  recipient: string;
}

class BridgeClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: 'https://api.polymarket.com/bridge',
      timeout: 30000,
    });
  }

  async getSupportedTokens(): Promise<Token[]> {
    const response = await this.client.get('/tokens');
    return response.data.tokens;
  }

  async getQuote(
    fromChainId: number,
    toChainId: number,
    fromToken: string,
    toToken: string,
    amount: string
  ): Promise<BridgeQuote> {
    const response = await this.client.post('/quote', {
      fromChainId,
      toChainId,
      fromToken,
      toToken,
      amount,
    });
    return response.data.quote;
  }

  async bridge(
    fromChainId: number,
    toChainId: number,
    fromToken: string,
    toToken: string,
    amount: string,
    recipient: string,
    wallet: ethers.Wallet
  ): Promise<BridgeTransaction> {
    // Create message to sign
    const message = ethers.utils.solidityKeccak256(
      ['uint256', 'uint256', 'address', 'address', 'uint256', 'address'],
      [fromChainId, toChainId, fromToken, toToken, amount, recipient]
    );

    // Sign message
    const signature = await wallet.signMessage(ethers.utils.arrayify(message));

    // Execute bridge
    const response = await this.client.post('/bridge', {
      fromChainId,
      toChainId,
      fromToken,
      toToken,
      amount,
      recipient,
      signature,
    });

    return response.data;
  }

  async getBridgeStatus(txHash: string): Promise<BridgeTransaction> {
    const response = await this.client.get(`/status/${txHash}`);
    return response.data;
  }
}

// Usage
const bridge = new BridgeClient();
```

### Example: Bridge USDC from Ethereum to Polygon

```typescript
async function bridgeUSDC() {
  const bridge = new BridgeClient();
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!);

  // USDC addresses
  const USDC_ETH = '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48';
  const USDC_POLYGON = '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174';

  // Get quote for $1000
  const amount = '1000000000'; // 1000 USDC (6 decimals)

  console.log('📊 Getting bridge quote...');
  const quote = await bridge.getQuote(
    1, // Ethereum
    137, // Polygon
    USDC_ETH,
    USDC_POLYGON,
    amount
  );

  console.log('💰 Quote:');
  console.log(`  You send: ${parseFloat(quote.fromAmount) / 1e6} USDC`);
  console.log(`  You receive: ${parseFloat(quote.toAmount) / 1e6} USDC`);
  console.log(`  Fee: ${parseFloat(quote.bridgeFee) / 1e6} USDC`);
  console.log(`  Estimated time: ${quote.estimatedTime}s`);

  // Execute bridge
  console.log('\n🌉 Executing bridge...');
  const tx = await bridge.bridge(
    1,
    137,
    USDC_ETH,
    USDC_POLYGON,
    amount,
    wallet.address,
    wallet
  );

  console.log(`✅ Bridge initiated: ${tx.txHash}`);

  // Monitor status
  let status = tx.status;
  while (status === 'pending') {
    await new Promise(resolve => setTimeout(resolve, 10000));
    const update = await bridge.getBridgeStatus(tx.txHash);
    status = update.status;
    console.log(`Status: ${status}`);
  }

  if (status === 'confirmed') {
    console.log('🎉 Bridge complete!');
  } else {
    console.log('❌ Bridge failed');
  }

  return tx;
}
```

### Example: Swap Tokens

```typescript
interface SwapQuote {
  fromToken: string;
  toToken: string;
  fromAmount: string;
  toAmount: string;
  exchangeRate: string;
  slippage: string;
  priceImpact: string;
}

async function swapTokens(
  fromToken: string,
  toToken: string,
  amount: string
): Promise<void> {
  const bridge = new BridgeClient();

  // Get swap quote
  const quote = await bridge.client.post<{ quote: SwapQuote }>('/swap/quote', {
    fromToken,
    toToken,
    amount,
  });

  console.log('🔄 Swap Quote:');
  console.log(`  Exchange Rate: ${quote.data.quote.exchangeRate}`);
  console.log(`  Price Impact: ${quote.data.quote.priceImpact}%`);
  console.log(`  Slippage: ${quote.data.quote.slippage}%`);

  // Execute swap
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!);
  
  // Sign swap transaction
  const message = ethers.utils.solidityKeccak256(
    ['address', 'address', 'uint256'],
    [fromToken, toToken, amount]
  );
  const signature = await wallet.signMessage(ethers.utils.arrayify(message));

  const swapTx = await bridge.client.post('/swap', {
    fromToken,
    toToken,
    amount,
    signature,
    slippageTolerance: 0.5, // 0.5%
  });

  console.log(`✅ Swap executed: ${swapTx.data.txHash}`);
}
```

---

## 🐍 Python Implementation

### Create Bridge Client

```python
import requests
from typing import Dict, List
from eth_account import Account
from eth_account.messages import encode_defunct

class BridgeClient:
    def __init__(self):
        self.base_url = "https://api.polymarket.com/bridge"
        self.session = requests.Session()
    
    def get_supported_tokens(self) -> List[Dict]:
        response = self.session.get(f"{self.base_url}/tokens")
        response.raise_for_status()
        return response.json()["tokens"]
    
    def get_quote(
        self,
        from_chain_id: int,
        to_chain_id: int,
        from_token: str,
        to_token: str,
        amount: str
    ) -> Dict:
        payload = {
            "fromChainId": from_chain_id,
            "toChainId": to_chain_id,
            "fromToken": from_token,
            "toToken": to_token,
            "amount": amount
        }
        
        response = self.session.post(f"{self.base_url}/quote", json=payload)
        response.raise_for_status()
        return response.json()["quote"]
    
    def bridge(
        self,
        from_chain_id: int,
        to_chain_id: int,
        from_token: str,
        to_token: str,
        amount: str,
        recipient: str,
        private_key: str
    ) -> Dict:
        # Sign message
        account = Account.from_key(private_key)
        message = encode_defunct(text=f"{from_chain_id}{to_chain_id}{from_token}{to_token}{amount}{recipient}")
        signed = account.sign_message(message)
        
        payload = {
            "fromChainId": from_chain_id,
            "toChainId": to_chain_id,
            "fromToken": from_token,
            "toToken": to_token,
            "amount": amount,
            "recipient": recipient,
            "signature": signed.signature.hex()
        }
        
        response = self.session.post(f"{self.base_url}/bridge", json=payload)
        response.raise_for_status()
        return response.json()
    
    def get_bridge_status(self, tx_hash: str) -> Dict:
        response = self.session.get(f"{self.base_url}/status/{tx_hash}")
        response.raise_for_status()
        return response.json()

# Usage
bridge = BridgeClient()

# Get quote
quote = bridge.get_quote(
    from_chain_id=1,
    to_chain_id=137,
    from_token="0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
    to_token="0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174",
    amount="1000000000"
)

print(f"💰 Bridge Quote:")
print(f"  Fee: {float(quote['bridgeFee']) / 1e6} USDC")
print(f"  Time: {quote['estimatedTime']}s")
```

---

## 🔐 Security Best Practices

### 1. Validate Quotes
```typescript
function validateQuote(quote: BridgeQuote, maxSlippage: number = 0.01): boolean {
  const slippage = 1 - parseFloat(quote.exchangeRate);
  
  if (slippage > maxSlippage) {
    console.warn(`⚠️ High slippage: ${(slippage * 100).toFixed(2)}%`);
    return false;
  }
  
  return true;
}
```

### 2. Check Allowances
```typescript
async function checkAllowance(
  token: string,
  owner: string,
  spender: string,
  provider: ethers.providers.Provider
): Promise<boolean> {
  const contract = new ethers.Contract(
    token,
    ['function allowance(address,address) view returns (uint256)'],
    provider
  );
  
  const allowance = await contract.allowance(owner, spender);
  return allowance.gt(0);
}
```

### 3. Monitor Transactions
```typescript
async function monitorBridge(txHash: string, bridge: BridgeClient) {
  const maxAttempts = 60; // 10 minutes
  let attempts = 0;
  
  while (attempts < maxAttempts) {
    const status = await bridge.getBridgeStatus(txHash);
    
    console.log(`⏳ Status: ${status.status} (${attempts + 1}/${maxAttempts})`);
    
    if (status.status === 'confirmed') {
      console.log('✅ Bridge successful!');
      return true;
    } else if (status.status === 'failed') {
      console.log('❌ Bridge failed');
      return false;
    }
    
    attempts++;
    await new Promise(resolve => setTimeout(resolve, 10000));
  }
  
  console.log('⏰ Bridge timeout');
  return false;
}
```

---

## ⚡ Use Cases

### 1. Onboarding Flow
```typescript
async function onboardUser(userAddress: string, amount: string) {
  const bridge = new BridgeClient();
  
  console.log('👋 Starting onboarding...');
  
  // Step 1: Get quote
  const quote = await bridge.getQuote(1, 137, USDC_ETH, USDC_POLYGON, amount);
  
  // Step 2: Execute bridge
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!);
  const tx = await bridge.bridge(1, 137, USDC_ETH, USDC_POLYGON, amount, userAddress, wallet);
  
  // Step 3: Monitor completion
  await monitorBridge(tx.txHash, bridge);
  
  console.log('🎉 Onboarding complete!');
}
```

### 2. Auto-Rebalance
```typescript
async function autoRebalance(targetBalance: number) {
  const bridge = new BridgeClient();
  const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
  
  // Check current balance on Polygon
  const balance = await provider.getBalance(wallet.address);
  const balanceUSDC = parseFloat(ethers.utils.formatUnits(balance, 6));
  
  if (balanceUSDC < targetBalance) {
    const deficit = targetBalance - balanceUSDC;
    const amount = ethers.utils.parseUnits(deficit.toString(), 6);
    
    console.log(`📊 Rebalancing ${deficit} USDC...`);
    await onboardUser(wallet.address, amount.toString());
  }
}
```

---

## 🔗 Related Documentation

- [CLOB API](./polymarket-clob.md)
- [CTF Framework](./polymarket-ctf.md)

---

## 📝 Additional Resources

- **Bridge Docs**: https://docs.polymarket.com/api-reference/bridge
