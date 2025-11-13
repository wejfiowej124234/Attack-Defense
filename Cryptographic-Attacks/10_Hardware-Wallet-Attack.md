# Hardware Wallet Attack

## 📋 Attack Information
**Category**: Physical + Firmware Attack | **Severity**: 🔴 High (CVSS 7.5)

## 🎯 Attack Principle
Attackers compromise hardware wallets via supply chain, firmware tampering, physical attacks, or social engineering.

## 💀 Attack Types

### 1. Supply Chain Attack
```
Attacker buys Ledger/Trezor
→ Installs malicious firmware
→ Pre-loads seed phrase
→ Repackages
→ Sells on eBay/Amazon as "new"
→ User imports assets
→ Attacker uses pre-loaded seed to steal
```

### 2. Firmware Tampering
```
Malicious firmware:
- Displays correct address, signs different address
- Uploads seed phrase to server
- Weak random number generation (predictable keys)
```

### 3. Fake Hardware Wallets
```
Complete counterfeit Ledger:
- Identical appearance
- Built-in trojan
- Generates known seed phrases
→ All user assets stolen
```

### 4. Physical Access Attacks
```
Side-Channel attacks:
- Power analysis
- Electromagnetic radiation analysis
- Timing attacks

Fault Injection:
- Voltage glitching
- Laser attacks
- Bypass PIN verification
```

## 🛡️ Defense Measures

### 1. Buy from Official Sources Only
```
✅ Safe Sources:
- Manufacturer website (ledger.com, trezor.io)
- Official authorized dealers
- Official Amazon store (verified)

❌ Dangerous Sources:
- eBay, second-hand markets
- Unauthorized dealers
- "Discounted" sales
- Pre-owned devices
```

### 2. Verify Device Authenticity
```
Ledger Verification:
1. Check packaging seal
2. Device self-test on boot
3. Verify firmware with Ledger Live
4. Check anti-tampering labels

Trezor Verification:
1. Packaging integrity
2. First boot generates NEW seed (not pre-loaded)
3. Verify bootloader signature
```

### 3. Generate Own Seed Phrase
```
✅ Correct Process:
1. Receive device, first boot
2. Select "Create new wallet"
3. Device generates seed phrase
4. Write down manually (don't photograph/type)

❌ Warning Signs:
- Device already has seed phrase
- Package includes "recovery card" with pre-filled seed
- Requests to enter seed on website for "verification"
```

### 4. Firmware Verification
```typescript
// Ledger device verification
import TransportWebUSB from '@ledgerhq/hw-transport-webusb';
import { getDeviceInfo } from '@ledgerhq/hw-app-eth';

async function verifyLedgerFirmware() {
  const transport = await TransportWebUSB.create();
  const deviceInfo = await getDeviceInfo(transport);
  
  // Check firmware version
  if (!OFFICIAL_VERSIONS.includes(deviceInfo.version)) {
    Alert.alert(
      '⚠️ Abnormal Firmware',
      `Detected unofficial firmware version: ${deviceInfo.version}`
    );
  }
  
  // Check if genuine
  if (!deviceInfo.isGenuine) {
    Alert.alert(
      '🚨 Device May Be Compromised',
      'Stop using immediately and contact Ledger support'
    );
  }
}
```

### 5. Transaction Verification
```
Every transaction signature:
1. ✅ Verify amount on hardware screen
2. ✅ Verify recipient address
3. ✅ Check network/chain ID
4. ❌ Don't blindly confirm

Note:
If screen differs from computer → computer compromised
If multiple different displays → hardware compromised
```

### 6. PIN Protection
```
✅ Set Strong PIN:
- At least 8 digits
- Not birthday/simple numbers
- Device resets after 3 wrong attempts

✅ Prevent Shoulder Surfing:
- Cover when entering PIN
- Use blind PIN (Ledger)
```

### 7. Multi-Signature
```
High-value assets:
Use multiple different brand hardware wallets

Example: 2-of-3 multisig
- 1 Ledger
- 1 Trezor
- 1 Coldcard

Single device compromised → assets still safe
```

## ✅ Checklist

```
New Device:
□ Packaging intact, no tampering
□ Anti-tampering labels normal
□ Purchased from official source
□ First boot, no pre-set content
□ Generate new seed phrase yourself
□ Firmware verification passed
□ Write seed phrase by hand, no photos/digital

During Use:
□ Update official firmware regularly
□ Carefully verify device screen on transactions
□ Don't connect to untrusted computers
□ Never enter seed phrase into any software
□ Set complex PIN
□ Prevent physical access

Abnormal Situations:
□ Abnormal screen display → Stop using
□ Requests seed phrase → Phishing
□ Firmware verification fails → Contact official support
□ Auto-signs without operation → Malicious firmware
```

**Created**: November 11, 2025

