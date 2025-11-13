# Double Spending Attack

## 📋 Attack Information
**Category**: Blockchain Consensus Attack | **Severity**: 🔴 Critical (CVSS 9.0)

## 🎯 Attack Principle
Spend same coins twice by creating competing blockchain forks or exploiting low confirmation requirements.

## 💀 Attack Scenarios

### Scenario 1: 51% Attack + Double Spend
```
Attacker controls >50% network hashrate:

1. Send 1000 BTC to exchange
2. Exchange credits account after 1 confirmation
3. Withdraw different asset (ETH)
4. Mine private fork without the BTC transaction
5. Publish longer fork
6. Network accepts attacker's fork
7. Original BTC transaction disappears
8. Attacker keeps both BTC and ETH
```

### Scenario 2: Race Attack (0-conf)
```
Merchant accepts 0-confirmation transactions:

1. Broadcast Tx1: Pay merchant 1 BTC
2. Merchant sees transaction, delivers goods
3. Immediately broadcast Tx2: Send same 1 BTC to own address (higher fee)
4. Miners include Tx2 (higher fee)
5. Tx1 becomes invalid
6. Attacker keeps goods + BTC
```

### Scenario 3: Finney Attack
```
Attacker is miner:

1. Mine block with Tx1: 1 BTC to own address
2. Don't broadcast block yet
3. Send Tx2: Same 1 BTC to merchant (0-conf)
4. Merchant delivers goods
5. Broadcast mined block with Tx1
6. Tx2 becomes invalid
```

## 🛡️ Defense Measures

### 1. Wait for Multiple Confirmations
```typescript
/**
 * Wait for sufficient confirmations
 */
const CONFIRMATION_REQUIREMENTS = {
  small: 1,      // <$100
  medium: 3,     // $100-$1000
  large: 6,      // $1000-$10000
  critical: 12,  // >$10000
};

async function waitForConfirmations(
  txHash: string,
  requiredConfs: number
): Promise<void> {
  let currentConfs = 0;
  
  while (currentConfs < requiredConfs) {
    const receipt = await provider.getTransactionReceipt(txHash);
    if (!receipt) {
      await sleep(5000);
      continue;
    }
    
    const latest = await provider.getBlockNumber();
    currentConfs = latest - receipt.blockNumber + 1;
    
    console.log(`Confirmations: ${currentConfs}/${requiredConfs}`);
    await sleep(15000); // Check every 15 seconds
  }
}
```

### 2. Monitor for Reorgs
```typescript
/**
 * Detect blockchain reorganization
 */
async function monitorForReorg(txHash: string): Promise<boolean> {
  const receipt = await provider.getTransactionReceipt(txHash);
  const originalBlock = receipt.blockHash;
  
  // Wait and check if block still exists
  await sleep(60000); // Wait 1 minute
  
  const block = await provider.getBlock(originalBlock);
  
  if (!block) {
    console.error('⚠️ Block reorganization detected!');
    return true; // Reorg happened
  }
  
  return false;
}
```

### 3. Check Network Hashrate
```typescript
/**
 * Verify network security
 */
async function checkNetworkSecurity(): Promise<SecurityStatus> {
  const latestBlock = await provider.getBlock('latest');
  const difficulty = latestBlock.difficulty;
  const hashrate = calculateHashrate(difficulty);
  
  // Check if hashrate is sufficient
  const MIN_SAFE_HASHRATE = ethers.utils.parseUnits('100', 'tera'); // 100 TH/s
  
  return {
    hashrate,
    secure: hashrate.gt(MIN_SAFE_HASHRATE),
    warning: hashrate.lt(MIN_SAFE_HASHRATE) 
      ? 'Network hashrate low, double-spend risk increased' 
      : null,
  };
}
```

### 4. Display Confirmation Status
```typescript
<TransactionStatus tx={transaction}>
  <ConfirmationProgress>
    <Progress 
      value={confirmations} 
      max={requiredConfirmations}
    />
    <Text>
      {confirmations}/{requiredConfirmations} confirmations
    </Text>
  </ConfirmationProgress>
  
  {confirmations < requiredConfirmations && (
    <Alert severity="warning">
      ⚠️ Transaction not yet finalized
      Wait for {requiredConfirmations} confirmations
      to ensure no double-spend
    </Alert>
  )}
  
  {confirmations >= requiredConfirmations && (
    <Alert severity="success">
      ✅ Transaction finalized
      Safe from double-spend
    </Alert>
  )}
</TransactionStatus>
```

## ✅ Best Practices
```
For Merchants:
□ Never accept 0-confirmation for valuable goods
□ Wait 1 conf for small amounts (<$100)
□ Wait 6 confs for large amounts (>$1000)
□ Monitor for blockchain reorgs
□ Use payment processors with double-spend protection

For Users:
□ Check transaction is included in block
□ Wait for confirmations before considering payment complete
□ Be aware of network congestion
□ Use appropriate gas/fee for timely inclusion

For Exchanges:
□ Require many confirmations for deposits (6-12)
□ Monitor network hashrate
□ Detect suspicious patterns
□ Have automated reorg detection
```

**Created**: November 11, 2025

