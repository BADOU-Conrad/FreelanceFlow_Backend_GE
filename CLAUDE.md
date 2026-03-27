# CLAUDE.md — FreelanceFlow Backend

> Fichier de contexte pour Claude Code (`/init`).
> À placer à la racine de `freelanceflow-backend/`.
> Repo : `github.com/FreelanceFlow-Team/freelanceflow-backend`

---

## 📋 Présentation du Projet

**FreelanceFlow** est une application web SaaS mono-tenant pour qu'un freelance gère son activité : clients, prestations, factures, génération PDF. Ce repo est le **backend uniquement**. Le frontend est dans un repo séparé (`freelanceflow-frontend`).

### Périmètre V1

1. Authentification JWT (register / login)
2. CRUD Clients — liés à l'utilisateur connecté
3. CRUD Services (Prestations) — intitulé + tarif horaire
4. CRUD Factures — calcul automatique HT/TVA/TTC, statuts (DRAFT/SENT/PAID/CANCELLED)
5. Génération PDF — facture complète streamée en `application/pdf`

---

## 🛠️ Stack & Versions — NE PAS DÉVIER

| Outil                   | Version         | Notes critiques                                                                       |
| ----------------------- | --------------- | ------------------------------------------------------------------------------------- |
| **Node.js**             | `22 LTS (22.x)` | Minimum 22.11.0 requis par Prisma 6                                                   |
| **NestJS**              | `11.1.x`        | SWC compilateur par défaut. Vitest remplace Jest. JSON logging natif.                 |
| **TypeScript**          | `5.7.x`         | Minimum 5.1 requis par Prisma 6                                                       |
| **Prisma ORM**          | `6.x`           | Client généré dans `src/generated/prisma/`. Import depuis `./generated/prisma/client` |
| **PostgreSQL**          | `17`            | Image Docker `postgres:17-alpine`                                                     |
| **@nestjs/jwt**         | `10.x`          |                                                                                       |
| **@nestjs/passport**    | `10.x`          |                                                                                       |
| **@nestjs/swagger**     | `8.x`           | Documentation auto sur `/api/docs`                                                    |
| **class-validator**     | `0.14.x`        | Validation DTOs                                                                       |
| **class-transformer**   | `0.5.x`         | Transformation globale                                                                |
| **@react-pdf/renderer** | `4.x`           | Génération PDF côté Node, React 19 compatible                                         |
| **bcrypt**              | `5.x`           | Hash mots de passe                                                                    |
| **Vitest**              | `3.x`           | Remplace Jest — `vi.fn()` pas `jest.fn()`                                             |
| **ESLint**              | `9.x`           | Flat config `eslint.config.mjs`                                                       |
| **Prettier**            | `3.x`           |                                                                                       |
| **Husky**               | `9.x`           | Git hooks                                                                             |
| **lint-staged**         | `15.x`          |                                                                                       |

---

## 🚨 Règles Breaking NestJS 11 & Prisma 6 — TOUJOURS APPLIQUER

### 1. Vitest remplace Jest — utiliser `vi` pas `jest`

```typescript
// ❌ INTERDIT
import { jest } from '@jest/globals';
(jest.fn(), jest.spyOn());

// ✅ OBLIGATOIRE
import { vi, describe, it, expect, beforeEach } from 'vitest';
(vi.fn(), vi.spyOn());
```

### 2. Import Prisma 6 — chemin généré

```typescript
// ❌ INTERDIT (Prisma 5 et avant)
import { PrismaClient } from '@prisma/client';

// ✅ OBLIGATOIRE (Prisma 6)
import { PrismaClient } from '../generated/prisma/client';
// ou avec .js pour ESM
import { PrismaClient } from '../generated/prisma/client.js';
```

### 3. Schema Prisma — output explicite

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../src/generated/prisma"  // ← obligatoire pour Prisma 6
}
```

### 4. JSON Logging NestJS 11

```typescript
// main.ts
const app = await NestFactory.create(AppModule, {
  logger: { json: true }, // ← nouvelle option NestJS 11
});
```

### 5. SWC compilateur par défaut

```json
// nest-cli.json — NestJS 11 utilise SWC par défaut
{
  "compilerOptions": {
    "builder": "swc"
  }
}
```

### 6. Scripts `package.json` — Vitest

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:cov": "vitest run --coverage"
  }
}
```

---

## 📁 Arborescence

```
freelanceflow-backend/
├── .github/workflows/ci.yml
│
├── prisma/
│   ├── schema.prisma                       # Schéma de données
│   ├── migrations/                         # Auto-générées
│   └── seed.ts
│
├── src/
│   ├── main.ts                             # Bootstrap + Swagger + CORS
│   ├── app.module.ts
│   │
│   ├── generated/
│   │   └── prisma/                         # ← Prisma 6 génère ici (NE PAS ÉDITER)
│   │       └── client.js
│   │
│   ├── config/
│   │   ├── app.config.ts
│   │   └── database.config.ts
│   │
│   ├── prisma/
│   │   ├── prisma.module.ts                # @Global() — disponible partout
│   │   └── prisma.service.ts               # extends PrismaClient
│   │
│   ├── common/
│   │   ├── decorators/current-user.decorator.ts
│   │   ├── filters/http-exception.filter.ts
│   │   ├── guards/jwt-auth.guard.ts
│   │   └── interceptors/transform.interceptor.ts
│   │
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts              # POST /api/auth/register, /api/auth/login
│   │   ├── auth.service.ts
│   │   ├── auth.service.spec.ts            # ← Tests Vitest OBLIGATOIRES
│   │   ├── strategies/jwt.strategy.ts
│   │   └── dto/
│   │       ├── register.dto.ts
│   │       └── login.dto.ts
│   │
│   ├── users/
│   │   ├── users.module.ts
│   │   ├── users.service.ts
│   │   ├── users.service.spec.ts
│   │   └── entities/user.entity.ts
│   │
│   ├── clients/
│   │   ├── clients.module.ts
│   │   ├── clients.controller.ts           # CRUD /api/clients
│   │   ├── clients.service.ts
│   │   ├── clients.service.spec.ts
│   │   └── dto/
│   │       ├── create-client.dto.ts
│   │       └── update-client.dto.ts
│   │
│   ├── services/                           # Module "Prestations"
│   │   ├── services.module.ts
│   │   ├── services.controller.ts          # CRUD /api/services
│   │   ├── services.service.ts
│   │   ├── services.service.spec.ts
│   │   └── dto/
│   │       ├── create-service.dto.ts
│   │       └── update-service.dto.ts
│   │
│   ├── invoices/
│   │   ├── invoices.module.ts
│   │   ├── invoices.controller.ts          # CRUD + GET /api/invoices/:id/pdf
│   │   ├── invoices.service.ts
│   │   ├── invoices.service.spec.ts
│   │   └── dto/
│   │       ├── create-invoice.dto.ts
│   │       ├── update-invoice.dto.ts
│   │       └── invoice-line.dto.ts
│   │
│   └── pdf/
│       ├── pdf.module.ts
│       ├── pdf.service.ts                  # @react-pdf/renderer v4
│       ├── pdf.service.spec.ts
│       └── templates/invoice.template.tsx  # Template JSX du PDF
│
├── test/app.e2e-spec.ts
├── .env.example
├── .husky/pre-commit
├── docker-compose.yml
├── Dockerfile
├── eslint.config.mjs
├── .prettierrc
├── vitest.config.ts
├── nest-cli.json
├── package.json
└── tsconfig.json
```

---

## 🗄️ Schéma Prisma Complet

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id           String    @id @default(cuid())
  email        String    @unique
  passwordHash String
  firstName    String
  lastName     String
  company      String?
  address      String?
  siret        String?
  createdAt    DateTime  @default(now())
  clients      Client[]
  services     Service[]
  invoices     Invoice[]
}

model Client {
  id        String    @id @default(cuid())
  name      String
  email     String
  company   String?
  address   String?
  userId    String
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  invoices  Invoice[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Service {
  id           String        @id @default(cuid())
  label        String
  hourlyRate   Decimal       @db.Decimal(10, 2)
  userId       String
  user         User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  invoiceLines InvoiceLine[]
  createdAt    DateTime      @default(now())
}

enum InvoiceStatus {
  DRAFT
  SENT
  PAID
  CANCELLED
}

model Invoice {
  id        String        @id @default(cuid())
  number    String        @unique   // Format : "FF-2024-001"
  status    InvoiceStatus @default(DRAFT)
  issuedAt  DateTime      @default(now())
  dueAt     DateTime?
  totalHT   Decimal       @db.Decimal(10, 2)
  vatRate   Decimal       @db.Decimal(5, 2)  @default(20)
  totalTVA  Decimal       @db.Decimal(10, 2)
  totalTTC  Decimal       @db.Decimal(10, 2)
  notes     String?
  userId    String
  user      User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  clientId  String
  client    Client        @relation(fields: [clientId], references: [id])
  lines     InvoiceLine[]
  createdAt DateTime      @default(now())
  updatedAt DateTime      @updatedAt
}

model InvoiceLine {
  id          String   @id @default(cuid())
  quantity    Decimal  @db.Decimal(10, 2)
  unitPrice   Decimal  @db.Decimal(10, 2)
  description String?
  invoiceId   String
  invoice     Invoice  @relation(fields: [invoiceId], references: [id], onDelete: Cascade)
  serviceId   String
  service     Service  @relation(fields: [serviceId], references: [id])
}
```

---

## 🔑 Fichiers de configuration clés

### `src/main.ts`

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: { json: true }, // NestJS 11
  });

  app.setGlobalPrefix('api');
  app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
  app.enableCors({
    origin: process.env.FRONTEND_URL ?? 'http://localhost:3000',
    credentials: true,
  });

  const config = new DocumentBuilder()
    .setTitle('FreelanceFlow API')
    .setVersion('1.0.0')
    .addBearerAuth()
    .build();
  SwaggerModule.setup(
    'api/docs',
    app,
    SwaggerModule.createDocument(app, config),
  );

  await app.listen(process.env.PORT ?? 3001);
}
bootstrap();
```

### `src/prisma/prisma.service.ts`

```typescript
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '../generated/prisma/client'; // Prisma 6

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  async onModuleInit() {
    await this.$connect();
  }
  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

### `src/prisma/prisma.module.ts`

```typescript
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

### `vitest.config.ts`

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['src/**/*.spec.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'lcov'],
    },
  },
});
```

---

## 🔌 API REST — Tous les endpoints

Préfixe global : `/api`. Toutes les routes sauf `/api/auth/*` nécessitent `Authorization: Bearer <token>`.

### Auth — public

| Méthode | Route                | Body                                       | Réponse                  |
| ------- | -------------------- | ------------------------------------------ | ------------------------ |
| `POST`  | `/api/auth/register` | `{ email, password, firstName, lastName }` | `{ access_token, user }` |
| `POST`  | `/api/auth/login`    | `{ email, password }`                      | `{ access_token, user }` |

### Clients — protégé

| Méthode  | Route              | Body                                  | Réponse    |
| -------- | ------------------ | ------------------------------------- | ---------- |
| `GET`    | `/api/clients`     | —                                     | `Client[]` |
| `GET`    | `/api/clients/:id` | —                                     | `Client`   |
| `POST`   | `/api/clients`     | `{ name, email, company?, address? }` | `Client`   |
| `PATCH`  | `/api/clients/:id` | `Partial<Client>`                     | `Client`   |
| `DELETE` | `/api/clients/:id` | —                                     | `204`      |

### Services (Prestations) — protégé

| Méthode  | Route               | Body                    | Réponse     |
| -------- | ------------------- | ----------------------- | ----------- |
| `GET`    | `/api/services`     | —                       | `Service[]` |
| `GET`    | `/api/services/:id` | —                       | `Service`   |
| `POST`   | `/api/services`     | `{ label, hourlyRate }` | `Service`   |
| `PATCH`  | `/api/services/:id` | `Partial`               | `Service`   |
| `DELETE` | `/api/services/:id` | —                       | `204`       |

### Invoices (Factures) — protégé

| Méthode  | Route                      | Body                                                                                              | Réponse                    |
| -------- | -------------------------- | ------------------------------------------------------------------------------------------------- | -------------------------- |
| `GET`    | `/api/invoices`            | —                                                                                                 | `Invoice[]`                |
| `GET`    | `/api/invoices/:id`        | —                                                                                                 | `Invoice` + client + lines |
| `POST`   | `/api/invoices`            | `{ clientId, lines: [{serviceId, quantity, unitPrice, description?}], vatRate?, dueAt?, notes? }` | `Invoice`                  |
| `PATCH`  | `/api/invoices/:id`        | `Partial`                                                                                         | `Invoice`                  |
| `PATCH`  | `/api/invoices/:id/status` | `{ status: InvoiceStatus }`                                                                       | `Invoice`                  |
| `DELETE` | `/api/invoices/:id`        | —                                                                                                 | `204`                      |
| `GET`    | `/api/invoices/:id/pdf`    | —                                                                                                 | `application/pdf` stream   |

### Format réponse uniforme

```typescript
// Succès
{ "data": { ... }, "message": "OK" }

// Erreur
{ "statusCode": 400, "message": "Validation failed", "errors": [...] }
```

---

## 🧪 Tests — Conventions Vitest

**Règle** : chaque `*.service.ts` doit avoir son `*.service.spec.ts`. Mocker `PrismaService` — jamais de vraie DB dans les tests unitaires.

```typescript
// Exemple : src/clients/clients.service.spec.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'; // ← vi, pas jest
import { Test, TestingModule } from '@nestjs/testing';
import { ClientsService } from './clients.service';
import { PrismaService } from '../prisma/prisma.service';

describe('ClientsService', () => {
  let service: ClientsService;
  let prisma: PrismaService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ClientsService,
        {
          provide: PrismaService,
          useValue: {
            client: {
              findMany: vi.fn(),
              findUnique: vi.fn(),
              create: vi.fn(),
              update: vi.fn(),
              delete: vi.fn(),
            },
          },
        },
      ],
    }).compile();

    service = module.get<ClientsService>(ClientsService);
    prisma = module.get<PrismaService>(PrismaService);
  });

  it('should return all clients for a user', async () => {
    const mockClients = [
      { id: '1', name: 'Acme Corp', email: 'acme@example.com' },
    ];
    vi.spyOn(prisma.client, 'findMany').mockResolvedValue(mockClients as any);

    const result = await service.findAll('user-123');
    expect(result).toEqual(mockClients);
    expect(prisma.client.findMany).toHaveBeenCalledWith({
      where: { userId: 'user-123' },
    });
  });
});
```

Coverage cible : > 70% sur les services.

---

## 🧮 Logique métier — Calcul facture

Le calcul HT/TVA/TTC se fait **exclusivement côté backend** dans `invoices.service.ts`.

```typescript
// Dans invoices.service.ts — create()
const totalHT = lines.reduce(
  (sum, line) => sum + Number(line.quantity) * Number(line.unitPrice),
  0,
);
const vatRate = dto.vatRate ?? 20;
const totalTVA = totalHT * (vatRate / 100);
const totalTTC = totalHT + totalTVA;

// Numérotation automatique des factures
const count = await this.prisma.invoice.count({ where: { userId } });
const number = `FF-${new Date().getFullYear()}-${String(count + 1).padStart(3, '0')}`;
```

---

## 🐳 Docker & Infrastructure Locale

### `docker-compose.yml`

```yaml
version: '3.9'

services:
  db:
    image: postgres:17-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: freelance
      POSTGRES_PASSWORD: freelance
      POSTGRES_DB: freelanceflow
    ports:
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U freelance -d freelanceflow']
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: postgresql://freelance:freelance@db:5432/freelanceflow
      JWT_SECRET: dev-secret-change-in-prod
      PORT: 3001
      FRONTEND_URL: http://localhost:3000
    ports:
      - '3001:3001'
    volumes:
      - .:/app
      - /app/node_modules

volumes:
  postgres_data:
```

### `Dockerfile` (multi-stage)

```dockerfile
FROM node:22-alpine AS base
WORKDIR /app
COPY package*.json ./

FROM base AS development
RUN npm ci
COPY . .
EXPOSE 3001
CMD ["npm", "run", "start:dev"]

FROM base AS builder
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

FROM node:22-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/src/generated ./src/generated
COPY package*.json ./
EXPOSE 3001
CMD ["node", "dist/main"]
```

---

## ⚙️ CI — GitHub Actions

```yaml
name: CI Backend
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17-alpine
        env:
          POSTGRES_USER: freelance
          POSTGRES_PASSWORD: freelance
          POSTGRES_DB: freelanceflow_test
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check
      - run: npx prisma generate
      - run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://freelance:freelance@localhost:5432/freelanceflow_test
      - run: npm run test       # Vitest dans NestJS 11
        env:
          DATABASE_URL: postgresql://freelance:freelance@localhost:5432/freelanceflow_test
          JWT_SECRET: ci-secret
      - run: npm run build
      - name: Build Docker image
        run: docker build --target production -t freelanceflow-backend:${{ github.sha }} .
      - name: Login & Push GHCR
        if: github.ref == 'refs/heads/main'
        uses: docker/login-action@v3
        with: { registry: ghcr.io, username: ${{ github.actor }}, password: ${{ secrets.GITHUB_TOKEN }} }
      - if: github.ref == 'refs/heads/main'
        run: |
          docker tag freelanceflow-backend:${{ github.sha }} ghcr.io/${{ github.repository_owner }}/freelanceflow-backend:latest
          docker push ghcr.io/${{ github.repository_owner }}/freelanceflow-backend:latest
      - name: Deploy
        if: github.ref == 'refs/heads/main'
        run: curl -X POST "${{ secrets.DEPLOY_WEBHOOK_URL }}"
```

---

## 🔀 Git

```
main      ← production (PR obligatoire)
develop   ← intégration
feature/<id>-<description>
fix/<id>-<description>
```

Commits : Conventional Commits — `feat:`, `fix:`, `chore:`, `test:`, `docs:`
PRs vers `develop` uniquement. Référencer l'issue avec `Closes #<id>`.

### `lint-staged` (`package.json`)

```json
{
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"],
    "*.{json,md,yml}": ["prettier --write"]
  }
}
```

---

## 📝 Variables d'environnement

### `.env.example`

```env
DATABASE_URL="postgresql://freelance:freelance@localhost:5432/freelanceflow"
JWT_SECRET="your-super-secret-jwt-key-change-in-production"
JWT_EXPIRES_IN="7d"
PORT=3001
FRONTEND_URL="http://localhost:3000"
NODE_ENV="development"
```

### Production (Railway)

```env
DATABASE_URL=<fourni par Railway>
JWT_SECRET=<secret long et aléatoire>
PORT=3001
FRONTEND_URL=https://freelanceflow.vercel.app
NODE_ENV=production
```

---

## 🔍 Points d'attention Claude Code

1. **Vitest** — toujours `vi.fn()`, jamais `jest.fn()`. Import depuis `'vitest'`
2. **Prisma 6** — import depuis `'../generated/prisma/client'`, jamais `'@prisma/client'`
3. **Toutes les routes** sauf `/api/auth/*` doivent avoir `@UseGuards(JwtAuthGuard)`
4. **Préfixe `/api`** configuré globalement dans `main.ts` — ne pas le répéter dans les controllers
5. **`@CurrentUser()`** pour récupérer l'utilisateur connecté dans les controllers
6. **Chaque ressource** (client, service, invoice) est filtrée par `userId` — un utilisateur ne voit que ses propres données
7. **Les montants** sont des `Decimal` dans Prisma — utiliser `Number()` pour les opérations arithmétiques, retourner en `string` dans les DTOs de réponse
8. **Calcul facture** dans `invoices.service.ts` uniquement — totalHT, totalTVA, totalTTC calculés au `create()`
9. **Numérotation** des factures : format `FF-YYYY-NNN` (ex: `FF-2024-001`)
10. **PDF** : `pdf.service.ts` avec `@react-pdf/renderer` v4 — retourner un `Buffer`, pas un fichier
11. **Tests** : mocker `PrismaService` avec `useValue` — jamais de vraie connexion DB dans les specs
12. **`src/generated/prisma/`** est dans `.gitignore` — `npx prisma generate` doit tourner dans la CI avant le build

---

_Stack : NestJS 11.1 / Prisma 6 / PostgreSQL 17 / Node.js 22 — Mars 2026_
