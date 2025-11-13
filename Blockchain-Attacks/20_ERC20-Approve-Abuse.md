# ERC20 Approve Abuse Attack (Unlimited Approval)

## 📋 Attack Information
**Category**: DeFi Smart Contract Attack | **Severity**: 🔴 Critical (CVSS 9.0)

## 🎯 Attack Principle
Users approve unlimited token allowance to DeFi contracts. If the contract is hacked or malicious, attackers can steal all tokens.

## 💀 Attack Scenarios

### Scenario 1: Malicious Contract
```solidity
// User needs to approve before Uniswap swap
await usdtContract.approve(
  uniswapRouter,
  ethers.constants.MaxUint256 // ❌ Unlimited approval
);

// If uniswapRouter has vulnerability or is upgraded to malicious contract
// Can transfer all user's USDT at any time
await usdtContract.transferFrom(
  user,
  attacker,
  userBalance // Steal everything
);
```

### Scenario 2: Phishing Website
```javascript
// Fake Uniswap website
// Tricks user to approve attacker's contract

const maliciousContract = '0xAttacker...';

await token.approve(
  maliciousContract,
  ethers.constants.MaxUint256
);

// Attacker immediately transfers
await token.transferFrom(victim, attacker, amount);
```

### Scenario 3: Forgotten Approvals
```
2020: Approved SushiSwap V1
2024: SushiSwap V1 vulnerability discovered
      or private key leaked
      → 4-year-old approval still valid
      → Attacker exploits vulnerability to steal tokens
```

## 🛡️ Defense Measures

### 1. Precise Approval Amount
```typescript
// ❌ DANGEROUS: Unlimited approval
await tokenContract.approve(
  spenderAddress,
  ethers.constants.MaxUint256 // 2^256-1
);

// ✅ SAFE: Only approve needed amount
const amountToSwap = ethers.utils.parseUnits('100', 18);

await tokenContract.approve(
  spenderAddress,
  amountToSwap // Only approve 100 tokens
);
```

### 2. Approval Management Interface
```typescript
/**
 * Display all user approvals
 */
interface Approval {
  token: string;
  tokenName: string;
  spender: string;
  spenderName: string;
  allowance: BigNumber;
  lastUpdated: number;
}

async function getAllApprovals(userAddress: string): Promise<Approval[]> {
  // Scan user's approve events
  const approvals: Approval[] = [];
  
  // Get all tokens user holds
  const tokens = await getTokenBalances(userAddress);
  
  for (const token of tokens) {
    const contract = new ethers.Contract(token.address, ERC20_ABI, provider);
    
    // Query approve events
    const filter = contract.filters.Approval(userAddress);
    const events = await contract.queryFilter(filter);
    
    for (const event of events) {
      const allowance = await contract.allowance(
        userAddress,
        event.args.spender
      );
      
      if (allowance.gt(0)) {
        approvals.push({
          token: token.address,
          tokenName: token.name,
          spender: event.args.spender,
          spenderName: await getContractName(event.args.spender),
          allowance,
          lastUpdated: event.blockNumber,
        });
      }
    }
  }
  
  return approvals;
}

// UI Display
<ApprovalManager>
  {approvals.map(approval => (
    <ApprovalItem key={`${approval.token}-${approval.spender}`}>
      <Token>{approval.tokenName}</Token>
      <Spender>{approval.spenderName}</Spender>
      <Amount>
        {approval.allowance.eq(ethers.constants.MaxUint256) 
          ? '♾️ Unlimited' 
          : formatUnits(approval.allowance)}
      </Amount>
      
      {approval.allowance.eq(ethers.constants.MaxUint256) && (
        <Warning>⚠️ Unlimited Approval Risk</Warning>
      )}
      
      <Button onClick={() => revokeApproval(approval)}>
        Revoke
      </Button>
    </ApprovalItem>
  ))}
</ApprovalManager>
```

### 3. Revoke Approval
```typescript
/**
 * Revoke token approval
 */
async function revokeApproval(
  tokenAddress: string,
  spenderAddress: string
) {
  const contract = new ethers.Contract(
    tokenAddress,
    ERC20_ABI,
    signer
  );
  
  // Set allowance to 0
  const tx = await contract.approve(spenderAddress, 0);
  await tx.wait();
  
  Alert.alert('✅ Approval Revoked');
}
```

### 4. Pre-Approval Warning
```typescript
/**
 * Show risk warning before approve
 */
const handleApprove = async (amount: BigNumber) => {
  const isUnlimited = amount.eq(ethers.constants.MaxUint256);
  
  if (isUnlimited) {
    const confirmed = await showConfirmDialog({
      title: '⚠️ Unlimited Approval Risk',
      message: `
        You are granting unlimited allowance to:
        ${spenderName}
        
        Risks:
        - If contract has vulnerability, you may lose all ${tokenName}
        - If contract is hacked, all tokens can be stolen
        - Approval requires manual revocation
        
        Recommendations:
        ✅ Approve only the amount needed for this transaction
        ❌ Do not use unlimited approval
        
        Continue?
      `,
      buttons: [
        { text: 'Cancel', style: 'cancel' },
        { text: 'Change to Exact Amount', onPress: () => approveExact() },
        { text: 'I Understand Risks, Continue', style: 'destructive' },
      ],
    });
    
    if (!confirmed) return;
  }
  
  // Execute approval
  await executeApprove(amount);
};
```

### 5. Use Permit Instead of Approve
```solidity
// ✅ ERC20 Permit: Off-chain signature, on-chain execution
// No separate approve transaction needed

// User signs
const signature = await signer._signTypedData(
  domain,
  {
    Permit: [
      { name: 'owner', type: 'address' },
      { name: 'spender', type: 'address' },
      { name: 'value', type: 'uint256' },
      { name: 'nonce', type: 'uint256' },
      { name: 'deadline', type: 'uint256' },
    ],
  },
  {
    owner: userAddress,
    spender: contractAddress,
    value: amount,
    nonce: await token.nonces(userAddress),
    deadline: Math.floor(Date.now() / 1000) + 3600, // Expires in 1 hour
  }
);

// Contract calls permit + actual operation in one transaction
await contract.swapWithPermit(
  amount,
  signature.v,
  signature.r,
  signature.s,
  deadline
);
```

## 🔧 Recommended Tools

```
Revoke.cash (https://revoke.cash)
- View all approvals
- One-click revoke
- Multi-chain support

Etherscan Token Approvals
- View approvals on Etherscan
- Direct revoke on website
```

## ✅ Best Practices

```typescript
// Before Transaction
1. ✅ Only approve needed amount
2. ✅ Verify contract address
3. ✅ Check if contract is audited

// After Transaction
4. ✅ Revoke unnecessary approvals
5. ✅ Regularly review all approvals
6. ✅ Use revoke.cash to clean old approvals
```

**Created**: November 11, 2025

