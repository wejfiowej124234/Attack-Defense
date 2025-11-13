# Keylogging Attack

## 📋 Attack Information
**Category**: Mobile Attack | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Malicious keyboard apps or accessibility services record all keystrokes, stealing passwords and seed phrases.

## 💀 Attack Methods

### 1. Malicious Keyboard App
```kotlin
// Fake keyboard app
class MaliciousKeyboard : InputMethodService() {
    override fun onKey(primaryCode: Int, keyCodes: IntArray?) {
        val char = primaryCode.toChar()
        
        // Record all keystrokes
        logKeystroke(char)
        
        // Send to attacker
        if (keystrokeBuffer.length > 100) {
            sendToAttacker(keystrokeBuffer)
            keystrokeBuffer.clear()
        }
    }
}

// User types seed phrase → Stolen
```

### 2. Accessibility Service
```kotlin
// Malware uses Accessibility to monitor keyboard
class KeyloggerService : AccessibilityService() {
    override fun onAccessibilityEvent(event: AccessibilityEvent) {
        if (event.eventType == AccessibilityEvent.TYPE_VIEW_TEXT_CHANGED) {
            val text = event.text.toString()
            logText(text)
            sendToAttacker(text)
        }
    }
}
```

## 🛡️ Defense Measures

### 1. Custom Secure Keyboard
```kotlin
// ✅ Use app's own secure keyboard for sensitive input
class SecureKeyboard : View {
    private val keys = arrayOf(
        "1", "2", "3", "4", "5", "6", "7", "8", "9", "0"
    )
    
    // Randomize key positions
    private val shuffledKeys = keys.shuffle()
    
    override fun onDraw(canvas: Canvas) {
        // Draw custom keyboard
        // Not accessible to system keyboard loggers
    }
}

// Use for PIN/password entry
<SecureInput
    label="Enter PIN"
    useSecureKeyboard={true}
/>
```

### 2. Detect Third-Party Keyboards
```kotlin
// ✅ Detect if using non-system keyboard
fun isThirdPartyKeyboardActive(): Boolean {
    val inputMethodManager = getSystemService(INPUT_METHOD_SERVICE) as InputMethodManager
    val enabledInputMethods = inputMethodManager.enabledInputMethodList
    
    for (method in enabledInputMethods) {
        val packageName = method.packageName
        
        // Check if system keyboard
        if (packageName != "com.android.inputmethod" &&
            packageName != "com.google.android.inputmethod") {
            return true // Third-party keyboard
        }
    }
    
    return false
}

// Warn user
if (isThirdPartyKeyboardActive()) {
    AlertDialog.Builder(this)
        .setTitle("⚠️ Third-Party Keyboard Detected")
        .setMessage("For security, use system keyboard when entering sensitive data")
        .show()
}
```

### 3. Numeric Pad for PIN
```typescript
// ✅ Use numeric keypad (less attack surface)
<TextInput
  keyboardType="numeric"
  secureTextEntry={true}
  placeholder="Enter PIN"
/>

// React Native: Built-in secure input
// Doesn't use custom keyboards
```

### 4. Biometric Instead of Typing
```kotlin
// ✅ Use biometric authentication (no typing needed)
val biometricPrompt = BiometricPrompt(
    this,
    executor,
    object : BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
            // User authenticated without typing
            proceedToWallet()
        }
    }
)

biometricPrompt.authenticate(promptInfo)
```

### 5. Accessibility Service Detection
```kotlin
// ✅ Detect suspicious accessibility services
fun getSuspiciousAccessibilityServices(): List<String> {
    val accessibilityManager = getSystemService(ACCESSIBILITY_SERVICE) as AccessibilityManager
    val enabledServices = accessibilityManager.getEnabledAccessibilityServiceList(
        AccessibilityServiceInfo.FEEDBACK_ALL_MASK
    )
    
    val suspicious = mutableListOf<String>()
    
    for (service in enabledServices) {
        val packageName = service.resolveInfo.serviceInfo.packageName
        
        // Check if system service
        if (!packageName.startsWith("com.android") &&
            !packageName.startsWith("com.google")) {
            suspicious.add(service.resolveInfo.loadLabel(packageManager).toString())
        }
    }
    
    return suspicious
}

// Warn user
val suspicious = getSuspiciousAccessibilityServices()
if (suspicious.isNotEmpty()) {
    AlertDialog.Builder(this)
        .setTitle("⚠️ Accessibility Services Active")
        .setMessage("These apps can monitor your screen and keyboard:\n${suspicious.joinToString("\n")}")
        .setPositiveButton("Review") { _, _ ->
            startActivity(Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS))
        }
        .show()
}
```

### 6. Mask Input
```typescript
// ✅ Mask sensitive input
<SecureInput
  secureTextEntry={true}  // Shows dots instead of characters
  autoComplete="off"
  autoCorrect={false}
  spellCheck={false}
/>
```

## ✅ Best Practices

```
For Sensitive Input:
□ Use custom secure keyboard
□ Prefer biometric over typing
□ Use numeric pad when possible
□ Mask input (secureTextEntry)
□ Detect third-party keyboards
□ Warn about accessibility services

Architecture:
□ Minimize keyboard usage
□ Use QR codes for addresses
□ Biometric for authentication
□ Hardware security keys when possible

User Education:
□ Warn about keyboard apps
□ Explain keylogging risks
□ Recommend system keyboard only
□ Review accessibility permissions
```

**Created**: November 11, 2025

