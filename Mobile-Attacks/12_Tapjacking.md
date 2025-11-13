# Tapjacking (Android Clickjacking)

## 📋 Attack Information
**Category**: Android Attack | **Severity**: 🔴 High (CVSS 7.0)

## 🎯 Attack Principle
Malicious app overlays transparent buttons over legitimate app's buttons to hijack user taps.

## 💀 Attack Example

```kotlin
// Malicious overlay
<Button
    android:alpha="0.01"  // Nearly transparent
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_gravity="center"
    android:onClick="maliciousAction"
/>

// Positioned over wallet's "Confirm Transfer" button
// User thinks they're tapping "Cancel"
// Actually tapping invisible "Approve" button
```

## 🛡️ Defense Measures

### 1. Detect Window Obscuring
```kotlin
// ✅ Check if window is obscured
override fun onFilterTouchEventForSecurity(event: MotionEvent): Boolean {
    if ((event.flags and MotionEvent.FLAG_WINDOW_IS_OBSCURED) != 0) {
        Toast.makeText(this, "Window obscured detected", Toast.LENGTH_LONG).show()
        return false // Reject tap
    }
    return super.onFilterTouchEventForSecurity(event)
}
```

### 2. Filter Touches When Obscured
```kotlin
// ✅ Sensitive buttons reject obscured touches
confirmButton.setFilterTouchesWhenObscured(true)

// If obscured, tap is ignored
```

### 3. React Native Protection
```typescript
/**
 * Native module to check obscuring
 */
import { NativeModules, Alert, Platform } from 'react-native';

const SecureButton = ({ onPress, title }) => {
  const handlePress = () => {
    if (Platform.OS === 'android') {
      // Check for obscuring
      NativeModules.SecurityModule.checkObscured()
        .then(obscured => {
          if (obscured) {
            Alert.alert('Security Warning', 'Screen obscured detected');
          } else {
            onPress();
          }
        });
    } else {
      onPress();
    }
  };
  
  return <Button onPress={handlePress} title={title} />;
};
```

## ✅ Best Practices

```
□ Implement onFilterTouchEventForSecurity
□ Use setFilterTouchesWhenObscured for critical buttons
□ Detect overlay permissions
□ Require user to disable overlay apps
□ Show visual feedback on important actions
```

**Created**: November 11, 2025

