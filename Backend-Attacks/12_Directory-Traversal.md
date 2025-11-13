# Directory Traversal Attack

## 📋 Attack Information
**Category**: Backend Attack | **Severity**: 🔴 High (CVSS 8.0) | **CWE**: CWE-22

## 🎯 Attack Principle
Use `../` to access files outside intended directory.

## 💀 Attack Example

```
Normal: GET /api/file?name=report.pdf
→ /uploads/report.pdf

Attack: GET /api/file?name=../../etc/passwd
→ /uploads/../../etc/passwd = /etc/passwd
```

## 🛡️ Defense

### Rust Implementation
```rust
use std::path::{Path, PathBuf};

fn sanitize_path(filename: &str, base_dir: &str) -> Result<PathBuf, Error> {
    // 1. Extract filename only
    let filename = Path::new(filename)
        .file_name()
        .ok_or(Error::InvalidPath)?
        .to_str()
        .ok_or(Error::InvalidPath)?;
    
    // 2. Validate characters
    if !filename.chars().all(|c| c.is_alphanumeric() || c == '.' || c == '-') {
        return Err(Error::InvalidCharacters);
    }
    
    // 3. Construct safe path
    let mut safe_path = PathBuf::from(base_dir);
    safe_path.push(filename);
    
    // 4. Ensure within base_dir
    let canonical = safe_path.canonicalize()?;
    if !canonical.starts_with(base_dir) {
        return Err(Error::PathTraversal);
    }
    
    Ok(canonical)
}
```

## ✅ Best Practices

```
□ Extract filename only (no paths)
□ Validate characters (alphanumeric + .-_)
□ Use whitelist of allowed files
□ Verify canonical path is within base directory
□ Never directly use user input in file paths
```

**Created**: November 11, 2025

