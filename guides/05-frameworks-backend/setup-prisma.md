# Setup — Prisma ORM en tu Proyecto Express

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.4)
> **Objetivo**: integrar Prisma en tu proyecto Express, definir el schema, generar el cliente tipado, y ejecutar migraciones.
> **Prerequisito**: proyecto Express (`setup-express-project.md`), PostgreSQL corriendo en Docker (`setup-postgres.md` de 06-databases).

---

## ¿Por qué Prisma?

Escribir SQL a mano funciona, pero es propenso a errores: typos en nombres de columnas, queries sin tipar, y migraciones manuales. Prisma genera un cliente TypeScript a partir de un schema declarativo — tu editor autocompleta cada query con tipos exactos. Si cambiás el schema y regenerás, el compilador te marca todo el código roto. Es type-safety real para tu base de datos.

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

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `Error: P1001: Can't reach database server` | PostgreSQL no está corriendo o el `DATABASE_URL` es incorrecto. Verificá con `docker ps` y probá la conexión con `psql`. |
| `Error: P4001: The migration directory is out of sync` | La DB y el schema.prisma están desincronizados. Ejecutá `npx prisma migrate resolve --applied <migration>` o reseteá con `npx prisma migrate reset`. |
| El cliente Prisma no tiene autocompletado | Ejecutá `npx prisma generate` después de cada cambio al schema. El cliente se regenera en `node_modules/.prisma/client`. |
| `PrismaClientValidationError` en runtime | Los datos que pasás no coinciden con el schema. Verificá los tipos y campos requeridos. Prisma valida en runtime además de compile-time. |

---

## Preguntas de repaso

- **P:** ¿Qué significa que Prisma sea "type-safe"?
  **R:** Que TypeScript conoce la estructura exacta de cada modelo. Si escribís `prisma.user.findUnique()`, el editor sabe que el resultado tiene `.id`, `.email`, `.name`, etc. — sin strings mágicos.

- **P:** ¿Qué hace `npx prisma migrate dev`?
  **R:** Compara el schema.prisma con el estado actual de la DB, genera SQL de migración, y lo aplica. En desarrollo puede resetear la DB si hay conflictos.

- **P:** ¿Qué diferencia hay entre `prisma migrate dev` y `prisma migrate deploy`?
  **R:** `dev` genera Y aplica migraciones (puede resetear). `deploy` solo aplica migraciones existentes — se usa en producción, nunca resetea datos.

- **P:** ¿Qué es `prisma studio`?
  **R:** Una UI web que se abre en el navegador para explorar y editar datos de tu base de datos. Útil para desarrollo y debugging.

- **P:** ¿Cuándo conviene Prisma sobre queries SQL raw?
  **R:** Cuando querés type-safety, autocompletado, y migraciones automáticas. SQL raw conviene para queries muy complejas (aggregations avanzadas, CTEs) que Prisma no soporta bien.

---

## Recursos

- [Prisma — Quickstart](https://www.prisma.io/docs/getting-started/quickstart)
- [Prisma — CRUD](https://www.prisma.io/docs/orm/prisma-client/queries/crud)