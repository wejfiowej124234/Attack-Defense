# Clipboard Monitoring Attack

## 📋 Attack Information
**Category**: Mobile Attack | **Severity**: 🟠 Medium (CVSS 5.0)

## 🎯 Attack Principle
Malicious apps monitor clipboard to steal cryptocurrency addresses and replace with attacker's address.

## 💀 Attack Flow

```kotlin
// Malicious app constantly monitors clipboard
class ClipboardMonitor : ClipboardManager.OnPrimaryClipChangedListener {
    override fun onPrimaryClipChanged() {
        val clipboard = getSystemService(Context.CLIPBOARD_SERVICE) as ClipboardManager
        val text = clipboard.primaryClip?.getItemAt(0)?.text?.toString()
        
        // Check if text is crypto address
        if (isEthereumAddress(text)) {
            // Replace with attacker's address
            val attackerAddress = "0xATTACKER..."
            
            val clip = ClipData.newPlainText("address", attackerAddress)
            clipboard.setPrimaryClip(clip)
            
            // User copies: 0xLEGITIMATE...
            // User pastes: 0xATTACKER...
        }
    }
}
```

## 🛡️ Defense Measures

### 1. Clipboard Verification
```typescript
/**
 * Verify clipboard hasn't been tampered
 */
let lastCopiedAddress: string | null = null;

async function securePaste(): Promise<string> {
  const clipboard = await Clipboard.getString();
  
  // Check if matches what we copied
  if (lastCopiedAddress && clipboard !== lastCopiedAddress) {
    Alert.alert(
      '⚠️ Clipboard Modified',
      `
      Expected: ${lastCopiedAddress.slice(0, 10)}...
      Found: ${clipboard.slice(0, 10)}...
      
      Your clipboard may have been hijacked!
      Please verify the address carefully.
      `,
      [
        { text: 'Use Expected', onPress: () => lastCopiedAddress },
        { text: 'Use Clipboard', onPress: () => clipboard },
        { text: 'Cancel' },
      ]
    );
  }
  
  return clipboard;
}

// Track copies
const handleCopy = (address: string) => {
  Clipboard.setString(address);
  lastCopiedAddress = address;
  
  Alert.alert('✅ Address Copied', 'Please verify when pasting');
};
```

### 2. Clear Clipboard After Use
```typescript
/**
 * Auto-clear clipboard
 */
import Clipboard from '@react-native-clipboard/clipboard';

async function copyAddress(address: string) {
  Clipboard.setString(address);
  
  Alert.alert('Address Copied', 'Clipboard will be cleared in 60 seconds');
  
  // Auto-clear after 60 seconds
  setTimeout(() => {
    Clipboard.setString('');
  }, 60000);
}
```

### 3. Visual Verification
```typescript
/**
 * Show address with visual identifier
 */
import blockies from 'ethereum-blockies';

<AddressCopy address={address}>
  <Blockies seed={address} size={8} scale={4} />
  <Address>{formatAddress(address)}</Address>
  
  <Button onPress={() => copyWithVerification(address)}>
    Copy Address
  </Button>
  
  <Alert severity="warning">
    ⚠️ Always verify full address after pasting!
    First 6: {address.slice(0, 8)}
    Last 4: {address.slice(-4)}
  </Alert>
</AddressCopy>
```

### 4. QR Code Preferred
```typescript
/**
 * Use QR codes instead of copy/paste
 */
<ReceiveAddress>
  <QRCode value={address} size={200} />
  
  <Text>Scan QR Code (More Secure)</Text>
  
  <Button onPress={() => copyAddress(address)}>
    Or Copy Address (Verify After Pasting)
  </Button>
</ReceiveAddress>
```

### 5. Paste Verification
```typescript
/**
 * Verify pasted address
 */
const handlePaste = async () => {
  const pasted = await Clipboard.getString();
  
  // Show confirmation with full address
  const confirmed = await showDialog({
    title: 'Verify Pasted Address',
    message: `
      Full address:
      ${pasted}
      
      First 6: ${pasted.slice(0, 8)}
      Last 4: ${pasted.slice(-4)}
      
      Is this correct?
    `,
    buttons: ['No, Re-enter', 'Yes, Correct'],
  });
  
  if (!confirmed) {
    return null;
  }
  
  return pasted;
};
```

## ✅ Best Practices

```
For Wallets:
□ Track copied addresses
□ Detect clipboard modifications
□ Auto-clear clipboard after time
□ Prefer QR codes over copy/paste
□ Require address verification before send
□ Show visual identifiers (blockies)

For Users:
□ Verify full address after pasting
□ Compare first/last characters
□ Use QR codes when possible
□ Check for clipboard-monitoring apps
□ Uninstall suspicious apps with clipboard permission
```

**Created**: November 11, 2025

