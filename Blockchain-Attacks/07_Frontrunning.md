# Frontrunning Attack

## 📋 Attack Information
**Category**: DeFi MEV Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Attacker monitors pending transactions in mempool, copies and submits with higher gas to execute first, profiting from price impact.

## 💀 Attack Process

```
1. User submits: Buy 100 ETH of UNI
   Gas: 50 Gwei
   
2. Frontrunning bot detects transaction in mempool

3. Bot submits identical transaction:
   Gas: 100 Gwei (higher priority)
   
4. Bot's transaction executes first
   
5. Price of UNI increases
   
6. User's transaction executes at higher price
   
7. Bot sells UNI for profit
```

## 💀 Common Targets

### DEX Trades
```typescript
// User wants to buy UNI on Uniswap
await router.swapExactETHForTokens(
  amountOutMin,
  [WETH, UNI],
  userAddress,
  deadline
);

// Frontrunner sees this in mempool
// Executes same trade with higher gas
// → Gets better price
// → User gets worse price
```

### NFT Drops
```solidity
// NFT mint
function mint() public payable {
    require(msg.value == 0.1 ether);
    _mint(msg.sender, nextTokenId++);
}

// Frontrunner sees mint transaction
// Submits with higher gas
// → Frontrunner gets NFT
// → User's transaction fails
```

### Liquidations
```solidity
// DeFi liquidation
function liquidate(address user) public {
    require(isUndercollateralized(user));
    // Liquidator gets 10% bonus
}

// Many bots compete to frontrun each other
// → Gas war
// → Miners profit
```

## 🛡️ Defense Measures

### 1. Private Transaction Pool
```typescript
/**
 * Use Flashbots to hide transaction
 */
import { FlashbotsBundleProvider } from '@flashbots/ethers-provider-bundle';

// ✅ Transaction bypasses public mempool
const flashbotsProvider = await FlashbotsBundleProvider.create(
  provider,
  authSigner,
  'https://relay.flashbots.net'
);

const signedTx = await wallet.signTransaction(transaction);

// Send directly to miners
await flashbotsProvider.sendPrivateTransaction({
  transaction: signedTx,
  maxBlockNumber: currentBlock + 5,
});

// Transaction not visible to frontrunners
```

### 2. Slippage Protection
```typescript
// ✅ Set minimum acceptable output
const expectedOutput = await router.getAmountsOut(amountIn, path);
const minOutput = expectedOutput[1].mul(995).div(1000); // 0.5% slippage

await router.swapExactTokensForTokens(
  amountIn,
  minOutput, // If frontrun too much, transaction reverts
  path,
  to,
  deadline
);
```

### 3. Commit-Reveal Pattern
```solidity
// ✅ Two-step process prevents frontrunning
contract CommitReveal {
    mapping(address => bytes32) public commits;
    
    // Step 1: Commit hash
    function commit(bytes32 hash) external {
        commits[msg.sender] = hash;
    }
    
    // Step 2: Reveal (after some blocks)
    function reveal(uint amount, bytes32 salt) external {
        require(
            commits[msg.sender] == keccak256(abi.encode(amount, salt)),
            "Invalid reveal"
        );
        
        // Execute action
        _execute(amount);
    }
}
```

### 4. Gas Price Limits
```typescript
/**
 * Warn if unusual gas price needed
 */
async function checkGasPrice(): Promise<void> {
  const currentGas = await provider.getGasPrice();
  const normalGas = await getAverageGasPrice();
  
  if (currentGas.gt(normalGas.mul(3))) {
    // Gas is 3x normal
    Alert.alert(
      '⚠️ High Gas Prices',
      `
      Current gas: ${ethers.utils.formatUnits(currentGas, 'gwei')} Gwei
      Normal gas: ${ethers.utils.formatUnits(normalGas, 'gwei')} Gwei
      
      Possible gas war / frontrunning activity.
      
      Consider:
      - Using private transaction pool
      - Waiting for gas to normalize
      - Accepting higher slippage
      `
    );
  }
}
```

### 5. MEV-Protected Protocols
```typescript
/**
 * Use protocols with built-in MEV protection
 */
const MEV_PROTECTED = [
  {
    name: 'CowSwap',
    protection: 'Batch auctions',
    mevSavings: '95%',
  },
  {
    name: '1inch Fusion',
    protection: 'Intent-based RFQ',
    mevSavings: '90%',
  },
  {
    name: 'Uniswap X',
    protection: 'Dutch auctions',
    mevSavings: '85%',
  },
];
```

## ✅ User Protection Tips

```
Before Trading:
□ Use private transaction pools (Flashbots)
□ Set reasonable slippage tolerance
□ Avoid trading during high gas
□ Use MEV-protected DEXs
□ Split large orders

Settings:
□ Enable MEV protection in wallet
□ Use moderate gas prices
□ Set strict deadlines
□ Monitor mempool activity
```

**Created**: November 11, 2025

