# Integer Overflow/Underflow Attack

## 📋 Attack Information
**Category**: Smart Contract Vulnerability | **Severity**: 🔴 Critical (CVSS 9.0)

## 🎯 Attack Principle
Exploit arithmetic overflow/underflow to bypass checks or create unlimited tokens.

## 💀 Famous Case: BEC Token (2018)

```solidity
// BEC Token - Market Cap Zeroed
contract BECToken {
    mapping(address => uint) public balances;
    
    function batchTransfer(address[] recipients, uint amount) public {
        // ❌ No overflow check in Solidity <0.8.0
        uint totalAmount = recipients.length * amount;
        
        require(balances[msg.sender] >= totalAmount);
        
        // Attack:
        // recipients.length = 2
        // amount = 2^255
        // totalAmount = 2 * 2^255 = 2^256 = 0 (overflow!)
        // require(balance >= 0) ✓ passes
        // But actually mints 2 * 2^255 tokens!
    }
}

// Result: Unlimited tokens minted → Market cap → $0
```

## 💀 Attack Scenarios

### Scenario 1: Balance Underflow
```solidity
// Solidity <0.8.0
contract Vulnerable {
    mapping(address => uint) public balances;
    
    function transfer(address to, uint amount) public {
        // Assume balances[msg.sender] = 0
        balances[msg.sender] -= amount; 
        // 0 - 1 = 2^256-1 (underflow!)
        // Attacker now has max uint tokens!
        
        balances[to] += amount;
    }
}
```

### Scenario 2: Time Lock Bypass
```solidity
function withdraw() public {
    require(block.timestamp >= unlockTime);
    
    // Attack: If unlockTime not properly validated
    unlockTime = block.timestamp - 1; // Underflows to huge number
    // Or manipulation in calculation
}
```

## 🛡️ Defense Measures

### 1. Solidity 0.8.0+ (Built-in Protection)
```solidity
// ✅ Solidity >=0.8.0 automatically checks overflow
pragma solidity ^0.8.0;

contract Safe {
    function transfer(address to, uint amount) public {
        balances[msg.sender] -= amount; // Auto-reverts on underflow
        balances[to] += amount; // Auto-reverts on overflow
    }
}
```

### 2. SafeMath Library (Pre-0.8.0)
```solidity
// For Solidity <0.8.0
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

contract Safe {
    using SafeMath for uint256;
    
    function transfer(address to, uint amount) public {
        balances[msg.sender] = balances[msg.sender].sub(amount); // Safe
        balances[to] = balances[to].add(amount); // Safe
    }
}
```

### 3. Input Validation
```solidity
// ✅ Validate inputs before calculations
function batchTransfer(address[] recipients, uint amount) public {
    // Check for potential overflow
    require(recipients.length > 0 && recipients.length < 256);
    require(amount > 0 && amount < type(uint128).max);
    
    // Safe calculation
    uint totalAmount;
    unchecked {
        totalAmount = recipients.length * amount;
    }
    
    // Verify no overflow occurred
    require(totalAmount / recipients.length == amount, "Overflow detected");
    require(balances[msg.sender] >= totalAmount, "Insufficient balance");
    
    // Execute transfers
}
```

### 4. Frontend Detection
```typescript
/**
 * Detect overflow vulnerability in contract
 */
async function checkOverflowProtection(address: string): Promise<boolean> {
  const code = await provider.getCode(address);
  const metadata = await getContractMetadata(address);
  
  // Check Solidity version
  if (metadata.compiler?.version) {
    const version = metadata.compiler.version;
    const major = parseInt(version.split('.')[1]);
    
    if (major >= 8) {
      return true; // ✅ 0.8.0+ has auto protection
    }
  }
  
  // Check if using SafeMath
  const usesSafeMath = code.includes('SafeMath') || 
                       metadata.imports?.includes('SafeMath');
  
  return usesSafeMath;
}

// Warn user
if (!hasProtection) {
  Alert.alert(
    '⚠️ Contract May Have Overflow Risk',
    `
    Solidity version: ${version}
    SafeMath: ${usesSafeMath ? 'Yes' : 'No'}
    
    Recommendation: Exercise caution
    `
  );
}
```

## ✅ Best Practices
```
For Smart Contract Developers:
□ Use Solidity >=0.8.0
□ Or use SafeMath for <0.8.0
□ Validate all inputs
□ Test with extreme values
□ Audit all math operations
□ Use static analysis tools (Slither, Mythril)

For Users/Auditors:
□ Check Solidity version
□ Verify SafeMath usage
□ Test edge cases
□ Review arithmetic operations
```

**Created**: November 11, 2025

