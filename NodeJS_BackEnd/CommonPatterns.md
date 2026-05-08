
Common Patterns irrespective of NodeJS Server Package or frameowkr

# Auth Strategy

## Authentication Best Practices

| Feature            | JWT      | Session      | API Key    |
| ------------------ | -------- | ------------ | ---------- |
| Stateless          | YES      | NO           | Usually NO |
| Server Storage     | Optional | Required     | Required   |
| Browser Friendly   | YES      | BEST         | NO         |
| Mobile Friendly    | BEST     | OK           | NO         |
| Revocation         | Harder   | Easy         | Easy       |
| Scaling            | Easy     | Redis needed | Easy       |
| CSRF Risk          | Low      | Higher       | Low        |
| B2B Integrations   | OK       | Bad          | BEST       |
| Third-party Access | Weak     | Bad          | BEST       |

Use:
- JWT for SPA/mobile auth
- Session cookies for SSR/admin apps
- API keys for integrations
This is the architecture many production systems use today.

### JWT Strategy
JWT (stateless): Generate a signed JWT on login, validate it on every request via middleware. No database lookup per request — scales well. Logout requires token blocklisting (Redis) or short expiry + refresh tokens.
#### Access Token
- short expiry (15m)
#### Refresh Token
- longer expiry
- stored securely
- rotate tokens

Recommended Packages
| Package        | Purpose                 |
| -------------- | ----------------------- |
| `jsonwebtoken` | JWT                     |
| `bcrypt`       | password hashing        |
| `passport`     | optional auth framework |
| `passport-jwt` | JWT strategy            |

- JWT Utils
```typescript
import jwt from "jsonwebtoken";

const JWT_SECRET =
  process.env.JWT_SECRET!;

export const signAccessToken = (
  payload
) => {
  return jwt.sign(
    payload,
    JWT_SECRET,
    {
      expiresIn: "15m",
    }
  );
};

export const verifyAccessToken = (
  token
) => {
  return jwt.verify(
    token,
    JWT_SECRET
  );
};
```

- JWT Middleware
```typescript
import jwt from "jsonwebtoken";

export const requireAuth = (
  req,
  res,
  next
) => {
  try {
    const authHeader =
      req.headers.authorization;

    if (
      !authHeader?.startsWith(
        "Bearer "
      )
    ) {
      return res.status(401).json({
        message: "Missing token",
      });
    }

    const token =
      authHeader.split(" ")[1];

    const decoded = jwt.verify(
      token,
      process.env.JWT_SECRET!
    );

    req.user = decoded;

    next();
  } catch {
    return res.status(401).json({
      message:
        "Invalid or expired token",
    });
  }
};
```
- Middleware
```typescript
export const apiKeyAuth = async (
  req,
  res,
  next
) => {
  const apiKey =
    req.header("x-api-key");

  if (!apiKey) {
    return res.status(401).json({
      message: "Missing API key",
    });
  }

  const key = await findApiKey(
    apiKey
  );

  if (!key) {
    return res.status(401).json({
      message: "Invalid API key",
    });
  }

  req.account = key.account;

  next();
};
```
### Session Cookie
Session-based (stateful): Store session data server-side (Redis or DB), send a session ID cookie. Supports instant revocation. Requires session store; more complex with multiple API instances.

#### How it works

1. User logs in
2. Server creates a session
3. Session stored in:
    - Redis
    - database
    - memory (dev only)
4. Server sends:
```Set-Cookie: sid=abc123```
5. Browser automatically sends cookie on every request
6. Server validates session ID

**Best Use Cases**

*Good for:*
- traditional web apps
- admin panels
- SSR apps
- dashboards
- enterprise systems

*Not ideal for:*
- public APIs
- mobile APIs
- microservices

Recommended Packages
| Package           | Purpose                    |
| ----------------- | -------------------------- |
| `express-session` | session middleware         |
| `connect-redis`   | Redis session store        |
| `redis`           | Redis client               |
| `cookie-parser`   | cookie handling            |
| `helmet`          | security headers           |
| `csurf`           | CSRF protection (optional) |

Reccommended Architecture
```
Browser
   ↓
Cookie: sid=abc123
   ↓
Express Server
   ↓
Redis Session Store
```

#### Basic Setup

Redis Client
```typescript
import { createClient } from "redis";

export const redisClient = createClient({
  url: process.env.REDIS_URL,
});

await redisClient.connect();
```
Session Config
```typescript
import session from "express-session";
import RedisStore from "connect-redis";

app.use(
  session({
    store: new RedisStore({
      client: redisClient,
    }),

    secret: process.env.SESSION_SECRET!,

    resave: false,
    saveUninitialized: false,

    cookie: {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      maxAge: 1000 * 60 * 60 * 24 * 7,
    },
  })
);
```

Login Example
```typescript
app.post("/login", async (req, res) => {
  const user = await validateUser(req.body);

  if (!user) {
    return res.status(401).json({
      message: "Invalid credentials",
    });
  }

  req.session.userId = user.id;

  res.json({
    success: true,
  });
});
```

Protected Route
```typescript
const requireAuth = (req, res, next) => {
  if (!req.session.userId) {
    return res.status(401).json({
      message: "Unauthorized",
    });
  }

  next();
};
```

Logout
Instant revocation works because sessions are stored server-side.
```typescript
app.post("/logout", (req, res) => {
  req.session.destroy(() => {
    res.clearCookie("connect.sid");

    res.json({
      success: true,
    });
  });
});
```
Always use one of Redis , Mongodb or any SQL for session storage , never use in memory store

#### Use CSRF protection
Cookies are vulnerable to CSRF.
Options:
- csrf
- SameSite=strict
- CSRF token strategy
- Generate CSRF
```typescript
import crypto from "crypto";

export const generateCsrfToken = (
  req,
  res,
  next
) => {
  const token = crypto.randomBytes(32).toString("hex");

  res.cookie("csrf_token", token, {
    httpOnly: false,
    secure: process.env.NODE_ENV === "production",
    sameSite: "strict",
  });

  res.locals.csrfToken = token;

  next();
};
```
- Validate CSRF
```typescript
export const validateCsrfToken = (
  req,
  res,
  next
) => {
  const cookieToken =
    req.cookies.csrf_token;

  const headerToken =
    req.headers["x-csrf-token"];

  if (
    !cookieToken ||
    cookieToken !== headerToken
  ) {
    return res.status(403).json({
      message: "Invalid CSRF token",
    });
  }

  next();
};
```
Usage:
```typescript
app.get(
  "/csrf-token",
  generateCsrfToken,
  (req, res) => {
    res.json({
      csrfToken: res.locals.csrfToken,
    });
  }
);

app.post(
  "/profile",
  validateCsrfToken,
  controller
);
```
#### Session Scaling
Redis is standard for multiple express server instances
```
Load Balancer
    ↓
Multiple Express Instances
    ↓
Shared Redis Session Store
```

### API Key Header
API keys (for B2B): Generate long-lived keys per user/team, store hashed in the database, validate on each request. Best for programmatic API access — Zapier integrations, CLI tools, third-party apps.

How it works:
client sends:  ```x-api-key: api_key_live_xxxxx```
Server:
1. hashes key
2. compares against DB
3. loads associated account/team

#### Best Use Cases

*Good for:*

1. third-party integrations
2. CLI tools
3. server-to-server APIs
4. internal tools
5. webhooks
6. partner APIs

*Not ideal for:*
1. browsers
2. user login systems

Recommended Packages:
| Package              | Purpose               |
| -------------------- | --------------------- |
| `crypto`             | secure key generation |
| `bcrypt` or `argon2` | hashing               |
| `express-rate-limit` | abuse prevention      |
| `helmet`             | security              |
| `redis`              | rate limiting/cache   |

- Auth Middleware
```typescript
const apiKeyAuth = async (req, res, next) => {
  const apiKey = req.header("x-api-key");

  if (!apiKey) {
    return res.status(401).json({
      message: "Missing API key",
    });
  }

  const keyRecord = await findKey(apiKey);

  if (!keyRecord) {
    return res.status(401).json({
      message: "Invalid API key",
    });
  }

  req.account = keyRecord.account;

  next();
};
```
- Add Permissions/Scopes
 ```json
 {
  "scopes": [
    "users.read",
    "payments.write"
  ]
}
 ```
- Add expiration: ```expires_at```
- Add Rate Limiting: 1. Redis , 2. express-rate-limit packages
- Rotate Keys:  
    Allow:
    - multiple active keys
    - revocation
    - regeneration

# Database Packages

## SQL Databse Package Comparisons
I personally recommend [`MikroORM`]((https://mikro-orm.io/)) for enterprise grade apps and [`DrizzleORM`](https://orm.drizzle.team/) for POCs , small and internal tool apps. Anyway leaving below the comparison for you to choose one among them.
 | ORM         | Performance | Flexibility    | Type Safety | Learning Curve | Best For                      |
| ----------- | ----------- | -------------- | ----------- | -------------- | ----------------------------- |
| [MikroORM](https://mikro-orm.io/)    | High        | Very High      | Excellent   | Medium         | Complex domain models         |
| [Drizzle ORM](https://orm.drizzle.team/) | Very High   | Extremely High | Excellent   | Medium         | Performance-first modern apps |
| [Prisma](https://www.prisma.io/)      | Medium–High | Medium         | Excellent   | Easy           | Fast product development      |
| [TypeORM](https://typeorm.io/)     | Medium–Low  | High           | Moderate    | Medium         | Legacy enterprise apps        |

## NoSQL Databse Recommendataion Package
 [`mongoose`](https://mongoosejs.com/docs/index.html) - The only best package even recommended by MongoDB development team itself. Widely used for complex applications requiring validation, schema definition, and complex data modeling.
