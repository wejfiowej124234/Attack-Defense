# Clickjacking Attack

## 📋 Attack Information
**Category**: Frontend Attack | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Attacker tricks users into clicking hidden buttons/links by overlaying transparent iframe over legitimate interface.

## 💀 Attack Example

```html
<!-- Attacker's site evil.com -->
<style>
  iframe {
    opacity: 0.0001; /* Nearly invisible */
    position: absolute;
    width: 500px;
    height: 500px;
    z-index: 2;
  }
  
  .fake-button {
    position: absolute;
    top: 300px;
    left: 200px;
    z-index: 1;
  }
</style>

<!-- Invisible iframe of wallet.com -->
<iframe src="https://wallet.com/transfer"></iframe>

<!-- Fake button underneath -->
<button class="fake-button">
  Click to Win $1000!
</button>

<!-- 
User thinks they click "Win $1000"
Actually clicks invisible "Confirm Transfer" button
→ Funds transferred to attacker
-->
```

## 🛡️ Defense Measures

### 1. X-Frame-Options Header
```typescript
// ✅ Prevent framing
app.use((req, res, next) => {
  res.setHeader('X-Frame-Options', 'DENY'); // Cannot be framed at all
  // Or: 'SAMEORIGIN' - only same origin can frame
  next();
});
```

### 2. CSP frame-ancestors
```html
<!-- ✅ Modern alternative to X-Frame-Options -->
<meta http-equiv="Content-Security-Policy" 
      content="frame-ancestors 'none'">

<!-- Or allow specific origins -->
<meta http-equiv="Content-Security-Policy" 
      content="frame-ancestors 'self' https://trusted.com">
```

### 3. Frame Busting (Client-Side)
```typescript
// ✅ Detect if page is framed
if (window.top !== window.self) {
  // Page is in iframe
  
  // Option 1: Break out
  window.top.location = window.self.location;
  
  // Option 2: Show warning
  document.body.innerHTML = `
    <div style="padding:50px;text-align:center;">
      <h1>⚠️ Security Warning</h1>
      <p>This page should not be displayed in a frame.</p>
      <p>Please visit directly: <a href="${window.location.href}">${window.location.href}</a></p>
    </div>
  `;
}
```

### 4. User Interaction Confirmation
```typescript
/**
 * Require explicit confirmation for sensitive actions
 */
async function confirmSensitiveAction(): Promise<boolean> {
  // Show modal that requires user interaction
  const confirmed = await showModal({
    title: 'Confirm Action',
    message: 'Please confirm you want to proceed',
    requirePassword: true,
    preventClickjacking: true, // Cannot be overlaid
  });
  
  return confirmed;
}
```

## ✅ Best Practices

```
Headers:
□ Set X-Frame-Options: DENY
□ Set CSP frame-ancestors 'none'
□ Use both for maximum compatibility

Frontend:
□ Implement frame-busting code
□ Check if page is framed
□ Require user confirmation for sensitive actions
□ Use click delay for critical buttons

Testing:
□ Try framing your site in iframe
□ Verify headers are set correctly
□ Test frame-busting works
```

**Created**: November 11, 2025

