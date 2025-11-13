# Weak Cryptographic Algorithms Attack

## 📋 Attack Information
**Category**: Cryptography | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Using outdated or weak cryptographic algorithms that can be broken by modern computing power.

## 💀 Weak Algorithms

### MD5 (Broken)
```javascript
// ❌ NEVER use MD5
import md5 from 'md5';
const hash = md5(password); // Collision attacks possible

// Attack: Rainbow tables, collision attacks
// MD5 collisions can be generated in seconds
```

### SHA-1 (Deprecated)
```javascript
// ❌ SHA-1 is deprecated
const hash = crypto.createHash('sha1').update(data).digest('hex');

// Google demonstrated collision in 2017
// Not suitable for security
```

### DES/3DES (Weak)
```javascript
// ❌ DES is broken, 3DES is deprecated
const cipher = crypto.createCipheriv('des-ede3', key, iv);

// Key length too short for modern standards
```

### ECB Mode (Insecure)
```javascript
// ❌ ECB mode reveals patterns
const cipher = crypto.createCipheriv('aes-256-ecb', key, null);

// Same plaintext → same ciphertext
// Patterns visible in encrypted data
```

## 🛡️ Secure Alternatives

### 1. Use AES-256-GCM
```typescript
// ✅ AES-256-GCM (authenticated encryption)
import crypto from 'crypto';

function encrypt(plaintext: string, key: Buffer): EncryptedData {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  
  let encrypted = cipher.update(plaintext, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag();
  
  return {
    encrypted,
    iv: iv.toString('hex'),
    authTag: authTag.toString('hex'),
  };
}

function decrypt(data: EncryptedData, key: Buffer): string {
  const decipher = crypto.createDecipheriv(
    'aes-256-gcm',
    key,
    Buffer.from(data.iv, 'hex')
  );
  
  decipher.setAuthTag(Buffer.from(data.authTag, 'hex'));
  
  let decrypted = decipher.update(data.encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
```

### 2. Use SHA-256/SHA-512
```typescript
// ✅ SHA-256 or SHA-512
import crypto from 'crypto';

const hash = crypto.createHash('sha256').update(data).digest('hex');
// Or SHA-512 for extra security
const hash512 = crypto.createHash('sha512').update(data).digest('hex');
```

### 3. Use Argon2 for Passwords
```typescript
// ✅ Argon2id (winner of Password Hashing Competition)
import argon2 from 'argon2';

// Hash password
const hash = await argon2.hash(password, {
  type: argon2.argon2id,
  memoryCost: 65536, // 64 MB
  timeCost: 3,
  parallelism: 4,
});

// Verify
const valid = await argon2.verify(hash, password);
```

### 4. React Native Secure Encryption
```typescript
/**
 * Mobile-optimized encryption
 */
import { NativeModules } from 'react-native';
import CryptoJS from 'crypto-js';

async function secureEncrypt(plaintext: string, password: string): Promise<string> {
  // Derive key using PBKDF2
  const salt = CryptoJS.lib.WordArray.random(128 / 8);
  const key = CryptoJS.PBKDF2(password, salt, {
    keySize: 256 / 32,
    iterations: 10000,
  });
  
  // Encrypt with AES-256
  const encrypted = CryptoJS.AES.encrypt(plaintext, key, {
    mode: CryptoJS.mode.GCM,
  });
  
  return JSON.stringify({
    encrypted: encrypted.toString(),
    salt: salt.toString(),
  });
}
```

## ✅ Recommended Algorithms

```
Hashing:
✅ SHA-256, SHA-512, SHA-3
✅ BLAKE2b, BLAKE3
✅ Argon2 (passwords)
❌ MD5, SHA-1

Encryption:
✅ AES-256-GCM
✅ ChaCha20-Poly1305
✅ XSalsa20-Poly1305
❌ DES, 3DES, RC4, ECB mode

Key Exchange:
✅ ECDH (Curve25519)
✅ X25519
❌ DH with small key sizes

Signatures:
✅ EdDSA (Ed25519)
✅ ECDSA (secp256k1 for Ethereum)
✅ RSA-4096+ with PSS padding
❌ RSA-1024, DSA
```

## ✅ Best Practices

```
□ Use AES-256-GCM for encryption
□ Use SHA-256+ for hashing
□ Use Argon2 for password hashing
□ Never use MD5, SHA-1, DES
□ Use authenticated encryption (GCM, Poly1305)
□ Keep crypto libraries updated
□ Follow NIST/OWASP recommendations
```

**Created**: November 11, 2025

