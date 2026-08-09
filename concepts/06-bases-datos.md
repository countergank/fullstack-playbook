# 6. Bases de Datos & Persistencia

> Objetivo: entender los dos paradigmas de bases de datos (SQL y NoSQL), cómo modelar datos en cada uno, y las estrategias de query, caching, índices y migrations que separan un backend amateur de uno profesional.

---

## 6.1 SQL vs NoSQL — El Falso Dilema

*En criollo:* La pregunta no es "¿SQL o NoSQL?". La pregunta es "¿este dato tiene estructura fija y relaciones entre sí, o es un documento autocontenido que cambia de forma según el caso?". La mayoría de las aplicaciones usan SQL para los datos transaccionales (usuarios, órdenes, pagos) y NoSQL para datos accesorios (logs, analytics, cachés, sesiones).

### Cuándo usar cada uno

| Criterio | SQL (PostgreSQL) | NoSQL (MongoDB) |
|----------|-----------------|-----------------|
| Estructura de datos | Fija, conocida de antemano | Variable, evoluciona con el tiempo |
| Relaciones | Muchas y complejas (joins) | Pocas, jerárquicas (documentos embebidos) |
| Consistencia | ACID — transaccional, todo o nada | Eventual — más rápido, menos garantías |
| Escalabilidad | Vertical (más CPU/RAM) | Horizontal (más nodos) |
| Consultas complejas | SQL expresivo, agregaciones, window functions | Aggregation pipeline (más verboso) |
| Ejemplos | Órdenes, pagos, inventario, usuarios | Catálogo de productos, logs, analytics, sesiones |

> **Regla práctica**: si los datos tienen dueño claro y relaciones predecibles → SQL. Si son documentos que cambian de forma o se leen junto con su dueño → NoSQL.

---

## 6.2 PostgreSQL — La Base SQL del Stack

*En criollo:* PostgreSQL es el Ferrari de las bases SQL open-source. No es "una base más" — tiene features que bases comerciales cobran fortuna: JSONB (documentos JSON indexables), full-text search, window functions, CTEs, extensions (PostGIS para datos geoespaciales), y un planner de queries que te dice exactamente por qué tu consulta es lenta.

### Tipos de datos esenciales

| Tipo | Uso | Ejemplo |
|------|-----|---------|
| `SERIAL` / `UUID` | IDs autoincrementales o universales | `id UUID DEFAULT gen_random_uuid()` |
| `VARCHAR(n)` / `TEXT` | Strings con o sin límite | `name VARCHAR(255)` |
| `INTEGER` / `BIGINT` / `DECIMAL` | Números | `price DECIMAL(10,2)` |
| `BOOLEAN` | Verdadero/falso | `active BOOLEAN DEFAULT true` |
| `TIMESTAMPTZ` | Fecha y hora con timezone | `created_at TIMESTAMPTZ DEFAULT NOW()` |
| `JSONB` | Documento JSON indexable | `metadata JSONB` |
| `ENUM` | Valores fijos | `role user_role DEFAULT 'user'` |

### Ejemplo de schema SQL

```sql
CREATE TABLE users (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email      VARCHAR(255) UNIQUE NOT NULL,
  name       VARCHAR(255),
  password   VARCHAR(255) NOT NULL,
  role       user_role DEFAULT 'user',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE posts (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title      VARCHAR(255) NOT NULL,
  content    TEXT,
  author_id  UUID REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Índice para búsqueda por email (login)
CREATE INDEX idx_users_email ON users(email);

-- Índice para listar posts de un autor ordenados por fecha
CREATE INDEX idx_posts_author_date ON posts(author_id, created_at DESC);
```

---

## 6.3 Modelado de Datos Relacional

### Las 3 formas normales (versión criolla)

1. **1FN**: Cada columna tiene UN solo valor. No guardes `"correo1, correo2"` en un solo campo.
2. **2FN**: Todo atributo depende de la clave primaria COMPLETA. Si tenés una tabla de detalle de orden, el nombre del producto va en la tabla de productos, no en el detalle.
3. **3FN**: Nada depende de otra cosa que no sea la clave. El precio total calculado no se guarda — se calcula con `quantity * unit_price`.

> En la práctica real, a veces **desnormalizás a propósito** (guardar el precio en la orden porque el precio del producto puede cambiar mañana y la orden debe reflejar el precio al momento de compra). Esto no es un error — es una decisión consciente.

### Relaciones

| Tipo | Ejemplo | En SQL |
|------|---------|--------|
| 1:1 | Usuario → Perfil | FK con UNIQUE |
| 1:N | Usuario → Posts | FK en el lado N |
| N:M | Estudiantes → Cursos | Tabla intermedia (junction table) |

---

## 6.4 Migrations — Evolución del Schema

*En criollo:* Una aplicación crece y la base de datos crece con ella. Las migrations son el historial de cambios del schema: "agregué tal columna", "renombré tal tabla", "creé tal índice". Son código versionable que se ejecuta en orden y garantiza que todos los entornos (dev, staging, prod) tengan exactamente la misma estructura.

### Con Prisma

```bash
# Crear migration desde cambios en schema.prisma
npx prisma migrate dev --name add-user-avatar

# Aplicar migrations pendientes en producción
npx prisma migrate deploy

# Volver atrás (rollback) — Prisma no tiene rollback automático.
# Crear una nueva migration que revierta el cambio manualmente.
```

### Buenas prácticas

- **Nunca edites una migration ya aplicada en producción.** Creá una nueva.
- **Las migrations son código — se commitean.**
- **Probá las migrations en un entorno idéntico antes de producción.**
- **Migraciones destructivas** (borrar columna/tabla): hacer en dos pasos. Paso 1: dejar de usar la columna en el código. Paso 2: borrar la columna. Así si hay rollback del código, no se rompe.

---

## 6.5 MongoDB — La Base Documental

*En criollo:* MongoDB guarda documentos JSON en colecciones. No hay schema fijo — cada documento en la misma colección puede tener campos distintos. Esto es poder y peligro al mismo tiempo: flexibilidad total a cambio de zero garantías de integridad a nivel base de datos.

### Modelado en MongoDB: ¿embebo o referencio?

| Estrategia | Cuándo | Ejemplo |
|------------|--------|---------|
| **Embebido** | Los datos se leen juntos el 90% del tiempo | Posts dentro de User (siempre mostrás posts con su autor) |
| **Referencia** | Los datos crecen sin límite o se consultan por separado | User referencia a Posts (si listás posts sin el autor frecuentemente) |

```js
// EMBEBIDO — todo junto (1 query = todo lo que necesitás)
{
  _id: ObjectId("..."),
  name: "Lean",
  email: "lean@ejemplo.com",
  posts: [
    { title: "Mi primer post", content: "..." },
    { title: "Otro post", content: "..." }
  ]
}

// REFERENCIA — separado (2 queries, pero posts pueden ser miles)
// users collection
{ _id: ObjectId("..."), name: "Lean", email: "lean@ejemplo.com" }
// posts collection
{ _id: ObjectId("..."), title: "...", authorId: ObjectId("...") }
```

### Aggregation Pipeline (el SQL de MongoDB)

```js
// Equivalente a: SELECT status, COUNT(*) FROM orders GROUP BY status
db.orders.aggregate([
  { $group: { _id: "$status", count: { $sum: 1 } } },
  { $sort: { count: -1 } }
]);
```

---

## 6.6 Estrategias de Query

### El problema N+1

*En criollo:* Hacés 1 query para traer 100 usuarios. Después, por cada usuario, hacés 1 query para traer sus posts. Resultado: 101 queries en vez de 2. Esto es EL error de performance más común en backends.

```js
// ❌ N+1 — 1 query de users + N queries de posts
const users = await db.users.find();
for (const user of users) {
  user.posts = await db.posts.find({ authorId: user.id });
}

// ✅ 2 queries — una para users, una para TODOS los posts
const users = await db.users.find();
const userIds = users.map(u => u.id);
const posts = await db.posts.find({ authorId: { $in: userIds } });
// Agrupar posts por userId en memoria (O(n))
```

**Con Prisma**: `include` resuelve el N+1 automáticamente (genera una query con JOIN).
**Con Mongoose**: `populate()` también, pero cuidado con populación anidada profunda.

### Paginación — dos enfoques

| | Offset-based | Cursor-based |
|---|---|---|
| Cómo | `LIMIT 20 OFFSET 40` | `WHERE id > 'last-seen-id' LIMIT 20` |
| Performance | Degrada con páginas altas | Constante |
| Saltar páginas | Sí | No |
| Datos que cambian | Duplicados/items perdidos | Consistente |
| Ideal para | Back office, páginas fijas | Feeds infinitos, timelines |

---

## 6.7 Redis — Mucho Más que un Caché

*En criollo:* Redis es una base de datos en RAM — microsegundos de latencia. Pero no es solo un caché: es una navaja suiza que también funciona como message broker, rate limiter, contador atómico, y lock distribuido.

### Estrategias de caché

| Estrategia | Cómo funciona | Cuándo |
|------------|--------------|--------|
| **Cache-aside** | App lee caché → si miss, lee DB y llena caché | La más común, control total |
| **Write-through** | App escribe en caché y DB al mismo tiempo | Datos que se leen apenas se escriben |
| **Write-behind** | App escribe en caché → async a DB | Alta velocidad de escritura, riesgo de pérdida |

### Invalidación — el problema más difícil en computación

> "There are only two hard things in Computer Science: cache invalidation and naming things." — Phil Karlton

```js
// Estrategia práctica: TTL + invalidación explícita
await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 300); // 5 min TTL

// Al actualizar el usuario, borrar la caché:
await redis.del(`user:${id}`);
// La próxima lectura será cache-miss → leer DB → poblar caché fresco
```

### Lo que NUNCA debés cachear
- Datos que cambian en tiempo real (saldo bancario, stock crítico).
- Datos que son únicos por request (paginación específica, resultados de búsqueda personalizados — salvo que la búsqueda sea costosa y aceptes staleness).
- Contraseñas, tokens, secretos (Redis no está diseñado para secretos — usá un vault).

---

## 6.8 Índices — Cómo Acelerar Queries

*En criollo:* Un índice es como el índice de un libro: en vez de leer todas las páginas para encontrar un capítulo, vas al índice, ves la página, y vas directo. Sin índice, la base lee TODA la tabla por cada query (full table scan). Con índice, va directo a las filas que necesita.

### Tipos de índices en PostgreSQL

| Tipo | Uso | Ejemplo |
|------|-----|---------|
| **B-tree** (default) | Igualdad, rangos, ORDER BY | `CREATE INDEX idx_users_email ON users(email);` |
| **Hash** | Solo igualdad (=) | Más rápido que B-tree para `WHERE email = 'x'` pero no soporta rangos |
| **Partial** | Indexar solo algunas filas | `CREATE INDEX idx_active_users ON users(email) WHERE active = true;` |
| **Composite** | Múltiples columnas | `CREATE INDEX idx_posts_author_date ON posts(author_id, created_at DESC);` |
| **GIN** | Full-text search, JSONB, arrays | `CREATE INDEX idx_posts_content ON posts USING GIN(to_tsvector('spanish', content));` |

### Reglas de índices

- **Indexá columnas que aparecen en WHERE, JOIN y ORDER BY.**
- **El orden en índices compuestos importa**: `(a, b)` sirve para `WHERE a = 1` y `WHERE a = 1 AND b = 2`, pero NO para `WHERE b = 2` solo.
- **Cada índice ralentiza INSERT/UPDATE/DELETE** (hay que mantenerlo). No indexes todo.
- **Medí antes de indexar**: `EXPLAIN ANALYZE` te dice exactamente si tu query usa el índice o hace full scan.

---

## 6.9 Backups y Restauración

*En criollo:* Los backups son el airbag de tu base de datos. No los necesitás... hasta que los necesitás. Y ese día, si no los tenés, perdiste todo.

### PostgreSQL

```bash
# Backup lógico (SQL portable)
pg_dump -U dev -d fullstack_dev > backup.sql

# Backup binario (más rápido, solo para misma versión de Postgres)
pg_dump -U dev -d fullstack_dev -Fc > backup.dump

# Restaurar
psql -U dev -d fullstack_dev < backup.sql
pg_restore -U dev -d fullstack_dev backup.dump
```

### MongoDB

```bash
# Backup
mongodump --db fullstack_dev --out ./backup/

# Restaurar
mongorestore --db fullstack_dev ./backup/fullstack_dev/
```

### Docker — snapshots de volumen

```bash
# Backup del volumen Docker
docker run --rm -v pgdata:/data -v $(pwd):/backup alpine tar czf /backup/pgdata-backup.tar.gz -C /data .

# Restaurar
docker run --rm -v pgdata:/data -v $(pwd):/backup alpine tar xzf /backup/pgdata-backup.tar.gz -C /data
```

---

> **Check de comprensión**:
> 1. ¿Cuándo modelarías datos como documentos embebidos en MongoDB y cuándo como referencias?
> 2. ¿Qué es el problema N+1? ¿Cómo lo resolvés con Prisma y con queries manuales?
> 3. ¿Por qué `LIMIT 20 OFFSET 1000` es más lento que `LIMIT 20 OFFSET 0`? ¿Qué alternativa existe?
> 4. ¿Qué pasa si borrás un usuario que tiene 50 posts con `ON DELETE CASCADE`? ¿Y sin él?
> 5. ¿Por qué no deberías cachear el saldo bancario de un usuario con TTL de 5 minutos?
