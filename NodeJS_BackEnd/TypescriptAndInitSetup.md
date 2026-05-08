## Clean, reference-style setup guide split into two independent sections:

1. NestJS (Express adapter)
2. Express + TypeScript only

Each includes:

* TypeScript initialization from scratch
* REST API starter setup
* env strategy (dev / test / staging / prod)
* build setup
* scripts for each mode
* test + staging clarity

---

# 1. NestJS (with Express adapter) — Best Practice Setup

---

## 1.1 Project Initialization

```bash id="n1"
npm i -g @nestjs/cli
nest new backend
```

Select:

* npm or pnpm
* TypeScript (default)

---

## 1.2 Core Structure (REST API baseline)

```txt id="n2"
src/
├── modules/
├── common/
├── config/
├── app.module.ts
└── main.ts
```

---

## 1.3 main.ts (Express-based Nest)

```ts id="n3"
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.setGlobalPrefix("api/v1");

  await app.listen(process.env.PORT || 3000);
}

bootstrap();
```

---

## 1.4 TypeScript Config (Nest default + recommended tweaks)

```json id="n4"
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "ES2022",
    "strict": true,
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true
  }
}
```

---

## 1.5 Environment Setup Strategy

### Install config

```bash id="n5"
npm install @nestjs/config
```

---

### Structure

```txt id="n6"
.env
.env.dev
.env.test
.env.staging
.env.prod
```

---

### Load config in AppModule

```ts id="n7"
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: `.env.${process.env.NODE_ENV}`,
    }),
  ],
})
export class AppModule {}
```

---

## 1.6 Scripts Setup (NestJS)

```json id="n8"
{
  "scripts": {
    "dev": "nest start --watch",
    "build": "nest build",
    "start": "node dist/main.js",

    "start:dev": "NODE_ENV=dev nest start --watch",
    "start:test": "NODE_ENV=test node dist/main.js",
    "start:staging": "NODE_ENV=staging node dist/main.js",
    "start:prod": "NODE_ENV=production node dist/main.js",

    "test": "jest",
    "test:watch": "jest --watch",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  }
}
```

---

## 1.7 Build Flow

```bash id="n9"
npm run build
npm run start:prod
```

Output:

```txt id="n10"
dist/main.js
```

---

## 1.8 Mode Summary (NestJS)

| Mode    | Command            | Env          |
| ------- | ------------------ | ------------ |
| Dev     | nest start --watch | .env.dev     |
| Test    | node dist          | .env.test    |
| Staging | node dist          | .env.staging |
| Prod    | node dist          | .env.prod    |

---

---

# 2. Express + TypeScript Only (REST API Setup)

---

## 2.1 Project Initialization

```bash id="e1"
mkdir backend && cd backend
npm init -y
```

---

## 2.2 Install Core Dependencies

```bash id="e2"
npm install express dotenv
```

---

## 2.3 Install Dev Dependencies

```bash id="e3"
npm install -D typescript ts-node-dev @types/node @types/express rimraf cross-env
```

---

## 2.4 TypeScript Init

```bash id="e4"
npx tsc --init
```

---

## 2.5 Recommended tsconfig.json

```json id="e5"
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",

    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,

    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 2.6 Project Structure

```txt id="e6"
src/
├── routes/
├── controllers/
├── services/
├── middlewares/
├── config/
├── app.ts
└── server.ts
```

---

## 2.7 Basic Express Setup

### app.ts

```ts id="e7"
import express from "express";

const app = express();

app.use(express.json());

export default app;
```

---

### server.ts

```ts id="e8"
import app from "./app";
import "dotenv/config";

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on ${PORT}`);
});
```

---

## 2.8 Environment Strategy

Same structure:

```txt id="e9"
.env.dev
.env.test
.env.staging
.env.prod
```

---

### Load environment manually

```ts id="e10"
import dotenv from "dotenv";

dotenv.config({
  path: `.env.${process.env.NODE_ENV}`,
});
```

---

## 2.9 Scripts Setup (Express)

```json id="e11"
{
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/server.ts",
    "build": "rimraf dist && tsc",
    "start": "node dist/server.js",

    "start:dev": "cross-env NODE_ENV=dev ts-node-dev src/server.ts",
    "start:test": "cross-env NODE_ENV=test node dist/server.js",
    "start:staging": "cross-env NODE_ENV=staging node dist/server.js",
    "start:prod": "cross-env NODE_ENV=production node dist/server.js",

    "typecheck": "tsc --noEmit"
  }
}
```

---

## 2.10 Build Flow

```bash id="e12"
npm run build
npm run start:prod
```

Output:

```txt id="e13"
dist/server.js
```

---

## 2.11 Mode Summary (Express)

| Mode    | Command           | Env          |
| ------- | ----------------- | ------------ |
| Dev     | ts-node-dev watch | .env.dev     |
| Test    | node dist         | .env.test    |
| Staging | node dist         | .env.staging |
| Prod    | node dist         | .env.prod    |

---

# Final Mental Model (Both)

## Dev

* hot reload
* ts-node-dev / nest --watch
* .env.dev

## Test

* compiled JS
* isolated env

## Staging

* production build
* .env.staging
* same as prod infra

## Production

* compiled JS only
* no TypeScript runtime
* .env.prod

---

# Key Rule (Very Important)

> Never run TypeScript directly in production.

Production always:

* build → `dist/`
* run → `node dist/...`

---

