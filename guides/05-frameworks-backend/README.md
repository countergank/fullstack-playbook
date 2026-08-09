# 05 — Frameworks & Herramientas Backend: Orden de Ejecución

> ⚠️ **El orden importa.**

## Prerequisito

Proyecto Express creado (`04-backend/setup-express-project.md`), PostgreSQL y Redis corriendo en Docker (`06-databases`).

## Paso a paso

1. **[setup-nestjs.md](setup-nestjs.md)** — NestJS con CLI oficial (`nest new` + `nest generate resource`).
2. **[setup-fastify.md](setup-fastify.md)** — Fastify + TypeScript con validación JSON Schema nativa.
3. **[setup-prisma.md](setup-prisma.md)** — Prisma ORM: schema, migraciones, cliente tipado, repository.
4. **[setup-mongoose.md](setup-mongoose.md)** — Mongoose ODM: schemas, hooks, populate, MongoDB.
5. **[setup-logging.md](setup-logging.md)** — Pino: request ID, niveles de log, pretty-print en desarrollo.
6. **[setup-queues.md](setup-queues.md)** — BullMQ: colas, workers, jobs asíncronos con Redis.
7. **[setup-websockets.md](setup-websockets.md)** — socket.io: eventos bidireccionales, rooms, broadcast.
8. **[setup-sse.md](setup-sse.md)** — Server-Sent Events: eventos unidireccionales, `EventSource`, broadcast.

---

## Verificación final

```bash
# Prisma
npx prisma studio                                      # UI muestra los modelos

# Logging
curl http://localhost:3000/health                       # log con request ID y duración

# Queues
npm run worker                                           # worker procesa jobs sin errores

# WebSockets (abrir test.html en navegador)
# Crear un usuario → la consola muestra "Nuevo usuario: ..."

# SSE (abrir test-sse.html en navegador)
# La consola muestra eventos cada 5 segundos
```

**Si los 5 funcionan → herramientas backend listas. ✅**
