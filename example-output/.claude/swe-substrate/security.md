# Segurança

**Gerado por**: `/swe-substrate-security`
**Data**: 2025-10-29

## Autenticação

### JWT (JSON Web Tokens)

**Biblioteca**: jsonwebtoken
**Algoritmo**: RS256 (asymmetric signing)

#### Token Structure

```typescript
// Access Token (15 min expiry)
{
  "sub": "user_123",
  "email": "user@example.com",
  "roles": ["customer"],
  "permissions": ["cart:write", "orders:read"],
  "type": "access",
  "iat": 1698765432,
  "exp": 1698766332
}

// Refresh Token (7 days expiry)
{
  "sub": "user_123",
  "type": "refresh",
  "jti": "refresh_abc123", // Unique token ID
  "iat": 1698765432,
  "exp": 1699370232
}
```

#### Token Generation

```typescript
import jwt from 'jsonwebtoken';
import fs from 'fs';

const privateKey = fs.readFileSync('private-key.pem');
const publicKey = fs.readFileSync('public-key.pem');

function generateAccessToken(user: User): string {
  return jwt.sign(
    {
      sub: user.id,
      email: user.email,
      roles: user.roles,
      permissions: user.permissions,
      type: 'access',
    },
    privateKey,
    {
      algorithm: 'RS256',
      expiresIn: '15m',
    }
  );
}

function generateRefreshToken(user: User): string {
  const tokenId = generateUUID();

  // Store refresh token ID in database for revocation
  await storeRefreshToken(user.id, tokenId);

  return jwt.sign(
    {
      sub: user.id,
      type: 'refresh',
      jti: tokenId,
    },
    privateKey,
    {
      algorithm: 'RS256',
      expiresIn: '7d',
    }
  );
}
```

#### Token Verification

```typescript
// Middleware for protected routes
async function authenticateToken(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  const token = authHeader?.split(' ')[1]; // Bearer TOKEN

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const decoded = jwt.verify(token, publicKey, {
      algorithms: ['RS256'],
    }) as JWTPayload;

    if (decoded.type !== 'access') {
      return res.status(401).json({ error: 'Invalid token type' });
    }

    req.user = decoded;
    next();
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(403).json({ error: 'Invalid token' });
  }
}
```

#### Token Refresh Flow

```typescript
app.post('/api/v1/auth/refresh', async (req, res) => {
  const { refreshToken } = req.body;

  try {
    const decoded = jwt.verify(refreshToken, publicKey) as JWTPayload;

    // Check if refresh token is revoked
    const isValid = await checkRefreshToken(decoded.sub, decoded.jti);
    if (!isValid) {
      return res.status(401).json({ error: 'Refresh token revoked' });
    }

    // Generate new access token
    const user = await getUserById(decoded.sub);
    const newAccessToken = generateAccessToken(user);

    res.json({ accessToken: newAccessToken });
  } catch (error) {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});
```

### OAuth 2.0 / Social Login

**Providers**: Google, Facebook, Apple

```typescript
import { OAuth2Client } from 'google-auth-library';

const googleClient = new OAuth2Client(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  process.env.GOOGLE_REDIRECT_URI
);

// Google OAuth flow
app.get('/auth/google', (req, res) => {
  const authUrl = googleClient.generateAuthUrl({
    access_type: 'offline',
    scope: ['profile', 'email'],
  });
  res.redirect(authUrl);
});

app.get('/auth/google/callback', async (req, res) => {
  const { code } = req.query;

  const { tokens } = await googleClient.getToken(code);
  const ticket = await googleClient.verifyIdToken({
    idToken: tokens.id_token,
    audience: process.env.GOOGLE_CLIENT_ID,
  });

  const payload = ticket.getPayload();

  // Find or create user
  let user = await findUserByEmail(payload.email);
  if (!user) {
    user = await createUser({
      email: payload.email,
      name: payload.name,
      provider: 'google',
      providerId: payload.sub,
    });
  }

  // Generate our tokens
  const accessToken = generateAccessToken(user);
  const refreshToken = generateRefreshToken(user);

  res.json({ accessToken, refreshToken });
});
```

### Password Hashing

**Biblioteca**: bcrypt
**Cost Factor**: 12 (2^12 iterations)

```typescript
import bcrypt from 'bcrypt';

// Hash password on registration
async function hashPassword(password: string): Promise<string> {
  const salt = await bcrypt.genSalt(12);
  return bcrypt.hash(password, salt);
}

// Verify password on login
async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

// Password requirements
function validatePassword(password: string): boolean {
  return (
    password.length >= 12 &&
    /[A-Z]/.test(password) &&  // At least one uppercase
    /[a-z]/.test(password) &&  // At least one lowercase
    /[0-9]/.test(password) &&  // At least one digit
    /[^A-Za-z0-9]/.test(password) // At least one special char
  );
}
```

### Multi-Factor Authentication (MFA)

**Type**: TOTP (Time-based One-Time Password)
**Library**: speakeasy

```typescript
import speakeasy from 'speakeasy';
import qrcode from 'qrcode';

// Enable MFA for user
async function enableMFA(userId: string) {
  const secret = speakeasy.generateSecret({
    name: `ShopFlow (${user.email})`,
    length: 32,
  });

  // Store secret in database (encrypted)
  await storeUserMFASecret(userId, secret.base32);

  // Generate QR code for user to scan
  const qrCodeUrl = await qrcode.toDataURL(secret.otpauth_url);

  return {
    secret: secret.base32,
    qrCode: qrCodeUrl,
  };
}

// Verify MFA token
async function verifyMFAToken(userId: string, token: string): Promise<boolean> {
  const secret = await getUserMFASecret(userId);

  return speakeasy.totp.verify({
    secret,
    encoding: 'base32',
    token,
    window: 1, // Allow 1 step before/after for clock skew
  });
}
```

## Autorização

### Role-Based Access Control (RBAC)

#### Roles

- **guest** - Unauthenticated user
- **customer** - Regular customer
- **premium_customer** - Premium subscription
- **admin** - Administrative access
- **super_admin** - Full system access

#### Permissions Matrix

```typescript
const PERMISSIONS = {
  // Products
  'products:read': ['guest', 'customer', 'premium_customer', 'admin', 'super_admin'],
  'products:write': ['admin', 'super_admin'],
  'products:delete': ['super_admin'],

  // Orders
  'orders:read:own': ['customer', 'premium_customer'],
  'orders:read:all': ['admin', 'super_admin'],
  'orders:write': ['customer', 'premium_customer', 'admin', 'super_admin'],
  'orders:cancel:own': ['customer', 'premium_customer'],
  'orders:cancel:all': ['admin', 'super_admin'],

  // Admin
  'admin:access': ['admin', 'super_admin'],
  'admin:users:manage': ['super_admin'],
};
```

#### Authorization Middleware

```typescript
function requirePermission(permission: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;

    if (!user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }

    const hasPermission = user.permissions?.includes(permission);
    if (!hasPermission) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }

    next();
  };
}

// Usage
app.get('/api/v1/admin/users',
  authenticateToken,
  requirePermission('admin:users:manage'),
  async (req, res) => {
    // Handler
  }
);
```

#### Resource-Level Authorization

```typescript
// Check if user can access specific order
async function canAccessOrder(userId: string, orderId: string): Promise<boolean> {
  const order = await getOrder(orderId);

  // User can access their own orders
  if (order.customerId === userId) {
    return true;
  }

  // Admins can access all orders
  const user = await getUser(userId);
  if (user.roles.includes('admin')) {
    return true;
  }

  return false;
}

app.get('/api/v1/orders/:orderId', authenticateToken, async (req, res) => {
  const { orderId } = req.params;

  const canAccess = await canAccessOrder(req.user.sub, orderId);
  if (!canAccess) {
    return res.status(403).json({ error: 'Cannot access this order' });
  }

  const order = await getOrder(orderId);
  res.json({ data: order });
});
```

## Secrets Management

**Tool**: AWS Secrets Manager

### Secret Rotation

```typescript
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

const secretsClient = new SecretsManagerClient({ region: 'us-east-1' });

async function getSecret(secretName: string): Promise<string> {
  const command = new GetSecretValueCommand({ SecretId: secretName });
  const response = await secretsClient.send(command);
  return response.SecretString;
}

// Database credentials with auto-rotation
const dbPassword = await getSecret('prod/database/password');

// API keys
const stripeKey = await getSecret('prod/stripe/api-key');
```

### Environment Variables

```bash
# .env (never commit to git)
NODE_ENV=production
DATABASE_URL=postgresql://user:password@host:5432/db
REDIS_URL=redis://host:6379
JWT_PRIVATE_KEY_PATH=/secrets/jwt-private-key.pem
JWT_PUBLIC_KEY_PATH=/secrets/jwt-public-key.pem
STRIPE_SECRET_KEY=${aws:secretsmanager:prod/stripe/api-key}
```

**Security Rules**:
- Never hardcode secrets in code
- Use AWS Secrets Manager or similar for sensitive data
- Rotate secrets regularly (90 days)
- Different secrets for each environment

## TLS/SSL

### Certificate Management

**Provider**: AWS Certificate Manager (ACM)
**Renewal**: Automatic (ACM handles renewal)

### Enforce HTTPS

```typescript
// Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (req.header('x-forwarded-proto') !== 'https' && process.env.NODE_ENV === 'production') {
    res.redirect(`https://${req.header('host')}${req.url}`);
  } else {
    next();
  }
});

// HSTS header
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  next();
});
```

### mTLS for Service-to-Service

```typescript
// Microservice communication with mutual TLS
import https from 'https';
import fs from 'fs';

const agent = new https.Agent({
  cert: fs.readFileSync('client-cert.pem'),
  key: fs.readFileSync('client-key.pem'),
  ca: fs.readFileSync('ca-cert.pem'),
});

// Make request to another service
const response = await fetch('https://inventory-service:8443/api/check-stock', {
  agent,
});
```

## Input Validation & Sanitization

**Library**: zod

```typescript
import { z } from 'zod';

// Define schemas
const CreateOrderSchema = z.object({
  items: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive().max(99),
    customization: z.object({
      text: z.string().max(50).optional(),
      color: z.enum(['red', 'blue', 'green']).optional(),
    }).optional(),
  })).min(1).max(50),
  shippingAddress: z.object({
    street: z.string().min(5).max(200),
    city: z.string().min(2).max(100),
    state: z.string().length(2),
    zipCode: z.string().regex(/^\d{5}-\d{3}$/),
  }),
  paymentMethod: z.enum(['credit_card', 'paypal', 'bank_slip']),
});

// Validate input
app.post('/api/v1/orders', authenticateToken, async (req, res) => {
  try {
    const validatedData = CreateOrderSchema.parse(req.body);

    const order = await createOrder(validatedData);
    res.status(201).json({ data: order });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.errors,
      });
    }
    throw error;
  }
});
```

### SQL Injection Prevention

```typescript
// Good - Parameterized queries
await db.query('SELECT * FROM products WHERE id = $1', [productId]);

// Bad - String concatenation
await db.query(`SELECT * FROM products WHERE id = '${productId}'`);
```

### XSS Prevention

```typescript
import DOMPurify from 'isomorphic-dompurify';

// Sanitize user-generated content
function sanitizeHTML(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href'],
  });
}

// CSP headers
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.stripe.com; style-src 'self' 'unsafe-inline'"
  );
  next();
});
```

## Rate Limiting & DDoS Protection

### Application-Level Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';

// General API rate limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

// Stricter limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true, // Only count failed attempts
});

app.use('/api/', apiLimiter);
app.use('/api/v1/auth', authLimiter);
```

### Infrastructure-Level Protection

- **AWS WAF**: Block malicious IPs, SQL injection, XSS attempts
- **CloudFlare**: DDoS protection, rate limiting, bot detection
- **API Gateway**: Throttling at 10,000 requests per second

## Security Headers

```typescript
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", 'https://cdn.stripe.com'],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https://cdn.shopflow.com'],
      connectSrc: ["'self'", 'https://api.stripe.com'],
      frameSrc: ["'none'"],
      objectSrc: ["'none'"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  xssFilter: true,
  noSniff: true,
  ieNoOpen: true,
  hidePoweredBy: true,
}));
```

## Security Auditing

### Dependency Scanning

```bash
# npm audit
npm audit --audit-level=moderate

# Snyk
snyk test
snyk monitor
```

### Penetration Testing

- **Frequency**: Quarterly
- **Scope**: All public-facing APIs and web applications
- **Provider**: External security firm

### Security Monitoring

```typescript
// Log suspicious activities
logger.warn('Suspicious login attempt', {
  userId,
  ip: req.ip,
  userAgent: req.headers['user-agent'],
  failedAttempts: loginAttempts,
});

// Alert on multiple failed login attempts
if (loginAttempts >= 5) {
  await notifySecurityTeam({
    type: 'multiple_failed_logins',
    userId,
    ip: req.ip,
  });

  // Temporarily block IP
  await blockIP(req.ip, { duration: '1h' });
}
```

## Compliance

### PCI DSS

- Never store full credit card numbers
- Use tokenization (Stripe tokens)
- Log all access to cardholder data
- Encrypt data at rest and in transit

### GDPR

- Right to access: API endpoint for data export
- Right to erasure: User data anonymization
- Consent management: Explicit opt-in for marketing
- Data portability: JSON export of user data

### LGPD (Brazil)

- Data processing transparency
- User consent for data collection
- Data protection officer designated
