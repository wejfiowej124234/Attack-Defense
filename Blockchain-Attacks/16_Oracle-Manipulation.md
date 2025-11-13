# Oracle Manipulation Attack

## 📋 Attack Information
**Category**: DeFi Attack | **Severity**: 🔴 Critical (CVSS 9.0)

## 🎯 Attack Principle
Manipulate price oracle data to exploit DeFi protocols for profit or unfair liquidations.

## 💀 Attack Scenarios

### Scenario 1: DEX Price Manipulation
```solidity
// ❌ Vulnerable: Using single DEX as oracle
contract Lending {
    function getPrice(address token) public view returns (uint) {
        return uniswap.getPrice(token); // Single source!
    }
    
    function liquidate(address user) public {
        uint collateralValue = getPrice(collateral) * amount;
        uint debtValue = getPrice(debt) * borrowed;
        
        if (collateralValue < debtValue) {
            // Liquidate user
        }
    }
}

// Attack:
// 1. Flash loan large amount of TOKEN
// 2. Dump on Uniswap → price crashes
// 3. Trigger liquidations
// 4. Buy collateral at discount
// 5. Repay flash loan
// 6. Profit
```

### Scenario 2: Chainlink Data Staleness
```solidity
// ❌ No freshness check
function getPrice() public view returns (uint) {
    (, int price, , uint updatedAt,) = priceFeed.latestRoundData();
    return uint(price); // Not checking updatedAt!
}

// If Chainlink stops updating (network issue)
// Old price used → can be exploited
```

## 🛡️ Defense Measures

### 1. Multiple Oracle Verification
```solidity
// ✅ Use multiple oracles
function getSecurePrice(address token) public view returns (uint) {
    uint chainlinkPrice = getChainlinkPrice(token);
    uint uniswapTWAP = getUniswapTWAP(token);
    uint curvePrice = getCurvePrice(token);
    
    // Calculate median
    uint[] memory prices = new uint[](3);
    prices[0] = chainlinkPrice;
    prices[1] = uniswapTWAP;
    prices[2] = curvePrice;
    
    uint median = getMedian(prices);
    
    // Check deviation
    require(
        abs(chainlinkPrice - median) < median / 20 && // 5%
        abs(uniswapTWAP - median) < median / 20 &&
        abs(curvePrice - median) < median / 20,
        "Price deviation too large"
    );
    
    return median;
}
```

### 2. TWAP (Time-Weighted Average Price)
```solidity
// ✅ Use Uniswap V3 TWAP
import '@uniswap/v3-periphery/contracts/libraries/OracleLibrary.sol';

function getTWAPPrice(address pool, uint32 period) public view returns (uint) {
    // Average price over 'period' seconds
    // Single transaction cannot manipulate
    (int24 tick,) = OracleLibrary.consult(pool, period);
    return OracleLibrary.getQuoteAtTick(tick, 1e18, token0, token1);
}
```

### 3. Data Freshness Check
```solidity
// ✅ Verify Chainlink data freshness
function getChainlinkPrice() public view returns (uint) {
    (
        uint80 roundId,
        int price,
        uint startedAt,
        uint updatedAt,
        uint80 answeredInRound
    ) = priceFeed.latestRoundData();
    
    // Check freshness
    require(updatedAt > block.timestamp - 3600, "Price too old");
    
    // Check round
    require(answeredInRound >= roundId, "Stale price");
    
    // Check validity
    require(price > 0, "Invalid price");
    
    return uint(price);
}
```

### 4. Frontend Protection
```typescript
/**
 * Verify price consistency before transaction
 */
async function verifyPrice(token: string): Promise<PriceCheck> {
  const prices = await Promise.all([
    chainlink.getPrice(token),
    uniswapV3.getTWAP(token, 3600), // 1 hour TWAP
    binance.getPrice(token),
    coinGecko.getPrice(token),
  ]);
  
  const median = calculateMedian(prices);
  const maxDeviation = Math.max(...prices.map(p => Math.abs(p - median) / median));
  
  return {
    prices,
    median,
    maxDeviation,
    consistent: maxDeviation < 0.05, // 5%
    warning: maxDeviation > 0.1 ? 'Price deviation >10%, possible manipulation' : null,
  };
}

// Warn user
if (!priceCheck.consistent) {
  Alert.alert(
    '⚠️ Price Anomaly Detected',
    `Price deviation: ${(priceCheck.maxDeviation * 100).toFixed(2)}%
    
    Chainlink: $${prices[0]}
    Uniswap: $${prices[1]}
    Binance: $${prices[2]}
    CoinGecko: $${prices[3]}
    
    Possible Causes:
    - Oracle manipulation
    - Flash loan attack in progress
    - Extreme market volatility
    
    Recommendation: Wait for prices to stabilize
    `
  );
}
```

## 💰 Real Cases
- Compound (2020): $90M at risk
- Harvest Finance (2020): $24M
- Cream Finance (2021): $130M

## ✅ Best Practices
```
For Protocols:
□ Use multiple oracle sources
□ Implement TWAP for DEX prices
□ Check data freshness
□ Set deviation thresholds
□ Have circuit breakers

For Users:
□ Check price consistency before large trades
□ Avoid protocols with single oracle
□ Be cautious during high volatility
```

**Created**: November 11, 2025

