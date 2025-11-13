# Memory Dump Attack

## 📋 Attack Information
**Category**: Mobile Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Attacker dumps app memory to extract private keys, passwords, or session tokens stored in RAM.

## 💀 Attack Methods

### Android Memory Dump
```bash
# Root required
adb shell
su

# Dump process memory
cat /proc/<PID>/maps  # Find memory regions
dd if=/proc/<PID>/mem of=/sdcard/dump.bin skip=<offset> count=<size>

# Search dump for sensitive data
strings dump.bin | grep -i "private\|key\|password"
```

### iOS Memory Dump
```bash
# Jailbreak required
# Use tools like:
- Fridump
- r2frida
- lldb

# Dump memory and search for keys
```

## 🛡️ Defense Measures

### 1. Never Store Sensitive Data in Memory Long-Term
```kotlin
// ❌ Dangerous: Keep in memory
class WalletManager {
    var privateKey: String = "" // Stays in memory!
    
    fun sign(data: ByteArray): ByteArray {
        return signWithKey(privateKey, data)
    }
}

// ✅ Secure: Retrieve when needed
class SecureWalletManager {
    suspend fun sign(data: ByteArray): ByteArray {
        // Get key from secure storage
        val privateKey = SecureStore.getPrivateKey()
        
        try {
            return signWithKey(privateKey, data)
        } finally {
            // Clear immediately
            privateKey.fill('\u0000')
        }
    }
}
```

### 2. Clear Sensitive Data After Use
```typescript
/**
 * Zero out sensitive data after use
 */
function clearSensitiveString(str: string): void {
  // JavaScript strings are immutable, use Buffer
  const buffer = Buffer.from(str, 'utf8');
  buffer.fill(0);
}

// Usage
let privateKey = await getPrivateKey();

try {
  const signature = await sign(data, privateKey);
  return signature;
} finally {
  // Clear from memory
  if (privateKey) {
    clearSensitiveString(privateKey);
    privateKey = null;
  }
}
```

### 3. Use Secure Enclave/Keystore
```kotlin
// ✅ Android: Hardware-backed keys never leave secure element
import android.security.keystore.KeyGenParameterSpec
import android.security.keystore.KeyProperties

fun generateSecureKey(alias: String) {
    val keyGenerator = KeyGenerator.getInstance(
        KeyProperties.KEY_ALGORITHM_AES,
        "AndroidKeyStore"
    )
    
    keyGenerator.init(
        KeyGenParameterSpec.Builder(
            alias,
            KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
        )
            .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .setUserAuthenticationRequired(true) // Requires biometric
            .build()
    )
    
    keyGenerator.generateKey()
    // Key stored in hardware, never in app memory
}
```

### 4. Memory Encryption
```kotlin
// ✅ Encrypt sensitive data even in memory
class EncryptedMemory {
    private val memoryKey = generateRandomKey()
    
    fun storeSecret(secret: String): ByteArray {
        // Encrypt before storing in memory
        return encrypt(secret, memoryKey)
    }
    
    fun retrieveSecret(encrypted: ByteArray): String {
        val decrypted = decrypt(encrypted, memoryKey)
        return decrypted
    }
}
```

### 5. Root Detection
```kotlin
// ✅ Detect rooted devices
if (isDeviceRooted()) {
    // Restrict functionality
    disableSensitiveFeatures()
    
    AlertDialog.Builder(this)
        .setTitle("Security Warning")
        .setMessage("This app cannot run on rooted devices")
        .setPositiveButton("Exit") { _, _ -> finish() }
        .show()
}
```

### 6. Flag Secure (Prevent Screenshots)
```kotlin
// ✅ Prevent screenshots and memory capture
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    // FLAG_SECURE prevents screenshots and screen recording
    window.setFlags(
        WindowManager.LayoutParams.FLAG_SECURE,
        WindowManager.LayoutParams.FLAG_SECURE
    )
}
```

### 7. React Native Protection
```typescript
/**
 * Clear sensitive data on app background
 */
import { AppState } from 'react-native';

AppState.addEventListener('change', (nextAppState) => {
  if (nextAppState === 'background') {
    // Clear sensitive data from memory
    clearPrivateKey();
    clearSessionData();
  }
});
```

## ✅ Best Practices

```
Key Management:
□ Use hardware keystore (Secure Enclave, AndroidKeyStore)
□ Never store keys in app memory long-term
□ Clear sensitive data immediately after use
□ Zero out buffers
□ Use biometric-protected keys

Memory Protection:
□ Enable FLAG_SECURE
□ Detect root/jailbreak
□ Encrypt data in memory
□ Clear on app background
□ Avoid heap allocations for keys

Architecture:
□ Minimize time keys in memory
□ Retrieve from secure storage when needed
□ Use hardware security modules
□ Implement anti-debugging
```

**Created**: November 11, 2025

