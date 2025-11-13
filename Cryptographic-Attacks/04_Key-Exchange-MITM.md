# Key Exchange MITM Attack

## 📋 Attack Information
**Category**: Cryptographic Protocol | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker intercepts key exchange, establishing separate encrypted channels with both parties.

## 💀 Attack Scenario

```
Normal:
Alice ←→ Bob (shared secret)

MITM:
Alice ←→ Attacker ←→ Bob

Alice thinks she's talking to Bob
Bob thinks he's talking to Alice
Attacker decrypts, reads, re-encrypts all messages
```

### Diffie-Hellman MITM
```
1. Alice sends: g^a to Bob
2. Attacker intercepts, sends: g^x to both
3. Alice computes: (g^x)^a = g^xa
4. Bob computes: (g^x)^b = g^xb
5. Attacker has both shared secrets
6. Can decrypt all traffic
```

## 🛡️ Defense Measures

### 1. Certificate-Based Authentication
```typescript
// ✅ Use TLS with certificate verification
import https from 'https';

const agent = new https.Agent({
  // Reject unauthorized certificates
  rejectUnauthorized: true,
  
  // Pin certificate
  ca: [fs.readFileSync('ca-cert.pem')],
});

https.get('https://api.rustwallet.com', { agent }, (res) => {
  // Connection authenticated
});
```

### 2. Public Key Infrastructure (PKI)
```typescript
// ✅ Verify server's public key
import { X509Certificate } from 'crypto';

function verifyCertificate(certPEM: string): boolean {
  const cert = new X509Certificate(certPEM);
  
  // Check issuer
  const trustedIssuers = ['DigiCert', 'Let\'s Encrypt'];
  if (!trustedIssuers.some(issuer => cert.issuer.includes(issuer))) {
    return false;
  }
  
  // Check expiry
  if (new Date() > cert.validTo) {
    return false;
  }
  
  // Check subject
  if (!cert.subject.includes('rustwallet.com')) {
    return false;
  }
  
  return true;
}
```

### 3. Certificate Pinning
```typescript
// ✅ Pin expected certificate
const EXPECTED_CERT_FINGERPRINT = 'sha256/AAAA...';

import { fetch } from 'react-native-ssl-pinning';

await fetch('https://api.rustwallet.com', {
  method: 'GET',
  sslPinning: {
    certs: ['rustwallet'],
  },
});

// Rejects if certificate doesn't match
// MITM attacker cannot present fake certificate
```

### 4. Authenticated Key Exchange
```typescript
// ✅ ECDH with authentication
import { generateKeyPair, createECDH } from 'crypto';

// Alice has certified public key
const alice = createECDH('secp256k1');
const alicePublic = alice.generateKeys();

// Bob verifies Alice's certificate before accepting public key
const bobSharedSecret = bob.computeSecret(alicePublicFromCert);

// MITM cannot impersonate without Alice's private key
```

## ✅ Best Practices

```
□ Always use TLS/SSL
□ Verify certificates
□ Implement certificate pinning
□ Use authenticated key exchange
□ Never ignore certificate warnings
□ Use strong cipher suites
□ Implement HSTS
```

**Created**: November 11, 2025

