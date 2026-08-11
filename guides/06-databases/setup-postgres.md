# Setup — PostgreSQL (Docker)

> **Tópico**: 6 — Bases de Datos & Persistencia
> **Objetivo**: PostgreSQL corriendo en Docker, accesible desde tu app Node, listo para usar con Prisma.
> **Prerequisito**: Docker (`setup-docker.md`).

---

## ¿Por qué PostgreSQL en Docker?

Un solo `docker compose up -d` y tenés PostgreSQL corriendo. Sin instalarlo en WSL, sin servicios que arrancar manualmente, sin conflictos de versiones. Cuando no lo necesitás, `docker compose down` y chau. Los datos persisten en un volumen Docker.

---

## Checklist

### 1. Agregar PostgreSQL a tu `compose.yaml`

Agregá este servicio a tu archivo `compose.yaml` (el que creaste en `setup-docker.md`):

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: fullstack_dev
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### 2. Levantar y verificar

```bash
docker compose up -d db        # solo PostgreSQL (sin redis ni mongo)
docker compose ps               # STATUS debe decir "Up" (healthy)

# Verificar que acepta conexiones
docker compose exec db psql -U dev -d fullstack_dev -c "SELECT version();"
```
- [ ] El `SELECT version()` devuelve la versión de PostgreSQL

### 3. Connect string para tu app

```
postgresql://dev:dev@localhost:5432/fullstack_dev
```

> Docker Desktop con backend WSL2 expone `localhost` automáticamente.

### 4. Prisma (cuando configures tu proyecto backend)

```bash
npm install prisma @prisma/client
npx prisma init
```

En el `.env` del proyecto:
```
DATABASE_URL="postgresql://dev:dev@localhost:5432/fullstack_dev"
```

```bash
npx prisma migrate dev --name init    # crea la migración inicial
npx prisma studio                      # UI para explorar la DB
```

### 5. Comandos útiles

```bash
docker compose up -d db                     # levantar PostgreSQL
docker compose down                         # apagar todo
docker compose exec db psql -U dev -d fullstack_dev  # shell SQL
docker compose logs -f db                   # logs en vivo

# Dentro de psql:
# \dt            listar tablas
# \d tabla       describir tabla
# \q             salir
```

---

## Verificación

```bash
docker compose ps | grep db            # "Up"
docker compose exec db psql -U dev -d fullstack_dev -c "SELECT 1;"  # 1
```

**Si el `SELECT 1` devuelve `1` → PostgreSQL listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Puerto 5432 ocupado | Otro PostgreSQL corriendo. `docker compose down` en otro proyecto, o cambiar `ports: - "5433:5432"` |
| "Connection refused" desde Node | El contenedor no está corriendo: `docker compose up -d db` |
| Prisma no conecta | Verificar que `DATABASE_URL` use `localhost` (no un IP de contenedor) |
| Datos se pierden al hacer `docker compose down` | Agregar `volumes:` como en el paso 1. Sin volumen, los datos mueren con el contenedor |
| "role dev does not exist" | Borrar el volumen y recrear: `docker compose down -v && docker compose up -d db` |

---

## Preguntas de repaso

- **P:** ¿Qué comando usás para levantar solo PostgreSQL sin los otros servicios del compose?
  **R:** `docker compose up -d db`.
- **P:** ¿Cuál es el connection string para conectar tu app Node a PostgreSQL en Docker?
  **R:** `postgresql://dev:dev@localhost:5432/fullstack_dev`.
- **P:** ¿Qué comando dentro de psql lista todas las tablas?
  **R:** `\dt`.
- **P:** ¿Por qué los datos se pierden al hacer `docker compose down` si no configuraste volumes?
  **R:** Porque sin volumen Docker, los datos viven solo dentro del contenedor efímero. Al destruir el contenedor, los datos se pierden.
- **P:** ¿Qué hace `npx prisma migrate deploy` y en qué entorno se usa?
  **R:** Aplica migrations pendientes sin crear nuevas. Se usa en producción/CI (a diferencia de `migrate dev` que es para desarrollo).

---

## Recursos

- [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- [Prisma — Getting Started](https://www.prisma.io/docs/getting-started)