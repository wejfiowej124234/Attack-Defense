# Prototype Pollution Attack

## 📋 Attack Information
**Category**: JavaScript Attack | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Attacker modifies JavaScript object prototypes to inject malicious properties affecting all objects.

## 💀 Attack Example

```javascript
// ❌ Vulnerable: Merge function
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === 'object') {
      target[key] = merge(target[key] || {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// Attack payload
const malicious = JSON.parse('{"__proto__": {"isAdmin": true}}');
merge({}, malicious);

// Now ALL objects have isAdmin = true!
const user = {};
console.log(user.isAdmin); // true (inherited from prototype)
```

## 💀 Wallet Context

```javascript
// If wallet checks admin status:
if (user.isAdmin) {
  // Allow privileged operations
}

// After prototype pollution:
// ANY user appears as admin!
```

## 🛡️ Defense Measures

### 1. Object.create(null)
```javascript
// ✅ Create object without prototype
const safeObject = Object.create(null);
safeObject.__proto__ = 'anything'; // Just a regular property
```

### 2. Validate Keys
```javascript
// ✅ Filter dangerous keys
function safeMerge(target, source) {
  const DANGEROUS_KEYS = ['__proto__', 'constructor', 'prototype'];
  
  for (let key in source) {
    // Skip dangerous keys
    if (DANGEROUS_KEYS.includes(key)) {
      continue;
    }
    
    if (source.hasOwnProperty(key)) {
      target[key] = source[key];
    }
  }
  
  return target;
}
```

### 3. Freeze Prototypes
```javascript
// ✅ Prevent prototype modification
Object.freeze(Object.prototype);
Object.freeze(Array.prototype);
```

### 4. Sanitize JSON
```typescript
/**
 * Sanitize parsed JSON
 */
export function sanitizeObject(obj: any): any {
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  const DANGEROUS_KEYS = ['__proto__', 'constructor', 'prototype'];
  
  if (Array.isArray(obj)) {
    return obj.map(sanitizeObject);
  }
  
  const sanitized: any = {};
  for (const key in obj) {
    if (DANGEROUS_KEYS.includes(key)) {
      continue; // Skip dangerous keys
    }
    
    if (obj.hasOwnProperty(key)) {
      sanitized[key] = sanitizeObject(obj[key]);
    }
  }
  
  return sanitized;
}

// Usage
const data = JSON.parse(response);
const safe = sanitizeObject(data);
```

## ✅ Best Practices

```
□ Validate all object keys
□ Use Object.create(null) for dictionaries
□ Filter __proto__, constructor, prototype
□ Use Map instead of objects for user data
□ Sanitize JSON before use
□ Use TypeScript for type safety
```

**Created**: November 11, 2025

