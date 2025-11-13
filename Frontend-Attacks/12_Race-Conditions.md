# Race Conditions Attack

## 📋 Attack Information
**Category**: Concurrency Attack | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Multiple simultaneous operations on shared state cause inconsistent results or security bypasses.

## 💀 Attack Scenarios

### Scenario 1: Double Spending
```typescript
// ❌ Vulnerable: No concurrency control
async function transfer(to: string, amount: number) {
  const balance = await getBalance();
  
  if (balance >= amount) {
    // Race condition: Multiple transfers can pass this check
    await executeTransfer(to, amount);
    await updateBalance(balance - amount);
  }
}

// Attack: Click transfer button 10 times rapidly
// All 10 checks pass before first transfer completes
// → Overspends balance
```

### Scenario 2: Double Withdrawal
```typescript
// User clicks "Withdraw All" twice quickly
// Both requests check balance at same time
// Both see sufficient funds
// Both execute
// → Double withdrawal
```

## 🛡️ Defense Measures

### 1. Operation Locks
```typescript
/**
 * Prevent concurrent operations
 */
class OperationLock {
  private locks = new Map<string, Promise<void>>();
  
  async acquire(key: string): Promise<void> {
    while (this.locks.has(key)) {
      await this.locks.get(key);
    }
    
    const releasePromise = new Promise<void>(resolve => {
      setTimeout(resolve, 0);
    });
    
    this.locks.set(key, releasePromise);
  }
  
  release(key: string): void {
    this.locks.delete(key);
  }
}

// Usage
const lock = new OperationLock();

async function transfer(to: string, amount: number) {
  const lockKey = `transfer_${userId}`;
  
  await lock.acquire(lockKey);
  
  try {
    const balance = await getBalance();
    if (balance >= amount) {
      await executeTransfer(to, amount);
    }
  } finally {
    lock.release(lockKey);
  }
}
```

### 2. Disable Button During Operation
```typescript
// ✅ Prevent multiple clicks
const [isTransferring, setIsTransferring] = useState(false);

const handleTransfer = async () => {
  if (isTransferring) return; // Already processing
  
  setIsTransferring(true);
  
  try {
    await transfer(to, amount);
  } finally {
    setIsTransferring(false);
  }
};

<Button 
  onPress={handleTransfer}
  disabled={isTransferring}
  loading={isTransferring}
>
  {isTransferring ? 'Processing...' : 'Transfer'}
</Button>
```

### 3. Idempotency Keys
```typescript
/**
 * Use idempotency keys for operations
 */
import { v4 as uuidv4 } from 'uuid';

async function transfer(to: string, amount: number) {
  const idempotencyKey = uuidv4();
  
  const response = await api.post('/transfer', {
    to,
    amount,
    idempotencyKey,
  });
  
  return response.data;
}

// Backend:
// Stores idempotency key
// If duplicate key received, returns cached result
// Prevents duplicate execution
```

### 4. Optimistic Locking
```typescript
// ✅ Version-based concurrency control
interface Wallet {
  balance: number;
  version: number;
}

async function transfer(walletId: string, amount: number) {
  const wallet = await getWallet(walletId);
  
  // Include version in update
  const result = await api.post('/transfer', {
    walletId,
    amount,
    expectedVersion: wallet.version,
  });
  
  if (result.error === 'VERSION_CONFLICT') {
    // Another operation modified wallet
    // Retry or show error
    Alert.alert('Please try again');
  }
}
```

## ✅ Best Practices

```
□ Implement operation locks
□ Disable buttons during operations
□ Use idempotency keys
□ Implement optimistic locking
□ Show loading states clearly
□ Backend must also handle concurrency
□ Test with concurrent requests
```

**Created**: November 11, 2025

