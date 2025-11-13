# Deep Link Hijacking Attack

## 📋 Attack Information
**Category**: Mobile Attack | **Severity**: 🟠 Medium (CVSS 6.0)

## 🎯 Attack Principle
Malicious app registers same deep link scheme, intercepting links intended for legitimate wallet app.

## 💀 Attack Scenario

```xml
<!-- Legitimate wallet app -->
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <data android:scheme="rustwallet" />
</intent-filter>

<!-- Malicious app registers SAME scheme -->
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <data android:scheme="rustwallet" /> <!-- Same! -->
</intent-filter>

<!-- When user clicks: rustwallet://send?to=0xABC&amount=100 -->
<!-- Android shows app chooser -->
<!-- User might pick malicious app -->
<!-- Malicious app steals transaction details or modifies them -->
```

## 🛡️ Defense Measures

### 1. Use App Links (Android)
```xml
<!-- ✅ Verified domain-based links -->
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    
    <data
        android:scheme="https"
        android:host="rustwallet.com"
        android:pathPrefix="/app" />
</intent-filter>

<!-- Requires proving domain ownership -->
<!-- Place assetlinks.json on website -->
<!-- Only verified app handles links -->
```

### 2. Universal Links (iOS)
```swift
// ✅ iOS Universal Links
// apple-app-site-association file on website
{
  "applinks": {
    "apps": [],
    "details": [{
      "appID": "TEAM_ID.com.rustwallet.app",
      "paths": ["/app/*"]
    }]
  }
}

// Only official app handles https://rustwallet.com/app/* links
```

### 3. Validate Deep Link Parameters
```kotlin
// ✅ Validate all deep link data
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    val data = intent?.data
    if (data != null) {
        // Validate scheme
        if (data.scheme != "https" || data.host != "rustwallet.com") {
            Toast.makeText(this, "Invalid link", Toast.LENGTH_SHORT).show()
            finish()
            return
        }
        
        // Validate parameters
        val to = data.getQueryParameter("to")
        val amount = data.getQueryParameter("amount")
        
        if (to != null && !isValidAddress(to)) {
            Toast.makeText(this, "Invalid address", Toast.LENGTH_SHORT).show()
            finish()
            return
        }
        
        // Show confirmation before executing
        showTransferConfirmation(to, amount)
    }
}
```

### 4. User Confirmation
```typescript
/**
 * Always confirm deep link actions
 */
const handleDeepLink = async (url: string) => {
  const params = parseDeepLink(url);
  
  // Show confirmation
  const confirmed = await showDialog({
    title: 'External Request',
    message: `
      Action: ${params.action}
      Recipient: ${params.to}
      Amount: ${params.amount}
      
      Verify this information carefully!
    `,
    destructive: true,
  });
  
  if (!confirmed) return;
  
  // Execute action
  await executeDeepLinkAction(params);
};
```

### 5. Display Link Source
```typescript
// ✅ Show where link came from
<DeepLinkConfirmation>
  <Alert severity="warning">
    ⚠️ Opened from external source
  </Alert>
  
  <Field label="Source App">
    {getReferrerApp()}
  </Field>
  
  <Field label="Action">
    {action}
  </Field>
  
  <Field label="Parameters">
    {JSON.stringify(params, null, 2)}
  </Field>
  
  <Button onClick={confirm}>
    I Verified, Proceed
  </Button>
</DeepLinkConfirmation>
```

## ✅ Best Practices

```
□ Use App Links (Android) / Universal Links (iOS)
□ Verify domain ownership
□ Validate all deep link parameters
□ Require user confirmation
□ Show action details before executing
□ Display source of deep link
□ Use HTTPS scheme (not custom schemes)
```

**Created**: November 11, 2025

