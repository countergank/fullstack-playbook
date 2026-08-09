# Setup — Proyecto Express + TypeScript

> **Tópico**: 4 & 5 — Backend Core + Frameworks
> **Objetivo**: crear un proyecto backend con Express + TypeScript, estructura en capas, y un endpoint de prueba funcionando.
> **Prerequisito**: Node (`setup-node.md`), Docker (`setup-docker.md`), PostgreSQL (`setup-postgres.md` de 06-databases).

---

## Checklist

### 1. Crear el proyecto

```bash
mkdir mi-backend && cd mi-backend
npm init -y
```

### 2. Instalar dependencias

```bash
# Producción
npm install express cors helmet
npm install -D typescript @types/express @types/node tsx

# Inicializar TypeScript
npx tsc --init
```

### 3. Configurar `tsconfig.json`

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"]
}
```

### 4. Agregar scripts a `package.json`

```jsonc
{
  "scripts": {
    "dev": "tsx watch src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js"
  }
}
```

### 5. Crear la estructura feature-based

```
src/
├── app.ts                          # entry point
├── config/
│   └── env.ts                      # variables de entorno validadas
├── modules/
│   └── health/
│       ├── health.controller.ts
│       └── health.routes.ts
└── shared/
    ├── middlewares/
    │   └── errorHandler.ts
    └── errors/
        └── AppError.ts
```

### 6. Escribir el entry point mínimo

```ts
// src/app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import { healthRoutes } from './modules/health/health.routes.js';
import { errorHandler } from './shared/middlewares/errorHandler.js';

const app = express();

app.use(helmet());
app.use(cors());
app.use(express.json());

app.use('/health', healthRoutes);

app.use(errorHandler);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));
```

### 7. Crear un endpoint de prueba

```ts
// src/modules/health/health.controller.ts
import type { Request, Response } from 'express';

export function getHealth(_req: Request, res: Response) {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
}
```

```ts
// src/modules/health/health.routes.ts
import { Router } from 'express';
import { getHealth } from './health.controller.js';

export const healthRoutes = Router();
healthRoutes.get('/', getHealth);
```

### 8. Error handler mínimo

```ts
// src/shared/middlewares/errorHandler.ts
import type { Request, Response, NextFunction } from 'express';

export function errorHandler(err: Error, _req: Request, res: Response, _next: NextFunction) {
  console.error(err);
  res.status(500).json({ error: { message: 'Internal server error' } });
}
```

### 9. Probar

```bash
npm run dev
```

```bash
# En otra terminal:
curl http://localhost:3000/health
# {"status":"ok","timestamp":"2026-08-09T15:00:00.000Z"}
```

---

## Verificación

```bash
curl http://localhost:3000/health | grep "ok"
```

**Si `curl` devuelve `{"status":"ok"}` → proyecto Express listo. ✅**

---

## Recursos

- [Express — Hello World](https://expressjs.com/en/starter/hello-world.html)
- [TypeScript — tsconfig reference](https://www.typescriptlang.org/tsconfig/)