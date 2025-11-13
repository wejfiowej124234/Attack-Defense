# NoSQL Injection Attack

## 📋 Attack Information
**Category**: Database Attack | **Severity**: 🔴 High (CVSS 8.5) | **CWE**: CWE-943

## 🎯 Attack Principle
Similar to SQL injection, but targeting NoSQL databases like MongoDB, Redis, CouchDB.

## 💀 Attack Example

### MongoDB
```javascript
// ❌ Dangerous
const user = await db.users.findOne({
  email: req.body.email,
  password: req.body.password,
});

// Attack payload:
{
  "email": {"$ne": null},
  "password": {"$ne": null}
}

// Query becomes: email != null AND password != null
// Returns first user (usually admin!)
```

### Operator Injection
```javascript
// Attack:
{
  "email": "admin@wallet.com",
  "password": {"$gt": ""}
}

// password > "" → always true
// Bypasses authentication
```

## 🛡️ Defense

### 1. Type Validation
```typescript
// ✅ Strict type checking
if (typeof email !== 'string' || typeof password !== 'string') {
  throw new Error('Invalid input type');
}

// Ensure no MongoDB operators
if (email.includes('$') || password.includes('$')) {
  throw new Error('Invalid characters');
}

const user = await User.findOne({ email, password });
```

### 2. Use ORM/ODM
```typescript
// ✅ Mongoose automatically handles this
import { User } from './models/User';

const user = await User.findOne({
  email: req.body.email,
  passwordHash: hashedPassword,
});

// Mongoose sanitizes inputs
```

### 3. Input Sanitization
```typescript
/**
 * Remove MongoDB operators
 */
function sanitizeMongoInput(obj: any): any {
  if (typeof obj !== 'object' || obj === null) {
    return obj;
  }
  
  const sanitized: any = {};
  for (const key in obj) {
    // Skip operators
    if (key.startsWith('$')) {
      continue;
    }
    
    sanitized[key] = sanitizeMongoInput(obj[key]);
  }
  
  return sanitized;
}

// Usage
const safe = sanitizeMongoInput(req.body);
const user = await db.users.findOne(safe);
```

## ✅ Best Practices

```
□ Validate input types
□ Use ORM/ODM (Mongoose, Prisma)
□ Sanitize inputs (remove $ operators)
□ Use parameterized queries
□ Principle of least privilege for DB user
```

**Created**: November 11, 2025

