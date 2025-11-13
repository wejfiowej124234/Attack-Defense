# Information Disclosure Attack

## 📋 Attack Information
**Category**: Information Leakage | **Severity**: 🟠 Medium (CVSS 5.0)

## 🎯 Attack Principle
Sensitive information leaked through error messages, debug outputs, or improper data handling.

## 💀 Information Leakage Sources

### 1. Detailed Error Messages
```javascript
// ❌ Dangerous
catch (error) {
  res.status(500).json({
    error: error.message, // May contain sensitive info
    stack: error.stack,   // Reveals code structure
    query: req.body,      // Contains user data
  });
}

// ✅ Safe
catch (error) {
  logger.error(error); // Log internally
  res.status(500).json({
    error: 'An error occurred', // Generic message
  });
}
```

### 2. Console Logs
```javascript
// ❌ Dangerous
console.log('Login attempt:', { email, password, token });
console.log('Private key:', privateKey);

// Visible in browser DevTools
// Recorded in logs
```

### 3. Source Maps in Production
```javascript
// ❌ Dangerous
// webpack.config.js
devtool: 'source-map' // In production!

// Attackers can read original source code
// See business logic, API keys, etc.
```

### 4. API Response Over-Sharing
```javascript
// ❌ Returns too much data
{
  "user": {
    "id": 123,
    "email": "user@example.com",
    "password_hash": "$2b$10$...",  // Shouldn't return
    "api_key": "sk_test_...",        // Shouldn't return
    "role": "admin"                   // Reveals privilege
  }
}
```

## 🛡️ Defense Measures

### 1. Error Message Whitelist
```typescript
/**
 * Sanitize error messages
 */
const SAFE_ERROR_MESSAGES = [
  'Invalid credentials',
  'Network error',
  'Transaction failed',
  'Insufficient funds',
];

export function sanitizeError(error: Error): string {
  const message = error.message.toLowerCase();
  
  // Check whitelist
  for (const safe of SAFE_ERROR_MESSAGES) {
    if (message.includes(safe.toLowerCase())) {
      return safe;
    }
  }
  
  // Default generic message
  return 'An error occurred. Please try again.';
}
```

### 2. Remove Console Logs
```javascript
// webpack.config.js
if (process.env.NODE_ENV === 'production') {
  config.optimization = {
    ...config.optimization,
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // ✅ Remove console.log
            drop_debugger: true,
          },
        },
      }),
    ],
  };
}
```

### 3. Disable Source Maps
```javascript
// ✅ Production config
module.exports = {
  mode: 'production',
  devtool: false, // No source maps in production
};
```

### 4. Selective API Responses
```typescript
/**
 * Only return necessary fields
 */
interface UserPublic {
  id: string;
  username: string;
  avatar: string;
}

function toPublicUser(user: UserFull): UserPublic {
  return {
    id: user.id,
    username: user.username,
    avatar: user.avatar,
    // Excludes: email, password_hash, api_key, etc.
  };
}

res.json({ user: toPublicUser(user) });
```

## ✅ Best Practices

```
□ Use generic error messages
□ Remove console.logs in production
□ Disable source maps in production
□ Return only necessary data
□ Sanitize all outputs
□ Use logging service for internal logs
□ Regular security audits
```

**Created**: November 11, 2025

