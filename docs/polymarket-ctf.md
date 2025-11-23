# 🟪 Conditional Token Framework (CTF)

The Conditional Token Framework (CTF) is the core smart contract system powering Polymarket's prediction markets.

---

## 📚 Official Documentation

- **CTF Overview**: https://docs.polymarket.com/developers/CTF/overview

---

## 🎯 Key Concepts

### What is CTF?

The Conditional Token Framework handles:
- **Splitting collateral** into outcome tokens
- **Merging outcome tokens** back to collateral
- **Redeeming** winning outcome tokens
- **Position management** across markets
- **Market resolution** logic

### Core Contracts

1. **ConditionalTokens** - Main CTF contract
2. **CTFExchange** - Trading interface
3. **NegRiskAdapter** - Negative risk handling
4. **FixedProductMarketMaker** - AMM for liquidity

---

## 📝 Contract Addresses (Polygon)

```typescript
const CONTRACTS = {
  ConditionalTokens: '0x4D97DCd97eC945f40cF65F87097ACe5EA0476045',
  CTFExchange: '0x4bFb41d5B3570DeFd03C39a9A4D8dE6Bd8B8982E',
  NegRiskAdapter: '0xd91E80cF2E7be2e162c6513ceD06f1dD0dA35296',
  CollateralToken: '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174', // USDC
};
```

---

## 💻 TypeScript Implementation

### Setup

```bash
npm install ethers @polymarket/ctf-contracts
```

### Create CTF Client

```typescript
import { ethers } from 'ethers';

const CTF_ABI = [
  'function balanceOf(address owner, uint256 id) view returns (uint256)',
  'function balanceOfBatch(address[] calldata owners, uint256[] calldata ids) view returns (uint256[] memory)',
  'function splitPosition(address collateralToken, bytes32 parentCollectionId, bytes32 conditionId, uint256[] calldata partition, uint256 amount)',
  'function mergePositions(address collateralToken, bytes32 parentCollectionId, bytes32 conditionId, uint256[] calldata partition, uint256 amount)',
  'function redeemPositions(address collateralToken, bytes32 parentCollectionId, bytes32 conditionId, uint256[] calldata indexSets)',
  'function getPositionId(address collateralToken, bytes32 collectionId) pure returns (uint256)',
  'function getCollectionId(bytes32 parentCollectionId, bytes32 conditionId, uint256 indexSet) pure returns (bytes32)',
  'function getOutcomeSlotCount(bytes32 conditionId) view returns (uint256)',
  'function payoutNumerators(bytes32 conditionId, uint256 index) view returns (uint256)',
];

class CTFClient {
  private contract: ethers.Contract;
  private provider: ethers.providers.Provider;

  constructor(provider: ethers.providers.Provider) {
    this.provider = provider;
    this.contract = new ethers.Contract(
      CONTRACTS.ConditionalTokens,
      CTF_ABI,
      provider
    );
  }

  async getBalance(owner: string, positionId: string): Promise<ethers.BigNumber> {
    return await this.contract.balanceOf(owner, positionId);
  }

  async getBalances(
    owners: string[],
    positionIds: string[]
  ): Promise<ethers.BigNumber[]> {
    return await this.contract.balanceOfBatch(owners, positionIds);
  }

  async splitPosition(
    collateralToken: string,
    parentCollectionId: string,
    conditionId: string,
    partition: number[],
    amount: ethers.BigNumber,
    wallet: ethers.Wallet
  ): Promise<ethers.ContractTransaction> {
    const contract = this.contract.connect(wallet);
    
    return await contract.splitPosition(
      collateralToken,
      parentCollectionId,
      conditionId,
      partition,
      amount
    );
  }

  async mergePositions(
    collateralToken: string,
    parentCollectionId: string,
    conditionId: string,
    partition: number[],
    amount: ethers.BigNumber,
    wallet: ethers.Wallet
  ): Promise<ethers.ContractTransaction> {
    const contract = this.contract.connect(wallet);
    
    return await contract.mergePositions(
      collateralToken,
      parentCollectionId,
      conditionId,
      partition,
      amount
    );
  }

  async redeemPositions(
    collateralToken: string,
    parentCollectionId: string,
    conditionId: string,
    indexSets: number[],
    wallet: ethers.Wallet
  ): Promise<ethers.ContractTransaction> {
    const contract = this.contract.connect(wallet);
    
    return await contract.redeemPositions(
      collateralToken,
      parentCollectionId,
      conditionId,
      indexSets
    );
  }

  getPositionId(collateralToken: string, collectionId: string): string {
    return ethers.utils.solidityKeccak256(
      ['address', 'bytes32'],
      [collateralToken, collectionId]
    );
  }

  getCollectionId(
    parentCollectionId: string,
    conditionId: string,
    indexSet: number
  ): string {
    return ethers.utils.solidityKeccak256(
      ['bytes32', 'bytes32', 'uint256'],
      [parentCollectionId, conditionId, indexSet]
    );
  }

  async getOutcomeSlotCount(conditionId: string): Promise<number> {
    const count = await this.contract.getOutcomeSlotCount(conditionId);
    return count.toNumber();
  }

  async getPayoutNumerators(
    conditionId: string,
    outcomeCount: number
  ): Promise<number[]> {
    const numerators: number[] = [];
    
    for (let i = 0; i < outcomeCount; i++) {
      const numerator = await this.contract.payoutNumerators(conditionId, i);
      numerators.push(numerator.toNumber());
    }
    
    return numerators;
  }
}

// Usage
const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
const ctf = new CTFClient(provider);
```

---

## 🔄 Core Operations

### 1. Splitting Collateral into Outcome Tokens

```typescript
async function splitIntoOutcomes(
  amount: number, // USDC amount
  conditionId: string,
  wallet: ethers.Wallet
) {
  const ctf = new CTFClient(wallet.provider!);
  
  console.log(`💰 Splitting ${amount} USDC into outcome tokens...`);

  // Amount in USDC (6 decimals)
  const amountBN = ethers.utils.parseUnits(amount.toString(), 6);

  // First, approve CTF contract to spend USDC
  const usdcContract = new ethers.Contract(
    CONTRACTS.CollateralToken,
    ['function approve(address spender, uint256 amount) returns (bool)'],
    wallet
  );

  const approveTx = await usdcContract.approve(
    CONTRACTS.ConditionalTokens,
    amountBN
  );
  await approveTx.wait();
  console.log('✅ USDC approved');

  // Split position (binary market: [1, 2] represents YES and NO)
  const partition = [1, 2];
  const parentCollectionId = ethers.constants.HashZero;

  const splitTx = await ctf.splitPosition(
    CONTRACTS.CollateralToken,
    parentCollectionId,
    conditionId,
    partition,
    amountBN,
    wallet
  );

  await splitTx.wait();
  console.log('✅ Position split complete');

  // Calculate position IDs
  const yesCollectionId = ctf.getCollectionId(parentCollectionId, conditionId, 1);
  const noCollectionId = ctf.getCollectionId(parentCollectionId, conditionId, 2);

  const yesPositionId = ctf.getPositionId(CONTRACTS.CollateralToken, yesCollectionId);
  const noPositionId = ctf.getPositionId(CONTRACTS.CollateralToken, noCollectionId);

  console.log(`YES Position ID: ${yesPositionId}`);
  console.log(`NO Position ID: ${noPositionId}`);

  return { yesPositionId, noPositionId };
}
```

### 2. Merging Outcome Tokens back to Collateral

```typescript
async function mergeBackToCollateral(
  amount: number,
  conditionId: string,
  wallet: ethers.Wallet
) {
  const ctf = new CTFClient(wallet.provider!);
  
  console.log(`🔄 Merging outcome tokens back to ${amount} USDC...`);

  const amountBN = ethers.utils.parseUnits(amount.toString(), 6);
  const partition = [1, 2];
  const parentCollectionId = ethers.constants.HashZero;

  const mergeTx = await ctf.mergePositions(
    CONTRACTS.CollateralToken,
    parentCollectionId,
    conditionId,
    partition,
    amountBN,
    wallet
  );

  await mergeTx.wait();
  console.log('✅ Positions merged back to USDC');
}
```

### 3. Redeeming Winning Positions

```typescript
async function redeemWinningPosition(
  conditionId: string,
  wallet: ethers.Wallet
) {
  const ctf = new CTFClient(wallet.provider!);
  
  console.log('💎 Checking if market is resolved...');

  // Get payout numerators (determines winning outcome)
  const outcomeCount = await ctf.getOutcomeSlotCount(conditionId);
  const payouts = await ctf.getPayoutNumerators(conditionId, outcomeCount);

  console.log(`Payouts: ${payouts}`);

  // Find winning outcome
  const winningIndex = payouts.findIndex(p => p > 0);
  
  if (winningIndex === -1) {
    console.log('❌ Market not yet resolved');
    return;
  }

  console.log(`✅ Winning outcome: ${winningIndex === 0 ? 'YES' : 'NO'}`);

  // Redeem winning position
  const indexSet = Math.pow(2, winningIndex);
  const parentCollectionId = ethers.constants.HashZero;

  const redeemTx = await ctf.redeemPositions(
    CONTRACTS.CollateralToken,
    parentCollectionId,
    conditionId,
    [indexSet],
    wallet
  );

  await redeemTx.wait();
  console.log('✅ Winning position redeemed!');
}
```

---

## 🐍 Python Implementation

### Setup

```bash
pip install web3
```

### Create CTF Client

```python
from web3 import Web3
from eth_account import Account

class CTFClient:
    def __init__(self, web3: Web3):
        self.web3 = web3
        self.contract_address = '0x4D97DCd97eC945f40cF65F87097ACe5EA0476045'
        
        self.abi = [
            {
                "inputs": [
                    {"name": "owner", "type": "address"},
                    {"name": "id", "type": "uint256"}
                ],
                "name": "balanceOf",
                "outputs": [{"type": "uint256"}],
                "stateMutability": "view",
                "type": "function"
            },
            # ... other functions
        ]
        
        self.contract = self.web3.eth.contract(
            address=self.contract_address,
            abi=self.abi
        )
    
    def get_balance(self, owner: str, position_id: int) -> int:
        balance = self.contract.functions.balanceOf(owner, position_id).call()
        return balance
    
    def split_position(
        self,
        collateral_token: str,
        parent_collection_id: bytes,
        condition_id: bytes,
        partition: list,
        amount: int,
        private_key: str
    ):
        account = Account.from_key(private_key)
        
        tx = self.contract.functions.splitPosition(
            collateral_token,
            parent_collection_id,
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
        
        print(f"✅ Split transaction: {tx_hash.hex()}")
        return tx_hash

# Usage
w3 = Web3(Web3.HTTPProvider('https://polygon-rpc.com'))
ctf = CTFClient(w3)
```

---

## 📊 Position Tracking

### Track User Positions

```typescript
interface Position {
  positionId: string;
  outcomeIndex: number;
  balance: ethers.BigNumber;
  value: number;
}

async function getUserPositions(
  userAddress: string,
  conditionId: string
): Promise<Position[]> {
  const ctf = new CTFClient(provider);
  
  const outcomeCount = await ctf.getOutcomeSlotCount(conditionId);
  const positions: Position[] = [];

  for (let i = 0; i < outcomeCount; i++) {
    const indexSet = Math.pow(2, i);
    const collectionId = ctf.getCollectionId(
      ethers.constants.HashZero,
      conditionId,
      indexSet
    );
    const positionId = ctf.getPositionId(CONTRACTS.CollateralToken, collectionId);
    
    const balance = await ctf.getBalance(userAddress, positionId);
    
    if (balance.gt(0)) {
      positions.push({
        positionId,
        outcomeIndex: i,
        balance,
        value: parseFloat(ethers.utils.formatUnits(balance, 6)),
      });
    }
  }

  console.log(`📊 Positions for ${userAddress}:`);
  positions.forEach(pos => {
    const outcome = pos.outcomeIndex === 0 ? 'YES' : 'NO';
    console.log(`  ${outcome}: ${pos.value.toFixed(2)} tokens`);
  });

  return positions;
}
```

---

## ⚡ Best Practices

### 1. Gas Optimization

```typescript
async function batchSplitPositions(
  amounts: number[],
  conditionIds: string[],
  wallet: ethers.Wallet
) {
  // Use multicall for batch operations
  const calls = amounts.map((amount, i) => ({
    target: CONTRACTS.ConditionalTokens,
    callData: ctfInterface.encodeFunctionData('splitPosition', [
      CONTRACTS.CollateralToken,
      ethers.constants.HashZero,
      conditionIds[i],
      [1, 2],
      ethers.utils.parseUnits(amount.toString(), 6),
    ]),
  }));

  // Execute via multicall contract
  // (implementation depends on multicall setup)
}
```

### 2. Position Validation

```typescript
async function validatePosition(
  userAddress: string,
  positionId: string,
  requiredAmount: ethers.BigNumber
): Promise<boolean> {
  const ctf = new CTFClient(provider);
  
  const balance = await ctf.getBalance(userAddress, positionId);
  
  if (balance.lt(requiredAmount)) {
    console.log(`❌ Insufficient balance`);
    console.log(`  Required: ${ethers.utils.formatUnits(requiredAmount, 6)}`);
    console.log(`  Available: ${ethers.utils.formatUnits(balance, 6)}`);
    return false;
  }
  
  return true;
}
```

---

## 🔗 Related Documentation

- [UMA Resolution](./polymarket-resolution.md)
- [CLOB API](./polymarket-clob.md)
- [Negative Risk](./polymarket-neg-risk.md)

---

## 📝 Additional Resources

- **CTF Specification**: https://docs.gnosis.io/conditionaltokens/
- **Smart Contracts**: https://github.com/Polymarket/ctf-exchange
