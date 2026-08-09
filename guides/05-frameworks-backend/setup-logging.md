# Setup — Logging Estructurado con Pino

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.6)
> **Objetivo**: reemplazar `console.log` por Pino con request ID, niveles de log, y pretty-printing en desarrollo.
> **Prerequisito**: proyecto Express (`setup-express-project.md`).

---

## Checklist

### 1. Instalar Pino

```bash
npm install pino pino-pretty
npm install -D @types/pino
```

### 2. Crear el logger compartido

```ts
// src/shared/logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  ...(process.env.NODE_ENV === 'development' && {
    transport: {
      target: 'pino-pretty',
      options: { colorize: true, translateTime: 'HH:MM:ss' },
    },
  }),
});
```

### 3. Middleware de request ID

```ts
// src/shared/middlewares/requestId.ts
import { v4 as uuid } from 'uuid';
import type { Request, Response, NextFunction } from 'express';

export function requestId(req: Request, _res: Response, next: NextFunction) {
  req.id = uuid();
  next();
}
```

```bash
npm install uuid
npm install -D @types/uuid
```

### 4. Agregar request ID al log

```ts
// src/shared/middlewares/requestLogger.ts
import { logger } from '../logger.js';
import type { Request, Response, NextFunction } from 'express';

export function requestLogger(req: Request, res: Response, next: NextFunction) {
  const start = Date.now();
  res.on('finish', () => {
    logger.info({
      requestId: req.id,
      method: req.method,
      url: req.originalUrl,
      status: res.statusCode,
      durationMs: Date.now() - start,
    }, 'request completed');
  });
  next();
}
```

### 5. Registrar los middlewares en orden

```ts
// src/app.ts
import { requestId } from './shared/middlewares/requestId.js';
import { requestLogger } from './shared/middlewares/requestLogger.js';

app.use(requestId);        // 1. Asignar ID a cada request
app.use(requestLogger);    // 2. Loggear al finalizar
```

### 6. Reemplazar `console.log` por `logger`

```ts
// Antes:
console.log('Server running');
console.error('DB connection failed', err);

// Ahora:
import { logger } from './shared/logger.js';
logger.info({ port: env.PORT }, 'Server running');
logger.error({ err, requestId }, 'DB connection failed');
```

### 7. Agregar `NODE_ENV` y `LOG_LEVEL` al `.env`

```
NODE_ENV=development
LOG_LEVEL=info
```

---

## Verificación

```bash
npm run dev
curl http://localhost:3000/health
```

**La terminal debe mostrar un log estructurado con request ID, método, URL, status y duración.** En desarrollo se ve coloreado (pino-pretty). En producción sería JSON puro.

---

## Recursos

- [Pino](https://getpino.io/)
- [pino-pretty](https://github.com/pinojs/pino-pretty)