# Fake Customer Support Scam

## 📋 Attack Information
**Category**: Social Engineering | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker impersonates official support via phone, email, or social media to trick users into revealing private keys or seed phrases.

## 💀 Attack Scenarios

### Scenario 1: Fake Telegram Support
```
User asks question in Telegram group

Seconds later, receives DM:
"Hello, I'm MetaMask official support
I saw your question
Please provide your seed phrase so I can check your wallet"

→ User provides seed phrase
→ Assets stolen
```

### Scenario 2: Fake Email
```
From: support@metamask.com (spoofed)
Subject: [URGENT] Your Wallet Has Security Risk

Content:
"Dear User,
We detected abnormal activity in your wallet.
To protect your assets, please immediately:
1. Click link: https://metamask-secure[.]com
2. Enter seed phrase to verify identity
3. Account will be frozen if not verified within 24 hours"

→ Phishing website
→ Seed phrase stolen
```

### Scenario 3: Fake Discord Admin
```
Discord DM:
"@everyone URGENT NOTICE
Our smart contract was hacked
Please immediately transfer assets to safe address:
0xAttackerAddress"

→ Fake admin
→ Users transfer funds
→ Funds stolen
```

## 🛡️ Defense Measures

### 1. Official Statement
```typescript
// ✅ Display security notice in app
<SecurityNotice permanent={true}>
  <h3>🔒 Official Security Statement</h3>
  
  <Alert severity="error">
    <strong>Official support will NEVER:</strong>
    <ul>
      <li>❌ Initiate direct messages to you</li>
      <li>❌ Ask for seed phrase</li>
      <li>❌ Ask for private keys</li>
      <li>❌ Ask for passwords</li>
      <li>❌ Ask to transfer to "safe address"</li>
      <li>❌ Ask for remote access to your device</li>
    </ul>
  </Alert>
  
  <Alert severity="success">
    <strong>Official Support Channels:</strong>
    <ul>
      <li>✅ Website: https://rustwallet.com</li>
      <li>✅ Email: support@rustwallet.com</li>
      <li>✅ Twitter: @RustWallet (verified ✓)</li>
    </ul>
  </Alert>
  
  <Alert severity="info">
    <strong>If you receive suspicious contact:</strong>
    <ol>
      <li>1️⃣ Don't reply</li>
      <li>2️⃣ Don't click links</li>
      <li>3️⃣ Take screenshots</li>
      <li>4️⃣ Report via official channels</li>
    </ol>
  </Alert>
</SecurityNotice>
```

### 2. Domain Verification Helper
```typescript
/**
 * Verify contact channel authenticity
 */
const OFFICIAL_CONTACTS = {
  website: 'rustwallet.com',
  email: ['support@rustwallet.com', 'security@rustwallet.com'],
  twitter: '@RustWallet',
  telegram: '@RustWalletOfficial',
  discord: 'discord.gg/rustwallet',
};

function verifyContact(contact: string, type: 'email' | 'website' | 'telegram'): boolean {
  switch (type) {
    case 'email':
      return OFFICIAL_CONTACTS.email.some(e => contact.endsWith(e));
    case 'website':
      try {
        const url = new URL(contact);
        return url.hostname === OFFICIAL_CONTACTS.website ||
               url.hostname.endsWith('.' + OFFICIAL_CONTACTS.website);
      } catch {
        return false;
      }
    case 'telegram':
      return contact === OFFICIAL_CONTACTS.telegram;
  }
}

// Usage
if (!verifyContact(emailAddress, 'email')) {
  showWarning('This is not an official email address!');
}
```

### 3. Phishing Detection
```typescript
/**
 * Detect common phishing domains
 */
const PHISHING_PATTERNS = [
  /metamask-.*\.com/,     // metamask-secure.com
  /.*-metamask\.com/,     // secure-metamask.com
  /metam[a4]sk\.com/,     // metam4sk.com
  /.*wallet.*-support/,   // wallet-support.com
  /.*airdrop.*\.com/,     // free-airdrop.com
];

function isProbablyPhishing(url: string): boolean {
  try {
    const parsed = new URL(url);
    return PHISHING_PATTERNS.some(pattern => 
      pattern.test(parsed.hostname)
    );
  } catch {
    return true; // Invalid URL also suspicious
  }
}
```

### 4. AI Scam Detection
```typescript
/**
 * AI assistant to detect scam messages
 */
function detectScamMessage(message: string): ScamAnalysis {
  const SCAM_KEYWORDS = [
    'seed phrase', 'mnemonic',
    'private key',
    'urgent', '24 hours',
    'verify identity', 'confirm',
    'safe address',
    'freeze', 'suspend',
  ];
  
  const suspiciousCount = SCAM_KEYWORDS.filter(kw =>
    message.toLowerCase().includes(kw.toLowerCase())
  ).length;
  
  return {
    isSuspicious: suspiciousCount >= 2,
    riskLevel: suspiciousCount >= 3 ? 'HIGH' : 'MEDIUM',
    matchedKeywords: SCAM_KEYWORDS.filter(kw => 
      message.toLowerCase().includes(kw.toLowerCase())
    ),
    recommendation: suspiciousCount >= 2 
      ? 'This is very likely a scam, do not reply'
      : 'Please be cautious',
  };
}
```

### 5. In-App Support System
```typescript
// ✅ Only contact support through app
<HelpCenter>
  <Section title="Official Support">
    <Button onPress={() => openInAppSupport()}>
      Contact Support In-App (Secure)
    </Button>
    
    <Alert severity="warning">
      ⚠️ Official support will never:
      - Initiate contact with you
      - Ask for seed phrase/private key
      - Request fund transfers
    </Alert>
  </Section>
  
  <Section title="Community Help">
    <Link href="https://discord.gg/rustwallet">
      Discord Community (verify link on website)
    </Link>
  </Section>
</HelpCenter>
```

### 6. Reporting System
```typescript
/**
 * Easy scam reporting
 */
<ReportScam>
  <Form>
    <Select label="Scam Type">
      <Option>Fake Support</Option>
      <Option>Phishing Website</Option>
      <Option>Malicious Airdrop</Option>
      <Option>Fake Exchange</Option>
    </Select>
    
    <TextInput
      label="Contact/URL"
      placeholder="Scammer's email, website, etc."
    />
    
    <ImagePicker
      label="Screenshot Evidence"
      multiple={true}
    />
    
    <Button onPress={submitReport}>
      Submit Report
    </Button>
  </Form>
</ReportScam>
```

## ✅ User Self-Protection Checklist
```
✅ Remember: Real support won't initiate contact
✅ Remember: No one needs your seed phrase
✅ Remember: Official won't ask to transfer to "safe address"
✅ Verify: Find official contact via website
✅ Verify: Check domain spelling
✅ Verify: Look for social media verification badge
✅ Report: Report scams immediately upon discovery
```

**Created**: November 11, 2025

