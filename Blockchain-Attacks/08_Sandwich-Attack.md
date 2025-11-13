# Sandwich Attack

## 📋 Attack Information
**Category**: DeFi MEV Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Attacker "sandwiches" victim's transaction between two of their own transactions, profiting from price manipulation.

## 💀 Attack Flow

```
Victim transaction: Buy 100 ETH of UNI

Sandwich Bot:
┌─────────────────────────────────┐
│ 1. Front-run: Buy UNI (↑ price)│
├─────────────────────────────────┤
│ 2. Victim: Buys at high price   │
├─────────────────────────────────┤
│ 3. Back-run: Sell UNI (↓ profit)│
└─────────────────────────────────┘

Result: 
- Victim suffers slippage loss
- Bot profits from price manipulation
```

## 💀 Detailed Example

### Step-by-Step
```
Initial state:
UNI price: $10

1. Victim submits: Buy $100,000 of UNI
   Gas: 50 Gwei
   Expected: 10,000 UNI at $10 each

2. Bot detects transaction in mempool

3. Bot Front-run transaction:
   Buy $50,000 of UNI
   Gas: 100 Gwei (executes first)
   Result: UNI price → $11

4. Victim's transaction executes:
   Buys at $11 per UNI
   Gets only 9,091 UNI (not 10,000!)
   
5. Bot Back-run transaction:
   Sells UNI at $11
   Profit: $50,000 * (11/10 - 1) = $5,000

6. Final:
   - Victim lost: $1,000 to slippage
   - Bot profit: $5,000
   - Price returns to $10
```

## 🛡️ Defense Measures

### 1. Slippage Protection
```typescript
// ✅ Strict slippage tolerance
const path = [WETH, UNI];
const amountIn = ethers.utils.parseEther('100');

// Calculate expected output
const amountsOut = await router.getAmountsOut(amountIn, path);
const expectedOut = amountsOut[1];

// Set acceptable slippage
const slippageTolerance = 0.005; // 0.5%
const minAmountOut = expectedOut.mul(1000 - slippageTolerance * 1000).div(1000);

await router.swapExactTokensForTokens(
  amountIn,
  minAmountOut, // If sandwich causes >0.5% slippage, tx reverts
  path,
  to,
  deadline
);
```

### 2. Private Transactions (Flashbots)
```typescript
/**
 * Send transaction privately
 */
import { FlashbotsBundleProvider } from '@flashbots/ethers-provider-bundle';

// Transaction not visible in public mempool
const flashbotsProvider = await FlashbotsBundleProvider.create(
  provider,
  authSigner
);

// Submit bundle (multiple transactions atomic)
const bundle = [
  {
    signedTransaction: await wallet.signTransaction(tx),
  },
];

const bundleSubmission = await flashbotsProvider.sendBundle(
  bundle,
  targetBlockNumber
);

// Cannot be sandwiched
```

### 3. Limit Orders
```typescript
/**
 * Use limit orders instead of market orders
 */
// ✅ Limit order executes at specific price or better
const limitOrder = {
  tokenIn: WETH,
  tokenOut: UNI,
  amountIn: ethers.utils.parseEther('100'),
  minPrice: ethers.utils.parseUnits('10', 18), // $10 per UNI
  expiry: Date.now() + 3600000, // 1 hour
};

// Order waits until price is favorable
// No sandwich risk
```

### 4. Split Orders
```typescript
/**
 * Split large orders into smaller chunks
 */
const totalAmount = ethers.utils.parseEther('100');
const chunks = 10;
const chunkSize = totalAmount.div(chunks);

// Execute 10 smaller trades instead of 1 large
for (let i = 0; i < chunks; i++) {
  await router.swapExactTokensForTokens(
    chunkSize,
    minAmountOut.div(chunks),
    path,
    to,
    deadline
  );
  
  // Wait between trades
  await sleep(30000); // 30 seconds
}

// Smaller price impact → less profitable for sandwich bots
```

### 5. MEV-Protected DEXs
```typescript
/**
 * Use DEXs with built-in sandwich protection
 */
const SANDWICH_RESISTANT_DEXS = [
  {
    name: 'CowSwap',
    method: 'Batch auctions - trades grouped',
    protection: 'No public mempool exposure',
  },
  {
    name: '1inch Fusion',
    method: 'RFQ with professional market makers',
    protection: 'Off-chain matching',
  },
  {
    name: 'Uniswap X',
    method: 'Intent-based, Dutch auction',
    protection: 'Fillers compete on price',
  },
];
```

### 6. Post-Transaction Analysis
```typescript
/**
 * Detect if transaction was sandwiched
 */
async function detectSandwich(txHash: string): Promise<boolean> {
  const tx = await provider.getTransaction(txHash);
  const receipt = await provider.getTransactionReceipt(txHash);
  const block = await provider.getBlock(receipt.blockNumber);
  
  const txIndex = receipt.transactionIndex;
  const prevTx = block.transactions[txIndex - 1];
  const nextTx = block.transactions[txIndex + 1];
  
  // Check if same token pair
  const samePairBefore = await isSameTokenPair(prevTx, tx);
  const samePairAfter = await isSameTokenPair(nextTx, tx);
  
  // Check if same sender for prev/next
  const sameSender = await isSameSender(prevTx, nextTx);
  
  if (samePairBefore && samePairAfter && sameSender) {
    // Likely sandwich attack
    const slippage = await calculateActualSlippage(tx, receipt);
    
    Alert.alert(
      '🥪 Sandwich Attack Detected',
      `
      Your transaction was sandwiched.
      Slippage: ${(slippage * 100).toFixed(2)}%
      
      Bot address: ${await getSender(prevTx)}
      
      Recommendation: Use private transactions or MEV-protected DEXs
      `
    );
    
    return true;
  }
  
  return false;
}
```

## ✅ Best Practices

```
For Users:
□ Use private transaction pools (Flashbots)
□ Set tight slippage tolerance (0.5%)
□ Use limit orders for large trades
□ Split large orders into smaller chunks
□ Trade on MEV-protected DEXs
□ Avoid trading during high volatility

For Developers:
□ Integrate Flashbots Protect RPC
□ Show MEV protection toggle
□ Display sandwich detection warnings
□ Recommend split orders for large amounts
□ Integrate with MEV-protected protocols
```

**Created**: November 11, 2025

