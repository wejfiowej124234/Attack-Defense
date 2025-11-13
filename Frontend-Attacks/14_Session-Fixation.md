# Session Fixation Attack

## 📋 Attack Information
**Category**: Session Management | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Attacker sets victim's session ID to known value, then after victim logs in, attacker uses same session ID to impersonate victim.

## 💀 Attack Flow

```
1. Attacker visits website, gets session ID
   Cookie: sessionId=ATTACKER123

2. Attacker sends this session ID to victim
   https://wallet.com?sessionId=ATTACKER123
   Or via email link

3. Victim clicks link, uses session ID ATTACKER123

4. Victim logs in (session ID unchanged!)

5. Attacker uses same session ID
   Cookie: sessionId=ATTACKER123
   → Already logged in! Impersonates victim
```

## 🛡️ Defense Measures

### 1. Regenerate Session on Login
```typescript
// ✅ Generate new session after login
app.post('/api/login', async (req, res) => {
  const user = await authenticate(req.body);
  
  if (user) {
    // ✅ Destroy old session
    req.session.destroy();
    
    // ✅ Create new session
    req.session.regenerate((err) => {
      if (err) {
        return res.status(500).json({ error: 'Login failed' });
      }
      
      req.session.userId = user.id;
      req.session.createdAt = Date.now();
      
      res.json({ success: true });
    });
  }
});
```

### 2. Don't Accept Session ID from URL
```typescript
// ❌ Dangerous: Accept session from URL
app.use((req, res, next) => {
  if (req.query.sessionId) {
    req.sessionId = req.query.sessionId; // Dangerous!
  }
  next();
});

// ✅ Safe: Only from cookie
app.use(session({
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
  },
  genid: () => crypto.randomUUID(),
}));
```

### 3. Bind Session to Device
```typescript
// ✅ Session tied to IP and User-Agent
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

## ✅ React Native Apps

```typescript
// ✅ Mobile apps use JWT tokens, not sessions
// Naturally protected from session fixation

const { token } = await login(credentials);

// Store securely
await SecureStore.setToken('auth_token', token, 3600);

// Each login generates new token
```

**Created**: November 11, 2025

