# 5. Frameworks & Herramientas Backend

> Objetivo: conocer los frameworks y herramientas que transforman Node.js crudo en un backend profesional — cuándo usar cada uno, cómo combinarlos, y qué aporta cada pieza del ecosistema.

---

## 5.1 Express — El Estándar

*En criollo:* Express es el abuelo de los frameworks Node. Minimalista, sin opiniones fuertes, te deja armar la arquitectura como quieras. No te impone estructura de carpetas, no te obliga a usar TypeScript, no te dice cómo organizar tus middlewares. Es un lienzo en blanco — y como todo lienzo en blanco, depende de VOS que el resultado sea una obra maestra o un desastre.

### Lo que Express hace bien
- **Simplicidad**: 5 líneas y tenés un servidor HTTP respondiendo.
- **Ecosistema**: miles de middlewares listos para usar (cors, helmet, morgan, rate-limit, multer, passport).
- **Flexibilidad**: armás la arquitectura como quieras (MVC, clean, hexagonal, o un solo archivo de 5000 líneas — Express no te juzga, pero tu equipo sí).
- **Curva de aprendizaje**: bajísima. Es el primer framework que todo dev Node debería tocar.

### Lo que Express NO hace (y tenés que resolver vos)
- Validación de inputs (necesitás Zod/Joi).
- Estructura de proyecto (necesitás disciplina).
- TypeScript nativo (funciona pero necesita config extra).
- Inyección de dependencias.
- Manejo de errores tipado.
- Documentación de API automática (necesitás swagger-jsdoc o similar).

### Middlewares — el superpoder de Express

```js
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';

const app = express();

// Orden IMPORTA: los middlewares se ejecutan en el orden en que se registran
app.use(helmet());                          // 1. Seguridad primero
app.use(cors({ origin: process.env.CORS_ORIGIN })); // 2. CORS
app.use(express.json());                    // 3. Parsear body
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 })); // 4. Rate limiting
app.use(requestLogger);                     // 5. Logging

// 6. Rutas
app.use('/api/users', userRoutes);
app.use('/api/auth', authRoutes);

// 7. Error handler SIEMPRE al final (4 parámetros)
app.use(errorHandler);
```

### Estructura recomendada con Express

```
src/
├── config/         # variables de entorno, configuración de DB, constantes
├── modules/        # feature-based: cada módulo contiene sus propias capas
│   ├── users/
│   │   ├── users.controller.ts   # recibe HTTP, valida input, arma respuesta
│   │   ├── users.service.ts      # lógica de negocio pura
│   │   ├── users.repository.ts   # acceso a datos (habla con Prisma/Mongoose)
│   │   ├── users.routes.ts       # define rutas y middlewares específicos
│   │   ├── users.schema.ts       # validación Zod para este módulo
│   │   └── users.test.ts
│   └── auth/
│       └── ...
├── shared/         # middlewares, errores, tipos, utils compartidos
│   ├── middlewares/
│   │   ├── auth.ts
│   │   ├── errorHandler.ts
│   │   └── validate.ts
│   ├── errors/
│   │   └── AppError.ts
│   └── types/
└── app.ts          # crea y configura la app Express
```

> **Regla de oro**: feature-based > layer-based. `users/` contiene TODO sobre users (controller + service + repository + routes + tests). No separar por capa (`controllers/`, `services/`, `routes/`). Feature-based escala, layer-based se vuelve un infierno de imports cruzados.

---

## 5.2 NestJS — El Framework Opinado

*En criollo:* Si Express es un lienzo en blanco, NestJS es un kit de construcción con manual de instrucciones. Viene con arquitectura predefinida (modulos, controladores, servicios, pipes, guards, interceptors), TypeScript nativo, inyección de dependencias, y una CLI que genera código boilerplate. Es el framework que usarías si venís de Angular o Spring Boot y querés esa misma experiencia en Node.

### Comparación Express vs NestJS

| Aspecto | Express | NestJS |
|---------|---------|--------|
| Filosofía | Minimalista, sin opiniones | Opinado, arquitectura predefinida |
| Estructura | La definís vos | Módulos, controladores, providers |
| TypeScript | Opcional (config manual) | Nativo, first-class |
| DI (Inyección) | No incluida | Built-in (inspirado en Angular) |
| CLI | No | `nest generate` (crea módulos, controladores, servicios) |
| Testing | Manual (Jest/Vitest) | Utilities integradas + TestingModule |
| Curva de aprendizaje | Baja | Media-alta |
| Mejor para | APIs simples, MVPs rápidos, microservicios livianos | Aplicaciones enterprise, equipos grandes, código que va a crecer mucho |

### Cuándo elegir cada uno

- **Express**: estás aprendiendo backend, el proyecto es chico/mediano, querés control total sobre la arquitectura, o el equipo ya conoce Express.
- **NestJS**: el proyecto es enterprise, va a crecer mucho, el equipo valora estructura predecible, venís de Angular/Spring/.NET y querés esa experiencia.
- **Fastify**: necesitás máxima performance, validación de schema nativa, y un plugin system elegante. Alternativa moderna a Express con mejor DX.

---

## 5.3 Fastify — La Alternativa de Alto Rendimiento

*En criollo:* Fastify es Express reescrito con foco en velocidad. Tiene validación de schemas nativa (JSON Schema), plugin system, y es significativamente más rápido que Express en benchmarks. Si empezás un proyecto nuevo en 2026, Fastify es una opción muy sólida.

| Aspecto | Express | Fastify |
|---------|---------|---------|
| Rendimiento | Bueno | Excelente (2-3x más rápido) |
| Validación | Necesitás Zod/Joi aparte | Nativa con JSON Schema |
| Plugins | Middlewares (función simple) | Plugin system con encapsulamiento |
| TypeScript | Manual | Mejor soporte out-of-the-box |
| Ecosistema | Enorme (miles de middlewares) | Creciente, más chico que Express |
| Comunidad | Masiva | Grande y activa |

> Si estás aprendiendo, arrancá con Express. Si empezás un proyecto nuevo y valorás performance + validación nativa, evaluá Fastify.

---

## 5.4 Prisma — ORM Type-Safe para SQL

*En criollo:* Prisma traduce tu base de datos a tipos de TypeScript. Definís el schema en un archivo `.prisma`, generás el cliente, y tu editor te autocompleta TODAS las queries con tipos exactos. Si escribís `prisma.user.findUnique({ where: { email } })`, TypeScript sabe que `user` tiene `.id`, `.name`, `.email`, etc. Sin magia, sin strings sueltos, sin errores en producción por un typo en el nombre de una columna.

### Schema-first: así funciona

```prisma
// prisma/schema.prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  password  String
  role      Role     @default(USER)
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
}

enum Role {
  USER
  ADMIN
}
```

```ts
// Uso (tipado completo, sin strings mágicos)
const user = await prisma.user.findUnique({
  where: { email: 'lean@ejemplo.com' },
  include: { posts: { orderBy: { createdAt: 'desc' } } }
});
// user?.posts[0].title  ← TypeScript sabe que esto es string | undefined
```

### Migraciones
```bash
npx prisma migrate dev --name add-user-role   # crea migración desde el schema
npx prisma migrate deploy                      # aplica migraciones en producción
npx prisma studio                              # UI para explorar/editar datos
```

---

## 5.5 Mongoose — ODM para MongoDB

*En criollo:* Mongoose es a MongoDB lo que Prisma es a SQL — pero para documentos. Definís schemas con tipos, validaciones, hooks (middlewares de Mongoose), y poblaciones (el equivalente a joins de SQL pero entre colecciones). Es el estándar para MongoDB en Node desde hace una década.

```js
import mongoose from 'mongoose';

const userSchema = new mongoose.Schema({
  email:    { type: String, required: true, unique: true, lowercase: true },
  name:     { type: String, trim: true },
  password: { type: String, required: true, select: false }, // nunca devuelto por defecto
  role:     { type: String, enum: ['user', 'admin'], default: 'user' }
}, { timestamps: true });

// Hook: hashear password antes de guardar
userSchema.pre('save', async function(next) {
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 12);
  }
  next();
});

export const User = mongoose.model('User', userSchema);
```

### Prisma + MongoDB: ¿cuándo?

El consenso que establecimos en el roadmap: **Prisma para SQL, Mongoose para MongoDB**. Prisma con Mongo funciona para CRUD simple pero no para aggregations avanzadas ni índices geoespaciales. Si tu proyecto es 100% MongoDB, quedate con Mongoose. Si es mixto (SQL principal + Mongo para ciertos datos), evaluá Prisma para unificar.

---

## 5.6 Logging — Structured Logging

*En criollo:* `console.log` es para debuggear en tu máquina. En producción necesitás logs estructurados (JSON) que un sistema como Datadog, Grafana o CloudWatch pueda indexar, buscar y graficar. Cada log debe tener: timestamp, nivel, mensaje, request ID, user ID, duración.

### Pino — el logger de Node más rápido

```js
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty', options: { colorize: true } }
    : undefined // JSON puro en producción
});

logger.info({ userId: 42, route: '/users' }, 'User list requested');
logger.error({ err, requestId }, 'Failed to connect to database');
```

### Niveles de log estándar
| Nivel | Cuándo |
|-------|--------|
| `fatal` | El servicio no puede arrancar (DB caída, config rota) |
| `error` | Algo falló para UN usuario/request (no crashea el server) |
| `warn` | Algo inesperado pero no crítico (rate limit alcanzado) |
| `info` | Eventos normales de negocio (usuario creado, orden completada) |
| `debug` | Información para desarrollo (no en producción) |
| `trace` | Máximo detalle (cada paso de un algoritmo) |

### Request ID — trazabilidad de punta a punta

```js
import { v4 as uuid } from 'uuid';

app.use((req, res, next) => {
  req.id = uuid();
  res.setHeader('X-Request-Id', req.id);
  next();
});

// Ahora cada log incluye requestId → podés seguir un request desde el frontend hasta la DB
logger.info({ requestId: req.id }, 'Processing request');
```

---

## 5.7 Jobs, Queues y Tareas Asíncronas

*En criollo:* Cuando un usuario hace clic en "comprar", no podés hacerlo esperar 10 segundos mientras mandás el email de confirmación, generás la factura PDF, y actualizás el stock en 3 servicios. Lo que hacés es ENCOLAR esas tareas para que se ejecuten después, y le respondés al usuario "compra confirmada" en 200ms. El queue es la lista de tareas pendientes, y los workers son los que las ejecutan una por una.

### BullMQ — el estándar en Node

BullMQ usa Redis como backend de colas. Soporta: reintentos, backoff exponencial, delays, prioridades, workers concurrentes, y UI de monitoreo.

```js
import { Queue, Worker } from 'bullmq';

// Crear cola
const emailQueue = new Queue('emails', { connection: { host: 'localhost', port: 6379 } });

// Encolar trabajo
await emailQueue.add('welcome-email', {
  userId: 42,
  email: 'lean@ejemplo.com'
});

// Worker que procesa los trabajos
const worker = new Worker('emails', async job => {
  await sendWelcomeEmail(job.data.email);
}, { connection: { host: 'localhost', port: 6379 } });

worker.on('completed', job => logger.info(`Email sent to ${job.data.email}`));
worker.on('failed', (job, err) => logger.error({ err }, `Email failed for ${job?.data.email}`));
```

### Patrones comunes con queues

| Patrón | Uso |
|--------|-----|
| Email / notificaciones | No bloquea el request del usuario |
| Generación de PDFs / reports | Pesado, se encola y se notifica cuando está listo |
| Procesamiento de imágenes | Redimensionar, optimizar, generar thumbnails |
| Webhooks a terceros | Reintentos con backoff si el tercero falla |
| Scheduled jobs | Ejecutar a una hora específica (recordatorios, limpieza) |

---

## 5.8 WebSockets

*En criollo:* HTTP es "pregunto y respondo". WebSocket es "mantenemos la línea abierta y hablamos cuando querramos". El servidor PUEDE mandarte datos sin que los pidas — ideal para chats, notificaciones en tiempo real, dashboards en vivo, juegos multiplayer.

### socket.io — el estándar en Node

```js
import { Server } from 'socket.io';

const io = new Server(httpServer, {
  cors: { origin: process.env.CORS_ORIGIN }
});

io.on('connection', (socket) => {
  logger.info({ socketId: socket.id }, 'Client connected');

  socket.on('chat:message', (msg) => {
    // Broadcast a todos MENOS al que envió
    socket.broadcast.emit('chat:message', msg);
  });

  socket.on('disconnect', () => {
    logger.info({ socketId: socket.id }, 'Client disconnected');
  });
});

// Enviar desde el servidor (ej: después de una operación REST)
app.post('/api/orders', async (req, res) => {
  const order = await createOrder(req.body);
  io.emit('order:created', order);  // notificar a todos los clientes conectados
  res.status(201).json(order);
});
```

### Eventos y rooms

```js
// Unir a un usuario a una room (ej: su dashboard personal)
socket.join(`user:${userId}`);

// Emitir SOLO a esa room
io.to(`user:${userId}`).emit('notification', { message: 'Tu orden fue confirmada' });

// Rooms dinámicas — ej: sala de chat de un proyecto
socket.join(`project:${projectId}`);
```

---

## 5.9 Server-Sent Events (SSE)

*En criollo:* SSE es el hermano menor de WebSockets. Solo va del servidor al cliente (one-way), pero es mucho más simple: usás HTTP común, el navegador se reconecta solo si se cae, y no necesitás una librería extra en el cliente. Ideal para feeds de noticias, barras de progreso, logs en tiempo real.

```js
app.get('/api/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.flushHeaders();

  const interval = setInterval(() => {
    res.write(`data: ${JSON.stringify({ time: new Date().toISOString() })}\n\n`);
  }, 1000);

  req.on('close', () => clearInterval(interval));
});
```

```js
// Cliente (navegador) — cero librerías
const source = new EventSource('/api/events');
source.onmessage = (e) => console.log(JSON.parse(e.data));
```

---

## 5.10 Elegir el Stack — Recomendación Final

| Si tu proyecto es... | Stack recomendado |
|----------------------|-------------------|
| API REST simple, aprendizaje | **Express + Prisma + PostgreSQL** |
| API REST enterprise, equipo grande | **NestJS + Prisma + PostgreSQL + BullMQ** |
| Máxima performance | **Fastify + Prisma + PostgreSQL** |
| Datos documentales puros (sin relaciones complejas) | **Express + Mongoose + MongoDB** |
| Tiempo real (chat, notificaciones) | Agregar **socket.io** al stack REST |
| Monolito que crece | Feature-based structure + queues para tareas pesadas |
| Microservicios | Express/Fastify + message broker (Redis, RabbitMQ) |

---

> **Check de comprensión**:
> 1. ¿Por qué feature-based (módulos por dominio) escala mejor que layer-based (carpetas por capa)?
> 2. ¿Cuándo elegirías NestJS sobre Express? ¿Y cuándo Fastify sobre ambos?
> 3. ¿Qué problema resuelve un job queue como BullMQ? ¿Por qué no mandar el email directamente en el controller?
> 4. ¿Cuál es la diferencia clave entre WebSockets y Server-Sent Events? ¿Cuándo usarías cada uno?
> 5. ¿Por qué `console.log` no es suficiente en producción? ¿Qué información debe tener cada log?
