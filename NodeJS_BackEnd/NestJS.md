# NestJS Backend (Express Adapter)

## Install

```bash id="x2m3jv"
npm install @nestjs/core @nestjs/common @nestjs/platform-express reflect-metadata rxjs
```

---

# Recommended Core Packages

| Package              | Purpose            |
| -------------------- | ------------------ |
| `@nestjs/config`     | environment config |
| `class-validator`    | DTO validation     |
| `class-transformer`  | DTO transformation |
| `helmet`             | security headers   |
| `compression`        | gzip compression   |
| `cookie-parser`      | cookie handling    |
| `express-rate-limit` | rate limiting      |
| `nestjs-pino`        | logging            |
| `@nestjs/swagger`    | API docs           |
| `cache-manager`      | caching            |
| `@nestjs/schedule`   | cron jobs          |
| `rxjs`               | reactive utilities |

---

# Recommended Project Structure

```txt id="k7k2p2"
src/
├── common/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   └── decorators/
│
├── config/
├── modules/
│   ├── users/
│   ├── auth/
│   └── health/
│
├── app.module.ts
└── main.ts
```

---

# Recommended Architecture

| Layer       | Purpose                   |
| ----------- | ------------------------- |
| Controller  | request/response          |
| Service     | business logic            |
| Module      | feature grouping          |
| Guard       | auth/permissions          |
| Pipe        | validation/transformation |
| Interceptor | logging/response shaping  |
| Filter      | exception handling        |

---

# Main Bootstrap (main.ts)

```ts id="r5m0q6"
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

import helmet from "helmet";
import compression from "compression";
import cookieParser from "cookie-parser";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.use(helmet());

  app.use(compression());

  app.use(cookieParser());

  app.enableCors({
    origin: true,
    credentials: true,
  });

  app.setGlobalPrefix("api/v1");

  await app.listen(3000);
}

bootstrap();
```

---

# Environment Config

## Install

```bash id="tm1zjl"
npm install @nestjs/config
```

---

## app.module.ts

```ts id="1xks7o"
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

---

# Validation Best Practice

## Install

```bash id="n0g2be"
npm install class-validator class-transformer
```

---

# DTO Example

```ts id="r79y3d"
import { IsEmail, MinLength } from "class-validator";

export class CreateUserDto {
  @IsEmail()
  email: string;

  @MinLength(8)
  password: string;
}
```

---

# Global Validation Pipe

```ts id="8kr2e9"
import { ValidationPipe } from "@nestjs/common";

app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }),
);
```

---

# Exception Handling

## Global Exception Filter

```ts id="jzjllw"
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
} from "@nestjs/common";

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();

    const response = ctx.getResponse();

    const status =
      exception instanceof HttpException ? exception.getStatus() : 500;

    response.status(status).json({
      success: false,
      message: exception.message || "Internal Server Error",
    });
  }
}
```

---

## Register Filter

```ts id="y6w0y7"
app.useGlobalFilters(new GlobalExceptionFilter());
```

---

# Logging

## Install

```bash id="6oyqz9"
npm install nestjs-pino pino pino-pretty
```

---

# Setup

```ts id="4mqutj"
import { LoggerModule } from "nestjs-pino";

@Module({
  imports: [LoggerModule.forRoot()],
})
export class AppModule {}
```

---

# Rate Limiting

## Install

```bash id="y6f8m5"
npm install @nestjs/throttler
```

---

# Setup

```ts id="4fnhwu"
import { ThrottlerModule } from "@nestjs/throttler";

@Module({
  imports: [
    ThrottlerModule.forRoot([
      {
        ttl: 60000,
        limit: 100,
      },
    ]),
  ],
})
export class AppModule {}
```

---

# Enable Globally

```ts id="h7vxv9"
import { ThrottlerGuard } from "@nestjs/throttler";

import { APP_GUARD } from "@nestjs/core";

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
  ],
})
export class AppModule {}
```

---

# Interceptors

## Response Interceptor

```ts id="g4ng5n"
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";

import { map } from "rxjs";

@Injectable()
export class ResponseInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    return next.handle().pipe(
      map((data) => ({
        success: true,
        data,
      })),
    );
  }
}
```

---

## Register Globally

```ts id="5w4dhf"
app.useGlobalInterceptors(new ResponseInterceptor());
```

---

# Guards

Used for:

- auth
- roles
- permissions
- API key validation

```ts id="z5tkr8"
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext) {
    const request = context.switchToHttp().getRequest();

    return !!request.user;
  }
}
```

---

# Custom Decorators

```ts id="v5s7ft"
import { createParamDecorator } from "@nestjs/common";

export const User = createParamDecorator((data, ctx) => {
  const request = ctx.switchToHttp().getRequest();

  return request.user;
});
```

Usage:

```ts id="v86b6z"
@Get("/me")
getMe(@User() user) {
  return user;
}
```

---

# API Documentation

## Install

```bash id="nhsrm7"
npm install @nestjs/swagger swagger-ui-express
```

---

# Setup

```ts id="5xll0u"
import { SwaggerModule, DocumentBuilder } from "@nestjs/swagger";

const config = new DocumentBuilder().setTitle("API").setVersion("1.0").build();

const document = SwaggerModule.createDocument(app, config);

SwaggerModule.setup("docs", app, document);
```

---

# Caching

## Install

```bash id="7zq2w6"
npm install cache-manager
```

---

# Schedule / Cron Jobs

## Install

```bash id="9m4zgr"
npm install @nestjs/schedule
```

---

# Setup

```ts id="5ypn7d"
import { ScheduleModule } from "@nestjs/schedule";

@Module({
  imports: [ScheduleModule.forRoot()],
})
export class AppModule {}
```

---

# Cron Example

```ts id="jlwm0m"
import { Cron } from "@nestjs/schedule";

@Cron("0 * * * *")
handleCron() {
  console.log("Runs hourly");
}
```

---

# Security Best Practices

## Use

```ts id="ukp0w3"
helmet()
compression()
cors()
rate limiting
validation pipes
```

---

# Recommended Global Setup

```ts id="u3h4s0"
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    transform: true,
  }),
);

app.useGlobalFilters(new GlobalExceptionFilter());

app.useGlobalInterceptors(new ResponseInterceptor());
```

---

# Recommended Response Format

```json id="yopaqg"
{
  "success": true,
  "data": {},
  "message": "OK"
}
```

---

# Recommended Production Setup

```txt id="dxy7w6"
NestJS
Express Adapter
TypeScript
Helmet
Pino
Swagger
ValidationPipe
Global Filters
Rate Limiting
Redis Cache
Cron Jobs
```

---

# Important Best Practices

| Best Practice  | Recommendation          |
| -------------- | ----------------------- |
| Validation     | global ValidationPipe   |
| Error handling | global exception filter |
| Logging        | pino                    |
| Config         | @nestjs/config          |
| Security       | helmet + throttler      |
| Docs           | swagger                 |
| API versioning | /api/v1                 |
| DTOs           | always use DTOs         |
| Business logic | services only           |
| Controllers    | thin controllers        |
| Shared code    | common module           |

---

# Common NestJS Pattern

```txt id="9lh25x"
Controller
   ↓
Service
   ↓
Repository / External API
```

Controllers should stay minimal. Business logic belongs in services.
