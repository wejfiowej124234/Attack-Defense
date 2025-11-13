# CSRF (Cross-Site Request Forgery) Attack

## 📋 Attack Information
**Category**: Frontend Attack | **Severity**: 🟠 Medium (CVSS 6.0) | **OWASP**: A01:2021

## 🎯 Attack Principle
Attacker tricks authenticated user's browser into executing unwanted actions on trusted website.

## 💀 Attack Scenarios

### Scenario 1: Unauthorized Transfer
```html
<!-- Attacker's website evil.com -->
<img src="https://wallet.com/api/transfer?to=ATTACKER&amount=1000" />

<!-- 
When victim (logged into wallet.com) visits evil.com:
- Browser automatically sends cookies for wallet.com
- Transfer executes without user knowledge
-->
```

### Scenario 2: Account Settings Change
```html
<!-- Hidden form auto-submits -->
<form action="https://wallet.com/api/change-email" method="POST">
  <input type="hidden" name="email" value="attacker@evil.com" />
</form>
<script>document.forms[0].submit();</script>

<!-- User's email changed to attacker's -->
```

### Scenario 3: Token Approval
```html
<!-- Malicious site -->
<script>
  // User is logged into wallet
  fetch('https://wallet.com/api/approve-token', {
    method: 'POST',
    credentials: 'include', // Sends cookies
    body: JSON.stringify({
      token: 'USDT',
      spender: 'ATTACKER_CONTRACT',
      amount: 'unlimited',
    }),
  });
</script>
```

## 🛡️ Defense Measures

### 1. CSRF Tokens
```typescript
// ✅ Generate unique token per session
import csrf from 'csurf';

const csrfProtection = csrf({ cookie: true });

app.use(csrfProtection);

// Send token to frontend
app.get('/api/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Validate on requests
app.post('/api/transfer', csrfProtection, async (req, res) => {
  // CSRF token validated automatically
  await executeTransfer(req.body);
});

// Frontend includes token
const csrfToken = await getCSRFToken();

await fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': csrfToken,
  },
  body: JSON.stringify(transferData),
});
```

### 2. SameSite Cookies
```typescript
// ✅ Set SameSite attribute
res.cookie('auth_token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict', // Cookie not sent in cross-site requests
});

// Options:
// - 'strict': Never sent cross-site (most secure)
// - 'lax': Sent on top-level navigation (GET)
// - 'none': Always sent (requires Secure)
```

### 3. Origin/Referer Validation
```typescript
// ✅ Verify request origin
app.use((req, res, next) => {
  const origin = req.headers.origin;
  const referer = req.headers.referer;
  
  const ALLOWED_ORIGINS = [
    'https://rustwallet.com',
    'https://www.rustwallet.com',
  ];
  
  if (req.method !== 'GET') {
    if (!origin || !ALLOWED_ORIGINS.includes(origin)) {
      return res.status(403).json({ error: 'Invalid origin' });
    }
  }
  
  next();
});
```

### 4. Custom Headers
```typescript
// ✅ Require custom header (cannot be set cross-site by forms)
app.use((req, res, next) => {
  if (req.method !== 'GET') {
    const customHeader = req.headers['x-requested-with'];
    
    if (customHeader !== 'XMLHttpRequest') {
      return res.status(403).json({ error: 'Missing custom header' });
    }
  }
  
  next();
});

// Frontend sets header
axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
```

### 5. Double Submit Cookie
```typescript
// ✅ CSRF token in both cookie and request
// Backend
res.cookie('csrf_token', token, { sameSite: 'strict' });
res.json({ csrfToken: token });

// Frontend
axios.post('/api/transfer', data, {
  headers: {
    'X-CSRF-Token': csrfToken, // Must match cookie
  },
});

// Backend verifies they match
if (req.cookies.csrf_token !== req.headers['x-csrf-token']) {
  return res.status(403).json({ error: 'CSRF validation failed' });
}
```

### 6. User Interaction Required
```typescript
/**
 * Require user confirmation for sensitive actions
 */
async function transfer(to: string, amount: number) {
  // ✅ Show confirmation dialog
  const confirmed = await showDialog({
    title: 'Confirm Transfer',
    message: `Send ${amount} ETH to ${to}?`,
    requirePassword: true, // Re-enter password
  });
  
  if (!confirmed) return;
  
  // Execute transfer
  await executeTransfer(to, amount);
}
```

## ✅ React Native Wallet Protection
```typescript
/**
 * Mobile apps naturally protected from CSRF
 */
// No cookies, uses tokens
const token = await SecureStore.getToken('auth_token');

axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;

// CSRF attacks cannot steal token from different origin
```

## ✅ Best Practices

```
Backend:
□ Use CSRF tokens for state-changing requests
□ Set SameSite=Strict on cookies
□ Validate Origin/Referer headers
□ Require custom headers
□ Use POST for state-changing operations

Frontend:
□ Include CSRF token in requests
□ Use axios/fetch (not forms for sensitive ops)
□ Implement user confirmation for critical actions
□ Display request origin to users

Mobile:
□ Use token-based auth (not cookies)
□ Implement biometric confirmation
□ Show transaction details before signing
```

**Created**: November 11, 2025

