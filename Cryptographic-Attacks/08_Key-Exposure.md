# Key Exposure - Environment Variables & Logs

## 📋 Attack Information
**Category**: Cryptographic Practice Error | **Severity**: 🔴 Critical (CVSS 9.0)

## 🎯 Exposure Vectors

### 1. Git Repository
```bash
# ❌ Dangerous: Committed to Git
.env
API_SECRET_KEY=sk_live_abc123...
PRIVATE_KEY=0x1234567890abcdef...

# Even if deleted, git history preserves it
git log --all --full-history -- .env
```

### 2. Log Files
```typescript
// ❌ Dangerous: Logging sensitive info
console.log('Signing with key:', privateKey);
logger.info(`API Key: ${apiKey}`);

// Logs stored/uploaded
// Attacker gets logs → Keys leaked
```

### 3. Error Messages
```javascript
try {
  decrypt(data, secretKey);
} catch (error) {
  // ❌ Error contains key
  throw new Error(`Decryption failed with key ${secretKey}: ${error}`);
}
```

## 🛡️ Defense Measures

### 1. Never Commit Keys
```bash
# .gitignore
.env
.env.*
*.key
secrets/
config/local.json
```

### 2. Log Sanitization
```typescript
// ✅ Sanitize logs - hide sensitive fields
function sanitizeLog(obj: any): any {
  const sensitive = ['privateKey', 'secretKey', 'password', 'token', 'mnemonic'];
  
  const result = { ...obj };
  for (const key of sensitive) {
    if (result[key]) {
      result[key] = '***REDACTED***';
    }
  }
  return result;
}

logger.info('User data:', sanitizeLog(user));
```

### 3. Environment Variable Management
```typescript
// ✅ Use secrets management service
import { SecretsManager } from 'aws-sdk';

const secretsManager = new SecretsManager();
const secret = await secretsManager.getSecretValue({
  SecretId: 'prod/wallet/signing-key',
}).promise();

const signingKey = JSON.parse(secret.SecretString).key;
```

### 4. Error Sanitization
```typescript
// ✅ Safe error handling
try {
  await sensitiveOperation(privateKey);
} catch (error) {
  // Don't include sensitive data in error
  logger.error('Operation failed', {
    operation: 'sensitiveOperation',
    timestamp: Date.now(),
    // NO private key here
  });
  
  throw new Error('Operation failed'); // Generic message
}
```

## ✅ Best Practices
```
Development:
□ Use .env files (never commit)
□ Add .env to .gitignore
□ Use environment variables
□ Sanitize all logs
□ Never log sensitive data
□ Use secrets management in production

Production:
□ Use AWS Secrets Manager / Vault
□ Rotate keys regularly
□ Monitor access logs
□ Encrypt logs at rest
□ Limited access to logs
```

**Created**: November 11, 2025

