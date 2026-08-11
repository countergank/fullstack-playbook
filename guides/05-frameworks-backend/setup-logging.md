# Setup — Logging Estructurado con Pino

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.6)
> **Objetivo**: reemplazar `console.log` por Pino con request ID, niveles de log, y pretty-printing en desarrollo.
> **Prerequisito**: proyecto Express (`setup-express-project.md`).

---

## ¿Por qué logging estructurado?

`console.log` sirve para debuggear localmente, pero en producción necesitás logs que un sistema de monitoreo pueda indexar, buscar y graficar. Pino produce JSON estructurado con niveles, timestamps, y contexto (request ID, user ID, duración). Sin logging estructurado, debuggear un error en producción es como buscar una aguja en un pajar sin imán.

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

- [ ] `NODE_ENV=development` y `LOG_LEVEL=info` en `.env`
- [ ] En desarrollo los logs se ven coloreados (pino-pretty)
- [ ] En producción (`NODE_ENV=production`) los logs son JSON puro

---

## Verificación

```bash
npm run dev
curl http://localhost:3000/health
```

**La terminal debe mostrar un log estructurado con request ID, método, URL, status y duración.** En desarrollo se ve coloreado (pino-pretty). En producción sería JSON puro.

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Los logs no muestran colores en desarrollo | Verificá que `NODE_ENV=development` en `.env` y que `pino-pretty` esté instalado (`npm ls pino-pretty`). |
| `req.id` es `undefined` en el request logger | El middleware `requestId` debe registrarse ANTES que `requestLogger` en `app.ts`. El orden importa. |
| `@types/pino` da error de tipos | Pino v9+ incluye tipos propios. Si usás v9+, remové `@types/pino`. Si usás v8, mantenelo. |
| Los logs no incluyen el request ID | Asegurate de que `req.id` se asigna en el middleware `requestId` y que se pasa al logger: `logger.info({ requestId: req.id }, 'msg')`. |

---

## Preguntas de repaso

- **P:** ¿Por qué `console.log` no es suficiente en producción?
  **R:** Produce texto plano sin estructura, sin niveles, sin request ID. Los sistemas de monitoreo no pueden indexarlo ni buscarlo eficientemente.

- **P:** ¿Qué información debe tener cada log de producción?
  **R:** Timestamp, nivel (info/error/warn), mensaje, request ID, y contexto relevante (método, URL, status, duración, user ID).

- **P:** ¿Para qué sirve el request ID?
  **R:** Identifica unívocamente cada request. Permite seguir todos los logs de un mismo request a través de múltiples servicios y encontrar la causa raíz de un error.

- **P:** ¿Qué diferencia hay entre los niveles `error` y `fatal`?
  **R:** `error` es algo que falló para un usuario/request específico (el server sigue funcionando). `fatal` es algo que impide que el servicio arranque o funcione (DB caída, config rota).

- **P:** ¿Por qué Pino es más rápido que `console.log`?
  **R:** Serializa a JSON de forma asíncrona con `sonic-boom` (I/O no bloqueante) y los transports corren en threads separados, sin bloquear el event loop.

---

## Recursos

- [Pino](https://getpino.io/)
- [pino-pretty](https://github.com/pinojs/pino-pretty)