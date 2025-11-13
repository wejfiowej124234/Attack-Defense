# API Key Leakage

## 📋 Attack Information
**Category**: Key Management | **Severity**: 🔴 Critical (CVSS 9.0)

## 🎯 Leakage Vectors

### 1. GitHub Exposure
```javascript
// ❌ Hardcoded in code
const INFURA_KEY = 'abc123def456...';
const ALCHEMY_KEY = 'xyz789...';

// Committed to GitHub
// → Auto-scanning tools find it
// → Attacker abuses API quota
```

### 2. Frontend Bundle Exposure
```javascript
// ❌ Visible in React bundle
const config = {
  infuraKey: 'YOUR_INFURA_KEY', // Plain text in bundle.js
  alchemyKey: 'YOUR_ALCHEMY_KEY',
};

// User opens DevTools →
// Searches bundle.js for 'infura' →
// Finds key
```

### 3. Mobile App Decompilation
```
APK decompilation →
strings.xml / config.json →
API keys found
```

## 🛡️ Defense Measures

### 1. Backend Proxy (Recommended)
```typescript
// ❌ Frontend direct call
const provider = new ethers.providers.InfuraProvider(
  'mainnet',
  'YOUR_INFURA_KEY' // Exposed!
);

// ✅ Backend proxy
// Frontend
const response = await fetch('https://api.yourwallet.com/rpc', {
  method: 'POST',
  body: JSON.stringify({
    jsonrpc: '2.0',
    method: 'eth_blockNumber',
    params: [],
  }),
});

// Backend
app.post('/rpc', async (req, res) => {
  const infuraKey = process.env.INFURA_KEY; // Not exposed
  
  const response = await fetch(
    `https://mainnet.infura.io/v3/${infuraKey}`,
    {
      method: 'POST',
      body: JSON.stringify(req.body),
    }
  );
  
  res.json(await response.json());
});
```

### 2. Domain Restrictions
```
Infura/Alchemy Dashboard Settings:
✅ Restrict to domains:
  - https://rustwallet.com
  - https://www.rustwallet.com

✅ Restrict to IPs:
  - Server IP whitelist

Even if key leaks, attacker can't use it
```

### 3. Rate Limiting
```typescript
// Backend rate limiting
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 minute
  max: 100, // 100 requests
  message: 'Too many requests',
});

app.use('/rpc', limiter);
```

### 4. Key Rotation
```typescript
/**
 * Periodic key rotation
 */
const API_KEYS = {
  primary: process.env.INFURA_KEY_PRIMARY,
  secondary: process.env.INFURA_KEY_SECONDARY,
  fallback: process.env.INFURA_KEY_FALLBACK,
};

let currentKeyIndex = 0;

async function makeRequest(method: string, params: any[]) {
  const keys = Object.values(API_KEYS);
  
  try {
    return await callInfura(keys[currentKeyIndex], method, params);
  } catch (error) {
    if (error.code === 429) { // Rate limit
      // Switch to next key
      currentKeyIndex = (currentKeyIndex + 1) % keys.length;
      return await callInfura(keys[currentKeyIndex], method, params);
    }
    throw error;
  }
}
```

### 5. Usage Monitoring
```typescript
/**
 * Monitor API key usage
 */
setInterval(async () => {
  const usage = await infura.getUsage();
  
  if (usage.requestsToday > EXPECTED_MAX) {
    // Abnormal usage
    await alert.send({
      title: '⚠️ API Key Abnormal Usage',
      message: `Requests today: ${usage.requestsToday} (expected <${EXPECTED_MAX})`,
      action: 'Rotate key immediately',
    });
    
    // Auto-rotate
    await rotateApiKey();
  }
}, 3600000); // Hourly check
```

## ✅ Best Practices
```
□ Never hardcode API keys in frontend
□ Use backend proxy for RPC calls
□ Set domain/IP restrictions
□ Implement rate limiting
□ Rotate keys periodically
□ Monitor usage for anomalies
□ Use .env files (never commit)
□ Add .env to .gitignore
```

**Created**: November 11, 2025

