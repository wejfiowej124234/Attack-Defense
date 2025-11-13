# Rug Pull Attack

## 📋 Attack Information
**Category**: DeFi Scam | **Severity**: 🔴 Critical (CVSS 9.5)

## 🎯 Attack Principle
Project team drains liquidity or exits with investor funds after building trust.

## 💀 Types of Rug Pulls

### 1. Liquidity Removal
```
1. Create token
2. Add liquidity on Uniswap (ETH + TOKEN)
3. Market and hype
4. Price increases
5. Remove ALL liquidity
6. Token price → $0
7. Investors can't sell
```

### 2. Code Backdoor
```solidity
// Hidden function in contract
function emergencyWithdraw() external {
    require(msg.sender == owner, "Not owner");
    
    // Owner can drain all funds anytime
    payable(owner).transfer(address(this).balance);
}

// Or: Malicious minting
function mint(address to, uint amount) external onlyOwner {
    // Owner mints unlimited tokens
    // Dumps on market
    // Price crashes
}
```

### 3. Sell Tax Manipulation
```solidity
contract RugToken {
    uint public sellTax = 1; // Shows 1%
    
    function setSellTax(uint _tax) external onlyOwner {
        sellTax = _tax; // Owner can change to 99%
    }
    
    function transfer(address to, uint amount) public {
        uint tax = amount * sellTax / 100;
        // When sellTax = 99%, only 1% transferred
    }
}
```

## 💀 Famous Cases
```
- Squid Game Token (2021): $3.4M
- AnubisDAO (2021): $60M
- Thodex (2021): $2B
- Thousands of small rug pulls: $10B+ total
```

## 🛡️ Defense Measures

### 1. Check Liquidity Lock
```typescript
/**
 * Verify liquidity is locked
 */
async function checkLiquidityLock(tokenAddress: string): Promise<LockInfo> {
  const pair = await factory.getPair(tokenAddress, WETH);
  const lpToken = new ethers.Contract(pair, LP_ABI, provider);
  
  // Check LP tokens locked in known lock contracts
  const unicrypt = await lpToken.balanceOf(UNICRYPT_ADDRESS);
  const teamFinance = await lpToken.balanceOf(TEAM_FINANCE_ADDRESS);
  
  const totalLocked = unicrypt.add(teamFinance);
  const totalLP = await lpToken.totalSupply();
  
  return {
    locked: totalLocked.gt(0),
    percentage: totalLocked.mul(100).div(totalLP).toNumber(),
    unlockDate: await getUnlockDate(lpToken),
  };
}

// Display warning
if (!lockInfo.locked) {
  Alert.alert(
    '⚠️ Liquidity Not Locked',
    'High rug pull risk! Team can remove liquidity anytime.'
  );
}
```

### 2. Contract Code Analysis
```typescript
/**
 * Detect rug pull features in contract
 */
async function detectRugFeatures(address: string): Promise<string[]> {
  const code = await provider.getCode(address);
  const risks: string[] = [];
  
  // Check for owner powers
  if (code.includes('mint') && code.includes('onlyOwner')) {
    risks.push('❌ Owner can mint unlimited tokens');
  }
  
  if (code.includes('pause') && code.includes('onlyOwner')) {
    risks.push('❌ Owner can pause trading');
  }
  
  if (code.includes('setTax') || code.includes('setFee')) {
    risks.push('❌ Owner can change tax rates');
  }
  
  if (code.includes('blacklist')) {
    risks.push('❌ Owner can blacklist addresses');
  }
  
  if (code.includes('withdraw') && code.includes('onlyOwner')) {
    risks.push('❌ Owner can withdraw funds');
  }
  
  return risks;
}
```

### 3. Team Verification
```typescript
interface ProjectAudit {
  team: {
    doxxed: boolean; // Public identities
    linkedin: string[];
    kyc: boolean;
  };
  contract: {
    audited: boolean;
    auditor: string;
    openSource: boolean;
    verified: boolean; // Etherscan verified
  };
  liquidity: {
    locked: boolean;
    lockDuration: number; // days
    lockPercentage: number;
  };
  tokenomics: {
    ownerShare: number;
    maxTx: number;
    maxWallet: number;
  };
}

function calculateRugRisk(audit: ProjectAudit): number {
  let score = 100;
  
  // Team
  if (!audit.team.doxxed) score -= 30;
  if (!audit.team.kyc) score -= 20;
  
  // Contract
  if (!audit.contract.audited) score -= 25;
  if (!audit.contract.openSource) score -= 15;
  
  // Liquidity
  if (!audit.liquidity.locked) score -= 40;
  if (audit.liquidity.lockDuration < 180) score -= 20;
  
  // Tokenomics
  if (audit.tokenomics.ownerShare > 0.2) score -= 20;
  
  return Math.max(score, 0);
}
```

### 4. Warning Signs
```typescript
const RUG_PULL_SIGNALS = [
  'Team Twitter deleted',
  'Telegram group closed',
  'Website down',
  'Large transfers from team wallet',
  'Sudden liquidity decrease',
  'GitHub activity stopped',
  'Admins disappeared',
  'Promises too good to be true',
  'Anonymous team',
  'No audit',
  'Unlocked liquidity',
];
```

## ✅ Investment Checklist
```
Before Investing:
□ Team is doxxed (public identities)
□ KYC completed
□ Contract audited by reputable firm
□ Liquidity locked for >6 months
□ Code is open source
□ Owner holds <10% of supply
□ No mint function
□ No pause function
□ No changeable tax rates
□ Active community
□ Clear roadmap
□ Real use case

Risk Score:
>80: Relatively safe
50-80: Medium risk
<50: High risk - avoid
```

**Created**: November 11, 2025

