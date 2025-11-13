# SQL Injection Attack

## 📋 Attack Information
**Category**: Backend Attack | **Severity**: 🔴 High (CVSS 8.5) | **OWASP**: A03:2021 | **CWE**: CWE-89

## 🎯 Attack Principle
Attacker injects malicious SQL code through user input to manipulate database queries.

## 💀 Attack Examples

### Classic SQL Injection
```sql
-- ❌ Vulnerable code
const query = `SELECT * FROM users WHERE email = '${email}' AND password = '${password}'`;

-- Attack input:
email: admin@wallet.com' OR '1'='1
password: anything

-- Resulting query:
SELECT * FROM users WHERE email = 'admin@wallet.com' OR '1'='1' AND password = 'anything'
-- '1'='1' is always true → bypasses authentication
```

### Data Extraction
```sql
-- Attack input:
email: ' UNION SELECT username, password_hash, api_key FROM admin_users--

-- Resulting query extracts admin data
```

### Database Deletion
```sql
-- Attack input:
email: '; DROP TABLE users;--

-- Deletes entire users table!
```

## 🛡️ Defense Measures

### 1. Parameterized Queries (Best)
```typescript
// ✅ SECURE: Use parameterized queries
import { Pool } from 'pg';

const pool = new Pool();

async function login(email: string, password: string) {
  // Parameters separated from SQL
  const result = await pool.query(
    'SELECT * FROM users WHERE email = $1 AND password_hash = $2',
    [email, hashedPassword] // Safe - cannot inject
  );
  
  return result.rows[0];
}
```

### 2. ORM Usage
```typescript
// ✅ Use ORM like TypeORM, Sequelize
import { User } from './entities/User';

async function login(email: string, password: string) {
  const user = await User.findOne({
    where: { email, password_hash: hashedPassword },
  });
  
  return user;
}

// ORM handles SQL safely
```

### 3. Input Validation
```typescript
/**
 * Validate inputs before database operations
 */
function validateEmail(email: string): boolean {
  // Strict email regex
  const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  
  if (!emailRegex.test(email)) {
    throw new Error('Invalid email format');
  }
  
  // Length limit
  if (email.length > 255) {
    throw new Error('Email too long');
  }
  
  return true;
}
```

### 4. Rust Backend (Type-Safe)
```rust
// ✅ Rust with sqlx - compile-time checked queries
use sqlx::{PgPool, FromRow};

#[derive(FromRow)]
struct User {
    id: i32,
    email: String,
}

async fn get_user(pool: &PgPool, email: &str) -> Result<User, sqlx::Error> {
    // Parameterized query - SQL injection impossible
    let user = sqlx::query_as::<_, User>(
        "SELECT id, email FROM users WHERE email = $1"
    )
    .bind(email)
    .fetch_one(pool)
    .await?;
    
    Ok(user)
}
```

### 5. Least Privilege
```sql
-- ✅ Database user has minimal permissions
CREATE USER wallet_app WITH PASSWORD 'secure_password';

-- Only grant necessary permissions
GRANT SELECT, INSERT, UPDATE ON users TO wallet_app;
GRANT SELECT, INSERT ON transactions TO wallet_app;

-- No DROP, DELETE on critical tables
-- No access to admin tables
```

### 6. Web Application Firewall (WAF)
```typescript
// ✅ Use WAF to detect SQL injection attempts
// Cloudflare, AWS WAF, etc.

// Block patterns like:
// - ' OR '1'='1
// - UNION SELECT
// - DROP TABLE
// - --, /* */ (SQL comments)
```

## 💰 Latest Attack Cases (2024-2025)

### 2024 SQL Injection Vulnerability Statistics
According to 360 Threat Intelligence Center, **SQL injection vulnerabilities reached 3,293 cases in 2024, accounting for 10% of total vulnerabilities**, demonstrating this traditional attack method remains widespread.

**Key Characteristics:**
- Proliferation of automated scanning tools lowering attack barriers
- Advanced injection techniques targeting ORM frameworks
- AI-generated polymorphic payloads bypassing WAF detection with 40% higher success rate

### AI-Driven SQL Injection Attacks (2025)
Research shows that AI-powered automated exploitation tools can complete **attack code generation, testing, and deployment in an average of 4 minutes 37 seconds** after discovering SQL injection vulnerabilities, improving efficiency by nearly 200x.

**Novel Attack Methods:**
```sql
-- AI-generated adversarial SQL injection (AdvSQLi)
-- Specifically designed to bypass WAF detection
email: admin'/**/UN%49ON/**/SEL%45CT/**/username,password/**/FROM/**/users--

-- Uses Unicode obfuscation and comment splitting
-- Successfully bypasses mainstream WAFs (Cloudflare, AWS WAF, etc.)
```

### Defense Against AI-Driven Attacks
```typescript
// ✅ AI-based detection system
import { AnomalyDetector } from 'ml-security';

const detector = new AnomalyDetector({
  model: 'sql-injection-detection-v2',
  threshold: 0.85,
});

app.use(async (req, res, next) => {
  const isAnomaly = await detector.predict({
    query: req.body,
    headers: req.headers,
    pattern: req.url,
  });
  
  if (isAnomaly.score > 0.85) {
    logger.warn('AI detected suspicious SQL injection attempt', { 
      ip: req.ip, 
      score: isAnomaly.score 
    });
    return res.status(403).json({ error: 'Request rejected' });
  }
  
  next();
});
```

## ✅ Best Practices

```
Input Handling:
□ Use parameterized queries (ALWAYS)
□ Use ORM with parameter binding
□ Validate all user inputs
□ Whitelist allowed characters
□ Limit input lengths

Database:
□ Use least privilege accounts
□ Separate read/write users
□ Disable dangerous functions
□ Regular security updates
□ Monitor for suspicious queries

AI-Enhanced Defense (2025):
□ Deploy AI-based anomaly detection
□ Use machine learning WAF rules
□ Monitor for AI-generated attack patterns
□ Real-time threat intelligence integration

Testing:
□ Test with SQL injection payloads
□ Use automated scanners (SQLMap)
□ Code review all queries
□ Regular penetration testing
□ Test against AI-generated payloads
```

**Created**: November 11, 2025  
**Last Updated**: November 12, 2025 (Added 2024-2025 latest cases and AI defense)

