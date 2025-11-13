# Authentication Bypass Attack

## 📋 Attack Information
**Category**: Authentication | **Severity**: 🔴 High (CVSS 8.5) | **OWASP**: A07:2021

## 🎯 Attack Principle
Attacker circumvents authentication mechanisms to gain unauthorized access.

## 💀 Common Bypasses

### 1. Missing Authentication Check
```typescript
// ❌ Vulnerable: No authentication check
app.get('/api/wallet/:id', async (req, res) => {
  const wallet = await Wallet.findById(req.params.id);
  res.json(wallet); // Anyone can access any wallet!
});

// ✅ Secure
app.get('/api/wallet/:id', authenticate, async (req, res) => {
  const wallet = await Wallet.findById(req.params.id);
  
  // Verify ownership
  if (wallet.userId !== req.user.id) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  res.json(wallet);
});
```

### 2. Weak JWT Secret
```javascript
// ❌ Weak secret
const secret = 'secret123';

// Attacker can brute-force or guess
// Generate valid tokens for any user
```

### 3. Algorithm Confusion
```javascript
// JWT header
{
  "alg": "none", // No signature!
}

// If server doesn't validate algorithm:
// Attacker creates unsigned token
// Server accepts it
```

### 4. SQL Injection Auth Bypass
```sql
-- Input: email: admin' OR '1'='1'--
SELECT * FROM users WHERE email = 'admin' OR '1'='1'--' AND password = ''
-- Always returns admin user
```

## 🛡️ Defense Measures

### 1. Strong JWT Implementation
```typescript
// ✅ Secure JWT
import jwt from 'jsonwebtoken';

const SECRET = process.env.JWT_SECRET; // Strong random secret

// Generate token
const token = jwt.sign(
  { 
    userId: user.id,
    role: user.role,
  },
  SECRET,
  { 
    expiresIn: '1h',
    algorithm: 'HS256', // Explicitly set
  }
);

// Verify token
function authenticate(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, SECRET, {
      algorithms: ['HS256'], // ✅ Whitelist algorithms
    });
    
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}
```

### 2. Mandatory Authentication Middleware
```typescript
// ✅ Apply authentication globally
app.use('/api/*', authenticate);

// Explicitly allow public routes
const PUBLIC_ROUTES = ['/api/health', '/api/login', '/api/register'];

app.use((req, res, next) => {
  if (PUBLIC_ROUTES.includes(req.path)) {
    return next();
  }
  
  // All other routes require authentication
  authenticate(req, res, next);
});
```

### 3. Multi-Factor Authentication
```typescript
// ✅ Require 2FA for sensitive operations
app.post('/api/transfer', authenticate, async (req, res) => {
  // Check 2FA
  const totpValid = await verifyTOTP(req.user.id, req.body.totpCode);
  
  if (!totpValid) {
    return res.status(403).json({ error: '2FA verification failed' });
  }
  
  // Execute transfer
  await executeTransfer(req.body);
  res.json({ success: true });
});
```

### 4. Rust Backend Security
```rust
// ✅ Rust type system prevents many auth bypasses
#[derive(Debug, Deserialize)]
struct AuthenticatedUser {
    id: i32,
    role: String,
}

#[get("/wallet/{wallet_id}")]
async fn get_wallet(
    wallet_id: web::Path<String>,
    user: web::ReqData<AuthenticatedUser>, // Extracted from JWT
    db: web::Data<Database>,
) -> Result<HttpResponse, Error> {
    let wallet = db.get_wallet(&wallet_id).await?;
    
    // Type system ensures user is authenticated
    if wallet.user_id != user.id {
        return Err(Error::Forbidden);
    }
    
    Ok(HttpResponse::Ok().json(wallet))
}
```

## ✅ Best Practices

```
Authentication:
□ Use strong JWT secrets (32+ random bytes)
□ Whitelist JWT algorithms
□ Set short token expiration
□ Implement token refresh mechanism
□ Never accept 'none' algorithm

Authorization:
□ Check authentication on all protected routes
□ Verify resource ownership
□ Implement role-based access control
□ Require 2FA for sensitive operations

Security:
□ Use parameterized queries
□ Validate all inputs
□ Use type-safe languages (Rust, TypeScript)
□ Regular security audits
```

**Created**: November 11, 2025

