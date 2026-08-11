# 5. Frameworks & Herramientas Backend

> Objetivo: conocer los frameworks y herramientas que transforman Node.js crudo en un backend profesional — cuándo usar cada uno, cómo combinarlos, y qué aporta cada pieza del ecosistema.

---

## 5.1 Express — El Estándar

*En criollo:* Express es el abuelo de los frameworks Node. Minimalista, sin opiniones fuertes, te deja armar la arquitectura como quieras. No te impone estructura de carpetas, no te obliga a usar TypeScript, no te dice cómo organizar tus middlewares. Es un lienzo en blanco — y como todo lienzo en blanco, depende de VOS que el resultado sea una obra maestra o un desastre.

*Técnicamente:* Express es una capa sobre el módulo `http` de Node.js. Intercepta requests entrantes, los pasa por una cadena de middleware functions `(req, res, next)`, y rutea al handler correspondiente según el HTTP method y path. No tiene compilador, no genera código, no impone estructura — es literalmente un router con middleware stack.

> [Express.js — Official Docs](https://expressjs.com/en/4x/api.html)
> [Express — Routing Guide](https://expressjs.com/en/guide/routing.html)

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

> **Check de comprensión**
> 1. ¿Por qué el orden de los middlewares en Express importa?
>    - R: Porque se ejecutan secuencialmente en el orden de registro. Si `express.json()` va después de tus rutas, los bodies llegan sin parsear.
> 2. ¿Qué diferencia hay entre `app.use()` y `app.get()`/`app.post()`?
>    - R: `app.use()` aplica a TODOS los métodos HTTP (middleware genérico), mientras que `app.get()`/`app.post()` solo responden a ese método específico (route handler).
> 3. ¿Por qué el error handler necesita exactamente 4 parámetros `(err, req, res, next)`?
>    - R: Express identifica los error handlers por la firma de 4 parámetros. Si tiene 3, Express lo trata como middleware normal y nunca captura errores.
> 4. ¿Qué problema resuelve la estructura feature-based vs layer-based?
>    - R: Feature-based agrupa todo lo de un dominio en un solo lugar, evitando imports cruzados entre carpetas lejanas. Layer-based dispersa un feature en múltiples carpetas (`controllers/`, `services/`, `routes/`).
> 5. ¿Cuál es la responsabilidad de un controller vs un service?
>    - R: El controller recibe HTTP, valida input, llama al service, y arma la respuesta HTTP. El service contiene lógica de negocio pura, sin saber de HTTP ni de Express.

---

## 5.2 NestJS — El Framework Opinado

*En criollo:* Si Express es un lienzo en blanco, NestJS es un kit de construcción con manual de instrucciones. Viene con arquitectura predefinida (modulos, controladores, servicios, pipes, guards, interceptors), TypeScript nativo, inyección de dependencias, y una CLI que genera código boilerplate. Es el framework que usarías si venís de Angular o Spring Boot y querés esa misma experiencia en Node.

*Técnicamente:* NestJS usa decorators de TypeScript (`@Controller()`, `@Injectable()`, `@Get()`) y un contenedor de inversión de control (IoC) para la inyección de dependencias. Por debajo usa Express por defecto como HTTP adapter (pero puede swaparse por Fastify). Los módulos son unidades de encapsulamiento que registran providers, controllers, e imports en un grafo de dependencias resuelto en bootstrap.

> [NestJS — Official Docs](https://docs.nestjs.com/)
> [NestJS — Providers & DI](https://docs.nestjs.com/providers)

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

> **Check de comprensión**
> 1. ¿Qué es la inyección de dependencias y por qué NestJS la incluye de fábrica?
>    - R: Es un patrón donde las dependencias se proveen externamente (por el contenedor IoC) en lugar de crearlas manualmente. NestJS lo incluye para que los servicios sean testables, desacoplados, y reutilizables.
> 2. ¿Qué diferencia hay entre un Module, un Controller y un Provider en NestJS?
>    - R: El Module agrupa y registra componentes. El Controller maneja requests HTTP (rutas). El Provider contiene lógica de negocio o servicios inyectables.
> 3. ¿Por qué NestJS usa Express por defecto pero permite swap a Fastify?
>    - R: NestJS abstract el HTTP layer con adapters. Express es el default por compatibilidad, pero `@nestjs/platform-fastify` permite usar Fastify como motor sin cambiar la lógica de tu app.
> 4. ¿Cuándo conviene NestJS sobre Express?
>    - R: Cuando el proyecto es grande, el equipo es numeroso, se necesita estructura predecible, o venís de Angular/Spring y querés esa experiencia.
> 5. ¿Qué hace `nest generate resource` que no podés hacer manualmente en Express?
>    - R: Genera automáticamente un módulo completo con controller, service, DTOs, entity, module y tests — todo conectado y con CRUD boilerplate listo.

---

## 5.3 Fastify — La Alternativa de Alto Rendimiento

*En criollo:* Fastify es Express reescrito con foco en velocidad. Tiene validación de schemas nativa (JSON Schema), plugin system, y es significativamente más rápido que Express en benchmarks. Si empezás un proyecto nuevo en 2026, Fastify es una opción muy sólida.

*Técnicamente:* Fastify usa `find-my-router` (un radix tree router) que es más eficiente que el router lineal de Express. La validación con JSON Schema se compila a funciones JavaScript optimizadas con `ajv`, evitando overhead runtime. Los plugins usan `avvio` para encapsulamiento: cada plugin tiene su propio scope de middlewares y rutas, evitando colisiones.

> [Fastify — Official Docs](https://fastify.dev/docs/latest/)
> [Fastify — Getting Started](https://fastify.dev/docs/latest/Guides/Getting-Started/)

| Aspecto | Express | Fastify |
|---------|---------|---------|
| Rendimiento | Bueno | Excelente (2-3x más rápido) |
| Validación | Necesitás Zod/Joi aparte | Nativa con JSON Schema |
| Plugins | Middlewares (función simple) | Plugin system con encapsulamiento |
| TypeScript | Manual | Mejor soporte out-of-the-box |
| Ecosistema | Enorme (miles de middlewares) | Creciente, más chico que Express |
| Comunidad | Masiva | Grande y activa |

> Si estás aprendiendo, arrancá con Express. Si empezás un proyecto nuevo y valorás performance + validación nativa, evaluá Fastify.

> **Check de comprensión**
> 1. ¿En qué se diferencia el sistema de validación de Fastify del de Express?
>    - R: Fastify tiene validación nativa con JSON Schema que rechaza requests inválidos automáticamente. Express no tiene validación built-in — necesitás Zod, Joi, o similar aparte.
> 2. ¿Qué es el encapsulamiento de plugins en Fastify y por qué importa?
>    - R: Cada plugin en Fastify tiene su propio scope aislado. Los decorators y hooks registrados dentro de un plugin no "escapan" al scope global, evitando colisiones accidentales.
> 3. ¿Por qué Fastify es 2-3x más rápido que Express en benchmarks?
>    - R: Usa un radix tree router más eficiente, serialización optimizada, y evita overhead de middleware innecesario. Además, los handlers async son nativos.
> 4. ¿Cuándo NO conviene Fastify?
>    - R: Cuando necesitás middlewares de Express que no tienen equivalente en Fastify, o cuando tu equipo ya tiene expertise consolidado en Express y el proyecto es chico.
> 5. ¿Qué diferencia hay entre el plugin system de Fastify y los middlewares de Express?
>    - R: Los middlewares de Express son funciones planas que se ejecutan en orden global. Los plugins de Fastify tienen encapsulamiento (scope propio), lifecycle hooks, y pueden registrar rutas y decorators de forma aislada.

---

## 5.4 Prisma — ORM Type-Safe para SQL

*En criollo:* Prisma traduce tu base de datos a tipos de TypeScript. Definís el schema en un archivo `.prisma`, generás el cliente, y tu editor te autocompleta TODAS las queries con tipos exactos. Si escribís `prisma.user.findUnique({ where: { email } })`, TypeScript sabe que `user` tiene `.id`, `.name`, `.email`, etc. Sin magia, sin strings sueltos, sin errores en producción por un typo en el nombre de una columna.

*Técnicamente:* Prisma genera un cliente TypeScript a partir del schema declarativo (`schema.prisma`). El engine de Prisma (binario escrito en Rust) traduce las llamadas del cliente a SQL nativo del proveedor (PostgreSQL, MySQL, SQLite, SQL Server). Las migraciones se generan comparando el schema actual con el estado de la DB, produciendo SQL idempotente.

> [Prisma — Official Docs](https://www.prisma.io/docs)
> [Prisma — Schema Reference](https://www.prisma.io/docs/orm/prisma-schema)

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

> **Check de comprensión**
> 1. ¿Qué significa que Prisma sea "type-safe" y por qué importa?
>    - R: Que TypeScript conoce la estructura exacta de cada modelo y sus relaciones. Si cambiás el schema y regenerás el cliente, el compilador te marca errores en todo el código que usa campos inexistentes.
> 2. ¿Qué hace `npx prisma migrate dev` vs `npx prisma migrate deploy`?
>    - R: `dev` crea y aplica migraciones en desarrollo (puede resetear la DB si hay conflictos). `deploy` solo aplica migraciones existentes en producción — nunca resetea datos.
> 3. ¿Qué es `prisma generate` y cuándo se ejecuta?
>    - R: Genera el cliente TypeScript a partir del schema. Se ejecuta automáticamente después de `prisma migrate` y también con `npx prisma generate` manual. Debe correrse después de cada cambio al schema.
> 4. ¿Qué diferencia hay entre `include` y `select` en una query de Prisma?
>    - R: `include` trae el modelo completo MÁS las relaciones especificadas. `select` trae SOLO los campos que pedís (proyección), omitiendo el resto.
> 5. ¿Por qué Prisma no es ideal para MongoDB con aggregations complejas?
>    - R: Prisma abstrae SQL de forma elegante pero su capa sobre MongoDB es más limitada. Aggregations, índices geoespaciales, y queries textuales avanzadas se hacen mejor con Mongoose nativo.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.1-postgresql) para profundizar en PostgreSQL y modelado relacional.

---

## 5.5 Mongoose — ODM para MongoDB

*En criollo:* Mongoose es a MongoDB lo que Prisma es a SQL — pero para documentos. Definís schemas con tipos, validaciones, hooks (middlewares de Mongoose), y poblaciones (el equivalente a joins de SQL pero entre colecciones). Es el estándar para MongoDB en Node desde hace una década.

*Técnicamente:* Mongoose es un ODM (Object Document Mapper) que agrega un schema layer sobre MongoDB, que es schemaless por naturaleza. Los schemas definen tipos, defaults, validaciones, y middleware hooks (`pre`/`post`). El método `populate()` resuelve referencias entre colecciones haciendo queries adicionales (no es un JOIN real de SQL).

> [Mongoose — Official Docs](https://mongoosejs.com/docs/)
> [Mongoose — Schemas & Models](https://mongoosejs.com/docs/guide.html)

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

> **Check de comprensión**
> 1. ¿Qué hace un hook `pre('save')` en Mongoose y cuándo se ejecuta?
>    - R: Se ejecuta ANTES de cada operación `save()` en un documento. Se usa comúnmente para hashear passwords, generar slugs, o validar datos antes de persistir.
> 2. ¿Qué significa `select: false` en un campo de schema?
>    - R: Que ese campo NO se devuelve en las queries por defecto. Se usa para datos sensibles como passwords — hay que pedirlo explícitamente con `.select('+password')`.
> 3. ¿Qué diferencia hay entre `populate()` y un JOIN de SQL?
>    - R: `populate()` hace una query SEPARADA para traer los documentos referenciados (N+1 queries potenciales). Un JOIN de SQL resuelve todo en una sola query a nivel de motor de base de datos.
> 4. ¿Por qué MongoDB es "schemaless" pero Mongoose agrega schemas?
>    - R: MongoDB acepta cualquier estructura de documento. Mongoose agrega schemas para validar datos, definir tipos, y dar estructura al código — es una capa de disciplina sobre flexibilidad nativa.
> 5. ¿Cuándo conviene Mongoose sobre Prisma para MongoDB?
>    - R: Cuando necesitás aggregations complejas, índices geoespaciales, text search, o queries que el adapter de Prisma para Mongo no soporta bien.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.2-mongodb) para profundizar en MongoDB y modelado documental.

---

## 5.6 Logging — Structured Logging

*En criollo:* `console.log` es para debuggear en tu máquina. En producción necesitás logs estructurados (JSON) que un sistema como Datadog, Grafana o CloudWatch pueda indexar, buscar y graficar. Cada log debe tener: timestamp, nivel, mensaje, request ID, user ID, duración.

*Técnicamente:* Pino serializa logs a JSON de forma asíncrona (usando `sonic-boom` para I/O no bloqueante), lo que lo hace ~5x más rápido que `console.log`. Los transports (como `pino-pretty`) corren en un thread separado vía `worker_threads`, evitando bloquear el event loop. El request ID permite correlacionar todos los logs de un mismo request a través de múltiples servicios.

> [Pino — Official Docs](https://getpino.io/#/)
> [Pino — Transports](https://getpino.io/#/docs/transports)

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

> **Check de comprensión**
> 1. ¿Por qué `console.log` no es suficiente en producción?
>    - R: Porque produce texto plano sin estructura, sin niveles, sin request ID. Un sistema de monitoreo no puede indexarlo, buscarlo, ni graficarlo eficientemente.
> 2. ¿Qué información debe tener cada log de producción como mínimo?
>    - R: Timestamp, nivel (info/error/warn), mensaje, request ID, y contexto relevante (user ID, ruta, duración).
> 3. ¿Para qué sirve el request ID y cómo se propaga?
>    - R: Identifica unívocamente cada request. Se genera en el primer middleware, se agrega al header `X-Request-Id`, y se incluye en cada log. Permite seguir un request a través de múltiples servicios.
> 4. ¿Qué nivel de log usarías para "usuario creado exitosamente"? ¿Y para "base de datos caída"?
>    - R: `info` para "usuario creado" (evento normal de negocio). `fatal` o `error` para "base de datos caída" (el servicio no puede funcionar).
> 5. ¿Por qué Pino es más rápido que `console.log`?
>    - R: Porque serializa a JSON de forma asíncrona usando `sonic-boom` (I/O no bloqueante) y los transports corren en threads separados, sin bloquear el event loop.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.5-manejo-de-errores) para el manejo de errores que complementa el logging.

---

## 5.7 Jobs, Queues y Tareas Asíncronas

*En criollo:* Cuando un usuario hace clic en "comprar", no podés hacerlo esperar 10 segundos mientras mandás el email de confirmación, generás la factura PDF, y actualizás el stock en 3 servicios. Lo que hacés es ENCOLAR esas tareas para que se ejecuten después, y le respondés al usuario "compra confirmada" en 200ms. El queue es la lista de tareas pendientes, y los workers son los que las ejecutan una por una.

*Técnicamente:* BullMQ usa Redis como broker de mensajes. Los jobs se serializan a JSON y se almacenan en listas ordenadas de Redis. Los workers hacen polling con `BRPOPLPUSH` (operación atómica de Redis) para consumir jobs. Soporta reintentos con backoff exponencial, delays programados, prioridades, y rate limiting por worker.

> [BullMQ — Official Docs](https://docs.bullmq.io/)
> [BullMQ — Patterns](https://docs.bullmq.io/patterns/)

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

> **Check de comprensión**
> 1. ¿Por qué no mandar un email de confirmación directamente en el controller?
>    - R: Porque bloquea la respuesta HTTP hasta que el email se envía (puede tardar segundos). Si el servicio de email falla, el usuario recibe un error. Encolar responde en 200ms y procesa en background.
> 2. ¿Qué es el backoff exponencial en reintentos?
>    - R: Es una estrategia donde el tiempo entre reintentos crece exponencialmente (1s, 2s, 4s, 8s...). Evita saturar un servicio externo que está fallando temporalmente.
> 3. ¿Qué pasa si un worker se cae mientras procesa un job?
>    - R: BullMQ detecta que el job no fue completado y lo reencola automáticamente después de un timeout. El job no se pierde.
> 4. ¿Por qué BullMQ usa Redis y no una base de datos SQL?
>    - R: Porque Redis ofrece operaciones atómicas de cola (`BRPOPLPUSH`) con latencia sub-milisegundo. Una DB SQL tendría overhead de transacciones y locks que harían el polling mucho más lento.
> 5. ¿Cuándo NO conviene usar un queue?
>    - R: Cuando la tarea es rápida (<100ms) y el usuario necesita el resultado inmediatamente. Los queues agregan complejidad infraestructural (Redis, workers, monitoreo).

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.3-manejo-de-estado) para entender por qué las tareas asíncronas son clave en arquitecturas escalables.

---

## 5.8 WebSockets

*En criollo:* HTTP es "pregunto y respondo". WebSocket es "mantenemos la línea abierta y hablamos cuando querramos". El servidor PUEDE mandarte datos sin que los pidas — ideal para chats, notificaciones en tiempo real, dashboards en vivo, juegos multiplayer.

*Técnicamente:* WebSocket inicia como un HTTP request con header `Upgrade: websocket`. El servidor responde `101 Switching Protocols` y la conexión se convierte en un canal bidireccional persistente sobre TCP. socket.io agrega fallback automático (polling → WebSocket), rooms, namespaces, y reconexión automática sobre el protocolo WebSocket nativo.

> [socket.io — Official Docs](https://socket.io/docs/v4/)
> [MDN — WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)

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

> **Check de comprensión**
> 1. ¿Qué diferencia fundamental hay entre HTTP y WebSockets?
>    - R: HTTP es request-response (el cliente siempre inicia). WebSocket es bidireccional y persistente — ambos lados pueden enviar datos en cualquier momento sin esperar un request.
> 2. ¿Qué son las "rooms" en socket.io y para qué sirven?
>    - R: Son grupos de sockets. Permiten emitir mensajes a un subconjunto de clientes conectados (ej: todos los usuarios de un proyecto) en lugar de broadcast a todos.
> 3. ¿Qué hace `socket.broadcast.emit()` vs `io.emit()`?
>    - R: `broadcast.emit()` envía a todos MENOS al socket que originó el evento. `io.emit()` envía a TODOS los clientes conectados, incluyendo al emisor.
> 4. ¿Por qué socket.io es preferible al WebSocket nativo del navegador?
>    - R: Porque agrega reconexión automática, fallback a HTTP polling si WebSocket falla, rooms, namespaces, y manejo de heartbeats — todo lo que tendrías que implementar manualmente.
> 5. ¿Cómo se integra WebSockets con una API REST existente?
>    - R: El REST maneja CRUD y operaciones síncronas. Los WebSockets notifican cambios en tiempo real. Ej: POST /api/orders crea la orden (REST) + `io.emit('order:created')` notifica a clientes (WS).

→ Ver [Tópico 5: SSE](#5.9-server-sent-events-sse) para la alternativa one-way más simple.

---

## 5.9 Server-Sent Events (SSE)

*En criollo:* SSE es el hermano menor de WebSockets. Solo va del servidor al cliente (one-way), pero es mucho más simple: usás HTTP común, el navegador se reconecta solo si se cae, y no necesitás una librería extra en el cliente. Ideal para feeds de noticias, barras de progreso, logs en tiempo real.

*Técnicamente:* SSE usa el header `Content-Type: text/event-stream` y mantiene la conexión HTTP abierta con `Connection: keep-alive`. El servidor escribe datos en formato `data: {...}\n\n`. El navegador usa `EventSource` API nativa que maneja reconexión automática con `Last-Event-ID` para reanudar desde donde quedó.

> [MDN — Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
> [MDN — EventSource API](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)

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

> **Check de comprensión**
> 1. ¿Cuál es la diferencia clave entre SSE y WebSockets?
>    - R: SSE es unidireccional (solo servidor → cliente), WebSockets es bidireccional. SSE usa HTTP común, WebSockets requiere un protocolo diferente después del upgrade.
> 2. ¿Por qué SSE se reconecta automáticamente y WebSockets no?
>    - R: Porque `EventSource` es una API nativa del navegador que implementa reconexión automática con backoff. WebSocket nativo no la incluye — hay que programarla manualmente o usar socket.io.
> 3. ¿Qué formato deben tener los datos enviados por SSE?
>    - R: `data: <payload>\n\n` — cada mensaje termina con doble newline. Se pueden usar `event: <nombre>` para tipos y `id: <id>` para reconexión con reanudación.
> 4. ¿Cuándo elegirías SSE sobre WebSockets?
>    - R: Cuando solo necesitás notificaciones del servidor al cliente (feeds, progreso, logs) y querés la solución más simple sin librerías extra ni infraestructura adicional.
> 5. ¿Qué headers son obligatorios para un endpoint SSE?
>    - R: `Content-Type: text/event-stream`, `Cache-Control: no-cache`, y `Connection: keep-alive`. Sin ellos, el navegador no trata la respuesta como stream de eventos.

→ Ver [Tópico 5: WebSockets](#5.8-websockets) para comunicación bidireccional cuando el cliente también necesita enviar datos en tiempo real.

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

> **Check de comprensión**
> 1. ¿Por qué la recomendación para aprendizaje es Express + Prisma + PostgreSQL?
>    - R: Porque Express tiene la curva más baja, Prisma da type-safety sin complejidad, y PostgreSQL es el SQL más usado en la industria. Juntos cubren el 80% de los casos reales.
> 2. ¿Qué agrega BullMQ al stack enterprise que no tiene el stack simple?
>    - R: Agrega procesamiento asíncrono de tareas pesadas (emails, reports, webhooks) sin bloquear requests. Es necesario cuando el volumen justifica separar trabajo del request-response cycle.
> 3. ¿Cuándo conviene Express + Mongoose + MongoDB sobre Express + Prisma + PostgreSQL?
>    - R: Cuando los datos son principalmente documentales (sin relaciones complejas), el schema cambia frecuentemente, o necesitás flexibilidad de estructura por registro.
> 4. ¿Por qué Fastify + Prisma es la recomendación para máxima performance?
>    - R: Fastify es 2-3x más rápido que Express en benchmarks, y Prisma agrega mínimo overhead comparado con queries raw. Juntos dan performance + type-safety.
> 5. ¿Qué significa "feature-based structure + queues para monolito que crece"?
>    - R: Organizar el código por dominio (users/, auth/, orders/) en lugar de por capa. Cuando crece, las tareas pesadas se mueven a queues para no bloquear el monolito, sin necesitar microservicios todavía.

