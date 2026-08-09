# 06 — Bases de Datos & Persistencia: Orden de Ejecución

> ⚠️ **El orden importa.** Las tres guías usan Docker. PostgreSQL va primero porque es la base SQL principal del stack.

## Prerequisito

Docker corriendo con backend WSL2 (`setup-docker.md` de 02-programming).

## Paso a paso

1. **[setup-postgres.md](setup-postgres.md)** — PostgreSQL 16 en Docker + Prisma.
2. **[setup-mongodb.md](setup-mongodb.md)** — MongoDB 7 en Docker + Mongoose.
3. **[setup-redis.md](setup-redis.md)** — Redis 7 en Docker para caché y colas.

> Las tres bases comparten el mismo `compose.yaml`. Al finalizar, un solo `docker compose up -d` levanta todo el stack.

---

## Verificación final

```bash
docker compose ps                                                       # postgres, mongo y redis "Up"
docker compose exec db psql -U dev -d fullstack_dev -c "SELECT 1;"       # 1
docker compose exec mongo mongosh --eval "db.runCommand({ ping: 1 })"    # { ok: 1 }
docker compose exec redis redis-cli ping                                 # PONG
```

**Si los 3 responden → bases de datos listas. ✅**
