# Setup — Jobs y Colas con BullMQ

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.7)
> **Objetivo**: configurar BullMQ para tareas asíncronas (emails, procesamiento) usando Redis como backend.
> **Prerequisito**: proyecto Express (`setup-express-project.md`), Redis corriendo en Docker (`setup-redis.md` de 06-databases).

---

## ¿Por qué colas de trabajos?

Cuando un usuario se registra, no debería esperar 5 segundos a que se envíe el email de bienvenida, se genere el PDF de bienvenida, y se actualice el CRM. Con una cola, el controller responde en 200ms y encola las tareas pesadas. Los workers las procesan en background, con reintentos automáticos si algo falla. Es la diferencia entre un server que responde instantáneamente y uno que bloquea al usuario.

---

## Checklist

### 1. Instalar BullMQ

```bash
npm install bullmq
```

### 2. Configurar la conexión a Redis

```ts
// src/shared/queue/connection.ts
import { Redis } from 'ioredis';

const connection = new Redis('redis://localhost:6379', { maxRetriesPerRequest: null });

export { connection };
```

### 3. Crear una cola

```ts
// src/shared/queue/emailQueue.ts
import { Queue } from 'bullmq';
import { connection } from './connection.js';

export interface EmailJob {
  to: string;
  subject: string;
  body: string;
}

export const emailQueue = new Queue<EmailJob>('emails', { connection });
```

### 4. Crear un worker

```ts
// src/workers/emailWorker.ts
import { Worker } from 'bullmq';
import { connection } from '../shared/queue/connection.js';
import { logger } from '../shared/logger.js';

const worker = new Worker('emails', async (job) => {
  const { to, subject, body } = job.data;

  logger.info({ jobId: job.id, to, subject }, 'Sending email');
  // await sendEmail(to, subject, body); ← acá iría la integración real
  logger.info({ jobId: job.id }, 'Email sent');
}, { connection });

worker.on('completed', (job) => {
  logger.info({ jobId: job.id }, 'Job completed');
});

worker.on('failed', (job, err) => {
  logger.error({ jobId: job?.id, err }, 'Job failed');
});
```

### 5. Agregar script para el worker

```jsonc
// package.json
{
  "scripts": {
    "dev": "tsx watch src/app.ts",
    "worker": "tsx watch src/workers/emailWorker.ts"
  }
}
```

### 6. Encolar trabajos desde un controller

```ts
// src/modules/users/users.controller.ts
import { emailQueue } from '../../shared/queue/emailQueue.js';
import type { Request, Response } from 'express';

export async function register(req: Request, res: Response) {
  const user = await userService.register(req.body);

  // Encolar email de bienvenida (no bloquea la respuesta)
  await emailQueue.add('welcome-email', {
    to: user.email,
    subject: 'Bienvenido!',
    body: `Hola ${user.name}, gracias por registrarte.`,
  });

  res.status(201).json({ data: user });
}
```

### 7. Ejecutar

```bash
# Terminal 1: el server
npm run dev

# Terminal 2: el worker
npm run worker
```

- [ ] Al crear un usuario, el worker procesa el email en background. La respuesta HTTP es instantánea.

---

## Verificación

```bash
# Redis debe estar corriendo
docker compose exec redis redis-cli ping   # PONG

# El worker loguea "Job completed" al procesar un trabajo
```

**Si el worker procesa trabajos sin bloquear el server → BullMQ listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `Error: connect ECONNREFUSED 127.0.0.1:6379` | Redis no está corriendo. Verificá con `docker compose exec redis redis-cli ping` — debe responder `PONG`. |
| El worker no procesa jobs | Asegurate de que el worker está corriendo en una terminal separada (`npm run worker`) y que el nombre de la cola coincide (`new Queue('emails')` vs `new Worker('emails')`). |
| Jobs fallidos se reintentan infinitamente | Configurá `attempts` y `backoff` en el job: `queue.add('job', data, { attempts: 3, backoff: { type: 'exponential', delay: 1000 } })`. |
| `maxRetriesPerRequest: null` warning en ioredis | Es requerido por BullMQ. Sin este setting, ioredis limita reintentos y BullMQ no puede manejar jobs fallidos correctamente. |

---

## Preguntas de repaso

- **P:** ¿Por qué encolar tareas en lugar de ejecutarlas directamente en el controller?
  **R:** Porque tareas pesadas (emails, PDFs, webhooks) bloquean la respuesta HTTP. Encolar responde en 200ms y procesa en background con reintentos automáticos.

- **P:** ¿Qué rol cumple Redis en BullMQ?
  **R:** Redis es el broker de mensajes. Almacena los jobs en listas ordenadas y los workers los consumen con operaciones atómicas (`BRPOPLPUSH`).

- **P:** ¿Qué es el backoff exponencial?
  **R:** Una estrategia de reintentos donde el tiempo entre intentos crece exponencialmente (1s, 2s, 4s, 8s...). Evita saturar un servicio externo que está fallando.

- **P:** ¿Qué pasa si un worker se cae mientras procesa un job?
  **R:** BullMQ detecta que el job no fue completado y lo reencola automáticamente después de un timeout. El job no se pierde.

- **P:** ¿Por qué el worker corre en un proceso separado del server?
  **R:** Para aislar el procesamiento de jobs del request-response cycle. Si un job pesado consume mucha CPU, no afecta la respuesta HTTP de otros usuarios.

---

## Recursos

- [BullMQ — Guide](https://docs.bullmq.io/)
- [BullMQ — Bull Board (UI)](https://github.com/felixmosh/bull-board)