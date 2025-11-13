# MEV Attack (Maximal Extractable Value)

## 📋 Attack Information
**Category**: DeFi Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Miners/validators reorder, insert, or censor transactions to extract maximum value, harming regular users.

## 💀 MEV Attack Types

### 1. Frontrunning
```
User submits: Buy 100 ETH of UNI at 50 Gwei

MEV bot detects →
Submits same transaction at 100 Gwei
→ Bot's transaction executes first
→ Price increases
→ User buys at higher price
→ Bot profits
```

### 2. Sandwich Attack
```
User transaction: Buy UNI
↓
MEV bot:
1. Front-run: Buy UNI first (increases price)
2. User transaction executes (at higher price)
3. Back-run: Sell UNI (profit)

Result: User suffers slippage loss
```

### 3. Liquidation Frontrunning
```
User's collateral < debt → liquidatable

MEV bots race to liquidate:
- Monitor mempool
- Detect liquidation opportunity
- High gas price to frontrun
- Get liquidation reward
```

## 💰 MEV Statistics
```
2021 MEV extracted: $690M
2022 MEV extracted: $750M
Single highest MEV: $1.4M

Daily thousands of MEV transactions on Ethereum
```

## 🛡️ Defense Measures

### 1. Private Transaction Pool (Flashbots Protect)
```typescript
/**
 * Use Flashbots to avoid public mempool
 */
import { FlashbotsBundleProvider } from '@flashbots/ethers-provider-bundle';

// ✅ Transaction not visible in public mempool
const flashbotsProvider = await FlashbotsBundleProvider.create(
  provider,
  authSigner,
  'https://relay.flashbots.net'
);

// Send private transaction
const signedTransaction = await wallet.signTransaction(transaction);

const flashbotsTransaction = {
  signedTransaction,
};

// Send directly to miners, not public
const bundleSubmission = await flashbotsProvider.sendRawBundle(
  [flashbotsTransaction],
  targetBlockNumber
);
```

### 2. Slippage Protection
```typescript
// ✅ Set reasonable slippage tolerance
const amountOutMin = expectedAmount * (1 - slippage);

await router.swapExactTokensForTokens(
  amountIn,
  amountOutMin, // Minimum acceptable output
  path,
  to,
  deadline
);

// Recommended slippage:
const SLIPPAGE_CONFIG = {
  stablecoin: 0.001, // 0.1%
  normal: 0.005,     // 0.5%
  volatile: 0.01,    // 1%
  large: 0.03,       // 3%
};
```

### 3. MEV Detection
```typescript
/**
 * Detect if transaction suffered MEV attack
 */
async function detectMEV(txHash: string): Promise<MEVAnalysis> {
  const tx = await provider.getTransaction(txHash);
  const receipt = await provider.getTransactionReceipt(txHash);
  const block = await provider.getBlock(receipt.blockNumber);
  
  // Find surrounding transactions
  const txIndex = receipt.transactionIndex;
  const prevTx = block.transactions[txIndex - 1];
  const nextTx = block.transactions[txIndex + 1];
  
  // Check for sandwich attack pattern
  const isSandwiched = 
    prevTx && nextTx &&
    await isSameTokenPair(prevTx, tx) &&
    await isSameTokenPair(nextTx, tx) &&
    await isSameSender(prevTx, nextTx);
  
  // Calculate actual slippage
  const expectedOutput = await calculateExpectedOutput(tx);
  const actualOutput = await getActualOutput(receipt);
  const slippage = (expectedOutput - actualOutput) / expectedOutput;
  
  return {
    isMEV: isSandwiched || slippage > 0.02,
    type: isSandwiched ? 'sandwich' : slippage > 0.02 ? 'frontrun' : null,
    loss: expectedOutput - actualOutput,
    slippage,
  };
}

// Alert user after transaction
const analysis = await detectMEV(txHash);
if (analysis.isMEV) {
  Alert.alert(
    '⚠️ MEV Attack Detected',
    `Your transaction may have suffered ${analysis.type} attack
    Loss: ${analysis.loss} USDT
    Actual slippage: ${(analysis.slippage * 100).toFixed(2)}%
    
    Recommendation: Use private transaction pool next time
    `
  );
}
```

### 4. Anti-MEV Configuration
```typescript
// ✅ Wallet transaction settings
const ANTI_MEV_CONFIG = {
  // 1. Use private RPC
  rpcUrl: 'https://rpc.flashbots.net', // Flashbots Protect RPC
  
  // 2. Reasonable gas price (don't overpay)
  maxPriorityFeePerGas: ethers.utils.parseUnits('2', 'gwei'),
  
  // 3. Strict deadline
  deadline: Math.floor(Date.now() / 1000) + 300, // 5 minutes
  
  // 4. Slippage protection
  slippage: 0.005, // 0.5%
  
  // 5. Split large orders
  splitLargeOrders: true,
  maxOrderSize: ethers.utils.parseEther('10'), // 10 ETH
};
```

### 5. Protocol Selection
```typescript
// ✅ Choose MEV-protected protocols
const MEV_PROTECTED_PROTOCOLS = [
  {
    name: 'CowSwap',
    protection: 'Batch Auction',
    score: 95,
  },
  {
    name: '1inch',
    protection: 'RFQ + Fusion',
    score: 90,
  },
  {
    name: 'Uniswap X',
    protection: 'Intent-based',
    score: 85,
  },
];
```

## ✅ User Protection
```typescript
<TransactionSettings>
  <Toggle
    label="MEV Protection (Recommended)"
    description="Send via private transaction pool to avoid frontrunning"
    value={useMEVProtection}
    onChange={setUseMEVProtection}
  />
  
  {!useMEVProtection && (
    <Alert severity="warning">
      ⚠️ MEV Protection Disabled
      
      Risks:
      - Transaction may be frontrun
      - May suffer sandwich attack
      - Higher slippage
      
      Large transactions strongly recommended to enable!
    </Alert>
  )}
  
  <SlippageSetting
    value={slippage}
    onChange={setSlippage}
    warning={slippage > 0.01 && "High slippage increases MEV risk"}
  />
</TransactionSettings>
```

**Created**: November 11, 2025

