Recommended Project Structure
```
    src/
    │
    ├── config/
    ├── controllers/
    ├── services/
    ├── repositories/
    ├── routes/
    ├── middlewares/
    ├── validators/
    ├── utils/
    ├── jobs/
    ├── db/
    ├── types/
    ├── app.ts
    └── server.ts
```

| Layer        | Responsibility          |
| ------------ | ----------------------- |
| routes       | define endpoints        |
| controllers  | handle req/res          |
| services     | business logic          |
| repositories | DB queries              |
| middlewares  | auth, logging, errors   |
| validators   | request validation      |
| config       | env, app configs        |
| jobs         | cron/background workers |


| Package                           | Purpose               |
| --------------------------------- | --------------------- |
| `express`                         | HTTP server           |
| `typescript`                      | type safety           |
| `dotenv`                          | environment variables |
| `cors`                            | CORS handling         |
| `helmet`                          | security headers      |
| `compression`                     | gzip compression      |
| `morgan` or `pino-http`           | logging               |
| `cookie-parser`                   | cookie handling       |
| `express-rate-limit`              | rate limiting         |
| `zod` or `joi`                    | validation            |
| `jsonwebtoken`                    | JWT auth              |
| `bcrypt`                          | password hashing      |
| `mongoose` / `prisma` / `drizzle` | DB layer              |
| `winston` / `pino`                | production logging    |

helmet, compression, proper logging, and rate limiting are considered standard production practices.

Recommending the Best Validation Package: [`Zod`](https://zod.dev/)
- TypeScript-first
- schema inference
- reusable DTOs
- frontend/backend sharing

Example:
 ```typescript
 import { z } from "zod";

export const createUserSchema = z.object({
  name: z.string().min(3),
  email: z.email(),
  password: z.string().min(8),
});
```
Middleware:
```typescript
const validate =
  (schema) => (req, res, next) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json(result.error);
    }

    req.body = result.data;
    next();
  };
```

Error Handling (Very Important):<br/>
Never repeat try/catch everywhere.

async handler
```typescript
export const asyncHandler =
  (fn) => (req, res, next) =>
    Promise.resolve(fn(req, res, next)).catch(next);
```

Global error middleware
```typescript
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({
    success: false,
    message: err.message,
  });
});
```
Custom Error Class
```typescript
// utils/AppError.ts — custom error class
export class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}

// middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { AppError } from '../utils/AppError';

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.code,
    });
  }

  // Prisma/ORM errors
  if (err.name === 'PrismaClientKnownRequestError') {
    return res.status(400).json({ error: 'Database constraint violation' });
  }

  // Unknown errors — don't expose internals
  console.error('Unhandled error:', err);
  res.status(500).json({ error: 'Internal server error' });
}
```



## Security Best Practices
### Essential Middleware
```typescript
app.use(helmet());
app.use(cors());
app.use(compression());
app.use(express.json());
app.use(cookieParser());
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));
app.use(pinoHttp());
```

Important Rules:
- never trust req.body
- validate everything
- hash passwords
- sanitize inputs
- avoid exposing stack traces
- use HTTPS
- store secrets in env vars

Environment Management:
```env
NODE_ENV=production
PORT=5000
DATABASE_URL=
JWT_SECRET=
```
Never Hardcode Secrets
Production configs should always set NODE_ENV=production.

## API Best Practices

- Use Versioning
```/api/v1/users```
- Consistent Response shape
```json
{
  "success": true,
  "data": {},
  "message": "User created"
}
```
- Pagination
```GET /users?page=1&limit=10```

## Testing Stack
I prefer [`Jest`](https://jestjs.io/) for converntional way and [`cypress`](cypress.io) for most modern end to end testing
| Package     | Usage              |
| ----------- | ------------------ |
| [`cypress`](https://www.npmjs.com/package/cypress) | End to End Testing |
| [`jest`](https://www.npmjs.com/package/jest)      | test runner        |
| [`supertest`](https://www.npmjs.com/package/supertest) | API testing        |
| [`vitest`](https://www.npmjs.com/package/vitest)    | modern alternative |

## Background Jobs and Queues
For emails, notifications, payments, cron jobs.
| Package     | Usage          |
| ----------- | -------------- |
| [`bullmq`](https://www.npmjs.com/package/bullmq)    | Redis queue    |
| [`node-cron`](npmjs.com/package/node-cron) | scheduled jobs |
| [`agenda`](https://www.npmjs.com/package/agenda)    | MongoDB jobs   |

## File Uploads
| Package          | Usage              |
| ---------------- | ------------------ |
| `multer`         | multipart upload   |
| `cloudinary` SDK | cloud storage      |
| `sharp`          | image optimization |

## API Docs
| Package              | Usage      |
| -------------------- | ---------- |
| `swagger-ui-express` | Swagger UI |
| `swagger-jsdoc`      | auto docs  |
| `redoc`              | API docs   |


Setting up Express Typescript API Server from Scratch

npm or pnpm commands
```
[p]npm init -y
[p]npm install express
[p]npm install -D typescript ts-node @types/express @types/node nodemon
[p]npx tsc --init
```

## Sample Server Init File
```typescript
// src/app.ts — Express with TypeScript
import express, { Application, Request, Response, NextFunction } from 'express';
import helmet from 'helmet';
import cors from 'cors';
import rateLimit from 'express-rate-limit';

export function createApp(): Application {
  const app = express();

  // Security middleware
  app.use(helmet());
  app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
    credentials: true,
  }));
  app.use(rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 100,
    standardHeaders: true,
  }));

  // Request parsing
  app.use(express.json({ limit: '10kb' }));
  app.use(express.urlencoded({ extended: true }));

  // Routes
  app.use('/api/v1/users', userRouter);
  app.use('/api/v1/posts', postRouter);

  // Global error handler — must be last
  app.use((err: Error, req: Request, res: Response, _next: NextFunction) => {
    console.error(err.stack);
    res.status(500).json({ error: 'Internal server error' });
  });

  return app;
}

// src/index.ts — start server
import { createApp } from './app';

const app = createApp();
const port = process.env.PORT || 4000;

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```
