# 🟧 Rewards System

Polymarket's Rewards system incentivizes liquidity provision and market making activities.

---

## 📚 Official Documentation

- **Rewards Overview**: https://docs.polymarket.com/developers/rewards/overview

---

## 🎯 Key Concepts

### What are Rewards?

The Rewards system provides:
- **Liquidity rewards** for market makers
- **Trading incentives** for active users
- **Market creation** bonuses
- **Referral rewards** for onboarding

### Reward Types

1. **Maker Rewards** - For providing liquidity
2. **Volume Rewards** - For trading activity
3. **Market Creator Rewards** - For creating popular markets
4. **Referral Rewards** - For bringing new users

---

## 🔌 API Endpoints

### Base URL
```
https://api.polymarket.com/rewards
```

### Get User Rewards

**Endpoint**: `GET /user/{address}`

**Example Request**:
```bash
curl "https://api.polymarket.com/rewards/user/0x..."
```

**Example Response**:
```json
{
  "address": "0x...",
  "totalRewards": "15000.50",
  "claimedRewards": "10000.00",
  "pendingRewards": "5000.50",
  "breakdown": {
    "makerRewards": "12000.00",
    "volumeRewards": "2500.50",
    "referralRewards": "500.00"
  }
}
```

### Get Rewards by Market

**Endpoint**: `GET /market/{marketId}`

**Example Request**:
```bash
curl "https://api.polymarket.com/rewards/market/0x..."
```

### Claim Rewards

**Endpoint**: `POST /claim`

**Body**:
```json
{
  "address": "0x...",
  "signature": "0x..."
}
```

---

## 💻 TypeScript Implementation

### Create Rewards Client

```typescript
import axios, { AxiosInstance } from 'axios';
import { ethers } from 'ethers';

interface UserRewards {
  address: string;
  totalRewards: string;
  claimedRewards: string;
  pendingRewards: string;
  breakdown: {
    makerRewards: string;
    volumeRewards: string;
    referralRewards: string;
  };
}

interface MarketRewards {
  marketId: string;
  totalRewards: string;
  rewardRate: string;
  eligibleUsers: number;
}

class RewardsClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: 'https://api.polymarket.com/rewards',
      timeout: 10000,
    });
  }

  async getUserRewards(address: string): Promise<UserRewards> {
    const response = await this.client.get(`/user/${address}`);
    return response.data;
  }

  async getMarketRewards(marketId: string): Promise<MarketRewards> {
    const response = await this.client.get(`/market/${marketId}`);
    return response.data;
  }

  async claimRewards(
    address: string,
    wallet: ethers.Wallet
  ): Promise<{ txHash: string; amount: string }> {
    // Create message to sign
    const message = `Claim rewards for ${address}`;
    const signature = await wallet.signMessage(message);

    const response = await this.client.post('/claim', {
      address,
      signature,
    });

    return response.data;
  }

  async getRewardsHistory(
    address: string,
    startDate?: Date,
    endDate?: Date
  ): Promise<any[]> {
    const params: any = {};
    if (startDate) params.startDate = startDate.toISOString();
    if (endDate) params.endDate = endDate.toISOString();

    const response = await this.client.get(`/history/${address}`, { params });
    return response.data;
  }
}

// Usage
const rewards = new RewardsClient();
```

### Example: Check User Rewards

```typescript
async function checkUserRewards(address: string) {
  const rewards = new RewardsClient();
  
  const userRewards = await rewards.getUserRewards(address);

  console.log('🎁 Rewards Summary:');
  console.log('═'.repeat(50));
  console.log(`Total Earned: $${parseFloat(userRewards.totalRewards).toLocaleString()}`);
  console.log(`Claimed: $${parseFloat(userRewards.claimedRewards).toLocaleString()}`);
  console.log(`Pending: $${parseFloat(userRewards.pendingRewards).toLocaleString()}`);
  
  console.log('\n📊 Breakdown:');
  console.log(`  Maker Rewards: $${parseFloat(userRewards.breakdown.makerRewards).toLocaleString()}`);
  console.log(`  Volume Rewards: $${parseFloat(userRewards.breakdown.volumeRewards).toLocaleString()}`);
  console.log(`  Referral Rewards: $${parseFloat(userRewards.breakdown.referralRewards).toLocaleString()}`);

  return userRewards;
}
```

### Example: Claim Rewards

```typescript
async function claimUserRewards(wallet: ethers.Wallet) {
  const rewards = new RewardsClient();
  
  // Check pending rewards
  const userRewards = await rewards.getUserRewards(wallet.address);
  const pending = parseFloat(userRewards.pendingRewards);

  if (pending === 0) {
    console.log('❌ No rewards to claim');
    return;
  }

  console.log(`💰 Claiming ${pending} USDC rewards...`);

  // Claim rewards
  const result = await rewards.claimRewards(wallet.address, wallet);

  console.log(`✅ Rewards claimed!`);
  console.log(`   Amount: $${result.amount}`);
  console.log(`   Tx: ${result.txHash}`);

  return result;
}
```

### Example: Track Market Maker Performance

```typescript
interface MakerMetrics {
  address: string;
  totalVolume: number;
  totalRewards: number;
  rewardRate: number;
  roi: number;
}

async function trackMakerPerformance(address: string): Promise<MakerMetrics> {
  const rewards = new RewardsClient();
  
  const userRewards = await rewards.getUserRewards(address);
  const history = await rewards.getRewardsHistory(address);

  const totalVolume = history.reduce((sum, entry) => sum + parseFloat(entry.volume), 0);
  const totalRewards = parseFloat(userRewards.breakdown.makerRewards);
  const rewardRate = totalVolume > 0 ? (totalRewards / totalVolume) * 100 : 0;

  const metrics: MakerMetrics = {
    address,
    totalVolume,
    totalRewards,
    rewardRate,
    roi: rewardRate,
  };

  console.log('📊 Market Maker Performance:');
  console.log(`  Total Volume: $${metrics.totalVolume.toLocaleString()}`);
  console.log(`  Total Rewards: $${metrics.totalRewards.toLocaleString()}`);
  console.log(`  Reward Rate: ${metrics.rewardRate.toFixed(3)}%`);
  console.log(`  ROI: ${metrics.roi.toFixed(2)}%`);

  return metrics;
}
```

---

## 🐍 Python Implementation

### Create Rewards Client

```python
import requests
from typing import Dict, List, Optional
from datetime import datetime
from eth_account import Account
from eth_account.messages import encode_defunct

class RewardsClient:
    def __init__(self):
        self.base_url = "https://api.polymarket.com/rewards"
        self.session = requests.Session()
    
    def get_user_rewards(self, address: str) -> Dict:
        response = self.session.get(f"{self.base_url}/user/{address}")
        response.raise_for_status()
        return response.json()
    
    def get_market_rewards(self, market_id: str) -> Dict:
        response = self.session.get(f"{self.base_url}/market/{market_id}")
        response.raise_for_status()
        return response.json()
    
    def claim_rewards(self, address: str, private_key: str) -> Dict:
        # Sign message
        account = Account.from_key(private_key)
        message = encode_defunct(text=f"Claim rewards for {address}")
        signed = account.sign_message(message)
        
        payload = {
            "address": address,
            "signature": signed.signature.hex()
        }
        
        response = self.session.post(f"{self.base_url}/claim", json=payload)
        response.raise_for_status()
        return response.json()
    
    def get_rewards_history(
        self,
        address: str,
        start_date: Optional[datetime] = None,
        end_date: Optional[datetime] = None
    ) -> List[Dict]:
        params = {}
        if start_date:
            params["startDate"] = start_date.isoformat()
        if end_date:
            params["endDate"] = end_date.isoformat()
        
        response = self.session.get(
            f"{self.base_url}/history/{address}",
            params=params
        )
        response.raise_for_status()
        return response.json()

# Usage
rewards = RewardsClient()

# Check rewards
user_rewards = rewards.get_user_rewards("0x...")
print(f"💰 Pending Rewards: ${user_rewards['pendingRewards']}")
```

---

## 📊 Rewards Calculation

### Maker Rewards Formula

```typescript
function calculateMakerRewards(
  providedLiquidity: number,
  totalLiquidity: number,
  rewardPool: number,
  timeProvided: number // in hours
): number {
  const liquidityShare = providedLiquidity / totalLiquidity;
  const timeWeight = Math.min(timeProvided / 24, 1); // Max weight at 24 hours
  
  const rewards = rewardPool * liquidityShare * timeWeight;
  
  return rewards;
}

// Example
const myRewards = calculateMakerRewards(
  10000, // My liquidity
  100000, // Total liquidity
  5000, // Reward pool
  48 // Hours provided
);

console.log(`Expected rewards: $${myRewards.toFixed(2)}`);
```

### Volume-Based Rewards

```typescript
function calculateVolumeRewards(
  userVolume: number,
  totalVolume: number,
  rewardPool: number
): number {
  const volumeShare = userVolume / totalVolume;
  const rewards = rewardPool * volumeShare;
  
  return rewards;
}
```

---

## 🎯 Optimization Strategies

### 1. Liquidity Optimization

```typescript
interface LiquidityStrategy {
  market: string;
  amount: number;
  expectedRewards: number;
  apr: number;
}

async function findBestLiquidityOpportunities(): Promise<LiquidityStrategy[]> {
  const rewards = new RewardsClient();
  
  // Get markets with rewards
  const markets = []; // Fetch from Gamma API
  
  const strategies: LiquidityStrategy[] = [];
  
  for (const market of markets) {
    const marketRewards = await rewards.getMarketRewards(market.id);
    
    const expectedRewards = parseFloat(marketRewards.rewardRate) * 1000; // For 1000 USDC
    const apr = (expectedRewards / 1000) * 365 * 100;
    
    strategies.push({
      market: market.id,
      amount: 1000,
      expectedRewards,
      apr,
    });
  }
  
  // Sort by APR
  strategies.sort((a, b) => b.apr - a.apr);
  
  console.log('🎯 Top Liquidity Opportunities:');
  strategies.slice(0, 5).forEach((strategy, i) => {
    console.log(`\n${i + 1}. Market: ${strategy.market.slice(0, 10)}...`);
    console.log(`   APR: ${strategy.apr.toFixed(2)}%`);
    console.log(`   Expected Daily: $${(strategy.expectedRewards / 365).toFixed(2)}`);
  });
  
  return strategies;
}
```

### 2. Referral Tracking

```typescript
interface ReferralStats {
  totalReferrals: number;
  activeReferrals: number;
  totalRewards: number;
  avgRewardPerReferral: number;
}

async function getReferralStats(address: string): Promise<ReferralStats> {
  const rewards = new RewardsClient();
  
  const userRewards = await rewards.getUserRewards(address);
  const history = await rewards.getRewardsHistory(address);
  
  const referralEntries = history.filter(entry => entry.type === 'referral');
  
  const stats: ReferralStats = {
    totalReferrals: referralEntries.length,
    activeReferrals: referralEntries.filter(e => e.active).length,
    totalRewards: parseFloat(userRewards.breakdown.referralRewards),
    avgRewardPerReferral: 0,
  };
  
  stats.avgRewardPerReferral = stats.totalRewards / stats.totalReferrals;
  
  console.log('👥 Referral Stats:');
  console.log(`  Total Referrals: ${stats.totalReferrals}`);
  console.log(`  Active: ${stats.activeReferrals}`);
  console.log(`  Total Rewards: $${stats.totalRewards.toLocaleString()}`);
  console.log(`  Avg per Referral: $${stats.avgRewardPerReferral.toFixed(2)}`);
  
  return stats;
}
```

---

## ⚡ Best Practices

### 1. Auto-Claim Strategy

```typescript
async function autoClaimRewards(wallet: ethers.Wallet, threshold: number = 100) {
  const rewards = new RewardsClient();
  
  setInterval(async () => {
    const userRewards = await rewards.getUserRewards(wallet.address);
    const pending = parseFloat(userRewards.pendingRewards);
    
    if (pending >= threshold) {
      console.log(`💰 Auto-claiming ${pending} USDC...`);
      await rewards.claimRewards(wallet.address, wallet);
      console.log('✅ Rewards claimed!');
    }
  }, 3600000); // Check every hour
}
```

### 2. Rewards Dashboard

```typescript
async function displayRewardsDashboard(address: string) {
  const rewards = new RewardsClient();
  
  const userRewards = await rewards.getUserRewards(address);
  const history = await rewards.getRewardsHistory(address);
  
  console.clear();
  console.log('╔═══════════════════════════════════════╗');
  console.log('║        POLYMARKET REWARDS             ║');
  console.log('╠═══════════════════════════════════════╣');
  console.log(`║ Total Earned:  $${parseFloat(userRewards.totalRewards).toFixed(2).padStart(18)} ║`);
  console.log(`║ Claimed:       $${parseFloat(userRewards.claimedRewards).toFixed(2).padStart(18)} ║`);
  console.log(`║ Pending:       $${parseFloat(userRewards.pendingRewards).toFixed(2).padStart(18)} ║`);
  console.log('╠═══════════════════════════════════════╣');
  console.log('║ Breakdown:                            ║');
  console.log(`║   Maker:     $${parseFloat(userRewards.breakdown.makerRewards).toFixed(2).padStart(20)} ║`);
  console.log(`║   Volume:    $${parseFloat(userRewards.breakdown.volumeRewards).toFixed(2).padStart(20)} ║`);
  console.log(`║   Referral:  $${parseFloat(userRewards.breakdown.referralRewards).toFixed(2).padStart(20)} ║`);
  console.log('╚═══════════════════════════════════════╝');
}
```

---

## 🔗 Related Documentation

- [CLOB API](./polymarket-clob.md)
- [Subgraph](./polymarket-subgraph.md)
- [Gamma API](./polymarket-gamma.md)

---

## 📝 Additional Resources

- **Rewards Program**: https://polymarket.com/rewards
- **Discord**: https://discord.gg/polymarket
