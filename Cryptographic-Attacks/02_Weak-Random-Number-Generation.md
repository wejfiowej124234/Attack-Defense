# Weak Random Number Generation Attack

## 📋 Attack Information
**Category**: Cryptography | **Severity**: 🔴 High (CVSS 8.5)

## 🎯 Attack Principle
Weak random number generators produce predictable values, allowing attackers to predict private keys or session tokens.

## 💀 Weak RNG Examples

### Math.random() (Predictable)
```javascript
// ❌ NEVER use for security
function generatePrivateKey() {
  let key = '';
  for (let i = 0; i < 64; i++) {
    key += Math.floor(Math.random() * 16).toString(16);
  }
  return key; // PREDICTABLE!
}

// Math.random() uses weak PRNG
// Attacker can predict sequence
```

### Timestamp-Based (Weak)
```javascript
// ❌ Predictable seed
const seed = Date.now();
const random = seededRandom(seed);

// Attacker knows approximate timestamp
// Can brute-force seed
```

### Insufficient Entropy
```javascript
// ❌ Too few random bytes
const sessionId = crypto.randomBytes(4).toString('hex'); // Only 32 bits
// Brute-forceable
```

## 🛡️ Defense Measures

### 1. Cryptographically Secure RNG
```typescript
// ✅ Use crypto.randomBytes (Node.js)
import crypto from 'crypto';

function generateSecureToken(): string {
  // 32 bytes = 256 bits of entropy
  return crypto.randomBytes(32).toString('hex');
}

// ✅ Web Crypto API (Browser)
const array = new Uint8Array(32);
crypto.getRandomValues(array);
const token = Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
```

### 2. Sufficient Entropy
```typescript
/**
 * Generate private key with proper entropy
 */
import * as bip39 from 'bip39';

// ✅ BIP39 uses cryptographically secure random
const mnemonic = bip39.generateMnemonic(256); // 24 words = 256 bits

// Derive private key
const seed = await bip39.mnemonicToSeed(mnemonic);
const hdwallet = hdkey.fromMasterSeed(seed);
const privateKey = hdwallet.derivePath("m/44'/60'/0'/0/0").getWallet().getPrivateKeyString();
```

### 3. React Native Secure Random
```typescript
// ✅ React Native Crypto
import { NativeModules } from 'react-native';
import CryptoJS from 'crypto-js';

async function generateSecureRandom(bytes: number): Promise<string> {
  // Use native secure random
  const random = await NativeModules.RNRandomBytes.randomBytes(bytes);
  return random;
}

// CryptoJS also provides secure random
const random = CryptoJS.lib.WordArray.random(32); // 32 bytes
```

### 4. Rust Backend (High Quality RNG)
```rust
// ✅ Rust rand crate with OsRng
use rand::rngs::OsRng;
use rand::RngCore;

fn generate_secure_token() -> String {
    let mut bytes = [0u8; 32];
    OsRng.fill_bytes(&mut bytes);
    
    hex::encode(bytes)
}

// OsRng uses OS's cryptographically secure RNG
// - /dev/urandom on Linux
// - CryptGenRandom on Windows
// - SecRandomCopyBytes on macOS/iOS
```

### 5. Test Randomness Quality
```typescript
/**
 * Basic randomness test
 */
function testRandomness(generator: () => number, samples: number): void {
  const results = new Array(10).fill(0);
  
  for (let i = 0; i < samples; i++) {
    const value = generator();
    const bucket = Math.floor(value * 10);
    results[bucket]++;
  }
  
  // Check distribution (should be roughly equal)
  const expected = samples / 10;
  const variance = results.reduce((sum, count) => 
    sum + Math.pow(count - expected, 2), 0
  ) / 10;
  
  console.log('Randomness variance:', variance);
  // Low variance = good distribution
}
```

## ✅ Sources of Entropy

```
Secure (✅):
- crypto.randomBytes (Node.js)
- crypto.getRandomValues (Browser)
- /dev/urandom (Linux)
- CryptGenRandom (Windows)
- SecRandomCopyBytes (iOS/macOS)
- OsRng (Rust)

Insecure (❌):
- Math.random()
- Date.now()
- process.pid
- Predictable seeds
- Custom PRNGs without crypto expertise
```

## ✅ Best Practices

```
Key Generation:
□ Use crypto.randomBytes or equivalent
□ Minimum 256 bits for private keys
□ Minimum 128 bits for session tokens
□ Never use Math.random()
□ Never use timestamps as seeds

Tokens:
□ Session IDs: 32+ bytes
□ CSRF tokens: 32+ bytes
□ API keys: 32+ bytes
□ Nonces: 16+ bytes

Validation:
□ Test randomness distribution
□ Use established libraries (BIP39, ethers.js)
□ Regular security audits
□ Follow standards (NIST, FIPS)
```

**Created**: November 11, 2025

