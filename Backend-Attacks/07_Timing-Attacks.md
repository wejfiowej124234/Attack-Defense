# Timing Attacks

## 📋 Attack Information
**Category**: Side-Channel Attack | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Attacker analyzes response times to extract sensitive information like passwords or tokens.

## 💀 Attack Example

### String Comparison Timing
```javascript
// ❌ Vulnerable: Early exit on mismatch
function comparePasswords(stored: string, provided: string): boolean {
  if (stored.length !== provided.length) return false;
  
  for (let i = 0; i < stored.length; i++) {
    if (stored[i] !== provided[i]) {
      return false; // Early exit - timing leak!
    }
  }
  
  return true;
}

// Attacker tries passwords character by character:
// 'a______' → 1ms (fails at position 0)
// 'p______' → 2ms (fails at position 1) ← 'p' is correct!
// 'pa_____' → 3ms (fails at position 2) ← 'a' is correct!
// ... continues until full password found
```

## 🛡️ Defense Measures

### 1. Constant-Time Comparison
```typescript
/**
 * ✅ Constant-time string comparison
 */
import crypto from 'crypto';

function constantTimeCompare(a: string, b: string): boolean {
  // crypto.timingSafeEqual is constant-time
  const bufA = Buffer.from(a, 'utf8');
  const bufB = Buffer.from(b, 'utf8');
  
  // Ensure same length to prevent length timing
  if (bufA.length !== bufB.length) {
    // Compare with dummy buffer of same length
    crypto.timingSafeEqual(bufA, Buffer.alloc(bufA.length));
    return false;
  }
  
  return crypto.timingSafeEqual(bufA, bufB);
}
```

### 2. Use bcrypt/Argon2
```typescript
// ✅ These have built-in constant-time comparison
import bcrypt from 'bcrypt';

// Hash password
const hash = await bcrypt.hash(password, 10);

// Verify (constant-time internally)
const valid = await bcrypt.compare(providedPassword, storedHash);
```

### 3. Add Randomized Delay
```typescript
// ✅ Add random delay to mask timing
async function login(email: string, password: string) {
  const startTime = Date.now();
  
  const user = await User.findOne({ email });
  const valid = user && await bcrypt.compare(password, user.passwordHash);
  
  // Add random delay (100-200ms)
  const elapsed = Date.now() - startTime;
  const targetTime = 150 + Math.random() * 50;
  
  if (elapsed < targetTime) {
    await sleep(targetTime - elapsed);
  }
  
  if (!valid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  return res.json({ token: generateToken(user) });
}
```

## ✅ Best Practices

```
□ Use constant-time comparison functions
□ Use bcrypt/Argon2 for password hashing
□ Add randomized delays
□ Don't leak information in error messages
□ Same response time for valid/invalid
□ Monitor for timing attack patterns
```

**Created**: November 11, 2025

