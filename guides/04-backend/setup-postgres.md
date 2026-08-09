# Setup — PostgreSQL (Instalación Local + Prisma)

> **Tópico**: 4 — Backend Core / 6 — Bases de Datos (PostgreSQL es la base SQL principal del stack)
> **Objetivo**: PostgreSQL corriendo en WSL, accesible desde tu app Node, listo para usar con Prisma.
> **Prerequisito**: WSL + terminal (`setup-wsl.md`, `setup-terminal.md`).

---

## ¿Por qué PostgreSQL?

Es la base SQL más usada en el stack moderno JS/TS (con Prisma es el combo más recomendado del mercado). Es robusta, open-source, con features avanzadas (JSONB, full-text search, extensions) que la hacen servir tanto para datos relacionales como para casos NoSQL-lite.

---

## Checklist

### 1. Instalar PostgreSQL en WSL
```bash
sudo apt update
sudo apt install -y postgresql postgresql-client
```
- [ ] `psql --version` responde

### 2. Arrancar el servicio
```bash
# En WSL los servicios no arrancan solos siempre — control manual:
sudo service postgresql start

# Verificar que está corriendo
sudo service postgresql status
# o
pg_lsclusters
```
- [ ] `pg_lsclusters` muestra el cluster con status `online`

### 3. Configurar acceso
```bash
# Cambiar al usuario postgres (el superusuario del sistema)
sudo -i -u postgres

# Crear tu usuario de desarrollo con password
psql -c "CREATE USER tu_usuario WITH PASSWORD 'tu_password';"

# Crear la base de desarrollo y asignar owner
psql -c "CREATE DATABASE fullstack_dev OWNER tu_usuario;"

# También podés crearte tu proprio rol con SUPERUSER para prácticas locales:
# ALTER USER tu_usuario WITH SUPERUSER;

# Salir
exit
```
- [ ] Puedés loguearte: `psql -U tu_usuario -d fullstack_dev -h localhost` (te pide password). 
  > Si no te deja, editá `sudo nano /etc/postgresql/*/main/pg_hba.conf` y cambiá `peer` a `md5` en la línea `host ... 127.0.0.1/32`.

### 4. Connect string (lo que usa tu app)
```
postgresql://tu_usuario:tu_password@localhost:5432/fullstack_dev
```

### 5. Prisma (cuando llegues al tópico 6)
```bash
# En tu proyecto Node:
npm install prisma @prisma/client
npx prisma init

# .env del proyecto:
DATABASE_URL="postgresql://tu_usuario:tu_password@localhost:5432/fullstack_dev"
```
- [ ] `npx prisma migrate dev --name init` corre sin error
- [ ] `npx prisma studio` abre la UI de Prisma viendo la base

### 6. Comandos útiles del día a día
```bash
# Listar bases
psql -l

# Entrar a una base
psql -U tu_usuario -d fullstack_dev

# Dentro de psql:
# \dt            listar tablas
# \d tabla       describir tabla
# \q             salir
# SELECT * FROM "User";   (Prisma usa nombres PascalCase entre comillas)
```

---

## Verificación

```bash
sudo service postgresql status          # online
psql -U tu_usuario -d fullstack_dev -h localhost -c "SELECT version();"
```
**Si `SELECT version()` devuelve la versión de Postgres → PostgreSQL listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `sudo service postgresql start` no persiste al reiniciar WSL | Agregá al final de `~/.zshrc`: `sudo service postgresql start` (o usá el snippet que inicia servicios al entrar a WSL) |
| "Peer authentication failed" | Es el `pg_hba.conf` (paso 3). Cambiá peer→md5 |
| Prisma no conecta a localhost | Asegurate que el servicio esté online y que el `DATABASE_URL` tenga `-h localhost` |
| Puerto 5432 ocupado | `sudo service postgresql stop`, cambiar puerto en `postgresql.conf` o resolver el conflicto |
| Puerto 5432 no accesible desde Docker/otra VM | En WSL2, expone el puerto nativamente. Si estás en contenedores, usar host.docker.internal |

---

## Recursos

- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [Prisma — Getting Started](https://www.prisma.io/docs/getting-started)
- [Explicación pg_hba.conf](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)