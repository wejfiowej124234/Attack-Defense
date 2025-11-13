# Reentrancy Attack

## 📋 Attack Information
**Category**: Smart Contract Attack | **Severity**: 🔴 Critical (CVSS 10.0)

## 🎯 Attack Principle
Attacker exploits external calls to recursively call vulnerable contract functions before state updates complete.

## 💀 Famous Case: The DAO Hack (2016)

```solidity
// Vulnerable Contract
contract Victim {
    mapping(address => uint) public balances;
    
    function withdraw() public {
        uint amount = balances[msg.sender];
        
        // ❌ External call BEFORE state update
        (bool success,) = msg.sender.call{value: amount}("");
        require(success);
        
        balances[msg.sender] = 0; // Too late!
    }
}

// Attacker Contract
contract Attacker {
    Victim victim;
    
    function attack() public {
        victim.withdraw(); // Start attack
    }
    
    receive() external payable {
        if (address(victim).balance >= 1 ether) {
            victim.withdraw(); // Recursive call!
        }
    }
}

// Attack Flow:
// 1. Attacker deposits 1 ETH
// 2. Calls withdraw()
// 3. victim.call() triggers attacker's receive()
// 4. receive() calls withdraw() again (balance not yet 0!)
// 5. Repeats until victim is drained
```

**The DAO Result**: $60M stolen, Ethereum hard fork

## 🛡️ Defense Measures

### 1. Checks-Effects-Interactions Pattern
```solidity
// ✅ SECURE: Update state BEFORE external calls
function withdraw() public {
    uint amount = balances[msg.sender];
    
    // Effect: Update state FIRST
    balances[msg.sender] = 0;
    
    // Interaction: External call LAST
    (bool success,) = msg.sender.call{value: amount}("");
    require(success);
}
```

### 2. ReentrancyGuard (OpenZeppelin)
```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract Secure is ReentrancyGuard {
    mapping(address => uint) public balances;
    
    // ✅ nonReentrant modifier prevents reentrancy
    function withdraw() public nonReentrant {
        uint amount = balances[msg.sender];
        balances[msg.sender] = 0;
        
        (bool success,) = msg.sender.call{value: amount}("");
        require(success);
    }
}
```

### 3. Pull Over Push Pattern
```solidity
// ✅ Users withdraw themselves (no external calls in logic)
contract Secure {
    mapping(address => uint) public pendingWithdrawals;
    
    function addCredit(address user, uint amount) internal {
        pendingWithdrawals[user] += amount;
    }
    
    function withdraw() public {
        uint amount = pendingWithdrawals[msg.sender];
        pendingWithdrawals[msg.sender] = 0;
        
        payable(msg.sender).transfer(amount);
    }
}
```

### 4. Frontend Detection
```typescript
// Detect suspicious contract behavior
async function checkReentrancyRisk(contractAddress: string): Promise<boolean> {
  const code = await provider.getCode(contractAddress);
  
  // Check for external calls before state changes
  const hasExternalCalls = code.includes('call') || code.includes('delegatecall');
  const hasStateChanges = code.includes('SSTORE');
  
  // Simple heuristic: if external calls exist, warn user
  if (hasExternalCalls) {
    return true; // Potential risk
  }
  
  return false;
}
```

## ✅ Prevention Checklist
```
□ Update state before external calls
□ Use ReentrancyGuard modifier
□ Prefer pull over push pattern
□ Audit all external call sites
□ Test with automated tools (Slither, Mythril)
```

**Created**: November 11, 2025

