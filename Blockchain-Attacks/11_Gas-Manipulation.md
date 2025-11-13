# Gas Manipulation Attack

## 📋 Attack Information
**Category**: Smart Contract Attack | **Severity**: 🟠 Medium (CVSS 5.0)

## 🎯 Attack Principle
Exploit gas consumption mechanisms to cause DoS or drain victim's funds through excessive gas usage.

## 💀 Attack Scenarios

### Scenario 1: Gas Limit DoS
```solidity
// ❌ Vulnerable: Unbounded loop
contract Auction {
    address[] public bidders;
    
    function refundAll() public {
        // If 10,000 bidders, may exceed block gas limit
        for (uint i = 0; i < bidders.length; i++) {
            payable(bidders[i]).transfer(bids[bidders[i]]);
        }
    }
}

// Attack: Register 10,000 fake bidders
// refundAll() becomes unexecutable
```

### Scenario 2: Gas Token Exploitation
```solidity
// Exploit gas refund mechanism (pre-London fork)
// Mint gas tokens when gas cheap
// Burn when gas expensive
// Get refund
```

## 🛡️ Defense

### 1. Pull Over Push
```solidity
// ✅ Let users withdraw themselves
mapping(address => uint) public refunds;

function withdraw() public {
    uint amount = refunds[msg.sender];
    refunds[msg.sender] = 0;
    payable(msg.sender).transfer(amount);
}
```

### 2. Gas Limits
```solidity
// ✅ Limit iterations
function batchProcess(uint start, uint count) public {
    require(count <= 50, "Batch too large");
    
    for (uint i = start; i < start + count && i < items.length; i++) {
        processItem(i);
    }
}
```

### 3. Frontend Gas Estimation
```typescript
// Warn about high gas
const estimatedGas = await contract.estimateGas.functionName();

if (estimatedGas.gt(5000000)) {
  Alert.alert('⚠️ High Gas Usage', 'This transaction will consume a lot of gas');
}
```

**Created**: November 11, 2025

