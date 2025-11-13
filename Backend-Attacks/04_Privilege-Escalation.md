# Privilege Escalation Attack

## 📋 Attack Information
**Category**: Authorization Attack | **Severity**: 🔴 High (CVSS 8.0) | **OWASP**: A01:2021

## 🎯 Attack Principle
Regular user gains admin/elevated privileges through vulnerabilities in role management.

## 💀 Attack Scenarios

### Scenario 1: Parameter Manipulation
```javascript
// ❌ Vulnerable: Trust client-provided role
app.post('/api/register', async (req, res) => {
  const user = await User.create({
    email: req.body.email,
    password: req.body.password,
    role: req.body.role, // ❌ Client controls role!
  });
});

// Attack: Register with role='admin'
{
  "email": "attacker@evil.com",
  "password": "password",
  "role": "admin" // ← Attacker sets this
}
```

### Scenario 2: Mass Assignment
```javascript
// ❌ Vulnerable
app.put('/api/user/profile', authenticate, async (req, res) => {
  await User.update(req.user.id, req.body); // ❌ Updates ALL fields
});

// Attack payload:
{
  "name": "Hacker",
  "role": "admin", // ← Not intended to be updatable
  "isVerified": true
}
```

### Scenario 3: Missing Authorization Check
```javascript
// ❌ Vulnerable: Checks authentication but not authorization
app.delete('/api/user/:id', authenticate, async (req, res) => {
  // User authenticated, but is it admin?
  await User.delete(req.params.id); // ❌ No role check
});

// Regular user can delete anyone
```

## 🛡️ Defense Measures

### 1. Server-Side Role Assignment
```typescript
// ✅ SECURE: Server assigns role
app.post('/api/register', async (req, res) => {
  const user = await User.create({
    email: req.body.email,
    password: hashedPassword,
    role: 'user', // ✅ Server-assigned, not from client
  });
  
  res.json({ user: toPublicUser(user) });
});
```

### 2. Whitelist Updatable Fields
```typescript
// ✅ Only allow specific fields
const ALLOWED_UPDATE_FIELDS = ['name', 'avatar', 'bio'];

app.put('/api/user/profile', authenticate, async (req, res) => {
  // Filter to allowed fields only
  const updates: any = {};
  for (const field of ALLOWED_UPDATE_FIELDS) {
    if (req.body[field] !== undefined) {
      updates[field] = req.body[field];
    }
  }
  
  await User.update(req.user.id, updates);
  res.json({ success: true });
});
```

### 3. Role-Based Access Control (RBAC)
```typescript
// ✅ Explicit role checking
const requireRole = (allowedRoles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }
    
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    
    next();
  };
};

// Usage
app.delete('/api/user/:id',
  authenticate,
  requireRole(['admin', 'superadmin']), // ✅ Explicit role check
  async (req, res) => {
    await User.delete(req.params.id);
    res.json({ success: true });
  }
);
```

### 4. Immutable Roles in JWT
```typescript
// ✅ Role in JWT payload (cannot be changed without secret)
const token = jwt.sign(
  {
    userId: user.id,
    role: user.role, // Signed with secret
  },
  SECRET,
  { expiresIn: '1h' }
);

// Middleware extracts role from token
function authenticate(req, res, next) {
  const decoded = jwt.verify(token, SECRET);
  req.user = {
    id: decoded.userId,
    role: decoded.role, // From token, not database
  };
  next();
}
```

### 5. Principle of Least Privilege
```typescript
// ✅ Define specific permissions
enum Permission {
  READ_OWN_WALLET = 'read:own:wallet',
  WRITE_OWN_WALLET = 'write:own:wallet',
  READ_ALL_WALLETS = 'read:all:wallets',
  DELETE_USER = 'delete:user',
}

const ROLE_PERMISSIONS = {
  user: [
    Permission.READ_OWN_WALLET,
    Permission.WRITE_OWN_WALLET,
  ],
  admin: [
    Permission.READ_OWN_WALLET,
    Permission.WRITE_OWN_WALLET,
    Permission.READ_ALL_WALLETS,
    Permission.DELETE_USER,
  ],
};

function hasPermission(user: User, permission: Permission): boolean {
  return ROLE_PERMISSIONS[user.role]?.includes(permission) || false;
}

// Check before action
if (!hasPermission(req.user, Permission.DELETE_USER)) {
  return res.status(403).json({ error: 'Insufficient permissions' });
}
```

## ✅ Best Practices

```
Role Management:
□ Server assigns roles, never trust client
□ Roles immutable (or require admin to change)
□ Use JWT to encode roles
□ Validate role on every request

Authorization:
□ Explicit permission checks on all routes
□ Implement RBAC or ABAC
□ Whitelist updatable fields
□ Principle of least privilege

Testing:
□ Test as different roles
□ Try parameter manipulation
□ Attempt mass assignment
□ Regular penetration testing
```

**Created**: November 11, 2025

