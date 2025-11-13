# Man-in-the-Middle (MITM) Attack

## 📋 Attack Information
**Category**: Network Attack | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker intercepts communication between user and server, reading or modifying data in transit.

## 💀 Attack Scenarios

### Scenario 1: Public WiFi MITM
```
User connects to "Free Airport WiFi"
→ Attacker controls router
→ All traffic goes through attacker
→ User visits wallet.com (HTTP)
→ Attacker injects JavaScript
→ Steals private key when entered
```

### Scenario 2: SSL Stripping
```
User types: wallet.com
→ Attacker intercepts
→ Downgrades to HTTP
→ User sees: http://wallet.com (no HTTPS)
→ Data transmitted in plain text
→ Attacker steals credentials
```

### Scenario 3: Fake Certificate
```
Attacker presents fake SSL certificate
→ Browser shows warning
→ User ignores warning (click "Proceed")
→ HTTPS established with attacker
→ Attacker decrypts all traffic
```

## 🛡️ Defense Measures

### 1. Force HTTPS
```typescript
// ✅ Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (!req.secure && process.env.NODE_ENV === 'production') {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

### 2. HSTS Header
```typescript
// ✅ Strict Transport Security
app.use((req, res, next) => {
  res.setHeader(
    'Strict-Transport-Security',
    'max-age=31536000; includeSubDomains; preload'
  );
  next();
});

// Browser enforces HTTPS for 1 year
// Prevents SSL stripping
```

### 3. Certificate Pinning (Mobile)
```typescript
// React Native
import { fetch } from 'react-native-ssl-pinning';

const PINNED_CERT = 'sha256/AAAA...';

await fetch('https://api.rustwallet.com', {
  method: 'GET',
  sslPinning: {
    certs: ['rustwallet'], // Certificate fingerprint
  },
});

// Only accepts specific certificate
// Fake certificates rejected
```

### 4. Certificate Validation
```typescript
/**
 * Check SSL certificate
 */
async function validateCertificate(url: string): Promise<boolean> {
  try {
    const response = await fetch(url);
    
    // Check if HTTPS
    if (!url.startsWith('https://')) {
      throw new Error('Not HTTPS');
    }
    
    // In production, certificate auto-validated by browser
    return true;
  } catch (error) {
    Alert.alert('⚠️ Certificate Error', 'Cannot verify server identity');
    return false;
  }
}
```

### 5. User Education
```typescript
<SecurityTips>
  <Alert severity="warning">
    <h4>⚠️ Public WiFi Warnings</h4>
    <ul>
      <li>Avoid accessing wallet on public WiFi</li>
      <li>Use VPN if necessary</li>
      <li>Never ignore certificate warnings</li>
      <li>Verify HTTPS lock icon</li>
      <li>Check URL spelling carefully</li>
    </ul>
  </Alert>
</SecurityTips>
```

## ✅ Best Practices

```
□ Always use HTTPS
□ Implement HSTS
□ Use certificate pinning (mobile)
□ Never ignore certificate warnings
□ Use VPN on public WiFi
□ Educate users about MITM risks
```

**Created**: November 11, 2025

