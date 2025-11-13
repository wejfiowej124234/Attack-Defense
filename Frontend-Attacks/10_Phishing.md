# Phishing Attack

## 📋 Attack Information
**Category**: Social Engineering + Technical | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker creates fake websites/emails mimicking legitimate services to steal credentials or private keys.

## 💀 Common Phishing Techniques

### 1. Domain Spoofing
```
Legitimate: metamask.io
Phishing:   metamask.com (wrong TLD)
           metarnask.io (rn looks like m)
           metamask-wallet.com (added words)
           metamask.io.evil.com (subdomain)
```

### 2. Look-Alike Interfaces
```
Fake site copies:
✓ Same logo
✓ Same colors
✓ Same layout
✓ Same wording

Users cannot tell difference
→ Enter seed phrase
→ Stolen
```

### 3. Email Phishing
```
From: support@metamask.io (spoofed)
Subject: [URGENT] Verify Your Wallet

"Your wallet will be locked in 24 hours.
Click here to verify: https://metamask-verify.com"

→ Fake website
→ Asks for seed phrase
→ Stolen
```

### 4. Unicode Homograph
```
Real: metamask.io
Fake: metаmask.io (а is Cyrillic, looks identical)

Browser shows: metamask.io
Actual domain: xn--metmask-4ue.io
```

## 🛡️ Defense Measures

### 1. Domain Verification
```typescript
/**
 * Verify domain authenticity
 */
const OFFICIAL_DOMAINS = [
  'rustwallet.com',
  'www.rustwallet.com',
  'app.rustwallet.com',
];

function verifyDomain(): boolean {
  const current = window.location.hostname;
  
  if (!OFFICIAL_DOMAINS.includes(current)) {
    // Show warning
    document.body.innerHTML = `
      <div style="text-align:center; padding:50px;">
        <h1 style="color:red;">⚠️ PHISHING SITE DETECTED</h1>
        <p>This is NOT the official Rust Wallet website!</p>
        <p>Official site: <a href="https://rustwallet.com">rustwallet.com</a></p>
      </div>
    `;
    return false;
  }
  
  return true;
}

// Run on page load
verifyDomain();
```

### 2. SSL Certificate Verification
```typescript
/**
 * Check SSL certificate
 */
function checkSSL(): void {
  if (window.location.protocol !== 'https:') {
    Alert.alert(
      '🚨 INSECURE CONNECTION',
      'This site is not using HTTPS! Do not enter sensitive information!'
    );
  }
  
  // Show SSL info to user
  <SecurityIndicator>
    <LockIcon /> Secure Connection
    <Button onClick={showCertificate}>
      View Certificate
    </Button>
  </SecurityIndicator>
}
```

### 3. Phishing Detection Database
```typescript
/**
 * Check against known phishing sites
 */
async function checkPhishingDatabase(domain: string): Promise<boolean> {
  // Check PhishTank, Google Safe Browsing
  const response = await fetch(
    `https://safebrowsing.googleapis.com/v4/threatMatches:find?key=${API_KEY}`,
    {
      method: 'POST',
      body: JSON.stringify({
        client: { clientId: 'rustwallet', clientVersion: '1.0' },
        threatInfo: {
          threatTypes: ['SOCIAL_ENGINEERING'],
          platformTypes: ['ANY_PLATFORM'],
          threatEntryTypes: ['URL'],
          threatEntries: [{ url: domain }],
        },
      }),
    }
  );
  
  const data = await response.json();
  return data.matches && data.matches.length > 0;
}
```

### 4. Bookmark Recommendation
```typescript
<Alert severity="info">
  <h4>🔖 Bookmark This Site</h4>
  <p>
    To avoid phishing, bookmark the official site and always 
    access via bookmark, not search engines or links.
  </p>
  
  <Button onClick={addBookmark}>
    Add Bookmark
  </Button>
</Alert>
```

### 5. User Education
```typescript
<PhishingWarning>
  <h3>🎣 How to Identify Phishing</h3>
  
  <h4>❌ Red Flags:</h4>
  <ul>
    <li>Urgent language ("Act now or lose funds!")</li>
    <li>Requests for private key/seed phrase</li>
    <li>Misspelled URL</li>
    <li>No HTTPS or certificate warnings</li>
    <li>Unexpected emails</li>
  </ul>
  
  <h4>✅ Always Verify:</h4>
  <ul>
    <li>Check exact URL spelling</li>
    <li>Look for HTTPS lock icon</li>
    <li>Verify certificate</li>
    <li>Use bookmarks</li>
    <li>Contact official support if unsure</li>
  </ul>
</PhishingWarning>
```

## ✅ Best Practices

```
For Users:
□ Bookmark official sites
□ Always check URL carefully
□ Verify HTTPS certificate
□ Never ignore certificate warnings
□ Don't click links in emails
□ Official support never asks for seed phrase

For Developers:
□ Implement domain verification
□ Use HSTS
□ Display security indicators
□ Educate users in-app
□ Report phishing sites
□ Monitor for fake domains
```

**Created**: November 11, 2025

