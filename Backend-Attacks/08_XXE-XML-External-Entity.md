# XXE (XML External Entity) Attack

## 📋 Attack Information
**Category**: Backend Attack | **Severity**: 🔴 High (CVSS 8.0) | **CWE**: CWE-611

## 🎯 Attack Principle
Exploit XML parsers that process external entities to read files, perform SSRF, or cause DoS.

## 💀 Attack Example

```xml
<!--? xml version="1.0"?-->
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<user>
  <name>&xxe;</name>
</user>

<!-- Server parses XML and includes /etc/passwd contents -->
```

### File Reading
```xml
<!ENTITY xxe SYSTEM "file:///app/config/database.yml">
<data>&xxe;</data>

<!-- Exposes database credentials -->
```

### SSRF via XXE
```xml
<!ENTITY xxe SYSTEM "http://internal-api/secret">
<data>&xxe;</data>

<!-- Access internal network resources -->
```

## 🛡️ Defense Measures

### 1. Disable External Entities
```javascript
// ✅ Secure XML parsing
const libxmljs = require('libxmljs');

const xml = libxmljs.parseXml(xmlString, {
  noent: false,   // ✅ Disable entity substitution
  nonet: true,    // ✅ Disable network access
  nocdata: true,
});
```

### 2. Use JSON Instead
```typescript
// ✅ Prefer JSON over XML
// JSON doesn't have entity concepts
app.post('/api/data', express.json(), async (req, res) => {
  // req.body is safely parsed JSON
  const data = req.body;
});
```

### 3. Input Validation
```typescript
// If must use XML, validate strictly
function validateXML(xml: string): boolean {
  // Reject if contains entity declarations
  if (xml.includes('<!ENTITY') || xml.includes('<!DOCTYPE')) {
    throw new Error('External entities not allowed');
  }
  
  return true;
}
```

## ✅ Best Practices

```
□ Disable XML external entity processing
□ Use JSON instead of XML when possible
□ Validate XML input
□ Use secure XML parsers
□ Update XML libraries regularly
```

**Created**: November 11, 2025

