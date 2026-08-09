# Setup — Prisma ORM en tu Proyecto Express

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.4)
> **Objetivo**: integrar Prisma en tu proyecto Express, definir el schema, generar el cliente tipado, y ejecutar migraciones.
> **Prerequisito**: proyecto Express (`setup-express-project.md`), PostgreSQL corriendo en Docker (`setup-postgres.md` de 06-databases).

---

## Checklist

### 1. Instalar Prisma

```bash
npm install prisma @prisma/client
npx prisma init
```

Esto crea `prisma/schema.prisma` y agrega `DATABASE_URL` a tu `.env`.

### 2. Configurar la conexión

En tu `.env`, asegurate de que apunte a tu PostgreSQL de Docker:
```
DATABASE_URL="postgresql://dev:dev@localhost:5432/fullstack_dev"
```

### 3. Definir tu primer modelo

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  password  String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

enum Role {
  USER
  ADMIN
}
```

### 4. Ejecutar la primera migración

```bash
npx prisma migrate dev --name init
```

- [ ] La migración crea la tabla `User` en PostgreSQL. Verificá con `npx prisma studio`.

### 5. Crear el cliente Prisma (singleton)

```ts
// src/shared/db/prisma.ts
import { PrismaClient } from '@prisma/client';

export const prisma = new PrismaClient();
```

### 6. Usar Prisma en un repository

```ts
// src/modules/users/users.repository.ts
import { prisma } from '../../shared/db/prisma.js';
import type { Prisma } from '@prisma/client';

export const userRepository = {
  findByEmail: (email: string) => prisma.user.findUnique({ where: { email } }),
  findById: (id: string) => prisma.user.findUnique({ where: { id } }),
  create: (data: Prisma.UserCreateInput) => prisma.user.create({ data }),
  list: (skip: number, take: number) => prisma.user.findMany({ skip, take }),
};
```

### 7. Verificar tipado

```ts
// TypeScript sabe que esto es { id, email, name, role, createdAt, updatedAt }
const user = await userRepository.findByEmail('test@ejemplo.com');
user?.createdAt; // ← autocompletado y tipado
```

---

## Verificación

```bash
npx prisma migrate dev --name init   # sin errores
npx prisma studio                     # UI abre mostrando la tabla User
```

**Si `prisma studio` muestra tu modelo → Prisma listo. ✅**

---

## Recursos

- [Prisma — Quickstart](https://www.prisma.io/docs/getting-started/quickstart)
- [Prisma — CRUD](https://www.prisma.io/docs/orm/prisma-client/queries/crud)