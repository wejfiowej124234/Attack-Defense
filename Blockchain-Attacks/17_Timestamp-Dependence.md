# Timestamp Dependence Attack

## 📋 Attack Information
**Category**: Smart Contract Attack | **Severity**: 🟠 Medium (CVSS 5.0)

## 🎯 Attack Principle
Miners can manipulate `block.timestamp` within ~15 second range, affecting contracts that depend on timestamps.

## 💀 Attack Scenarios

### Scenario 1: Timestamp as Randomness
```solidity
// ❌ Vulnerable: Using timestamp as random source
contract Lottery {
    function draw() public {
        // Miner can manipulate timestamp
        uint random = uint(keccak256(abi.encodePacked(block.timestamp)));
        uint winner = random % players.length;
        
        // Miner can adjust timestamp to control winner
    }
}
```

### Scenario 2: Time-Locked Functions
```solidity
// Timestamp can be shifted ±15 seconds
contract TimeLock {
    uint public unlockTime;
    
    function withdraw() public {
        require(block.timestamp >= unlockTime);
        // Miner can make this pass slightly early
    }
}
```

### Scenario 3: Deadline Bypass
```solidity
// Vulnerable to miner manipulation
function executeBeforeDeadline() public {
    require(block.timestamp < deadline); // Miner controls this
    // Execute action
}
```

## 🛡️ Defense Measures

### 1. Use Block Number Instead
```solidity
// ✅ Use block.number instead of block.timestamp
contract SafeTimeLock {
    uint public unlockBlock; // Block number, not timestamp
    
    function withdraw() public {
        require(block.number >= unlockBlock);
        // Miners cannot manipulate block number
    }
}

// Convert time to blocks
// Ethereum: ~12 seconds per block
uint blocksPerDay = 7200;
uint unlockBlock = block.number + (7 * blocksPerDay); // 7 days
```

### 2. Chainlink VRF for Randomness
```solidity
// ✅ Use Chainlink VRF for secure randomness
import "@chainlink/contracts/src/v0.8/VRFConsumerBase.sol";

contract SecureLottery is VRFConsumerBase {
    function requestRandomness() public {
        requestRandomness(keyHash, fee);
    }
    
    function fulfillRandomness(bytes32 requestId, uint randomness) internal override {
        uint winner = randomness % players.length;
        // Secure random selection
    }
}
```

### 3. Accept Timestamp Tolerance
```solidity
// ✅ Only use timestamp when ±15 seconds is acceptable
contract Auction {
    uint public endTime;
    
    function bid() public payable {
        // ±15 seconds acceptable for auction
        require(block.timestamp < endTime, "Auction ended");
        // Process bid
    }
}
```

### 4. External Time Sources
```solidity
// ✅ Use oracle for precise timing
import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

// Get real-world timestamp from oracle
// More reliable than block.timestamp
```

## ✅ Best Practices
```
□ Never use block.timestamp as randomness source
□ Use block.number for time locks when possible
□ Use Chainlink VRF for randomness
□ Only use timestamp when ±15s tolerance acceptable
□ For critical timing, use external oracles
□ Test with various timestamp values
```

**Created**: November 11, 2025

