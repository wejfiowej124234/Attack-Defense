# Broken Access Control (IDOR) Attack

## 📋 Attack Information
**Category**: Authorization | **Severity**: 🔴 Critical (CVSS 9.0) | **OWASP**: A01:2021

## 🎯 Attack Principle
User bypasses authorization by manipulating requests to access unauthorized resources (Insecure Direct Object Reference).

## 💀 Attack Scenarios

### Scenario 1: IDOR
```typescript
// ❌ Vulnerable: Only checks authentication, not ownership
app.get('/api/wallet/:id', authenticate, async (req, res) => {
  const wallet = await Wallet.findById(req.params.id);
  res.json(wallet); // ❌ Doesn't verify if user owns wallet
});

// Attack:
GET /api/wallet/user123  // My wallet ✓
GET /api/wallet/user456  // Someone else's wallet - also accessible!
```

### Scenario 2: Path Traversal Authorization
```typescript
// ❌ Doesn't verify parent resource ownership
app.get('/api/wallet/:walletId/transaction/:txId', async (req, res) => {
  const tx = await Transaction.findById(req.params.txId);
  res.json(tx); // ❌ Doesn't verify tx belongs to walletId
});

// Attack:
GET /api/wallet/my-wallet/transaction/someone-else-tx
→ See someone else's transaction
```

### Scenario 3: Function-Level Access Control
```typescript
// ❌ Frontend hides admin button, but backend doesn't restrict
app.delete('/api/admin/user/:id', async (req, res) => {
  // ❌ No admin check!
  await User.deleteById(req.params.id);
});

// Regular user directly calls:
DELETE /api/admin/user/victim_id
```

## 🛡️ Defense Measures

### 1. Ownership Verification
```rust
// ✅ Rust backend
#[get("/wallet/{wallet_id}")]
async fn get_wallet(
    wallet_id: web::Path<String>,
    user: AuthenticatedUser, // From JWT
    db: web::Data<Database>,
) -> Result<HttpResponse, Error> {
    let wallet = db.get_wallet(&wallet_id).await?;
    
    // ✅ Verify ownership
    if wallet.user_id != user.id {
        return Err(Error::Forbidden("Not your wallet"));
    }
    
    Ok(HttpResponse::Ok().json(wallet))
}
```

### 2. Resource-Based Queries
```typescript
// ✅ Query by userId, don't accept resource ID directly
app.get('/api/wallets/:walletId', authenticate, async (req, res) => {
  const wallet = await Wallet.findOne({
    _id: req.params.walletId,
    userId: req.user.id, // ✅ Auto-filter by user
  });
  
  if (!wallet) {
    return res.status(404).json({ error: 'Wallet not found' });
  }
  
  res.json(wallet);
});
```

### 3. Role-Based Access Control (RBAC)
```typescript
// ✅ Middleware checks role
const requireRole = (roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    
    next();
  };
};

// Usage
app.delete('/api/admin/user/:id',
  authenticate,
  requireRole(['admin', 'superadmin']), // ✅ Explicit role check
  async (req, res) => {
    await User.deleteById(req.params.id);
    res.json({ success: true });
  }
);
```

### 4. Attribute-Based Access Control (ABAC)
```typescript
// ✅ Policy-based access control
interface AccessPolicy {
  resource: string;
  action: string;
  condition: (user: User, resource: any) => boolean;
}

const policies: AccessPolicy[] = [
  {
    resource: 'wallet',
    action: 'transfer',
    condition: (user, wallet) =>
      wallet.userId === user.id &&      // Owns wallet
      wallet.status === 'active' &&     // Wallet active
      !user.isSuspended &&              // User not suspended
      user.kycVerified,                 // KYC verified
  },
];

function checkAccess(user: User, resource: any, action: string): boolean {
  const policy = policies.find(p =>
    p.resource === resource.type &&
    p.action === action
  );
  
  return policy ? policy.condition(user, resource) : false;
}
```

## ✅ Best Practices

```
Every Endpoint:
□ Authentication check (who are you?)
□ Authorization check (can you do this?)
□ Resource ownership verification
□ Never trust client-provided IDs directly

Architecture:
□ Query by userId to filter resources
□ Implement RBAC or ABAC
□ Explicit permission checks
□ Least privilege principle

Testing:
□ Try accessing other users' resources
□ Try admin operations as regular user
□ Test with various roles
□ Automated authorization testing
```

**Created**: November 11, 2025

