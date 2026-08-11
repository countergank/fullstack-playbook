# Setup — DBeaver Community

> **Tópico**: 6 — Bases de Datos & Persistencia
> **Objetivo**: DBeaver Community instalado y conectado a PostgreSQL en Docker, listo para explorar las tablas de Prisma.
> **Prerequisito**: Docker + PostgreSQL corriendo (`setup-postgres.md`).

---

## ¿Por qué DBeaver?

Un cliente universal de bases de datos: gratis, multiplataforma y con editor SQL cómodo. Lo elegimos porque te sirve para PostgreSQL, MySQL, SQLite y más — un solo IDE para todo lo que sea SQL. Prisma Studio está bueno, pero DBeaver te muestra la base de verdad, sin abstracciones.

---

## Checklist

### 1. Descargar e instalar DBeaver Community

Andá a [dbeaver.io/download](https://dbeaver.io/download/).

- En **Windows** bajá el installer **.exe** de "Windows x86_64" (edición Community, la gratis).
- En **Linux** bajá el AppImage o el paquete `.deb` de tu distro.

Instalá con los valores por defecto.

- [ ] DBeaver abre sin errores

### 2. Conectar a PostgreSQL en Docker

1. Asegurate de que PostgreSQL esté corriendo: `docker compose up -d db`.
2. En DBeaver: **File → New → Database Connection** → elegí **PostgreSQL** → **Next**.
3. Completá los mismos datos del `compose.yaml` de `setup-postgres.md`:
   - **Host**: `localhost`
   - **Port**: `5432`
   - **Database**: `fullstack_dev`
   - **Username**: `dev`
   - **Password**: `dev`
4. Si el wizard te muestra **Driver properties** con SSL, desactivá SSL (dejalo en *disable*): el Docker local no usa SSL.
5. **Test Connection** → debe decir "Connected" → **Finish**.

- [ ] La conexión de prueba es exitosa

### 3. Explorar las tablas que creó Prisma

Después de correr `npx prisma migrate dev --name init`, Prisma crea su propia tabla y las de tus modelos:

- En el panel izquierdo: **conexión → Databases → fullstack_dev → Schemas → public → Tables**.
- Vas a ver `_prisma_migrations` (el historial de migraciones) y las tablas de tus modelos (ej: `User`, `Post`).
- Hacé **doble clic** en una tabla → pestaña **Datos** para ver las filas.

### 4. Ejecutar SQL

1. Clic derecho sobre la conexión → **Herramientas SQL** → **Abrir consola SQL**.
2. Escribí la query y ejecutá con **Ctrl+Enter** (o el ícono ▶).

```sql
SELECT * FROM "User" LIMIT 10;
```

---

## Verificación

Conectate y corré esta query en la consola SQL:

```sql
SELECT version();
```

**Si ves las tablas de Prisma (`_prisma_migrations`, tus modelos) o `SELECT version()` responde → DBeaver listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Connection refused" | PostgreSQL no está corriendo: `docker compose up -d db` |
| Error de SSL al conectar | En "Driver properties" desactivá SSL — el Docker local no usa SSL |
| "password authentication failed" | Usá las credenciales del compose: user `dev`, password `dev` |
| No aparecen las tablas de Prisma | Corré `npx prisma migrate dev --name init` antes de explorar |

---

## Preguntas de repaso

- **P:** ¿Qué datos de conexión usás en DBeaver para PostgreSQL en Docker?
  **R:** Host: `localhost`, Port: `5432`, Database: `fullstack_dev`, Username: `dev`, Password: `dev`.
- **P:** ¿Por qué debés desactivar SSL en las Driver properties al conectar a Docker local?
  **R:** Porque el contenedor Docker de PostgreSQL no está configurado con SSL por defecto. Intentar conectar con SSL causa error de handshake.
- **P:** ¿Cómo ejecutás una query SQL en DBeaver?
  **R:** Clic derecho sobre la conexión → Herramientas SQL → Abrir consola SQL, escribir la query y ejecutar con Ctrl+Enter.
- **P:** ¿Qué tabla especial crea Prisma automáticamente y para qué sirve?
  **R:** `_prisma_migrations`, que trackea el historial de migrations aplicadas con su nombre, checksum y fecha de ejecución.
- **P:** ¿Qué ventaja tiene DBeaver sobre Prisma Studio?
  **R:** DBeaver muestra la base de datos real sin abstracciones del ORM, soporta múltiples motores SQL, y tiene editor SQL completo con autocompletado.

---

## Recursos

- [DBeaver — Download](https://dbeaver.io/download/)
- [DBeaver — Docs](https://dbeaver.io/docs/)
