# Insecure File Upload Attack

## 📋 Attack Information
**Category**: Backend Attack | **Severity**: 🔴 High (CVSS 8.5) | **CWE**: CWE-434

## 🎯 Attack Principle
Upload malicious files (web shells, viruses, oversized files) to execute code or cause DoS.

## 💀 Attack Scenarios

### 1. Web Shell Upload
```php
// Attacker uploads avatar.php
<?php system($_GET['cmd']); ?>

// Visits: https://wallet.com/uploads/avatar.php?cmd=cat /etc/passwd
// Server executes command
```

### 2. Path Traversal
```
Upload filename: ../../etc/passwd
Save path: /var/www/uploads/../../etc/passwd
→ Overwrites system files
```

## 🛡️ Defense Measures

### 1. File Type Validation
```typescript
const ALLOWED_TYPES = ['image/jpeg', 'image/png'];
const ALLOWED_EXTENSIONS = ['.jpg', '.jpeg', '.png'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

async function validateFile(file: File): Promise<boolean> {
  // 1. Check MIME type
  if (!ALLOWED_TYPES.includes(file.type)) {
    throw new Error('Invalid file type');
  }
  
  // 2. Check extension
  const ext = file.name.toLowerCase().match(/\.[^.]+$/)?.[0];
  if (!ext || !ALLOWED_EXTENSIONS.includes(ext)) {
    throw new Error('Invalid file extension');
  }
  
  // 3. Check size
  if (file.size > MAX_SIZE) {
    throw new Error('File too large');
  }
  
  // 4. Check magic bytes
  const header = await readFileHeader(file);
  const validHeaders = {
    jpg: [0xFF, 0xD8, 0xFF],
    png: [0x89, 0x50, 0x4E, 0x47],
  };
  
  return Object.values(validHeaders).some(h => 
    h.every((byte, i) => header[i] === byte)
  );
}
```

### 2. Safe Filename
```rust
// ✅ Sanitize filename
use std::path::Path;
use uuid::Uuid;

fn sanitize_filename(name: &str) -> String {
    // 1. Remove path
    let name = Path::new(name)
        .file_name()
        .and_then(|n| n.to_str())
        .unwrap_or("unknown");
    
    // 2. Only keep safe characters
    let safe: String = name.chars()
        .filter(|c| c.is_alphanumeric() || *c == '.' || *c == '-')
        .collect();
    
    // 3. Generate unique name
    let uuid = Uuid::new_v4();
    format!("{}_{}", uuid, safe)
}
```

### 3. Store on CDN/Object Storage
```typescript
// ✅ Upload to S3/CDN, not app server
import AWS from 'aws-sdk';

const s3 = new AWS.S3();

async function uploadFile(file: File): Promise<string> {
  const filename = sanitizeFilename(file.name);
  
  await s3.upload({
    Bucket: 'rustwallet-uploads',
    Key: `avatars/${filename}`,
    Body: file,
    ACL: 'public-read',
    ContentType: file.type,
  }).promise();
  
  return `https://cdn.rustwallet.com/avatars/${filename}`;
}
```

### 4. Virus Scanning
```typescript
// ✅ Scan uploaded files
import ClamScan from 'clamscan';

const clam = await new ClamScan().init();

const { isInfected, viruses } = await clam.scanFile(filePath);

if (isInfected) {
  await fs.unlink(filePath);
  throw new Error('Infected file detected');
}
```

## ✅ Best Practices

```
□ Validate file type (MIME + extension + magic bytes)
□ Limit file size
□ Sanitize filenames
□ Store on CDN, not app server
□ Never execute uploaded files
□ Scan for viruses
□ Set proper permissions
```

**Created**: November 11, 2025

