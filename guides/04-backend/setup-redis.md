# Setup — Redis (Caché y Colas)

> **Tópico**: 4 — Backend Core / 6 — Bases de Datos (caching)
> **Objetivo**: Redis corriendo en WSL, entendido en sus dos roles principales: caché de lecturas y cola/rate-limit.
> **Prerequisito**: WSL + terminal (`setup-wsl.md`, `setup-terminal.md`).

---

## ¿Por qué Redis?

*En criollo:* Redis es una base de datos EN MEMORIA — vive en RAM, no en disco. Por eso es rapidísima (microsegundos). No es para tus datos de negocio (¿y si se apaga el server?), es para los datos que necesitás MUY rápido: cachés, sesiones, rate limits, colas de trabajos.

---

## Checklist

### 1. Instalar Redis
```bash
sudo apt update
sudo apt install -y redis-server redis-tools
```
- [ ] `redis-server --version` responde

### 2. Configurar + arrancar
```bash
# Ajustar para que no pida systemd (WSL):
sudo sed -i 's/^supervised auto/supervised no/' /etc/redis/redis.conf

# Arrancar
sudo service redis-server start

# Verificar
redis-cli ping    # debe responder PONG
```

### 3. Verificar persistencia por defecto
Redis guarda en disco (snapshots RDB por defecto). Verificá que el dumpfile existe tras una escritura:
```bash
redis-cli set test "hola"
redis-cli get test          # "hola"
ls /var/lib/redis/          # deberías ver dump.rdb tras un snapshot
```

### 4. Uso en tu app Node (cuando llegues a caching real)
```bash
npm install ioredis
```
```js
import Redis from 'ioredis';
const redis = new Redis('redis://localhost:6379');

// Caché simple
await redis.set('user:42', JSON.stringify(user), 'EX', 60); // expira en 60s
const cached = await redis.get('user:42');

// Cache-aside pattern (el estándar):
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);
  const user = await db.users.find(id);      // miss → leer de la DB
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 60);
  return user;
}
```

### 5. Comandos útiles
```bash
redis-cli SET nombre valor EX 60   # set con expiración
redis-cli GET nombre               # get
redis-cli TTL nombre               # tiempo de vida restante (-1 = sin exp)
redis-cli EXISTS nombre            # 1 si existe
redis-cli DEL nombre               # borrar
redis-cli KEYS 'user:*'            # buscar keys (no usar en prod con muchos keys)
redis-cli FLUSHALL                 # BORRA TODO — ojo
redis-cli MONITOR                  # ver todos los comandos en vivo (debug)
```

### 6. Patrones que vas a usar
| Patrón | Uso |
|--------|-----|
| **Cache-aside** | Leer caché → si miss, leer DB y llenar caché |
| **Rate limiting** | `INCR key` + `EXPIRE` por ventana (ej: 10 req/min por IP) |
| **Sesiones** | Guardar sesión de usuario en vez de memoria del server |
| **Pub/Sub** | Comunicación entre servicios (menos usado que colas) |
| **Colas** | Con `BullMQ`/`ioredis` para jobs async (emails, procesamiento) |
| **Distributed lock** | Coordinar procesos: una sola instancia hace X |

---

## Verificación

```bash
redis-cli ping        # PONG
redis-cli set test 1  # OK
redis-cli get test    # 1
```
**Si `ping` devuelve PONG y podés set/get → Redis listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `redis-server` se cae en WSL | Verificar memoria (`free -h`) y ajustar `.wslconfig`. Redis es liviano, 1GB alcanza |
| Port 6379 ocupado | `sudo lsof -i :6379`, cambiar `port` en `/etc/redis/redis.conf` |
| No persiste al reiniciar | RDB por defecto snapshotea cada N secs; si querés más seguridad configurá AOF en `redis.conf` |
| KEYS lento en producción | Usar `SCAN` en vez de `KEYS`. O delegar a una estructura indexada |
| No conecta desde Node | Asegurate `redis-server` corriendo + host `localhost`/`127.0.0.1` |

---

## Recursos

- [Redis Docs](https://redis.io/docs/)
- [ioredis (Node client)](https://github.com/redis/ioredis)