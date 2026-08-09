# 04 — Backend Core: Orden de Ejecución

> ⚠️ **El orden importa.** Cada guía asume Docker ya instalado (guía 02).

## Paso a paso

1. **[setup-postgres.md](setup-postgres.md)** — PostgreSQL corriendo en Docker (o nativo WSL si preferís).
2. **[setup-mongodb.md](setup-mongodb.md)** — MongoDB corriendo en Docker (o nativo WSL).
3. **[setup-redis.md](setup-redis.md)** — Redis corriendo en Docker (o nativo WSL).

> Las tres guías incluyen opciones para instalación nativa Y con Docker. Si ya completaste `setup-docker.md`, usar Docker es más simple: `docker compose up -d` y listo.

---

## Verificación final

```bash
docker compose ps                        # los 3 servicios corriendo
psql -h localhost -U dev -d fullstack_dev -c "SELECT version();"  # PostgreSQL
mongosh --eval "db.runCommand({ ping: 1 })"                        # MongoDB
redis-cli ping                                                      # Redis
```

**Si los 3 responden → el entorno del tópico 4 está completo. ✅**
