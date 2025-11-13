# DOM-Based Attacks

## 📋 Attack Information
**Category**: Frontend Attack | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Exploit client-side JavaScript that unsafely processes user-controlled data from URL, localStorage, or other sources.

## 💀 Attack Examples

### DOM XSS
```javascript
// ❌ Vulnerable
const name = new URLSearchParams(location.search).get('name');
document.getElementById('welcome').innerHTML = `Welcome ${name}`;

// Attack URL:
?name=<img src=x onerror=alert(document.cookie)>
```

### DOM-based Open Redirect
```javascript
// ❌ Vulnerable
const redirectUrl = location.hash.substring(1);
location.href = redirectUrl;

// Attack: #https://evil.com
```

## 🛡️ Defense

```typescript
// ✅ Use textContent instead of innerHTML
element.textContent = userInput;

// ✅ Sanitize
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);

// ✅ Validate URLs
function isSafeUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    return ['http:', 'https:'].includes(parsed.protocol);
  } catch {
    return false;
  }
}
```

**Created**: November 11, 2025

