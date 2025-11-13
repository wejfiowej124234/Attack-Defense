# App Reverse Engineering Attack

## 📋 Attack Information
**Category**: Mobile Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Attacker decompiles mobile app to extract API keys, secrets, vulnerabilities, and business logic.

## 💀 Attack Process

### Android APK Reverse Engineering
```bash
# 1. Download APK
adb pull /data/app/com.wallet.app/base.apk

# 2. Decompile with APKTool
apktool d base.apk

# 3. Convert DEX to JAR
d2j-dex2jar base.apk

# 4. Decompile Java
jd-gui base-dex2jar.jar

# 5. Extract secrets
grep -r "api_key" .
grep -r "private_key" .
grep -r "secret" .

# Found:
# - API keys
# - Backend URLs
# - Encryption keys
# - Business logic
```

### iOS IPA Reverse Engineering
```bash
# 1. Extract IPA
unzip app.ipa

# 2. Find binary
cd Payload/App.app

# 3. Disassemble
otool -tV App > disassembly.txt

# 4. Or use Hopper/IDA Pro
# 5. Extract strings
strings App | grep -i "key\|secret\|api"
```

## 🛡️ Defense Measures

### 1. Code Obfuscation
```kotlin
// Android: ProGuard/R8
// build.gradle
android {
    buildTypes {
        release {
            minifyEnabled true // ✅ Enable obfuscation
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                         'proguard-rules.pro'
        }
    }
}

// proguard-rules.pro
-keep class com.wallet.** { *; }
-keepclassmembers class * {
    @com.wallet.annotations.Keep *;
}
-repackageclasses ''
-allowaccessmodification
```

### 2. Native Code (C/C++)
```kotlin
// ✅ Store sensitive logic in native code
class CryptoModule {
    external fun encryptPrivateKey(key: String): ByteArray
    external fun sign(data: ByteArray): ByteArray
    
    companion object {
        init {
            System.loadLibrary("crypto-native")
        }
    }
}

// crypto-native.cpp (harder to reverse)
extern "C" JNIEXPORT jbyteArray JNICALL
Java_com_wallet_CryptoModule_encryptPrivateKey(
    JNIEnv* env, jobject, jstring key) {
    // Native implementation
    // Much harder to reverse than Java
}
```

### 3. String Encryption
```typescript
// ❌ Dangerous: Plain text strings
const API_KEY = "sk_live_abc123...";

// ✅ Encrypted strings
import { decrypt } from './crypto';

const ENCRYPTED_API_KEY = "U2FsdGVkX1..."; // Encrypted at build time

function getApiKey(): string {
    return decrypt(ENCRYPTED_API_KEY, getDeviceKey());
}
```

### 4. Root/Jailbreak Detection
```kotlin
// ✅ Detect rooted devices
fun isDeviceRooted(): Boolean {
    // Check for su binary
    val paths = arrayOf(
        "/system/app/Superuser.apk",
        "/sbin/su",
        "/system/bin/su",
        "/system/xbin/su"
    )
    
    for (path in paths) {
        if (File(path).exists()) return true
    }
    
    // Check for root management apps
    val rootApps = arrayOf(
        "com.noshufou.android.su",
        "com.thirdparty.superuser",
        "eu.chainfire.supersu"
    )
    
    for (pkg in rootApps) {
        if (isPackageInstalled(pkg)) return true
    }
    
    return false
}

// Warn or restrict functionality
if (isDeviceRooted()) {
    AlertDialog.Builder(this)
        .setTitle("Rooted Device Detected")
        .setMessage("This app cannot run on rooted devices for security reasons")
        .setPositiveButton("Exit") { _, _ -> finish() }
        .show()
}
```

### 5. Integrity Checks
```kotlin
// ✅ Verify app hasn't been tampered
fun verifyAppIntegrity(): Boolean {
    val context = applicationContext
    val packageInfo = context.packageManager.getPackageInfo(
        context.packageName,
        PackageManager.GET_SIGNATURES
    )
    
    val signature = packageInfo.signatures[0]
    val md = MessageDigest.getInstance("SHA-256")
    val certFingerprint = md.digest(signature.toByteArray())
    
    // Compare with official signature
    val officialFingerprint = "AA:BB:CC:DD..." // Official SHA-256
    val actualFingerprint = certFingerprint.toHex()
    
    return actualFingerprint == officialFingerprint
}
```

### 6. Runtime Protection
```typescript
// React Native: Detect debugging
import { NativeModules } from 'react-native';

async function detectDebugging(): Promise<boolean> {
    // Check if debugger is attached
    const isDebugged = await NativeModules.SecurityModule.isDebugged();
    
    if (isDebugged) {
        Alert.alert(
            'Debugging Detected',
            'This app cannot run while being debugged'
        );
        return true;
    }
    
    return false;
}

// Check on app start
useEffect(() => {
    detectDebugging();
}, []);
```

## ✅ Best Practices
```
Code Protection:
□ Enable ProGuard/R8 obfuscation
□ Use native code for sensitive logic
□ Encrypt all strings
□ No hardcoded secrets
□ Implement tamper detection

Runtime Protection:
□ Detect root/jailbreak
□ Verify app signature
□ Check for debugging
□ Monitor for hooking frameworks
□ Certificate pinning

Architecture:
□ Critical logic on backend
□ Use secure enclave/keystore
□ Hardware-backed keys
□ Minimal secrets in app
□ Regular security updates
```

**Created**: November 11, 2025

