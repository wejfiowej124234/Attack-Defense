# SSL Pinning Bypass (MITM on Mobile)

## 📋 Attack Information
**Category**: Mobile Network Attack | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker bypasses SSL certificate pinning to perform man-in-the-middle attacks on mobile apps.

## 💀 Attack Tools & Methods

### 1. Frida + SSL Kill Switch
```javascript
// Frida script to bypass SSL pinning
Java.perform(function() {
    // Hook SSLContext
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    
    SSLContext.init.overload(
        '[Ljavax.net.ssl.KeyManager;',
        '[Ljavax.net.ssl.TrustManager;',
        'java.security.SecureRandom'
    ).implementation = function(keyManager, trustManager, secureRandom) {
        // Install custom TrustManager that accepts all certificates
        this.init(keyManager, [CustomTrustManager], secureRandom);
    };
});

// All SSL certificates accepted → MITM possible
```

### 2. Objection (iOS/Android)
```bash
# Bypass SSL pinning
objection -g com.wallet.app explore
android sslpinning disable

# App now accepts any certificate
# Attacker can intercept with Burp Suite/Charles Proxy
```

### 3. Xposed Modules
```
Install JustTrustMe or SSLUnpinning module
→ Globally disables SSL pinning
→ All apps vulnerable to MITM
```

## 🛡️ Defense Measures

### 1. Multi-Layer Certificate Pinning
```kotlin
// ✅ Android: OkHttp Certificate Pinning
import okhttp3.CertificatePinner

val certificatePinner = CertificatePinner.Builder()
    .add("api.rustwallet.com", "sha256/AAAA...")  // Primary certificate
    .add("api.rustwallet.com", "sha256/BBBB...")  // Backup certificate
    .build()

val client = OkHttpClient.Builder()
    .certificatePinner(certificatePinner)
    .build()

// Connection fails if certificate doesn't match
```

### 2. Root/Jailbreak Detection
```kotlin
// ✅ Detect compromised devices
fun isDeviceCompromised(): Boolean {
    // Check for root
    val rootIndicators = arrayOf(
        "/system/app/Superuser.apk",
        "/sbin/su",
        "/system/bin/su",
        "/system/xbin/su",
        "/data/local/xbin/su",
        "/data/local/bin/su"
    )
    
    for (path in rootIndicators) {
        if (File(path).exists()) return true
    }
    
    // Check for Xposed
    try {
        throw Exception()
    } catch (e: Exception) {
        for (line in e.stackTrace) {
            if (line.className.contains("Xposed")) return true
        }
    }
    
    // Check for Frida
    val fridaLibs = arrayOf("frida-agent", "frida-gadget")
    for (lib in fridaLibs) {
        if (checkLibraryLoaded(lib)) return true
    }
    
    return false
}

// Restrict functionality on compromised devices
if (isDeviceCompromised()) {
    AlertDialog.Builder(this)
        .setTitle("Security Warning")
        .setMessage("Cannot run on rooted/jailbroken device")
        .setPositiveButton("Exit") { _, _ -> finish() }
        .show()
}
```

### 3. Runtime Integrity Checks
```kotlin
// ✅ Detect hooking frameworks
fun detectHooking(): Boolean {
    try {
        // Check if classes are modified
        val method = OkHttpClient::class.java.getMethod("certificatePinner")
        val declaringClass = method.declaringClass
        
        // If class loader is suspicious
        if (declaringClass.classLoader.toString().contains("Xposed")) {
            return true
        }
    } catch (e: Exception) {
        // Method not found or error - possible tampering
        return true
    }
    
    return false
}
```

### 4. Network Security Config (Android)
```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.rustwallet.com</domain>
        <pin-set>
            <!-- Pin multiple certificates -->
            <pin digest="SHA-256">AAAA...</pin>
            <pin digest="SHA-256">BBBB...</pin>
        </pin-set>
    </domain-config>
</network-security-config>

<!-- AndroidManifest.xml -->
<application
    android:networkSecurityConfig="@xml/network_security_config">
```

### 5. iOS Certificate Pinning
```swift
// ✅ iOS: URLSession with pinning
class PinningDelegate: NSObject, URLSessionDelegate {
    let pinnedCerts: [Data]
    
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge,
        completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void
    ) {
        guard let serverTrust = challenge.protectionSpace.serverTrust,
              let serverCert = SecTrustGetCertificateAtIndex(serverTrust, 0)
        else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        let serverCertData = SecCertificateCopyData(serverCert) as Data
        
        // Check if certificate matches pinned
        if pinnedCerts.contains(serverCertData) {
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

### 6. React Native SSL Pinning
```typescript
// ✅ React Native
import { fetch } from 'react-native-ssl-pinning';

await fetch('https://api.rustwallet.com', {
  method: 'GET',
  sslPinning: {
    certs: ['rustwallet'], // Certificate in assets
  },
});

// Rejects if certificate doesn't match
```

### 7. Obfuscate Pinning Logic
```kotlin
// ✅ Make pinning logic harder to find
// Use native code for pinning checks
// Obfuscate with ProGuard/R8
// Split logic across multiple methods
```

## ✅ Best Practices

```
Implementation:
□ Implement certificate pinning
□ Pin multiple certificates (current + backup)
□ Use system-level pinning (network_security_config)
□ Implement in native code
□ Obfuscate pinning logic

Runtime Protection:
□ Detect root/jailbreak
□ Detect hooking frameworks (Frida, Xposed)
□ Verify app integrity
□ Monitor for SSL bypass attempts

User Education:
□ Warn about rooting/jailbreaking
□ Explain MITM risks
□ Recommend using trusted networks only
```

**Created**: November 11, 2025

