# Setup — Redis (Docker)

> **Tópico**: 6 — Bases de Datos & Persistencia (caching)
> **Objetivo**: Redis corriendo en Docker, listo para caché, rate limiting y colas.
> **Prerequisito**: Docker (`setup-docker.md`).

---

## ¿Por qué Redis en Docker?

Redis vive en RAM y es rapidísimo. Un `docker compose up -d redis` y está listo. Sin instalación, sin configuración del sistema.

---

## Checklist

### 1. Agregar Redis a tu `compose.yaml`

```yaml
services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data

volumes:
  redisdata:
```

### 2. Levantar y verificar

```bash
docker compose up -d redis
docker compose ps

# Verificar que responde
docker compose exec redis redis-cli ping
```
- [ ] Devuelve `PONG`

### 3. Probar persistencia

```bash
docker compose exec redis redis-cli set test "hola"
docker compose exec redis redis-cli get test   # "hola"

# Apagar y volver a levantar
docker compose down && docker compose up -d redis
docker compose exec redis redis-cli get test   # "hola" (persistió)
```

### 4. Uso en tu app Node (ioredis)

```bash
npm install ioredis
```

```js
import Redis from 'ioredis';
const redis = new Redis('redis://localhost:6379');

// Cache-aside pattern
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);
  const user = await db.users.find(id);
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 60);
  return user;
}
```

### 5. Comandos útiles

```bash
docker compose up -d redis                            # levantar Redis
docker compose exec redis redis-cli                   # CLI interactivo
docker compose exec redis redis-cli MONITOR            # ver todos los comandos en vivo (debug)

# Dentro de redis-cli:
# SET nombre valor EX 60    set con expiración de 60 segundos
# GET nombre
# TTL nombre                tiempo de vida restante
# DEL nombre                borrar key
# FLUSHALL                  BORRA TODO — ojo
```

### 6. Patrones comunes

| Patrón | Uso |
|--------|-----|
| **Cache-aside** | Leer caché → si miss, leer DB y llenar caché |
| **Rate limiting** | `INCR` + `EXPIRE` por ventana (ej: 10 req/min por IP) |
| **Sesiones** | Guardar sesión en Redis en vez de memoria del server |
| **Colas (BullMQ)** | Redis como backend de jobs asíncronos |

---

## Verificación

```bash
docker compose exec redis redis-cli ping   # PONG
docker compose exec redis redis-cli set test 1  # OK
docker compose exec redis redis-cli get test    # "1"
```

**Si `ping` devuelve PONG → Redis listo. ✅**

---

## Recursos

- [Redis Docker Image](https://hub.docker.com/_/redis)
- [ioredis (Node client)](https://github.com/redis/ioredis)