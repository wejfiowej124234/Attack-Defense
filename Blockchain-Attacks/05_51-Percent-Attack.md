# 51% Attack

## 📋 Attack Information
**Category**: Blockchain Consensus Attack | **Severity**: 🔴 Critical (CVSS 9.5)

## 🎯 Attack Principle
Attacker controls over 50% of network's mining/staking power, enabling blockchain manipulation, double-spending, and transaction censorship.

## 💀 Attack Capabilities

### What Attacker CAN Do
```
✓ Double-spend transactions
✓ Reverse recent transactions
✓ Prevent new transactions from confirming
✓ Prevent other miners from mining blocks
✓ Monopolize mining rewards
```

### What Attacker CANNOT Do
```
✗ Steal coins from other wallets
✗ Change past blocks (too deep)
✗ Create coins from nothing
✗ Change block rewards
✗ Reverse very old transactions
```

## 💀 Attack Scenarios

### Scenario 1: Double-Spending
```
Attacker controls 51% hashrate:

1. Send 1000 BTC to exchange
2. Exchange credits account after 6 confirmations
3. Withdraw different asset (ETH)
4. Attacker mines private fork WITHOUT the 1000 BTC transaction
5. Private fork becomes longer than public chain
6. Network accepts attacker's fork
7. Original 1000 BTC transaction disappears
8. Attacker keeps both BTC and ETH
```

### Scenario 2: Selfish Mining
```
Attacker mines blocks but doesn't broadcast:

1. Mine block N
2. Keep it secret
3. Start mining block N+1 on private chain
4. When public network mines block N:
   - If attacker also has N+1, broadcast both
   - Attacker's chain becomes longest
   - Network reorgs to attacker's chain
5. Attacker gets more rewards than fair share
```

## 💰 Real Cases & Costs

### Successful 51% Attacks
```
- Ethereum Classic (2019): $1.1M stolen
- Ethereum Classic (2020): Multiple attacks
- Bitcoin Gold (2018): $18M stolen
- Verge (2018): $1M+ stolen
```

### Attack Cost (2024)
```
Bitcoin: $20B+ (practically impossible)
Ethereum: $40B+ (practically impossible)
Small PoW coins: $1,000 - $100,000 per hour

Attack cost calculator:
https://www.crypto51.app
```

## 🛡️ Defense Measures

### 1. Wait for More Confirmations
```typescript
/**
 * Adjust confirmations based on transaction value
 */
const CONFIRMATION_REQUIREMENTS = {
  low: 6,        // <$1,000 (standard)
  medium: 12,    // $1,000 - $10,000
  high: 24,      // $10,000 - $100,000
  critical: 50,  // >$100,000
};

// For smaller coins, require MORE confirmations
const NETWORK_SECURITY_MULTIPLIER = {
  bitcoin: 1.0,      // 6 confs sufficient
  ethereum: 1.0,     // 12 confs sufficient
  smallCoin: 5.0,    // 30+ confs needed
};
```

### 2. Monitor Network Hashrate
```typescript
/**
 * Check if network is under attack
 */
async function detectAttack(): Promise<AttackWarning> {
  const currentHashrate = await getCurrentHashrate();
  const historicalAverage = await getHistoricalHashrate(30); // 30 days
  
  // Sudden hashrate drop could indicate attack
  const hashrateDropPercent = 
    (historicalAverage - currentHashrate) / historicalAverage;
  
  if (hashrateDropPercent > 0.3) { // 30% drop
    return {
      warning: true,
      message: 'Network hashrate dropped 30%, possible 51% attack',
      recommendation: 'Wait for more confirmations',
    };
  }
  
  // Check for unusual reorgs
  const recentReorgs = await getRecentReorgs(24); // 24 hours
  if (recentReorgs.length > 3) {
    return {
      warning: true,
      message: 'Multiple blockchain reorgs detected',
      recommendation: 'Network may be under attack',
    };
  }
  
  return { warning: false };
}
```

### 3. Use Established Networks
```typescript
/**
 * Rate network security
 */
interface NetworkSecurity {
  name: string;
  hashrate: number;
  attackCost: number; // USD per hour
  securityScore: number; // 0-100
}

const NETWORKS: NetworkSecurity[] = [
  {
    name: 'Bitcoin',
    hashrate: 400_000_000, // TH/s
    attackCost: 20_000_000_000,
    securityScore: 100,
  },
  {
    name: 'Ethereum',
    hashrate: 1_000_000, // GH/s (PoS now)
    attackCost: 40_000_000_000,
    securityScore: 100,
  },
  {
    name: 'SmallCoin',
    hashrate: 100, // TH/s
    attackCost: 10_000,
    securityScore: 20,
  },
];

// Warn users about low-security networks
if (network.securityScore < 50) {
  Alert.alert(
    '⚠️ Low Network Security',
    `${network.name} has low hashrate ($${network.attackCost}/hour to attack).
    
    Recommendations:
    - Wait for 50+ confirmations
    - Use for small amounts only
    - Consider using more secure network
    `
  );
}
```

### 4. Proof of Stake (PoS)
```
Ethereum's Solution:
- Switched from PoW to PoS
- Attack requires 51% of staked ETH ($40B+)
- Economic penalties for attackers
- Slashing mechanism destroys attacker's stake
- Makes attack economically irrational
```

## ✅ Best Practices

```
For Users:
□ Use major networks (Bitcoin, Ethereum)
□ Wait for sufficient confirmations
□ Monitor network health
□ Be extra cautious on small coins
□ Higher value → more confirmations

For Exchanges:
□ Require many confirmations (12-50+)
□ Monitor network hashrate
□ Detect abnormal reorgs
□ Suspend deposits if attack detected
□ Use multiple confirmation thresholds

For Developers:
□ Display network security score
□ Warn about low-security networks
□ Implement reorg detection
□ Allow manual confirmation requirements
```

**Created**: November 11, 2025

