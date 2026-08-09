# 05 — Frameworks & Herramientas Backend: Orden de Ejecución

> ⚠️ **El orden importa.** Prisma va primero (es la base de datos de la app), después logging (transversal), después queues y WebSockets (features adicionales).

## Prerequisito

Proyecto Express creado (`04-backend/setup-express-project.md`), PostgreSQL y Redis corriendo en Docker (`06-databases`).

## Paso a paso

1. **[setup-prisma.md](setup-prisma.md)** — Prisma ORM: schema, migraciones, cliente tipado, repository.
2. **[setup-logging.md](setup-logging.md)** — Pino: request ID, niveles de log, pretty-print en desarrollo.
3. **[setup-queues.md](setup-queues.md)** — BullMQ: colas, workers, jobs asíncronos con Redis.
4. **[setup-websockets.md](setup-websockets.md)** — socket.io: eventos en tiempo real, rooms, broadcast.

---

## Verificación final

```bash
# Prisma
npx prisma studio                              # UI muestra los modelos

# Logging
curl http://localhost:3000/health               # log con request ID y duración

# Queues
npm run worker                                   # worker procesa jobs sin errores

# WebSockets (abrir test.html en navegador)
# Crear un usuario → la consola del navegador muestra "Nuevo usuario: ..."
```

**Si los 4 funcionan → herramientas backend listas. ✅**
