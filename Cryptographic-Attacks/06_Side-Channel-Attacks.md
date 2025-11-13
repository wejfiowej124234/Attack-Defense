# Side-Channel Attacks

## 📋 Attack Information
**Category**: Physical Cryptanalysis | **Severity**: 🟠 Medium (CVSS 5.0)

## 🎯 Attack Principle
Extract secrets by analyzing physical characteristics of device operation (power, timing, EM radiation).

## 💀 Attack Types

### 1. Power Analysis
```
Simple Power Analysis (SPA):
- Monitor power consumption during crypto operations
- Different operations have different power signatures
- Can reveal key bits

Differential Power Analysis (DPA):
- Statistical analysis of many operations
- Correlate power consumption with key guesses
- Recovers full key
```

### 2. Electromagnetic Analysis
```
Monitor EM radiation from device
- CPU emits EM during operations
- Different instructions emit different patterns
- Can reconstruct executed code and keys
```

### 3. Cache Timing
```javascript
// AES implementation detail leaks through cache
// Attacker measures cache access times
// Infers key bits from timing patterns
```

### 4. Acoustic Cryptanalysis
```
Listen to device sounds:
- Keyboard typing patterns
- CPU coil whine
- Hard drive seek patterns

Can extract:
- Typed passwords
- Cryptographic keys
```

## 🛡️ Defense Measures

### 1. Use Hardware Security Modules
```typescript
// ✅ Hardware-isolated crypto operations
// Android Keystore
import { NativeModules } from 'react-native';

async function signWithSecureElement(data: string): Promise<string> {
  // Key never leaves secure element
  // Operations performed in hardware
  // Immune to software-based side-channel attacks
  const signature = await NativeModules.KeystoreModule.sign(data);
  return signature;
}
```

### 2. Constant-Time Algorithms
```rust
// ✅ Rust: Use constant-time crypto libraries
use subtle::ConstantTimeEq;

fn verify_mac(expected: &[u8], provided: &[u8]) -> bool {
    expected.ct_eq(provided).into()
    // Comparison takes same time regardless of match position
}
```

### 3. Blinding Techniques
```
Add random noise to operations:
- Random delays
- Dummy operations
- Randomized operation order

Makes timing analysis harder
```

### 4. Physical Security
```
For High-Value Operations:
✅ Use hardware wallets (Ledger, Trezor)
   - Physically isolated
   - Hardened against side-channel
   - Specialized secure chips

✅ Secure environment:
   - Faraday cage (blocks EM)
   - White noise (masks acoustic)
   - Physical access control
```

## ✅ Best Practices

```
For Developers:
□ Use hardware security modules
□ Use constant-time crypto libraries
□ Avoid branching on secret data
□ Use blinding techniques
□ Keep crypto libraries updated

For High-Security Users:
□ Use hardware wallets
□ Secure physical environment
□ Limit physical access to devices
□ Use HSMs for critical keys

Architecture:
□ Isolate crypto operations
□ Use dedicated hardware
□ Minimize software crypto
□ Regular security audits
```

**Created**: November 11, 2025

