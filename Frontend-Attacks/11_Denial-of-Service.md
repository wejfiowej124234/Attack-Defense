# Denial of Service (DoS) Attack

## 📋 Attack Information
**Category**: Availability Attack | **Severity**: 🟠 Medium (CVSS 5.0)

## 🎯 Attack Principle
Overwhelm frontend with excessive requests or resource-intensive operations, making application unusable.

## 💀 Attack Scenarios

### 1. API Flooding
```javascript
// Attacker sends thousands of requests
for (let i = 0; i < 100000; i++) {
  fetch('https://wallet.com/api/balance');
}

// Server overwhelmed
// Legitimate users cannot access
```

### 2. Frontend Resource Exhaustion
```javascript
// Malicious payload causes infinite loop
const data = { /* deeply nested object */ };

// Triggers expensive operation
JSON.stringify(data); // Hangs browser
```

### 3. Regex DoS (ReDoS)
```javascript
// ❌ Vulnerable regex
const emailRegex = /^([a-zA-Z0-9]+)+@/;

// Attack input:
const malicious = 'aaaaaaaaaaaaaaaaaaaaaaaaaaa!';
emailRegex.test(malicious); // Takes forever (exponential backtracking)
```

## 🛡️ Defense Measures

### 1. Rate Limiting (Frontend)
```typescript
/**
 * Client-side rate limiting
 */
class RateLimiter {
  private requests: number[] = [];
  private readonly maxRequests: number;
  private readonly windowMs: number;
  
  constructor(maxRequests: number, windowMs: number) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
  }
  
  canMakeRequest(): boolean {
    const now = Date.now();
    
    // Remove old requests outside window
    this.requests = this.requests.filter(
      time => now - time < this.windowMs
    );
    
    // Check if under limit
    if (this.requests.length >= this.maxRequests) {
      return false;
    }
    
    this.requests.push(now);
    return true;
  }
}

// Usage
const limiter = new RateLimiter(10, 60000); // 10 requests per minute

async function fetchBalance() {
  if (!limiter.canMakeRequest()) {
    Alert.alert('Too many requests, please wait');
    return;
  }
  
  await api.getBalance();
}
```

### 2. Input Size Limits
```typescript
// ✅ Limit input sizes
const MAX_INPUT_LENGTH = 1000;

function validateInput(input: string): boolean {
  if (input.length > MAX_INPUT_LENGTH) {
    throw new Error('Input too long');
  }
  
  return true;
}
```

### 3. Safe Regex
```typescript
// ❌ Vulnerable
const emailRegex = /^([a-zA-Z0-9]+)+@/;

// ✅ Safe - no nested quantifiers
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
```

### 4. Request Deduplication
```typescript
/**
 * Prevent duplicate requests
 */
let pendingRequests = new Set<string>();

async function fetchWithDedup(url: string) {
  if (pendingRequests.has(url)) {
    return; // Already requesting
  }
  
  pendingRequests.add(url);
  
  try {
    const response = await fetch(url);
    return response.json();
  } finally {
    pendingRequests.delete(url);
  }
}
```

## ✅ Best Practices

```
□ Implement client-side rate limiting
□ Limit input sizes
□ Use safe regex patterns
□ Debounce user actions
□ Deduplicate requests
□ Show loading states
□ Backend rate limiting essential
```

**Created**: November 11, 2025

