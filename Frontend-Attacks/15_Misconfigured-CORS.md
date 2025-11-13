# Misconfigured CORS Attack

## 📋 Attack Information
**Category**: Configuration Error | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Misconfigured CORS (Cross-Origin Resource Sharing) allows malicious sites to access sensitive data from your API.

## 💀 Dangerous Configuration

```javascript
// ❌ DANGEROUS
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*'); // Allow all origins!
  res.header('Access-Control-Allow-Credentials', 'true'); // With cookies!
  next();
});

// Attack from evil.com:
fetch('https://wallet.com/api/user/info', {
  credentials: 'include', // Sends user's cookies
})
.then(r => r.json())
.then(data => {
  // Steal user info
  sendToAttacker(data);
});
```

## 🛡️ Defense

### 1. Strict Origin Whitelist
```typescript
// ✅ SECURE
const ALLOWED_ORIGINS = [
  'https://rustwallet.com',
  'https://www.rustwallet.com',
  'https://app.rustwallet.com',
];

app.use((req, res, next) => {
  const origin = req.headers.origin;
  
  if (origin && ALLOWED_ORIGINS.includes(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
    res.header('Access-Control-Allow-Credentials', 'true');
  }
  
  next();
});
```

### 2. Dynamic Origin Validation
```typescript
// ✅ Validate origin pattern
function isAllowedOrigin(origin: string): boolean {
  try {
    const url = new URL(origin);
    
    // Must be HTTPS
    if (url.protocol !== 'https:') return false;
    
    // Must match domain
    const allowedDomains = ['rustwallet.com'];
    return allowedDomains.some(domain => 
      url.hostname === domain || 
      url.hostname.endsWith('.' + domain)
    );
  } catch {
    return false;
  }
}

app.use((req, res, next) => {
  const origin = req.headers.origin;
  
  if (origin && isAllowedOrigin(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
  }
  
  next();
});
```

### 3. No Credentials with Wildcard
```typescript
// ❌ NEVER do this
res.header('Access-Control-Allow-Origin', '*');
res.header('Access-Control-Allow-Credentials', 'true');
// This combination is blocked by browsers anyway

// ✅ If must use wildcard (public API), no credentials
res.header('Access-Control-Allow-Origin', '*');
// Don't set Allow-Credentials
```

## ✅ Best Practices

```
□ Use strict origin whitelist
□ Validate HTTPS protocol
□ No wildcard with credentials
□ Validate origin format
□ Test CORS configuration
□ Use CORS middleware (express-cors)
```

**Created**: November 11, 2025

