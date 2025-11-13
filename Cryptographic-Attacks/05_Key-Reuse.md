# Key Reuse Attack

## 📋 Attack Information
**Category**: Cryptographic Practice | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Reusing same key/nonce pairs in encryption compromises security, potentially revealing plaintext or keys.

## 💀 Attack Scenarios

### 1. Nonce Reuse in AES-GCM
```typescript
// ❌ DANGEROUS: Reusing same nonce
const key = crypto.randomBytes(32);
const nonce = crypto.randomBytes(12); // Generated once

// Multiple encryptions with SAME nonce
const cipher1 = crypto.createCipheriv('aes-256-gcm', key, nonce);
const encrypted1 = cipher1.update(plaintext1);

const cipher2 = crypto.createCipheriv('aes-256-gcm', key, nonce); // ❌ Same nonce!
const encrypted2 = cipher2.update(plaintext2);

// Attacker can XOR ciphertexts to get plaintext XOR
// If one plaintext known, can recover the other
```

### 2. Same Key for Different Purposes
```typescript
// ❌ Using same key for encryption and HMAC
const key = crypto.randomBytes(32);

const encrypted = encrypt(data, key);
const mac = hmac(encrypted, key); // Same key!

// Key reuse weakens both operations
```

### 3. Static IV
```javascript
// ❌ Hardcoded IV
const IV = Buffer.from('0000000000000000', 'hex');

// Multiple encryptions with same key+IV
// Reveals patterns, allows attacks
```

## 🛡️ Defense Measures

### 1. Generate New Nonce Every Time
```typescript
// ✅ New nonce for each encryption
function encrypt(plaintext: string, key: Buffer): EncryptedData {
  const nonce = crypto.randomBytes(12); // ✅ New random nonce
  
  const cipher = crypto.createCipheriv('aes-256-gcm', key, nonce);
  let encrypted = cipher.update(plaintext, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  return {
    encrypted,
    nonce: nonce.toString('hex'), // Include nonce
    authTag: cipher.getAuthTag().toString('hex'),
  };
}
```

### 2. Separate Keys for Different Purposes
```typescript
// ✅ Derive separate keys
import crypto from 'crypto';

function deriveKeys(masterKey: Buffer): DerivedKeys {
  // Use HKDF to derive separate keys
  const encryptionKey = crypto.hkdfSync(
    'sha256',
    masterKey,
    Buffer.from(''),
    Buffer.from('encryption'),
    32
  );
  
  const macKey = crypto.hkdfSync(
    'sha256',
    masterKey,
    Buffer.from(''),
    Buffer.from('mac'),
    32
  );
  
  return { encryptionKey, macKey };
}

// Use separate keys
const keys = deriveKeys(masterKey);
const encrypted = encrypt(data, keys.encryptionKey);
const mac = hmac(encrypted, keys.macKey);
```

### 3. Key Rotation
```typescript
/**
 * Rotate keys periodically
 */
class KeyManager {
  private currentKey: Buffer;
  private keyVersion: number;
  private lastRotation: number;
  
  async rotateIfNeeded(): Promise<void> {
    const KEY_ROTATION_PERIOD = 30 * 24 * 3600 * 1000; // 30 days
    
    if (Date.now() - this.lastRotation > KEY_ROTATION_PERIOD) {
      // Generate new key
      this.currentKey = crypto.randomBytes(32);
      this.keyVersion++;
      this.lastRotation = Date.now();
      
      // Re-encrypt existing data with new key
      await this.reEncryptData();
    }
  }
}
```

### 4. Counter-Based Nonce
```typescript
// ✅ If must use deterministic nonce, use counter
class NonceCounter {
  private counter: bigint = 0n;
  
  getNext(): Buffer {
    this.counter++;
    
    // 96-bit nonce (12 bytes)
    const nonce = Buffer.alloc(12);
    nonce.writeBigUInt64BE(this.counter, 4);
    
    return nonce;
  }
}

// Each nonce is unique (never reused)
```

## ✅ Best Practices

```
Nonces:
□ Generate new random nonce for each encryption
□ Never reuse key+nonce pair
□ Use 96-bit (12 byte) nonces for AES-GCM
□ Store nonce with ciphertext

Keys:
□ Use separate keys for different purposes
□ Derive keys from master key using HKDF
□ Rotate keys periodically
□ Never reuse keys across contexts

Validation:
□ Test nonce uniqueness in code
□ Implement key rotation
□ Monitor for key reuse
□ Follow cryptographic standards
```

**Created**: November 11, 2025

