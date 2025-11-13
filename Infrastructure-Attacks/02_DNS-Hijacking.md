# DNS Hijacking Attack

## 📋 Attack Information
**Category**: Network Layer Attack | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker manipulates DNS resolution to redirect legitimate domain names to malicious IP addresses.

## 💀 Attack Scenarios

### Scenario 1: DNS Cache Poisoning
```
User visits: wallet.com

Normal DNS: wallet.com → 1.2.3.4 (official server)

Hijacked:
wallet.com → 5.6.7.8 (attacker's server)
→ Phishing website (looks identical)
→ User enters seed phrase
→ Stolen
```

### Scenario 2: Router DNS Change
```
Attacker compromises home router
→ Changes DNS settings
→ All DNS queries go through attacker
→ wallet.com resolves to phishing site
→ User doesn't notice (HTTPS cert invalid but ignored)
```

### Scenario 3: ISP-Level Hijacking
```
Government or attacker compromises ISP DNS
→ Affects all ISP customers
→ Large-scale phishing campaign
```

## 🛡️ Defense Measures

### 1. DNSSEC
```
Enable DNSSEC to verify DNS response authenticity
Prevents DNS spoofing attacks
```

### 2. HTTPS + HSTS
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

→ Browser forces HTTPS
→ Certificate mismatch triggers warning
→ User alerted to potential attack
```

### 3. Certificate Pinning
```typescript
/**
 * Pin expected certificate
 */
const EXPECTED_CERT_FINGERPRINT = 'sha256/AAAA...';

// React Native
import { fetch } from 'react-native-ssl-pinning';

await fetch('https://api.rustwallet.com', {
  method: 'GET',
  sslPinning: {
    certs: ['rustwallet'], // Certificate in assets
  },
});

// If certificate doesn't match → connection refused
```

### 4. IP Address Verification
```typescript
/**
 * Compare with known official IPs
 */
const OFFICIAL_IPS = ['1.2.3.4', '5.6.7.8'];

const resolvedIP = await dns.resolve('api.rustwallet.com');

if (!OFFICIAL_IPS.includes(resolvedIP)) {
  Alert.alert(
    '⚠️ DNS Resolution Anomaly',
    `
    Expected IP: ${OFFICIAL_IPS.join(' or ')}
    Resolved IP: ${resolvedIP}
    
    Your DNS may have been hijacked!
    
    Actions:
    1. Check router DNS settings
    2. Use trusted DNS (8.8.8.8, 1.1.1.1)
    3. Contact ISP if issue persists
    `
  );
}
```

### 5. Trusted DNS Servers
```typescript
/**
 * Configure trusted DNS
 */
// Use Google DNS, Cloudflare DNS
const TRUSTED_DNS = [
  '8.8.8.8',     // Google
  '8.8.4.4',     // Google
  '1.1.1.1',     // Cloudflare
  '1.0.0.1',     // Cloudflare
];

// Instructions for users
<DNSSettings>
  <h4>Recommended DNS Settings</h4>
  
  <Setting>
    <Label>Primary DNS:</Label>
    <Code>8.8.8.8</Code> (Google)
  </Setting>
  
  <Setting>
    <Label>Secondary DNS:</Label>
    <Code>1.1.1.1</Code> (Cloudflare)
  </Setting>
  
  <Button onPress={openDNSGuide}>
    How to Change DNS Settings
  </Button>
</DNSSettings>
```

### 6. DoH (DNS over HTTPS)
```typescript
/**
 * Use encrypted DNS
 */
// Cloudflare DoH
const DOH_ENDPOINT = 'https://cloudflare-dns.com/dns-query';

async function secureDNSLookup(domain: string): Promise<string> {
  const response = await fetch(`${DOH_ENDPOINT}?name=${domain}&type=A`, {
    headers: { 'Accept': 'application/dns-json' },
  });
  
  const data = await response.json();
  return data.Answer[0].data; // Encrypted, can't be intercepted
}
```

### 7. Certificate Transparency Monitoring
```typescript
/**
 * Monitor for fraudulent certificates
 */
// Check Certificate Transparency logs
// Alert if someone issues cert for your domain

async function monitorCertificates(domain: string): Promise<void> {
  const certs = await fetch(`https://crt.sh/?q=${domain}&output=json`);
  const data = await certs.json();
  
  // Check for unexpected certificates
  const suspicious = data.filter(cert => 
    !EXPECTED_ISSUERS.includes(cert.issuer_name)
  );
  
  if (suspicious.length > 0) {
    alert('⚠️ Unexpected SSL certificate issued for your domain!');
  }
}
```

## ✅ Best Practices

```
For Users:
□ Use trusted DNS (Google 8.8.8.8, Cloudflare 1.1.1.1)
□ Enable DNSSEC if possible
□ Always check HTTPS certificate
□ Bookmark important sites
□ Don't ignore certificate warnings
□ Use VPN with trusted DNS

For Developers:
□ Implement HSTS with preload
□ Use certificate pinning
□ Monitor Certificate Transparency logs
□ Use DoH/DoT
□ Display certificate info to users
□ Have alternate access methods

For Admins:
□ Configure DNSSEC
□ Use HSTS preload list
□ Monitor DNS resolution
□ Have DDoS protection
□ Regular security audits
```

**Created**: November 11, 2025

