# Malicious Permission Abuse

## 📋 Attack Information
**Category**: Mobile Attack | **Severity**: 🔴 High (CVSS 7.0)

## 🎯 Attack Principle
Malicious apps request excessive permissions to steal data, monitor activities, or gain device control.

## 💀 Dangerous Permissions

### High-Risk Permissions
```xml
<!-- ❌ Dangerous if abused -->
<uses-permission android:name="android.permission.READ_CONTACTS" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.READ_CALL_LOG" />
<uses-permission android:name="android.permission.GET_ACCOUNTS" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />

<!-- Malware could:
- Steal contacts for phishing
- Record camera/audio
- Read 2FA SMS codes
- Track location
- Steal phone identity
-->
```

## 🛡️ Defense Measures

### 1. Minimal Permissions
```xml
<!-- ✅ Only request necessary permissions -->
<manifest>
    <!-- Wallet typically only needs: -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" /> <!-- For QR scanning -->
    <uses-permission android:name="android.permission.VIBRATE" />
    
    <!-- NO SMS, Contacts, Location, etc. -->
</manifest>
```

### 2. Runtime Permission with Context
```kotlin
// ✅ Request with clear explanation
fun requestCameraPermission() {
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
        != PackageManager.PERMISSION_GRANTED) {
        
        // Explain why permission needed
        AlertDialog.Builder(this)
            .setTitle("Camera Permission")
            .setMessage("Camera is needed to scan QR codes for addresses")
            .setPositiveButton("Grant") { _, _ ->
                ActivityCompat.requestPermissions(
                    this,
                    arrayOf(Manifest.permission.CAMERA),
                    CAMERA_PERMISSION_CODE
                )
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
}
```

### 3. Permission Auditing
```typescript
/**
 * Audit app permissions
 */
async function auditPermissions(): Promise<PermissionReport> {
  const granted = await PermissionsAndroid.getGrantedPermissions();
  
  const REQUIRED = ['CAMERA', 'INTERNET'];
  const SUSPICIOUS = ['READ_SMS', 'READ_CONTACTS', 'ACCESS_FINE_LOCATION'];
  
  const suspicious = granted.filter(p => 
    SUSPICIOUS.some(s => p.includes(s))
  );
  
  return {
    granted,
    required: REQUIRED,
    suspicious,
    warning: suspicious.length > 0 
      ? 'App has unnecessary permissions' 
      : null,
  };
}
```

### 4. User Permission Review
```typescript
<PermissionSettings>
  <Section title="App Permissions">
    {permissions.map(perm => (
      <Permission key={perm.name}>
        <Name>{perm.name}</Name>
        <Status granted={perm.granted} />
        <Reason>{perm.reason}</Reason>
        
        {!perm.required && (
          <Button onClick={() => revokePermission(perm)}>
            Revoke
          </Button>
        )}
      </Permission>
    ))}
  </Section>
  
  <Alert severity="info">
    Review and revoke unnecessary permissions regularly
  </Alert>
</PermissionSettings>
```

## ✅ Best Practices

```
Development:
□ Request minimal permissions
□ Explain each permission clearly
□ Request at point of use (not on startup)
□ Handle permission denial gracefully
□ Regular permission audits

User Protection:
□ Review app permissions before install
□ Revoke unnecessary permissions
□ Be suspicious of excessive permission requests
□ Uninstall apps with unexplained permissions
```

**Created**: November 11, 2025

