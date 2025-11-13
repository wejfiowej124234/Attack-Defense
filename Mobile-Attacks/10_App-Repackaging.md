# App Repackaging Attack

## 📋 Attack Information
**Category**: Mobile Attack | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker decompiles official app, injects malicious code, repackages, and uploads to third-party stores to trick users into downloading.

## 💀 Attack Flow
```
1. Download official wallet APK
2. Decompile using APKTool
3. Inject malicious code:
   - Private key theft code
   - Transaction address modification
   - User data upload
4. Re-sign and repackage
5. Upload to third-party app stores
6. SEO optimization for high ranking
7. Users download fake app
8. Users create wallet → private keys stolen
```

## 💀 Real Cases
- **Fake MetaMask**: Multiple fake MetaMask apps found on Google Play
- **Fake imToken**: Numerous fake apps on third-party stores
- **Fake Trust Wallet**: Losses of millions of dollars

## 🛡️ Defense Measures

### 1. Signature Verification
```kotlin
// Android: Verify app signature
fun verifyAppSignature(context: Context): Boolean {
    val packageInfo = context.packageManager.getPackageInfo(
        context.packageName,
        PackageManager.GET_SIGNATURES
    )
    
    val signature = packageInfo.signatures[0]
    val md = MessageDigest.getInstance("SHA-256")
    val certFingerprint = md.digest(signature.toByteArray())
    
    // ✅ Compare with official signature fingerprint
    val officialFingerprint = "AA:BB:CC:DD:EE:FF..." // Official SHA-256
    val actualFingerprint = certFingerprint.toHex()
    
    if (actualFingerprint != officialFingerprint) {
        // Signature mismatch - possibly fake app
        AlertDialog.Builder(context)
            .setTitle("⚠️ Security Warning")
            .setMessage("Invalid app signature! This may be a fake app!")
            .setPositiveButton("Uninstall") { _, _ ->
                uninstallApp()
            }
            .setCancelable(false)
            .show()
        
        return false
    }
    
    return true
}

// Verify on app startup
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    if (!verifyAppSignature(this)) {
        finish()
    }
}
```

### 2. Server-Side Verification
```typescript
/**
 * Verify app legitimacy when connecting to API
 */
// Server-side
app.use(async (req, res, next) => {
  const appSignature = req.headers['x-app-signature'];
  const appVersion = req.headers['x-app-version'];
  
  // ✅ Verify app signature
  const validSignatures = [
    'SHA256:AAAA...', // Android release
    'SHA256:BBBB...', // iOS release
    'SHA256:CCCC...', // Android beta
  ];
  
  if (!validSignatures.includes(appSignature)) {
    return res.status(403).json({
      error: 'Invalid app signature',
      message: 'Unofficial app detected, please download from official source',
    });
  }
  
  next();
});

// Client sends signature
const appSignature = await getAppSignature();

axios.defaults.headers.common['X-App-Signature'] = appSignature;
axios.defaults.headers.common['X-App-Version'] = APP_VERSION;
```

### 3. Code Integrity Check
```kotlin
// ✅ Runtime verification of DEX file
fun verifyDexIntegrity(): Boolean {
    val context = applicationContext
    val apkPath = context.packageCodePath
    
    // Calculate APK SHA-256
    val apkFile = File(apkPath)
    val digest = MessageDigest.getInstance("SHA-256")
    val hash = digest.digest(apkFile.readBytes())
    
    // Compare with official hash
    val officialHash = "expected_hash_here"
    val actualHash = hash.toHex()
    
    return actualHash == officialHash
}
```

### 4. Google Play Integrity API
```kotlin
// ✅ Use Google Play verification
import com.google.android.play.core.integrity.IntegrityManagerFactory

val integrityManager = IntegrityManagerFactory.create(context)

val tokenRequest = IntegrityTokenRequest.builder()
    .setNonce(generateNonce())
    .build()

integrityManager.requestIntegrityToken(tokenRequest)
    .addOnSuccessListener { response ->
        val token = response.token()
        // Send to server for verification
        verifyWithServer(token)
    }
    .addOnFailureListener { error ->
        // Verification failed - possibly fake app
        showWarning("App integrity verification failed")
    }
```

### 5. User Education
```typescript
<Alert severity="error">
  <h4>⚠️ Download from Official Sources Only</h4>
  
  <h5>✅ Official Download Sources:</h5>
  <ul>
    <li>Google Play Store</li>
    <li>Apple App Store</li>
    <li>Official website: https://rustwallet.com</li>
  </ul>
  
  <h5>❌ Don't Download From:</h5>
  <ul>
    <li>❌ Third-party app stores</li>
    <li>❌ APK download websites</li>
    <li>❌ Social media links</li>
    <li>❌ Email attachments</li>
  </ul>
  
  <h5>🔍 How to Verify Authenticity?</h5>
  <ul>
    <li>Check developer name</li>
    <li>Check download count and ratings</li>
    <li>Compare app icon and interface</li>
    <li>Verify link on official website</li>
  </ul>
</Alert>
```

### 6. First Launch Verification
```typescript
/**
 * Verify with official server on first launch
 */
async function firstRunVerification() {
  const deviceId = await getDeviceId();
  const appSignature = await getAppSignature();
  const appVersion = APP_VERSION;
  
  try {
    const response = await axios.post('https://api.rustwallet.com/verify-app', {
      deviceId,
      appSignature,
      appVersion,
      platform: Platform.OS,
    });
    
    if (!response.data.verified) {
      Alert.alert(
        '⚠️ App Verification Failed',
        `
        Cannot verify app legitimacy
        Reason: ${response.data.reason}
        
        Please ensure you downloaded from official source:
        - Google Play
        - App Store
        - https://rustwallet.com
        `
      );
      
      // Disable sensitive functions
      disableWalletCreation();
    }
  } catch (error) {
    // Network error - warn but allow continue
    showWarning('Cannot connect to verification server, please be cautious');
  }
}
```

## ✅ Release Checklist
```
Before Release:
□ Use official signing certificate
□ Enable ProGuard obfuscation
□ Remove debug code
□ Record APK SHA-256
□ Publish signature fingerprint on website

During Release:
□ Upload to official stores only
□ Configure app store verification
□ Release announcement (website + social media)

After Release:
□ Monitor for fake apps
□ Report fake apps
□ User warnings
```

**Created**: November 11, 2025

