# Address Manipulation Attack

## 📋 Attack Information
**Category**: Frontend/Display Attack | **Severity**: 🔴 Critical (CVSS 9.0)

## 🎯 Attack Principle
Attackers manipulate displayed addresses to trick users into sending funds to wrong addresses.

## 💀 Attack Vectors

### 1. Clipboard Hijacking
```javascript
// Malware running on user's computer
setInterval(() => {
  navigator.clipboard.readText().then(text => {
    // Check if text is crypto address
    if (isEthereumAddress(text)) {
      // Replace with attacker's address
      navigator.clipboard.writeText(ATTACKER_ADDRESS);
    }
  });
}, 100);

// User flow:
// 1. User copies legitimate address: 0xABCD...1234
// 2. Malware detects and replaces with: 0xATTACKER
// 3. User pastes → sends to attacker
```

### 2. Similar Address Attack
```
Legitimate: 0xABCD1234efgh5678ijkl9012mnop3456qrst7890
Attacker:   0xABCD1234efgh5678ijkl9012mnop3456qrst7891
                                                    ↑ Only last digit different!

User checks first/last few characters → looks correct
→ Sends to wrong address
```

### 3. Vanity Address Generation
```javascript
// Attacker generates addresses matching first/last chars
while (true) {
  const wallet = ethers.Wallet.createRandom();
  const address = wallet.address;
  
  // Match first 6 and last 4 characters
  if (address.startsWith('0xABCD12') && 
      address.endsWith('7890')) {
    console.log('Found match:', address);
    // Use this to impersonate victim
    break;
  }
}
```

### 4. UI Injection
```javascript
// XSS vulnerability allows address replacement
document.querySelectorAll('.wallet-address').forEach(el => {
  el.textContent = ATTACKER_ADDRESS;
});

// User sees attacker's address everywhere
```

## 🛡️ Defense Measures

### 1. Address Verification
```typescript
/**
 * Verify full address before transaction
 */
async function verifyAddress(displayedAddress: string): Promise<boolean> {
  // Show full address to user
  const confirmed = await showDialog({
    title: 'Confirm Recipient Address',
    message: `
      Full address:
      ${displayedAddress}
      
      ⚠️ Verify ALL characters!
      
      Common check:
      First 6: ${displayedAddress.slice(0, 8)}
      Last 4: ${displayedAddress.slice(-4)}
    `,
    copyButton: true,
    compareButton: true, // Allow user to compare with source
  });
  
  return confirmed;
}
```

### 2. Address Book / Contacts
```typescript
/**
 * Save trusted addresses with labels
 */
interface Contact {
  address: string;
  label: string;
  verified: boolean;
  lastUsed: Date;
}

// Show contacts instead of manual entry
<RecipientSelector>
  <Tabs>
    <Tab label="Contacts">
      {contacts.map(contact => (
        <ContactItem onClick={() => selectRecipient(contact)}>
          <Avatar>{contact.label[0]}</Avatar>
          <Label>{contact.label}</Label>
          <Address>{formatAddress(contact.address)}</Address>
          {contact.verified && <Badge>✓ Verified</Badge>}
        </ContactItem>
      ))}
    </Tab>
    
    <Tab label="Manual Entry">
      <Input 
        placeholder="Enter address"
        onChange={handleAddressInput}
      />
      <Warning>
        ⚠️ Double-check address before sending!
      </Warning>
    </Tab>
  </Tabs>
</RecipientSelector>
```

### 3. ENS/Domain Names
```typescript
/**
 * Use ENS instead of raw addresses
 */
// User enters: vitalik.eth
const address = await provider.resolveName('vitalik.eth');
// Returns: 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045

// Display both
<RecipientDisplay>
  <ENS>vitalik.eth</ENS>
  <Address>{address}</Address>
  <VerifyButton onClick={() => verifyOnEtherscan(address)}>
    Verify on Etherscan
  </VerifyButton>
</RecipientDisplay>
```

### 4. Visual Verification (Blockies/Identicons)
```typescript
/**
 * Show unique visual representation of address
 */
import blockies from 'ethereum-blockies';

function AddressDisplay({ address }: { address: string }) {
  // Generate unique icon for address
  const icon = blockies.create({
    seed: address.toLowerCase(),
    size: 8,
    scale: 16,
  });
  
  return (
    <div>
      <img src={icon.toDataURL()} />
      <span>{formatAddress(address)}</span>
    </div>
  );
}

// User can visually recognize addresses
```

### 5. Clipboard Protection
```typescript
/**
 * Detect clipboard manipulation
 */
let lastCopiedAddress: string | null = null;

async function secureAddressPaste(): Promise<string> {
  const clipboard = await navigator.clipboard.readText();
  
  // Compare with last copied address
  if (lastCopiedAddress && clipboard !== lastCopiedAddress) {
    const different = await showDialog({
      title: '⚠️ Address Mismatch',
      message: `
        Copied: ${lastCopiedAddress}
        Pasted: ${clipboard}
        
        Addresses don't match!
        Your clipboard may have been hijacked!
      `,
    });
    
    if (!different) {
      throw new Error('Clipboard hijacking detected');
    }
  }
  
  return clipboard;
}

// Track copies
document.addEventListener('copy', (e) => {
  const selection = window.getSelection()?.toString();
  if (isAddress(selection)) {
    lastCopiedAddress = selection;
  }
});
```

### 6. Transaction Confirmation
```typescript
/**
 * Multi-step confirmation for large amounts
 */
async function confirmTransaction(tx: Transaction) {
  // Step 1: Show full details
  await showTransactionDetails(tx);
  
  // Step 2: Verify address
  if (tx.value > LARGE_AMOUNT_THRESHOLD) {
    const verified = await verifyAddressManually(tx.to);
    if (!verified) throw new Error('Address verification failed');
  }
  
  // Step 3: Type confirmation
  const userTyped = await promptInput('Type recipient address to confirm:');
  if (userTyped.toLowerCase() !== tx.to.toLowerCase()) {
    throw new Error('Address mismatch');
  }
  
  // Step 4: Final confirmation
  const confirmed = await showDialog({
    title: 'Final Confirmation',
    message: `Send ${tx.value} ETH to ${tx.to}?`,
    destructive: true,
  });
  
  if (!confirmed) throw new Error('User cancelled');
}
```

## ✅ Best Practices
```
For Users:
□ Use address book for frequent recipients
□ Verify full address (not just first/last chars)
□ Use ENS names when possible
□ Check address visual (blockie/identicon)
□ Be cautious with clipboard
□ Use hardware wallet for large amounts

For Developers:
□ Display full address
□ Show address visual representation
□ Implement address book
□ Support ENS resolution
□ Warn on large transactions
□ Provide address verification tools
```

**Created**: November 11, 2025

