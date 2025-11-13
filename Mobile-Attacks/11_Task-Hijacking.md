# Task Hijacking Attack (Android)

## 📋 Attack Information
**Category**: Android-Specific Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Exploit Android task stack management to overlay phishing screens when user opens legitimate wallet app.

## 💀 Attack Scenario

```kotlin
// Malicious app's Activity
class PhishingActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Displays fake login screen
    }
}

// AndroidManifest.xml
<activity
    android:name=".PhishingActivity"
    android:taskAffinity="com.rustwallet.app"  <!-- ❌ Hijacks task stack -->
    android:launchMode="singleTask"
    android:excludeFromRecents="true"  <!-- Hidden from recent apps -->
>

// Attack flow:
// 1. User installs malware (disguised as other app)
// 2. User taps wallet icon
// 3. Android launches wallet app
// 4. Malware detects, launches PhishingActivity
// 5. User sees fake login screen
// 6. User enters password → Stolen
```

## 🛡️ Defense Measures

### 1. Correct TaskAffinity Configuration
```xml
<!-- AndroidManifest.xml -->
<activity
    android:name=".MainActivity"
    android:taskAffinity="${applicationId}"  <!-- ✅ Unique taskAffinity -->
    android:launchMode="singleTop"
    android:excludeFromRecents="false"
>
```

### 2. Verify Task Integrity
```kotlin
/**
 * Verify current Activity is in correct task
 */
fun verifyTaskIntegrity(): Boolean {
    val activityManager = getSystemService(ACTIVITY_MANAGER_SERVICE) as ActivityManager
    val tasks = activityManager.getRunningTasks(1)
    
    if (tasks.isNotEmpty()) {
        val topTask = tasks[0]
        
        // ✅ Verify task belongs to this app
        if (topTask.topActivity?.packageName != packageName) {
            AlertDialog.Builder(this)
                .setTitle("Security Warning")
                .setMessage("Abnormal task stack detected, possible hijacking")
                .setPositiveButton("Exit") { _, _ -> finish() }
                .setCancelable(false)
                .show()
            
            return false
        }
    }
    
    return true
}

// Call in every Activity's onResume
override fun onResume() {
    super.onResume()
    if (!verifyTaskIntegrity()) {
        finish()
    }
}
```

### 3. Screen Obscured Check
```kotlin
/**
 * Check if screen is being obscured
 */
fun isScreenObscured(): Boolean {
    return window.attributes.flags and 
           WindowManager.LayoutParams.FLAG_WINDOW_IS_OBSCURED != 0
}

// On sensitive operations
override fun onFilterTouchEventForSecurity(event: MotionEvent): Boolean {
    if (isScreenObscured()) {
        Toast.makeText(this, "Screen obscured, operation cancelled", Toast.LENGTH_LONG).show()
        return false // Reject touch event
    }
    return super.onFilterTouchEventForSecurity(event)
}
```

### 4. Verify Activity Origin
```kotlin
/**
 * Verify who launched this Activity
 */
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    // ✅ Check caller
    val referrer = referrer
    if (referrer != null && referrer.host != packageName) {
        AlertDialog.Builder(this)
            .setTitle("Security Warning")
            .setMessage("Activity launched by external app")
            .setPositiveButton("Exit") { _, _ -> finish() }
            .show()
    }
}
```

### 5. React Native Protection
```typescript
// React Native: Use native module
import { NativeModules } from 'react-native';

const SecurityModule = NativeModules.SecurityModule;

// Check before sensitive operations
const showPrivateKey = async () => {
  const isSafe = await SecurityModule.checkTaskIntegrity();
  
  if (!isSafe) {
    Alert.alert(
      'Security Warning',
      'Abnormal app environment detected, cannot display sensitive information'
    );
    return;
  }
  
  // Display private key
  setShowKey(true);
};
```

## ✅ Best Practices

```
□ Use unique taskAffinity (${applicationId})
□ Verify task stack integrity
□ Check for screen obscuring
□ Verify Activity launch origin
□ Implement anti-tampering
□ Regular security checks in onResume
□ User education about overlay attacks
```

**Created**: November 11, 2025

