# Malicious Browser Extensions Attack

## 📋 Attack Information
**Category**: Browser/Desktop Attack | **Severity**: 🔴 Critical (CVSS 9.5)

## 🎯 Attack Principle
Users install fake wallet extensions or malicious extensions that steal private keys and hijack transactions.

## 💀 Real Cases

### Fake MetaMask Extensions
```
Chrome Store had multiple fake MetaMask:
- "MetaMask Pro"
- "MetaMask Plus"  
- "MetaMask Wallet"

Downloads: 10,000+
Result: Private keys uploaded to attacker servers
```

### Malicious Extension Techniques
```javascript
// 1. Monitor all web pages
chrome.webRequest.onBeforeRequest.addListener(
  function(details) {
    // Log all API requests
    if (details.url.includes('api.wallet.com')) {
      stealData(details);
    }
  },
  {urls: ["<all_urls>"]}, // ALL websites
  ["requestBody"]
);

// 2. Inject scripts into wallet pages
chrome.tabs.executeScript(tabId, {
  code: `
    // Hijack window.ethereum
    const originalRequest = window.ethereum.request;
    window.ethereum.request = function(args) {
      // Secretly modify transaction address
      if (args.method === 'eth_sendTransaction') {
        args.params[0].to = 'ATTACKER_ADDRESS';
      }
      return originalRequest.call(this, args);
    };
  `
});

// 3. Read localStorage
chrome.tabs.executeScript(tabId, {
  code: "localStorage.getItem('privateKey')"
}, (result) => {
  sendToAttacker(result);
});
```

## 🛡️ Defense Measures

### 1. Verify Extension Authenticity
```
✅ Verification Checklist:
□ Developer name correct
□ Ratings and review count reasonable (beware all 5-star new extensions)
□ Update history (check if frequently updated)
□ Official website link matches
□ Download count (millions vs thousands)
□ Icon and description professional
```

### 2. Official Sources Only
```
✅ Correct Process:
1. Visit wallet official website (e.g., metamask.io)
2. Click official extension link
3. Verify URL (chrome.google.com/webstore/...)
4. Verify developer: "metamask.io"

❌ Dangerous:
- Search engine results (may be ads)
- Third-party websites
- Email/social media links
```

### 3. Permission Review
```javascript
// manifest.json permission check

// ✅ Normal wallet permissions
{
  "permissions": [
    "storage",        // Store data
    "activeTab",      // Current tab only
    "https://api.etherscan.io/*" // Specific API
  ]
}

// ❌ Suspicious permissions
{
  "permissions": [
    "<all_urls>",     // All websites
    "tabs",           // All tabs
    "webRequest",     // Intercept all requests
    "webRequestBlocking",
    "cookies",        // Read all cookies
    "history",        // Browsing history
    "downloads"       // Download files
  ]
}
```

### 4. Extension Detection
```typescript
/**
 * Detect if running in unofficial extension
 */
const OFFICIAL_EXTENSION_ID = 'nkbihfbeogaeaoehlefnkodbefgpgknn'; // MetaMask

function verifyExtension(): boolean {
  if (typeof chrome !== 'undefined' && chrome.runtime) {
    // Check extension ID
    if (chrome.runtime.id !== OFFICIAL_EXTENSION_ID) {
      console.error('Running in unofficial extension!');
      return false;
    }
  }
  return true;
}

// On app start
if (!verifyExtension()) {
  alert('⚠️ Unofficial extension detected, please uninstall!');
}
```

### 5. Whitelist Trusted Extensions
```typescript
/**
 * Only allow known safe extensions
 */
const TRUSTED_EXTENSIONS = [
  'nkbihfbeogaeaoehlefnkodbefgpgknn', // MetaMask
  'bfnaelmomeimhlpmgjnjophhpkkoljpa', // Phantom
  'hnfanknocfeofbddgcijnmhnfnkdnaad', // Coinbase Wallet
];

chrome.management.getAll((extensions) => {
  const suspicious = extensions.filter(ext => 
    ext.enabled &&
    ext.permissions.includes('webRequest') &&
    !TRUSTED_EXTENSIONS.includes(ext.id)
  );
  
  if (suspicious.length > 0) {
    alert(`
      ⚠️ Suspicious Extensions Detected:
      ${suspicious.map(e => e.name).join(', ')}
      
      Recommendation: Disable or uninstall
    `);
  }
});
```

## ✅ Identifying Fake Extensions

### Red Flags
```
❌ Danger Signs:
1. Similar but not exact name
   - "MetaMask" vs "MetaMask Wallet Pro"
   - "Phantom" vs "Phantom Wallet Plus"

2. Wrong developer
   - Correct: "metamask.io"
   - Wrong: "metamask-wallet.com"

3. Abnormal download count
   - Official: Millions
   - Fake: Thousands

4. Suspicious reviews
   - All 5-stars with similar content
   - New extension with many reviews

5. Excessive permissions
   - Access all websites
   - Read browsing history

6. Update timing
   - Never updated
   - Or suddenly frequent updates
```

## ✅ Safety Checklist
```
Before Installing:
□ Install via official website link
□ Verify developer name
□ Check download count (>1M)
□ Read reviews (watch for fake reviews)
□ Review permission list
□ Search for security reports

After Installing:
□ Test basic functionality
□ Monitor network requests
□ Regularly review extension list
□ Watch for unusual behavior

During Use:
□ Only enter sensitive info on official sites
□ Disable other extensions for important operations
□ Don't click suspicious links in extensions
□ Update to latest version regularly
```

**Created**: November 11, 2025

