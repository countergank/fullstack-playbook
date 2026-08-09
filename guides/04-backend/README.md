# 04 — Backend Core: Orden de Ejecución

> ⚠️ **El orden importa.** Cada guía asume la anterior completada.

## Prerequisito

Antes de empezar, asegurate de tener Docker con las bases de datos corriendo (guías `06-databases`):

```bash
docker compose ps   # postgres, mongo y redis deben estar "Up"
```

## Paso a paso

1. **[setup-express-project.md](setup-express-project.md)** — Crear proyecto Express + TypeScript con estructura en capas, endpoint de prueba.
2. **[setup-env-config.md](setup-env-config.md)** — Variables de entorno tipadas con Zod, `.env` + `.env.example`, fail-fast al arranque.

---

## Verificación final

```bash
curl http://localhost:3000/health   # {"status":"ok"}
npm run dev                          # arranca sin errores
```

**Si el endpoint de health responde y el server arranca → backend listo. ✅**
