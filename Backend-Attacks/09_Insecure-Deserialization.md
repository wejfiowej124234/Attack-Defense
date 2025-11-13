# Insecure Deserialization Attack

## 📋 Attack Information
**Category**: Backend Attack | **Severity**: 🔴 High (CVSS 8.5) | **OWASP**: A08:2021

## 🎯 Attack Principle
Attacker manipulates serialized objects to execute arbitrary code or bypass authentication.

## 💀 Attack Example

```javascript
// ❌ Dangerous: Deserializing user data
const userData = deserialize(req.cookies.user);

// Attacker crafts malicious serialized object:
// Contains code that executes on deserialization
// → Remote code execution
```

## 🛡️ Defense

### 1. Use JSON (Not Native Serialization)
```typescript
// ✅ JSON is safe (no code execution)
const data = JSON.parse(jsonString);

// ❌ Dangerous: Native serialization
const data = eval(serialized); // Never do this!
```

### 2. Validate Deserialized Data
```typescript
// ✅ Validate structure and types
interface UserData {
  id: number;
  email: string;
}

function validateUserData(data: any): data is UserData {
  return (
    typeof data.id === 'number' &&
    typeof data.email === 'string' &&
    !data.hasOwnProperty('__proto__')
  );
}

const data = JSON.parse(input);
if (!validateUserData(data)) {
  throw new Error('Invalid data');
}
```

### 3. Never Deserialize Untrusted Data
```typescript
// ✅ Don't accept serialized objects from clients
// Use simple JSON with validation
```

## ✅ Best Practices

```
□ Use JSON instead of native serialization
□ Never deserialize untrusted data
□ Validate all deserialized data
□ Use type checking
□ Avoid eval, Function constructor
```

**Created**: November 11, 2025

