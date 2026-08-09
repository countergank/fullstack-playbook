# Setup — Jobs y Colas con BullMQ

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.7)
> **Objetivo**: configurar BullMQ para tareas asíncronas (emails, procesamiento) usando Redis como backend.
> **Prerequisito**: proyecto Express (`setup-express-project.md`), Redis corriendo en Docker (`setup-redis.md` de 06-databases).

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

## Recursos

- [BullMQ — Guide](https://docs.bullmq.io/)
- [BullMQ — Bull Board (UI)](https://github.com/felixmosh/bull-board)