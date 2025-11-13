# Screen Recording Attack

## 📋 Attack Information
**Category**: Mobile Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Malicious apps or system features record screen while user enters seed phrase or views private keys.

## 💀 Attack Scenarios

### 1. Malicious Screen Recording App
```kotlin
// Malicious app requests screen recording permission
val mediaProjection = mediaProjectionManager.getMediaProjection(resultCode, data)

// Records everything user does
// When user opens wallet and views seed phrase
// → Attacker gets video of seed phrase
```

### 2. Accessibility Service Abuse
```kotlin
// Malware uses Accessibility Service
class MaliciousAccessibilityService : AccessibilityService() {
    override fun onAccessibilityEvent(event: AccessibilityEvent) {
        // Can read all screen content
        if (event.packageName == "com.wallet.app") {
            // Capture all text user sees
            val text = event.text
            sendToAttacker(text)
        }
    }
}
```

### 3. Screen Mirroring
```
User enables screen mirroring to untrusted device/screen
→ Opens wallet
→ Seed phrase visible on mirrored screen
→ Attacker records
```

## 🛡️ Defense Measures

### 1. FLAG_SECURE (Android)
```kotlin
// ✅ Prevent screenshots and screen recording
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    // Set FLAG_SECURE on window
    window.setFlags(
        WindowManager.LayoutParams.FLAG_SECURE,
        WindowManager.LayoutParams.FLAG_SECURE
    )
    
    // App screens cannot be captured
    // Screenshots blocked
    // Screen recording shows black screen
}
```

### 2. iOS Screenshot Prevention
```swift
// ✅ Detect screenshots (iOS)
NotificationCenter.default.addObserver(
    forName: UIApplication.userDidTakeScreenshotNotification,
    object: nil,
    queue: .main
) { _ in
    // User took screenshot
    self.handleScreenshotDetected()
}

func handleScreenshotDetected() {
    // Hide sensitive content
    self.hideSensitiveViews()
    
    // Show warning
    let alert = UIAlertController(
        title: "Screenshot Detected",
        message: "Taking screenshots of seed phrases is not secure!",
        preferredStyle: .alert
    )
    present(alert, animated: true)
}
```

### 3. Blur on Background (iOS)
```swift
// ✅ Blur app when backgrounded
func applicationWillResignActive(_ application: UIApplication) {
    // Add blur view
    let blurEffect = UIBlurEffect(style: .light)
    let blurView = UIVisualEffectView(effect: blurEffect)
    blurView.frame = window!.bounds
    blurView.tag = 999
    window!.addSubview(blurView)
}

func applicationDidBecomeActive(_ application: UIApplication) {
    // Remove blur
    window?.viewWithTag(999)?.removeFromSuperview()
}

// Prevents seed phrase visibility in app switcher
```

### 4. React Native Protection
```typescript
/**
 * Prevent screen recording in React Native
 */
import { NativeModules, AppState } from 'react-native';

// Set FLAG_SECURE on Android
useEffect(() => {
  if (Platform.OS === 'android') {
    NativeModules.PreventScreenshotModule.forbid();
  }
}, []);

// Hide sensitive content on background
useEffect(() => {
  const subscription = AppState.addEventListener('change', (state) => {
    if (state === 'background' || state === 'inactive') {
      setSensitiveContentVisible(false);
    }
  });
  
  return () => subscription.remove();
}, []);
```

### 5. User Warnings
```typescript
<SeedPhraseView>
  <Alert severity="error">
    <h4>⚠️ SECURITY WARNING</h4>
    
    <ul>
      <li>❌ Do NOT take screenshots</li>
      <li>❌ Do NOT screen record</li>
      <li>❌ Do NOT screen mirror</li>
      <li>✅ Write on paper only</li>
      <li>✅ Store in secure location</li>
    </ul>
  </Alert>
  
  <SeedPhrase words={seedWords} />
  
  <Checkbox required>
    I understand and will not screenshot
  </Checkbox>
</SeedPhraseView>
```

### 6. Detect Screen Recording (iOS 11+)
```swift
// ✅ Detect active screen recording
if #available(iOS 11.0, *) {
    if UIScreen.main.isCaptured {
        // Screen is being recorded or mirrored
        showWarning("Screen recording detected!")
        hideSensitiveContent()
    }
    
    // Listen for changes
    NotificationCenter.default.addObserver(
        forName: UIScreen.capturedDidChangeNotification,
        object: nil,
        queue: .main
    ) { _ in
        if UIScreen.main.isCaptured {
            self.hideSensitiveContent()
        } else {
            self.showSensitiveContent()
        }
    }
}
```

## ✅ Best Practices

```
Implementation:
□ Enable FLAG_SECURE (Android)
□ Detect screenshots (iOS)
□ Blur app on background
□ Detect screen recording
□ Hide sensitive content on background

User Education:
□ Warn against screenshots
□ Explain risks of screen recording
□ Recommend paper backup only
□ Alert when screenshot detected

Architecture:
□ Minimize time sensitive data displayed
□ Require biometric to view seed phrase
□ Show warnings before displaying secrets
□ Implement view timeouts
```

**Created**: November 11, 2025

