# Open Redirect Attack

## 📋 Attack Information
**Category**: Frontend Attack | **Severity**: 🟡 Low (CVSS 4.0)

## 🎯 Attack Principle
Attacker uses legitimate website's redirect functionality to send users to phishing sites.

## 💀 Attack Example

```javascript
// ❌ Vulnerable redirect
https://wallet.com/redirect?url=https://evil.com

// Server:
app.get('/redirect', (req, res) => {
  const url = req.query.url;
  res.redirect(url); // No validation!
});

// Phishing email:
"Click to login: https://wallet.com/redirect?url=https://wallet-phishing.com"

// Users trust wallet.com domain
// → Click link
// → Redirected to phishing site
```

## 🛡️ Defense

### 1. Whitelist URLs
```typescript
// ✅ Only allow specific URLs
const ALLOWED_REDIRECTS = [
  'https://rustwallet.com',
  'https://docs.rustwallet.com',
];

app.get('/redirect', (req, res) => {
  const url = req.query.url as string;
  
  if (!ALLOWED_REDIRECTS.some(allowed => url.startsWith(allowed))) {
    return res.status(400).json({ error: 'Invalid redirect URL' });
  }
  
  res.redirect(url);
});
```

### 2. Confirmation Page
```typescript
// ✅ Show confirmation before redirect
<RedirectConfirmation url={redirectUrl}>
  <Alert>
    You are being redirected to:
    <strong>{redirectUrl}</strong>
    
    <Button onClick={confirmRedirect}>Continue</Button>
    <Button onClick={cancel}>Cancel</Button>
  </Alert>
</RedirectConfirmation>
```

**Created**: November 11, 2025

