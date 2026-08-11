# Setup — Redis Insight

> **Tópico**: 6 — Bases de Datos & Persistencia (caching)
> **Objetivo**: Redis Insight instalado y conectado a Redis en Docker, listo para ver keys, valores y TTL.
> **Prerequisito**: Docker + Redis corriendo (`setup-redis.md`).

---

## ¿Por qué Redis Insight?

Es el cliente GUI oficial de Redis, gratis y multiplataforma. Con `redis-cli` podés hacer todo, pero para VER qué hay en la base — qué keys, de qué tipo, con cuánto TTL — Insight te lo muestra de una. Ojo: DBeaver también soporta Redis, pero Insight es el cliente oficial y con la mejor UI para este caso.

---

## Checklist

### 1. Descargar e instalar Redis Insight

Andá a [redis.io/insight](https://redis.io/insight/) y bajá el instalador de tu sistema operativo (gratis). Instalá con los valores por defecto.

- [ ] Redis Insight abre sin errores

### 2. Conectar a Redis en Docker

1. Asegurate de que Redis esté corriendo: `docker compose up -d redis`.
2. En Insight: **Add Redis Database** → **Enter manually**.
3. Completá:
   - **Host**: `localhost`
   - **Port**: `6379`
   - **Database**: dejá la default (0)
4. El `compose.yaml` de `setup-redis.md` no define password → dejá el campo de contraseña vacío.
5. **Add Redis Database**.

- [ ] La conexión es exitosa

### 3. Ver keys y valores

- En el browser de keys ves todas las keys de la base.
- Clic en una key → ves el **valor y el tipo** (string, hash, list, etc.).

### 4. Ver el TTL de una key

- Clic en una key con expiración → Insight te muestra el **TTL** en segundos.
- Probálo con tu app: cuando tu código Node hace `redis.set('user:1', JSON.stringify(user), 'EX', 60)` (el ejemplo de `setup-redis.md`), la key aparece con TTL de 60 segundos que va bajando en vivo.

### 5. Crear una key manual para probar

Si todavía no tenés keys, creá una desde el contenedor y mirala aparecer en Insight:

```bash
docker compose exec redis redis-cli set test "hola"
```

---

## Verificación

```bash
docker compose up -d redis
docker compose exec redis redis-cli set test "hola"
```

Abrí Redis Insight, conectá a `localhost:6379` y buscá la key `test`.

**Si ves la key `test` (o las keys que creó tu app) con su valor y TTL → Redis Insight listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Connection refused" | Redis no está corriendo: `docker compose up -d redis` |
| Pide password y no tengo | El compose de `setup-redis.md` no define `requirepass`: dejá el password vacío |
| No veo ninguna key | La base está vacía: creá una con `redis-cli set test "hola"` o corré tu app |
| DBeaver también soporta Redis | Cierto, pero Insight es el cliente oficial con la mejor UI para ver TTL y tipos |

---

## Preguntas de repaso

- **P:** ¿Qué datos usás para conectar Redis Insight a Redis en Docker?
  **R:** Host: `localhost`, Port: `6379`, Database: 0 (default), sin password.
- **P:** ¿Por qué dejás el campo de contraseña vacío en Redis Insight?
  **R:** Porque el `compose.yaml` de `setup-redis.md` no define `requirepass`, así que Redis arranca sin autenticación.
- **P:** ¿Cómo creás una key de prueba desde la terminal para verla en Insight?
  **R:** `docker compose exec redis redis-cli set test "hola"`.
- **P:** ¿Qué información te muestra Redis Insight sobre una key con expiración?
  **R:** El valor, el tipo de dato, y el TTL en segundos que va bajando en tiempo real.
- **P:** ¿Qué diferencia hay entre Redis Insight y DBeaver para conectar a Redis?
  **R:** Insight es el cliente oficial de Redis con la mejor UI para ver TTL, tipos y memoria. DBeaver también soporta Redis pero con una interfaz más genérica.

---

## Recursos

- [Redis Insight](https://redis.io/insight/)
- [Redis Docker Image](https://hub.docker.com/_/redis)
