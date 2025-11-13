# Password Brute Force Attack

## 📋 Attack Information
**Category**: Authentication Attack | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Attacker tries many password combinations to guess correct credentials.

## 💀 Attack Methods

### Dictionary Attack
```python
# Common passwords list
passwords = ['123456', 'password', 'admin', '12345678', ...]

for password in passwords:
    response = requests.post('https://wallet.com/api/login', {
        'email': 'target@example.com',
        'password': password
    })
    if response.status == 200:
        print(f'Found: {password}')
        break
```

### Credential Stuffing
```
Using leaked credentials from other breaches:
- Have I Been Pwned database
- Dark web credential lists
- Previous data breaches

Try same email+password on wallet site
```

## 🛡️ Defense Measures

### 1. Rate Limiting
```typescript
// ✅ Limit login attempts
import rateLimit from 'express-rate-limit';

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  skipSuccessfulRequests: true,
  message: 'Too many login attempts, try again later',
});

app.post('/api/login', loginLimiter, async (req, res) => {
  // Login logic
});
```

### 2. Account Lockout
```typescript
// ✅ Lock account after failed attempts
const MAX_ATTEMPTS = 5;
const LOCKOUT_DURATION = 30 * 60 * 1000; // 30 minutes

interface LoginAttempt {
  count: number;
  lockedUntil?: number;
}

const attempts = new Map<string, LoginAttempt>();

app.post('/api/login', async (req, res) => {
  const { email, password } = req.body;
  
  // Check if locked
  const attempt = attempts.get(email) || { count: 0 };
  
  if (attempt.lockedUntil && Date.now() < attempt.lockedUntil) {
    return res.status(429).json({
      error: 'Account temporarily locked',
      retryAfter: attempt.lockedUntil - Date.now(),
    });
  }
  
  const user = await authenticate(email, password);
  
  if (!user) {
    attempt.count++;
    
    if (attempt.count >= MAX_ATTEMPTS) {
      attempt.lockedUntil = Date.now() + LOCKOUT_DURATION;
      
      // Send email notification
      await sendEmail(email, 'Account locked due to multiple failed login attempts');
    }
    
    attempts.set(email, attempt);
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Success - reset counter
  attempts.delete(email);
  res.json({ token: generateToken(user) });
});
```

### 3. Strong Password Policy
```typescript
/**
 * Enforce strong passwords
 */
function validatePassword(password: string): boolean {
  // OWASP recommendations
  if (password.length < 12) {
    throw new Error('Password must be at least 12 characters');
  }
  
  // Check complexity
  const hasUpperCase = /[A-Z]/.test(password);
  const hasLowerCase = /[a-z]/.test(password);
  const hasNumber = /[0-9]/.test(password);
  const hasSpecial = /[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(password);
  
  const complexityCount = [hasUpperCase, hasLowerCase, hasNumber, hasSpecial]
    .filter(Boolean).length;
  
  if (complexityCount < 3) {
    throw new Error('Password must contain uppercase, lowercase, numbers, and special characters');
  }
  
  // Check against common passwords
  if (COMMON_PASSWORDS.includes(password.toLowerCase())) {
    throw new Error('Password is too common');
  }
  
  return true;
}
```

### 4. Multi-Factor Authentication
```typescript
// ✅ Require 2FA
app.post('/api/login', async (req, res) => {
  const user = await authenticate(req.body.email, req.body.password);
  
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Require 2FA code
  if (user.totpEnabled) {
    const totpValid = await verifyTOTP(user.id, req.body.totpCode);
    
    if (!totpValid) {
      return res.status(401).json({ error: '2FA verification failed' });
    }
  }
  
  res.json({ token: generateToken(user) });
});
```

### 5. CAPTCHA
```typescript
// ✅ Require CAPTCHA after failures
app.post('/api/login', async (req, res) => {
  const failedCount = await getFailedAttempts(req.ip);
  
  if (failedCount >= 3) {
    // Verify CAPTCHA
    const captchaValid = await verifyCaptcha(req.body.captchaToken);
    
    if (!captchaValid) {
      return res.status(400).json({ error: 'Invalid CAPTCHA' });
    }
  }
  
  // Proceed with login
});
```

## ✅ Best Practices

```
□ Implement rate limiting (5 attempts per 15 minutes)
□ Lock accounts after failures
□ Enforce strong password policy
□ Require multi-factor authentication
□ Use CAPTCHA after failed attempts
□ Monitor for brute-force patterns
□ Send email alerts on failed attempts
□ Use exponential backoff
```

**Created**: November 11, 2025

