# SSRF (Server-Side Request Forgery) Attack

## 📋 Attack Information
**Category**: Backend Attack | **Severity**: 🟠 Medium (CVSS 6.5) | **OWASP**: A10:2021

## 🎯 Attack Principle
Attacker tricks server into making requests to unintended destinations, accessing internal resources.

## 💀 Attack Scenarios

### 1. Internal Network Access
```javascript
// ❌ Vulnerable: Server fetches user-provided URL
app.get('/api/fetch-price', async (req, res) => {
  const url = req.query.url;
  const response = await fetch(url); // No validation!
  res.json(await response.json());
});

// Attack:
GET /api/fetch-price?url=http://192.168.1.1/admin

// Server accesses internal admin panel
// Returns data to attacker
```

### 2. Cloud Metadata Access
```bash
# AWS metadata endpoint
GET /api/fetch-price?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Returns AWS credentials
# Attacker gains cloud access
```

### 3. Port Scanning
```javascript
// Attacker uses server to scan internal network
for (let port = 1; port < 65535; port++) {
  fetch(`/api/fetch-price?url=http://internal-server:${port}`);
  // Analyze response times to find open ports
}
```

## 🛡️ Defense Measures

### 1. URL Whitelist
```typescript
// ✅ Only allow specific URLs
const ALLOWED_DOMAINS = [
  'api.coingecko.com',
  'api.binance.com',
  'api.coinbase.com',
];

function validateUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    
    // Check protocol
    if (!['http:', 'https:'].includes(parsed.protocol)) {
      return false;
    }
    
    // Check domain
    if (!ALLOWED_DOMAINS.includes(parsed.hostname)) {
      return false;
    }
    
    return true;
  } catch {
    return false;
  }
}

app.get('/api/fetch-price', async (req, res) => {
  const url = req.query.url as string;
  
  if (!validateUrl(url)) {
    return res.status(400).json({ error: 'Invalid URL' });
  }
  
  const response = await fetch(url);
  res.json(await response.json());
});
```

### 2. Block Internal IPs
```typescript
// ✅ Block private IP ranges
import { isPrivateIP } from 'private-ip';
import dns from 'dns/promises';

async function isSafeUrl(url: string): Promise<boolean> {
  const parsed = new URL(url);
  
  // Resolve hostname to IP
  const addresses = await dns.resolve4(parsed.hostname);
  
  // Block private IPs
  for (const ip of addresses) {
    if (isPrivateIP(ip)) {
      return false; // Block 192.168.x.x, 10.x.x.x, etc.
    }
    
    // Block localhost
    if (ip === '127.0.0.1' || ip.startsWith('127.')) {
      return false;
    }
    
    // Block AWS metadata
    if (ip === '169.254.169.254') {
      return false;
    }
  }
  
  return true;
}
```

### 3. Use Proxy Service
```typescript
// ✅ Dedicated proxy with restrictions
import axios from 'axios';

const proxyClient = axios.create({
  proxy: {
    host: 'secure-proxy.internal',
    port: 8080,
  },
  timeout: 5000,
  maxRedirects: 0, // Don't follow redirects
});

// Proxy enforces whitelist
```

## ✅ Best Practices

```
□ Whitelist allowed URLs/domains
□ Block private IP ranges
□ Resolve DNS before checking IP
□ Disable redirects or limit to same domain
□ Set connection timeout
□ Use dedicated proxy with restrictions
□ Monitor outbound requests
```

**Created**: November 11, 2025

