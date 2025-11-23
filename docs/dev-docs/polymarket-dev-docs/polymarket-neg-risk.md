# 🟦 Negative Risk

Polymarket's Negative Risk system handles markets with exclusive outcomes where buying "YES" on one outcome is equivalent to selling "NO" on others.

---

## 📚 Official Documentation

- **Negative Risk Overview**: https://docs.polymarket.com/developers/neg-risk/overview

---

## 🎯 Key Concepts

### What is Negative Risk?

Negative Risk (NegRisk) is a market structure where:
- **Outcomes are mutually exclusive**
- Buying one outcome implies betting against all others
- More efficient for multi-outcome markets
- Reduces required collateral

### Example: Presidential Election

In a market with outcomes [Trump, Harris, Other]:
- Buying "Trump YES" = Selling "Harris NO" + Selling "Other NO"
- Only need to hold one position instead of multiple

---

## 📐 Mathematical Model

### Standard CTF vs NegRisk

**Standard CTF** (requires full collateral):
```
Position Value = Collateral × Outcome Probability
Total Collateral = Sum of all outcome positions
```

**NegRisk** (optimized collateral):
```
Position Value = Collateral × (1 - Sum of other probabilities)
Total Collateral = Max position value
```

### Example Calculation

Market: [A: 40%, B: 35%, C: 25%]

**Standard CTF**:
- Buy 100 of A: Requires 100 USDC
- Buy 100 of B: Requires 100 USDC
- Total: 200 USDC

**NegRisk**:
- Buy 100 of A: Requires 100 USDC
- Buy 100 of B: Uses same 100 USDC collateral
- Total: 100 USDC

---

## 💻 TypeScript Implementation

### NegRisk Adapter Contract

```typescript
import { ethers } from 'ethers';

const NEGRISK_ADAPTER = '0xd91E80cF2E7be2e162c6513ceD06f1dD0dA35296';

const NEGRISK_ABI = [
  'function splitPosition(address collateralToken, bytes32 conditionId, uint256[] calldata partition, uint256 amount)',
  'function mergePositions(address collateralToken, bytes32 conditionId, uint256[] calldata partition, uint256 amount)',
  'function getPositionId(bytes32 conditionId, uint256 outcomeIndex) pure returns (uint256)',
  'function getOutcomeSlotCount(bytes32 conditionId) view returns (uint256)',
];

class NegRiskClient {
  private contract: ethers.Contract;
  private provider: ethers.providers.Provider;

  constructor(provider: ethers.providers.Provider) {
    this.provider = provider;
    this.contract = new ethers.Contract(
      NEGRISK_ADAPTER,
      NEGRISK_ABI,
      provider
    );
  }

  async splitPosition(
    collateralToken: string,
    conditionId: string,
    partition: number[],
    amount: ethers.BigNumber,
    wallet: ethers.Wallet
  ): Promise<ethers.ContractTransaction> {
    const contract = this.contract.connect(wallet);
    
    return await contract.splitPosition(
      collateralToken,
      conditionId,
      partition,
      amount
    );
  }

  async mergePositions(
    collateralToken: string,
    conditionId: string,
    partition: number[],
    amount: ethers.BigNumber,
    wallet: ethers.Wallet
  ): Promise<ethers.ContractTransaction> {
    const contract = this.contract.connect(wallet);
    
    return await contract.mergePositions(
      collateralToken,
      conditionId,
      partition,
      amount
    );
  }

  getPositionId(conditionId: string, outcomeIndex: number): string {
    return ethers.utils.solidityKeccak256(
      ['bytes32', 'uint256'],
      [conditionId, outcomeIndex]
    );
  }

  async getOutcomeSlotCount(conditionId: string): Promise<number> {
    const count = await this.contract.getOutcomeSlotCount(conditionId);
    return count.toNumber();
  }
}

// Usage
const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
const negRisk = new NegRiskClient(provider);
```

---

## 🔄 NegRisk Operations

### 1. Split Collateral (NegRisk)

```typescript
async function splitPositionNegRisk(
  amount: number,
  conditionId: string,
  wallet: ethers.Wallet
) {
  const negRisk = new NegRiskClient(wallet.provider!);
  
  console.log(`💰 Splitting ${amount} USDC (NegRisk mode)...`);

  const amountBN = ethers.utils.parseUnits(amount.toString(), 6);

  // Approve USDC
  const usdcContract = new ethers.Contract(
    '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174', // USDC on Polygon
    ['function approve(address spender, uint256 amount) returns (bool)'],
    wallet
  );

  const approveTx = await usdcContract.approve(NEGRISK_ADAPTER, amountBN);
  await approveTx.wait();

  // Get outcome count
  const outcomeCount = await negRisk.getOutcomeSlotCount(conditionId);
  
  // Create partition (all outcomes)
  const partition = Array.from({ length: outcomeCount }, (_, i) => i + 1);

  // Split position
  const splitTx = await negRisk.splitPosition(
    '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174',
    conditionId,
    partition,
    amountBN,
    wallet
  );

  await splitTx.wait();
  console.log('✅ NegRisk position split complete');

  // Get position IDs
  const positionIds = partition.map(index =>
    negRisk.getPositionId(conditionId, index)
  );

  console.log('Position IDs:', positionIds);
  return positionIds;
}
```

### 2. Merge Positions (NegRisk)

```typescript
async function mergePositionNegRisk(
  amount: number,
  conditionId: string,
  wallet: ethers.Wallet
) {
  const negRisk = new NegRiskClient(wallet.provider!);
  
  console.log(`🔄 Merging NegRisk positions back to ${amount} USDC...`);

  const amountBN = ethers.utils.parseUnits(amount.toString(), 6);
  const outcomeCount = await negRisk.getOutcomeSlotCount(conditionId);
  const partition = Array.from({ length: outcomeCount }, (_, i) => i + 1);

  const mergeTx = await negRisk.mergePositions(
    '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174',
    conditionId,
    partition,
    amountBN,
    wallet
  );

  await mergeTx.wait();
  console.log('✅ NegRisk positions merged');
}
```

---

## 📊 NegRisk vs Standard Comparison

### Collateral Efficiency

```typescript
interface CollateralComparison {
  standardRequired: number;
  negRiskRequired: number;
  savings: number;
  efficiency: number;
}

function compareCollateralRequirements(
  positions: { outcome: string; amount: number }[]
): CollateralComparison {
  // Standard CTF: sum of all positions
  const standardRequired = positions.reduce((sum, pos) => sum + pos.amount, 0);

  // NegRisk: maximum single position
  const negRiskRequired = Math.max(...positions.map(pos => pos.amount));

  const savings = standardRequired - negRiskRequired;
  const efficiency = (savings / standardRequired) * 100;

  console.log('💰 Collateral Comparison:');
  console.log(`  Standard CTF: ${standardRequired} USDC`);
  console.log(`  NegRisk: ${negRiskRequired} USDC`);
  console.log(`  Savings: ${savings} USDC (${efficiency.toFixed(1)}%)`);

  return {
    standardRequired,
    negRiskRequired,
    savings,
    efficiency,
  };
}

// Example
const comparison = compareCollateralRequirements([
  { outcome: 'Trump', amount: 1000 },
  { outcome: 'Harris', amount: 800 },
  { outcome: 'Other', amount: 200 },
]);
```

---

## 🎯 Use Cases

### 1. Multi-Outcome Sports Markets

```typescript
interface SportsOutcome {
  team: string;
  odds: number;
  betAmount: number;
}

async function betOnMultipleTeams(
  outcomes: SportsOutcome[],
  conditionId: string,
  wallet: ethers.Wallet
) {
  const negRisk = new NegRiskClient(wallet.provider!);

  // Calculate total required collateral (max bet)
  const maxBet = Math.max(...outcomes.map(o => o.betAmount));
  
  console.log(`🏈 Betting on ${outcomes.length} teams`);
  console.log(`💰 Collateral needed: ${maxBet} USDC (vs ${outcomes.reduce((s, o) => s + o.betAmount, 0)} in standard)`);

  // Split position once
  await splitPositionNegRisk(maxBet, conditionId, wallet);

  console.log('✅ Bets placed on:');
  outcomes.forEach(outcome => {
    console.log(`  ${outcome.team}: ${outcome.betAmount} USDC @ ${outcome.odds}`);
  });
}
```

### 2. Political Elections

```typescript
interface Candidate {
  name: string;
  currentPrice: number;
  targetPrice: number;
  allocation: number;
}

async function diversifiedElectionBet(
  candidates: Candidate[],
  totalBudget: number,
  conditionId: string,
  wallet: ethers.Wallet
) {
  console.log('🗳️ Diversified Election Bet:');
  console.log(`Total Budget: ${totalBudget} USDC`);

  // With NegRisk, we can bet on multiple candidates with the same collateral
  for (const candidate of candidates) {
    const betAmount = totalBudget * candidate.allocation;
    console.log(`\n${candidate.name}:`);
    console.log(`  Allocation: ${(candidate.allocation * 100).toFixed(1)}%`);
    console.log(`  Amount: ${betAmount.toFixed(2)} USDC`);
    console.log(`  Current Price: ${candidate.currentPrice}`);
    console.log(`  Target Price: ${candidate.targetPrice}`);
  }

  // Execute single split for all positions
  await splitPositionNegRisk(totalBudget, conditionId, wallet);
  
  console.log('\n✅ All positions opened with NegRisk optimization');
}
```

---

## 🐍 Python Implementation

### NegRisk Client

```python
from web3 import Web3
from eth_account import Account

class NegRiskClient:
    def __init__(self, web3: Web3):
        self.web3 = web3
        self.adapter_address = '0xd91E80cF2E7be2e162c6513ceD06f1dD0dA35296'
        
        self.abi = [
            {
                "inputs": [
                    {"name": "collateralToken", "type": "address"},
                    {"name": "conditionId", "type": "bytes32"},
                    {"name": "partition", "type": "uint256[]"},
                    {"name": "amount", "type": "uint256"}
                ],
                "name": "splitPosition",
                "outputs": [],
                "type": "function"
            }
        ]
        
        self.contract = self.web3.eth.contract(
            address=self.adapter_address,
            abi=self.abi
        )
    
    def split_position(
        self,
        collateral_token: str,
        condition_id: bytes,
        partition: list,
        amount: int,
        private_key: str
    ):
        account = Account.from_key(private_key)
        
        tx = self.contract.functions.splitPosition(
            collateral_token,
            condition_id,
            partition,
            amount
        ).build_transaction({
            'from': account.address,
            'nonce': self.web3.eth.get_transaction_count(account.address),
            'gas': 500000,
            'gasPrice': self.web3.eth.gas_price
        })
        
        signed = self.web3.eth.account.sign_transaction(tx, private_key)
        tx_hash = self.web3.eth.send_raw_transaction(signed.rawTransaction)
        
        print(f"✅ NegRisk split: {tx_hash.hex()}")
        return tx_hash

# Usage
w3 = Web3(Web3.HTTPProvider('https://polygon-rpc.com'))
neg_risk = NegRiskClient(w3)
```

---

## 📈 Advanced Strategies

### Portfolio Hedging

```typescript
async function hedgePortfolio(
  primaryBet: { outcome: string; amount: number },
  hedges: { outcome: string; amount: number }[],
  conditionId: string,
  wallet: ethers.Wallet
) {
  console.log('🛡️ Portfolio Hedging Strategy:');
  console.log(`Primary: ${primaryBet.outcome} - ${primaryBet.amount} USDC`);
  
  hedges.forEach(hedge => {
    console.log(`Hedge: ${hedge.outcome} - ${hedge.amount} USDC`);
  });

  const totalCollateral = Math.max(
    primaryBet.amount,
    ...hedges.map(h => h.amount)
  );

  console.log(`\n💰 Total Collateral: ${totalCollateral} USDC`);
  console.log(`📊 Savings vs Standard: ${(primaryBet.amount + hedges.reduce((s, h) => s + h.amount, 0)) - totalCollateral} USDC`);

  await splitPositionNegRisk(totalCollateral, conditionId, wallet);
  console.log('✅ Hedged portfolio created');
}
```

---

## ⚡ Best Practices

### 1. Validate Partition

```typescript
function validateNegRiskPartition(
  partition: number[],
  outcomeCount: number
): boolean {
  // Check all outcomes are included
  if (partition.length !== outcomeCount) {
    console.error('❌ Partition must include all outcomes');
    return false;
  }

  // Check no duplicates
  if (new Set(partition).size !== partition.length) {
    console.error('❌ Partition contains duplicates');
    return false;
  }

  // Check valid range
  if (partition.some(p => p < 1 || p > outcomeCount)) {
    console.error('❌ Invalid outcome index');
    return false;
  }

  return true;
}
```

### 2. Calculate Optimal Allocation

```typescript
function calculateOptimalAllocation(
  outcomes: { name: string; probability: number }[],
  budget: number
): { name: string; allocation: number }[] {
  // Kelly Criterion for optimal bet sizing
  return outcomes.map(outcome => ({
    name: outcome.name,
    allocation: budget * outcome.probability,
  }));
}
```

---

## 🔗 Related Documentation

- [CTF Framework](./polymarket-ctf.md)
- [CLOB API](./polymarket-clob.md)
- [UMA Resolution](./polymarket-resolution.md)

---

## 📝 Additional Resources

- **NegRisk Whitepaper**: https://docs.polymarket.com/neg-risk-whitepaper
- **Smart Contracts**: https://github.com/Polymarket/neg-risk-ctf-adapter
