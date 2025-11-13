# Timing Attacks (Cryptographic)

## 📋 Attack Information
**Category**: Side-Channel Attack | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Measure execution time to extract secret information like keys or passwords.

## 💀 Attack Example

### Password Comparison Timing
```javascript
// ❌ Vulnerable: Early exit comparison
function compareTokens(stored: string, provided: string): boolean {
  if (stored.length !== provided.length) return false;
  
  for (let i = 0; i < stored.length; i++) {
    if (stored[i] !== provided[i]) {
      return false; // Early exit - timing leak!
    }
  }
  return true;
}

// Attacker measures response time:
// 'aaaa' → 1ms (fails at position 0)
// 'paaa' → 2ms (fails at position 1) ← 'p' is correct!
// 'praa' → 3ms (fails at position 2) ← 'r' is correct!
// Continues until full token recovered
```

## 🛡️ Defense Measures

### 1. Constant-Time Comparison
```typescript
// ✅ crypto.timingSafeEqual
import crypto from 'crypto';

function constantTimeCompare(a: string, b: string): boolean {
  const bufA = Buffer.from(a, 'utf8');
  const bufB = Buffer.from(b, 'utf8');
  
  // Ensure same length
  if (bufA.length !== bufB.length) {
    // Compare with dummy to maintain constant time
    crypto.timingSafeEqual(bufA, Buffer.alloc(bufA.length));
    return false;
  }
  
  // Constant-time comparison
  return crypto.timingSafeEqual(bufA, bufB);
}
```

### 2. Hash-Based Comparison
```typescript
// ✅ Compare hashes (constant-time)
function secureCompare(a: string, b: string): boolean {
  const hashA = crypto.createHash('sha256').update(a).digest();
  const hashB = crypto.createHash('sha256').update(b).digest();
  
  return crypto.timingSafeEqual(hashA, hashB);
}
```

### 3. Add Random Delay
```typescript
// ✅ Randomized timing
async function authenticate(token: string): Promise<boolean> {
  const startTime = Date.now();
  
  const valid = constantTimeCompare(storedToken, token);
  
  // Add random delay (100-200ms)
  const elapsed = Date.now() - startTime;
  const targetTime = 150 + Math.random() * 50;
  
  if (elapsed < targetTime) {
    await sleep(targetTime - elapsed);
  }
  
  return valid;
}
```

### 4. bcrypt/Argon2 (Built-in Protection)
```typescript
// ✅ These have constant-time comparison built-in
import bcrypt from 'bcrypt';

const hash = await bcrypt.hash(password, 10);
const valid = await bcrypt.compare(password, hash);
// bcrypt.compare is timing-safe
```

## ✅ Best Practices

```
□ Use crypto.timingSafeEqual for comparison
□ Use bcrypt/Argon2 for passwords
□ Add random delays to mask timing
□ Never use early-exit comparisons
□ Hash before comparing when possible
□ Same execution time for all code paths
```

**Created**: November 11, 2025

