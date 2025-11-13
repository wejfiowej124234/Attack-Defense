# Dust Attack

## 📋 Attack Information
**Category**: Privacy Attack | **Severity**: 🟡 Low (CVSS 3.0)

## 🎯 Attack Principle
Attacker sends tiny amounts ("dust") to many addresses, then tracks dust movement to link addresses and de-anonymize users.

## 💀 Attack Flow
```
1. Attacker selects 10,000 Bitcoin addresses
2. Sends 0.00000546 BTC (dust) to each
3. Waits for users to spend wallets (combining UTXOs)
4. Tracks dust UTXO usage
5. Links multiple addresses to same user
6. De-anonymizes user
7. Possible uses:
   - Targeted phishing
   - Blackmail
   - Regulatory tracking
```

## 💀 Real Cases
- **Binance (2019)**: Many users received dust attacks
- **Samourai Wallet**: Developed dust protection features

## 🛡️ Defense Measures

### 1. Dust Detection
```typescript
/**
 * Detect dust UTXOs
 */
const DUST_THRESHOLD = {
  BTC: 546,        // 546 satoshi
  ETH: 0.0001,     // 0.0001 ETH
  USDT: 1,         // 1 USDT
};

function isDust(amount: number, token: string): boolean {
  const threshold = DUST_THRESHOLD[token] || 0.0001;
  return amount <= threshold;
}

// Scan wallet
async function detectDustUTXOs(address: string): Promise<UTXO[]> {
  const utxos = await getUTXOs(address);
  return utxos.filter(utxo => isDust(utxo.value, 'BTC'));
}
```

### 2. Dust Isolation
```typescript
/**
 * Don't combine dust UTXOs
 */
async function selectUTXOs(address: string, targetAmount: number): Promise<UTXO[]> {
  const utxos = await getUTXOs(address);
  
  // ✅ Filter out dust
  const cleanUTXOs = utxos.filter(utxo => !isDust(utxo.value, 'BTC'));
  
  // Select from clean UTXOs only
  return coinSelect(cleanUTXOs, targetAmount);
}

// Dust never participates in transactions, cannot be tracked
```

### 3. Dust Labeling
```typescript
// ✅ Mark dust transactions in UI
<TransactionList>
  {transactions.map(tx => (
    <TransactionItem key={tx.hash}>
      <Amount value={tx.value} />
      
      {isDust(tx.value, tx.token) && (
        <Badge color="warning">⚠️ Dust Transaction</Badge>
      )}
      
      {tx.isDustAttack && (
        <Tooltip title="Possible dust attack, recommend not using this UTXO">
          <Icon>info</Icon>
        </Tooltip>
      )}
    </TransactionItem>
  ))}
</TransactionList>
```

### 4. Privacy Coin Mixing
```typescript
/**
 * Use CoinJoin or privacy coins
 */
// CoinJoin: Multiple users combine transactions
// Tracking becomes difficult

// Or use privacy coins:
// - Monero (XMR): Default privacy
// - Zcash (ZEC): Optional privacy
```

### 5. HD Wallet Strategy
```typescript
/**
 * Use HD wallet, generate new address each time
 */
async function getReceiveAddress(walletId: string): Promise<string> {
  // ✅ Generate new address each time (BIP44)
  const index = await getNextAddressIndex(walletId);
  const address = deriveAddress(walletId, index);
  
  return address;
  
  // Benefits:
  // - Each transaction uses different address
  // - Dust cannot link multiple addresses
  // - Increased privacy
}

// UI
<View>
  <Text>Receive Address (changes each time)</Text>
  <QRCode value={receiveAddress} />
  <Button onPress={generateNewAddress}>
    Generate New Address
  </Button>
</View>
```

## ✅ User Recommendations
```
□ Use new receiving address each time
□ Don't combine dust UTXOs
□ Ignore tiny incoming amounts
□ Use privacy-enhancing wallets
□ Consider CoinJoin for Bitcoin
```

**Created**: November 11, 2025

