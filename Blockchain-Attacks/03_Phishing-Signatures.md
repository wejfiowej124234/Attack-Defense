# Phishing Signatures Attack

## 📋 Attack Information
**Category**: Social Engineering + Smart Contract | **Severity**: 🔴 Critical (CVSS 9.5)

## 🎯 Attack Principle
Fake websites trick users into signing malicious transactions that appear legitimate but transfer assets to attackers.

## 💀 Attack Scenarios

### Scenario 1: Fake Approval
```javascript
// Fake Uniswap website shows: "Approve USDT for swap"
// Actual transaction:
await usdtContract.approve(
  attackerAddress, // ❌ Not Uniswap!
  ethers.constants.MaxUint256 // Unlimited
);

// User thinks: Approving Uniswap
// Reality: Approving attacker
// → All USDT stolen
```

### Scenario 2: NFT Phishing
```javascript
// Fake OpenSea: "List your NFT for sale"
// Actual transaction:
await nftContract.setApprovalForAll(
  attackerAddress, // ❌ Not OpenSea!
  true
);

// → Attacker transfers all NFTs
```

### Scenario 3: Blind Signing
```javascript
// Website shows: "Claim airdrop of 1000 tokens"
// User clicks "Sign"
// Actual transaction hidden:
{
  from: userAddress,
  to: userAddress,
  value: 0,
  data: "0x..." // transferFrom(user, attacker, ALL_TOKENS)
}
```

## 🛡️ Defense Measures

### 1. Transaction Simulation
```typescript
/**
 * Simulate transaction before signing
 */
import { Tenderly } from '@tenderly/sdk';

async function simulateTransaction(tx: Transaction): Promise<SimulationResult> {
  const tenderly = new Tenderly();
  
  // Simulate execution
  const simulation = await tenderly.simulate({
    from: tx.from,
    to: tx.to,
    value: tx.value,
    data: tx.data,
  });
  
  // Analyze state changes
  const warnings: string[] = [];
  
  for (const change of simulation.stateChanges) {
    // Check for approvals
    if (change.type === 'approval') {
      if (change.amount === 'unlimited') {
        warnings.push(`⚠️ Unlimited approval to ${change.spender}`);
      }
    }
    
    // Check for transfers
    if (change.type === 'transfer') {
      warnings.push(`💸 Transfer ${change.amount} ${change.token} to ${change.to}`);
    }
  }
  
  return { simulation, warnings };
}

// Show to user before signing
const result = await simulateTransaction(tx);
if (result.warnings.length > 0) {
  Alert.alert('⚠️ Transaction Warning', result.warnings.join('\n'));
}
```

### 2. Human-Readable Transaction Display
```typescript
/**
 * Decode transaction to human-readable format
 */
import { Interface } from 'ethers/lib/utils';

async function decodeTransaction(tx: Transaction): Promise<DecodedTx> {
  // Get contract ABI
  const abi = await getContractABI(tx.to);
  const iface = new Interface(abi);
  
  try {
    const decoded = iface.parseTransaction({ data: tx.data });
    
    return {
      function: decoded.name,
      params: decoded.args,
      readable: formatReadable(decoded),
    };
  } catch (error) {
    return { 
      function: 'Unknown',
      warning: '⚠️ Cannot decode - possible scam!' 
    };
  }
}

// Display
<TransactionPreview>
  <h3>Review Transaction</h3>
  
  <Field label="Function">
    {decoded.function}
  </Field>
  
  <Field label="Contract">
    {tx.to}
    <VerifiedBadge verified={isVerified(tx.to)} />
  </Field>
  
  {decoded.function === 'approve' && (
    <Alert severity="warning">
      ⚠️ You are granting token approval
      Spender: {decoded.params.spender}
      Amount: {decoded.params.amount}
    </Alert>
  )}
  
  <Button onClick={sign}>
    I understand, sign transaction
  </Button>
</TransactionPreview>
```

### 3. URL Verification
```typescript
/**
 * Verify website authenticity
 */
const OFFICIAL_DOMAINS = {
  uniswap: ['app.uniswap.org'],
  opensea: ['opensea.io'],
  metamask: ['metamask.io'],
};

function verifyWebsite(): void {
  const currentDomain = window.location.hostname;
  
  // Check against known phishing sites
  const phishingList = await fetchPhishingList();
  if (phishingList.includes(currentDomain)) {
    document.body.innerHTML = `
      <div style="text-align:center; padding:50px;">
        <h1>🚨 PHISHING SITE DETECTED</h1>
        <p>This is a fake website!</p>
        <p>Official site: ${getOfficialSite()}</p>
      </div>
    `;
    return;
  }
  
  // Check SSL certificate
  if (window.location.protocol !== 'https:') {
    showWarning('⚠️ Insecure connection');
  }
}
```

### 4. Wallet Protection
```typescript
/**
 * Built-in phishing detection in wallet
 */
// MetaMask-style warning
async function checkTransactionSafety(tx: Transaction): Promise<void> {
  // 1. Check if recipient is known scammer
  const isKnownScammer = await checkScammerDatabase(tx.to);
  if (isKnownScammer) {
    throw new Error('⚠️ Recipient address flagged as scammer!');
  }
  
  // 2. Check for unlimited approvals
  if (tx.data.includes('approve')) {
    const decoded = decodeApprove(tx.data);
    if (decoded.amount === MAX_UINT256) {
      const confirm = await showDialog({
        title: '⚠️ Unlimited Approval',
        message: 'You are approving unlimited spending. Continue?',
      });
      if (!confirm) throw new Error('User rejected');
    }
  }
  
  // 3. Check transaction origin
  if (tx.origin !== 'wallet-initiated') {
    showWarning('Transaction requested by website');
  }
}
```

## ✅ User Protection Tips
```
Before Signing:
□ Verify website URL (check spelling!)
□ Look for HTTPS and valid certificate
□ Read transaction details carefully
□ Check contract address on Etherscan
□ If unsure, reject the transaction

Red Flags:
□ Urgent language ("Act now or lose tokens!")
□ Too good to be true ("Free 1000 ETH")
□ Unfamiliar websites
□ Requests for private key/seed phrase
□ Unexpected transaction requests
```

**Created**: November 11, 2025

