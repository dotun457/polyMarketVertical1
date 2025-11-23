# 🟥 UMA Resolution System

Polymarket uses UMA's Optimistic Oracle for decentralized market resolution.

---

## 📚 Official Documentation

- **UMA Resolution System**: https://docs.polymarket.com/developers/resolution/UMA

---

## 🎯 Key Concepts

### What is UMA?

UMA (Universal Market Access) provides:
- **Optimistic Oracle** - Dispute resolution mechanism
- **Data Verification Module (DVM)** - Final arbitration
- **Economic guarantees** - Bond-based dispute system
- **Decentralized truth** - Community-verified outcomes

### How Polymarket Uses UMA

1. **Price Requests** - Market creators request outcome resolution
2. **Proposal Period** - Data is proposed with bond
3. **Challenge Period** - Disputes can be raised
4. **DVM Voting** - If disputed, UMA token holders vote
5. **Settlement** - Finalized outcomes resolve markets

---

## 🔄 Resolution Flow

```
Market Ends
    ↓
Price Request Initiated
    ↓
Proposer Submits Answer (with bond)
    ↓
Challenge Period (2 hours)
    ↓
    ├─→ No Dispute → Answer Accepted
    └─→ Dispute Raised → DVM Vote
            ↓
        UMA Token Holders Vote
            ↓
        Final Resolution
            ↓
        Market Settlement
```

---

## 💻 TypeScript Implementation

### UMA Contract Interaction

```typescript
import { ethers } from 'ethers';

const OPTIMISTIC_ORACLE_V3 = '0xfb55F43fB9F48F63f9269DB7Dde3BbBe1ebDC0dE';

interface PriceRequest {
  identifier: string;
  timestamp: number;
  ancillaryData: string;
  requester: string;
  currency: string;
  reward: string;
  finalFee: string;
}

class UMAResolver {
  private provider: ethers.providers.Provider;
  private oracleContract: ethers.Contract;

  constructor(provider: ethers.providers.Provider) {
    this.provider = provider;
    
    const OOV3_ABI = [
      'function requestPrice(bytes32 identifier, uint256 timestamp, bytes ancillaryData, address currency, uint256 reward) returns (uint256)',
      'function proposePrice(address requester, bytes32 identifier, uint256 timestamp, bytes ancillaryData, int256 proposedPrice) returns (uint256)',
      'function disputePrice(address requester, bytes32 identifier, uint256 timestamp, bytes ancillaryData) returns (uint256)',
      'function settle(address requester, bytes32 identifier, uint256 timestamp, bytes ancillaryData) returns (int256)',
      'function getRequest(address requester, bytes32 identifier, uint256 timestamp, bytes ancillaryData) view returns (tuple)',
      'event PriceRequestAdded(address indexed requester, bytes32 indexed identifier, uint256 timestamp, bytes ancillaryData, address currency, uint256 reward, uint256 finalFee)',
      'event PriceProposed(address indexed requester, address indexed proposer, bytes32 indexed identifier, uint256 timestamp, bytes ancillaryData, int256 proposedPrice)',
      'event PriceDisputed(address indexed requester, address indexed disputer, bytes32 indexed identifier, uint256 timestamp, bytes ancillaryData)',
      'event PriceSettled(address indexed requester, bytes32 indexed identifier, uint256 timestamp, bytes ancillaryData, int256 price)',
    ];

    this.oracleContract = new ethers.Contract(
      OPTIMISTIC_ORACLE_V3,
      OOV3_ABI,
      provider
    );
  }

  async requestPrice(
    identifier: string,
    timestamp: number,
    ancillaryData: string,
    currency: string,
    reward: string,
    wallet: ethers.Wallet
  ): Promise<ethers.ContractTransaction> {
    const contract = this.oracleContract.connect(wallet);
    
    const identifierBytes = ethers.utils.formatBytes32String(identifier);
    const ancillaryDataBytes = ethers.utils.toUtf8Bytes(ancillaryData);

    const tx = await contract.requestPrice(
      identifierBytes,
      timestamp,
      ancillaryDataBytes,
      currency,
      reward
    );

    console.log(`📝 Price request submitted: ${tx.hash}`);
    await tx.wait();
    
    return tx;
  }

  async proposePrice(
    requester: string,
    identifier: string,
    timestamp: number,
    ancillaryData: string,
    proposedPrice: number,
    wallet: ethers.Wallet
  ): Promise<ethers.ContractTransaction> {
    const contract = this.oracleContract.connect(wallet);
    
    const identifierBytes = ethers.utils.formatBytes32String(identifier);
    const ancillaryDataBytes = ethers.utils.toUtf8Bytes(ancillaryData);
    const priceScaled = ethers.utils.parseEther(proposedPrice.toString());

    const tx = await contract.proposePrice(
      requester,
      identifierBytes,
      timestamp,
      ancillaryDataBytes,
      priceScaled
    );

    console.log(`💡 Price proposed: ${proposedPrice}`);
    await tx.wait();
    
    return tx;
  }

  async disputePrice(
    requester: string,
    identifier: string,
    timestamp: number,
    ancillaryData: string,
    wallet: ethers.Wallet
  ): Promise<ethers.ContractTransaction> {
    const contract = this.oracleContract.connect(wallet);
    
    const identifierBytes = ethers.utils.formatBytes32String(identifier);
    const ancillaryDataBytes = ethers.utils.toUtf8Bytes(ancillaryData);

    const tx = await contract.disputePrice(
      requester,
      identifierBytes,
      timestamp,
      ancillaryDataBytes
    );

    console.log(`⚠️ Price disputed: ${tx.hash}`);
    await tx.wait();
    
    return tx;
  }

  async settlePrice(
    requester: string,
    identifier: string,
    timestamp: number,
    ancillaryData: string,
    wallet: ethers.Wallet
  ): Promise<number> {
    const contract = this.oracleContract.connect(wallet);
    
    const identifierBytes = ethers.utils.formatBytes32String(identifier);
    const ancillaryDataBytes = ethers.utils.toUtf8Bytes(ancillaryData);

    const tx = await contract.settle(
      requester,
      identifierBytes,
      timestamp,
      ancillaryDataBytes
    );

    const receipt = await tx.wait();
    
    // Parse settled price from event
    const settledEvent = receipt.events?.find(e => e.event === 'PriceSettled');
    const settledPrice = settledEvent?.args?.price;

    console.log(`✅ Price settled: ${ethers.utils.formatEther(settledPrice)}`);
    
    return parseFloat(ethers.utils.formatEther(settledPrice));
  }

  async getPriceRequest(
    requester: string,
    identifier: string,
    timestamp: number,
    ancillaryData: string
  ): Promise<PriceRequest> {
    const identifierBytes = ethers.utils.formatBytes32String(identifier);
    const ancillaryDataBytes = ethers.utils.toUtf8Bytes(ancillaryData);

    const request = await this.oracleContract.getRequest(
      requester,
      identifierBytes,
      timestamp,
      ancillaryDataBytes
    );

    return request;
  }

  async monitorPriceRequest(
    requester: string,
    identifier: string,
    timestamp: number,
    ancillaryData: string
  ): Promise<void> {
    const identifierBytes = ethers.utils.formatBytes32String(identifier);
    
    console.log('👀 Monitoring price request...');

    // Listen for proposal
    this.oracleContract.on(
      this.oracleContract.filters.PriceProposed(requester, null, identifierBytes),
      (req, proposer, id, ts, data, price) => {
        console.log(`💡 Price proposed: ${ethers.utils.formatEther(price)} by ${proposer}`);
      }
    );

    // Listen for dispute
    this.oracleContract.on(
      this.oracleContract.filters.PriceDisputed(requester, null, identifierBytes),
      (req, disputer, id, ts, data) => {
        console.log(`⚠️ Price disputed by ${disputer}`);
      }
    );

    // Listen for settlement
    this.oracleContract.on(
      this.oracleContract.filters.PriceSettled(requester, identifierBytes),
      (req, id, ts, data, price) => {
        console.log(`✅ Price settled: ${ethers.utils.formatEther(price)}`);
      }
    );
  }
}

// Usage
const provider = new ethers.providers.JsonRpcProvider(process.env.RPC_URL);
const resolver = new UMAResolver(provider);
```

### Example: Request Market Resolution

```typescript
async function requestMarketResolution(
  marketQuestion: string,
  resolutionSource: string
) {
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
  const resolver = new UMAResolver(provider);

  // Prepare ancillary data
  const ancillaryData = JSON.stringify({
    question: marketQuestion,
    resolutionSource: resolutionSource,
    timestamp: Date.now(),
  });

  // Request price
  await resolver.requestPrice(
    'YES_OR_NO_QUERY',
    Math.floor(Date.now() / 1000),
    ancillaryData,
    '0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174', // USDC on Polygon
    ethers.utils.parseUnits('100', 6).toString(), // 100 USDC reward
    wallet
  );

  console.log('✅ Resolution requested');
}
```

### Example: Propose Resolution

```typescript
async function proposeResolution(
  requesterAddress: string,
  marketQuestion: string,
  outcome: 'YES' | 'NO'
) {
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
  const resolver = new UMAResolver(provider);

  const ancillaryData = JSON.stringify({
    question: marketQuestion,
    resolutionSource: 'https://...',
    timestamp: Date.now(),
  });

  // Propose outcome (1 for YES, 0 for NO)
  const proposedPrice = outcome === 'YES' ? 1 : 0;

  await resolver.proposePrice(
    requesterAddress,
    'YES_OR_NO_QUERY',
    Math.floor(Date.now() / 1000),
    ancillaryData,
    proposedPrice,
    wallet
  );

  console.log(`✅ Proposed: ${outcome}`);
}
```

---

## 🐍 Python Implementation

### UMA Resolver

```python
from web3 import Web3
from eth_account import Account
import json

class UMAResolver:
    def __init__(self, web3: Web3):
        self.web3 = web3
        self.oracle_address = '0xfb55F43fB9F48F63f9269DB7Dde3BbBe1ebDC0dE'
        
        self.abi = [
            {
                "inputs": [
                    {"name": "identifier", "type": "bytes32"},
                    {"name": "timestamp", "type": "uint256"},
                    {"name": "ancillaryData", "type": "bytes"},
                    {"name": "currency", "type": "address"},
                    {"name": "reward", "type": "uint256"}
                ],
                "name": "requestPrice",
                "outputs": [{"type": "uint256"}],
                "type": "function"
            },
            {
                "inputs": [
                    {"name": "requester", "type": "address"},
                    {"name": "identifier", "type": "bytes32"},
                    {"name": "timestamp", "type": "uint256"},
                    {"name": "ancillaryData", "type": "bytes"},
                    {"name": "proposedPrice", "type": "int256"}
                ],
                "name": "proposePrice",
                "outputs": [{"type": "uint256"}],
                "type": "function"
            }
        ]
        
        self.contract = self.web3.eth.contract(
            address=self.oracle_address,
            abi=self.abi
        )
    
    def request_price(
        self,
        identifier: str,
        timestamp: int,
        ancillary_data: str,
        currency: str,
        reward: int,
        private_key: str
    ):
        account = Account.from_key(private_key)
        
        identifier_bytes = Web3.toBytes(text=identifier).ljust(32, b'\0')
        ancillary_bytes = Web3.toBytes(text=ancillary_data)
        
        tx = self.contract.functions.requestPrice(
            identifier_bytes,
            timestamp,
            ancillary_bytes,
            currency,
            reward
        ).build_transaction({
            'from': account.address,
            'nonce': self.web3.eth.get_transaction_count(account.address),
            'gas': 500000,
            'gasPrice': self.web3.eth.gas_price
        })
        
        signed = self.web3.eth.account.sign_transaction(tx, private_key)
        tx_hash = self.web3.eth.send_raw_transaction(signed.rawTransaction)
        
        print(f"📝 Price request submitted: {tx_hash.hex()}")
        return tx_hash
    
    def propose_price(
        self,
        requester: str,
        identifier: str,
        timestamp: int,
        ancillary_data: str,
        proposed_price: float,
        private_key: str
    ):
        account = Account.from_key(private_key)
        
        identifier_bytes = Web3.toBytes(text=identifier).ljust(32, b'\0')
        ancillary_bytes = Web3.toBytes(text=ancillary_data)
        price_scaled = Web3.toWei(proposed_price, 'ether')
        
        tx = self.contract.functions.proposePrice(
            requester,
            identifier_bytes,
            timestamp,
            ancillary_bytes,
            price_scaled
        ).build_transaction({
            'from': account.address,
            'nonce': self.web3.eth.get_transaction_count(account.address),
            'gas': 500000,
            'gasPrice': self.web3.eth.gas_price
        })
        
        signed = self.web3.eth.account.sign_transaction(tx, private_key)
        tx_hash = self.web3.eth.send_raw_transaction(signed.rawTransaction)
        
        print(f"💡 Price proposed: {proposed_price}")
        return tx_hash

# Usage
w3 = Web3(Web3.HTTPProvider('https://polygon-rpc.com'))
resolver = UMAResolver(w3)
```

---

## 📊 Resolution Monitoring

### Monitor Resolution Status

```typescript
interface ResolutionStatus {
  state: 'Requested' | 'Proposed' | 'Disputed' | 'Settled';
  proposedPrice?: number;
  proposer?: string;
  disputeTime?: number;
  settledPrice?: number;
}

class ResolutionMonitor {
  private resolver: UMAResolver;

  constructor(resolver: UMAResolver) {
    this.resolver = resolver;
  }

  async getStatus(
    requester: string,
    identifier: string,
    timestamp: number,
    ancillaryData: string
  ): Promise<ResolutionStatus> {
    const request = await this.resolver.getPriceRequest(
      requester,
      identifier,
      timestamp,
      ancillaryData
    );

    // Determine state based on request data
    if (request.finalFee === '0') {
      return { state: 'Requested' };
    }

    // Additional logic to determine state
    return { state: 'Settled' };
  }

  async waitForResolution(
    requester: string,
    identifier: string,
    timestamp: number,
    ancillaryData: string,
    maxWaitTime: number = 7200 // 2 hours
  ): Promise<number> {
    const startTime = Date.now();

    while (Date.now() - startTime < maxWaitTime * 1000) {
      try {
        const price = await this.resolver.settlePrice(
          requester,
          identifier,
          timestamp,
          ancillaryData,
          new ethers.Wallet(process.env.PRIVATE_KEY!, this.resolver['provider'])
        );

        return price;
      } catch (error) {
        // Not yet settleable
        console.log('⏳ Waiting for resolution...');
        await new Promise(resolve => setTimeout(resolve, 60000)); // Check every minute
      }
    }

    throw new Error('Resolution timeout');
  }
}
```

---

## 🔐 Security & Economics

### Bond Requirements

```typescript
interface BondConfig {
  proposalBond: string;
  disputeBond: string;
  minimumBond: string;
}

async function getBondRequirements(): Promise<BondConfig> {
  // Typical bonds
  return {
    proposalBond: '10000000000', // 10,000 USDC
    disputeBond: '20000000000', // 20,000 USDC (2x proposal)
    minimumBond: '1000000000', // 1,000 USDC
  };
}
```

### Dispute Economics

```typescript
function calculateDisputeIncentive(
  proposalBond: number,
  correctOutcome: boolean
): number {
  // If disputer is correct, they earn the proposal bond
  // If disputer is wrong, they lose their dispute bond
  
  if (correctOutcome) {
    return proposalBond; // Profit
  } else {
    return -proposalBond * 2; // Loss (their bond)
  }
}
```

---

## ⚡ Best Practices

### 1. Ancillary Data Format
```typescript
interface AncillaryData {
  question: string;
  resolutionSource: string;
  resolutionCriteria: string;
  timestamp: number;
  additionalInfo?: Record<string, any>;
}

function formatAncillaryData(data: AncillaryData): string {
  return JSON.stringify(data, null, 0);
}
```

### 2. Validation
```typescript
function validateProposal(
  question: string,
  proposedOutcome: number,
  evidence: string[]
): boolean {
  // Validate that proposal has sufficient evidence
  if (evidence.length === 0) {
    console.error('❌ No evidence provided');
    return false;
  }

  // Validate outcome is binary (0 or 1)
  if (proposedOutcome !== 0 && proposedOutcome !== 1) {
    console.error('❌ Invalid outcome value');
    return false;
  }

  return true;
}
```

---

## 🔗 Related Documentation

- [CTF Framework](./polymarket-ctf.md)
- [Gamma Markets](./polymarket-gamma.md)

---

## 📝 Additional Resources

- **UMA Docs**: https://docs.uma.xyz/
- **Optimistic Oracle**: https://docs.uma.xyz/protocol-overview/how-does-umas-oracle-work
