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

**Técnicamente:** PostgreSQL usa MVCC (Multi-Version Concurrencia Control) para permitir lecturas sin bloquear escrituras — cada transacción ve un snapshot consistente de la base. MongoDB usa WiredTiger storage engine con document-level locking. La diferencia fundamental está en el teorema CAP: SQL prioriza Consistency y Partition tolerance (CP), mientras MongoDB en replica set prioriza Availability y Partition tolerance (AP). Consultá el [PostgreSQL Feature Matrix](https://www.postgresql.org/about/featurematrix/) y el [MongoDB Document Model](https://www.mongodb.com/docs/manual/core/document/) para las diferencias técnicas completas.

> **Check de comprensión**:
> 1. ¿Qué criterio usarías para decidir entre SQL y NoSQL para un sistema de inventario con relaciones entre productos, proveedores y categorías?
>    - R: SQL, porque los datos tienen estructura fija (SKU, precio, stock) y relaciones predecibles (producto → proveedor, producto → categoría).
> 2. ¿Por qué MongoDB escala horizontalmente más fácilmente que PostgreSQL?
>    - R: MongoDB usa sharding nativo para distribuir colecciones entre nodos, mientras PostgreSQL requiere soluciones externas (Citus, pgpool) para sharding horizontal.
> 3. ¿Qué significa ACID y por qué importa en transacciones de pago?
>    - R: Atomicity, Consistency, Isolation, Durability. En pagos, Atomicity garantiza que si falla un paso (cobrar tarjeta pero no registrar orden), toda la transacción se revierte.
> 4. Si tu app necesita full-text search en documentos JSON, ¿qué base usarías y por qué?
>    - R: PostgreSQL con JSONB + índices GIN, porque soporta búsqueda full-text nativa sobre documentos JSON indexables sin necesidad de un motor externo.
> 5. ¿Cuándo tiene sentido usar AMBAS bases en la misma aplicación?
>    - R: Cuando tenés datos transaccionales (usuarios, pagos → SQL) Y datos accesorios (logs, analytics, sesiones → NoSQL). Es el patrón polyglot persistence.

→ Ver [6.2 PostgreSQL](#62-postgresql--la-base-sql-del-stack) para detalles de SQL, [6.5 MongoDB](#65-mongodb--la-base-documental) para NoSQL, y [6.3 Modelado de Datos Relacional](#63-modelado-de-datos-relacional) para normalización.

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

**Técnicamente:** PostgreSQL usa un cost-based optimizer (CBO) que evalúa múltiples planes de ejecución y elige el de menor costo estimado usando estadísticas de tabla (`pg_statistic`). Las estadísticas se actualizan con `ANALYZE` o automáticamente vía autovacuum. El tipo JSONB almacena datos en formato binario decomposed, permitiendo indexación con GIN y acceso eficiente a keys específicas sin parsear el JSON completo. La documentación oficial de [tipos de datos](https://www.postgresql.org/docs/current/datatype.html) y [CREATE TABLE](https://www.postgresql.org/docs/current/sql-createtable.html) cubre todos los detalles.

> **Check de comprensión**:
> 1. ¿Por qué `TIMESTAMPTZ` es preferible a `TIMESTAMP` sin timezone?
>    - R: Porque almacena la fecha/hora en UTC internamente y convierte al timezone del cliente al leer, evitando confusiones con horarios de verano o servidores en distintas zonas.
> 2. ¿Cuándo usar `UUID` en vez de `SERIAL` como primary key?
>    - R: UUID es preferible cuando necesitás IDs únicos distribuidos (microservicios, merge de bases, IDs predecibles no deseados). SERIAL es más compacto y rápido para apps monolíticas.
> 3. ¿Qué ventaja tiene JSONB sobre TEXT para guardar JSON?
>    - R: JSONB parsea el JSON al guardarlo, permite indexación con GIN, y soporta queries sobre keys internas (`metadata->>'key'`). TEXT solo guarda el string sin parsear.
> 4. ¿Qué hace `ON DELETE CASCADE` en la FK `author_id REFERENCES users(id)`?
>    - R: Cuando se borra un usuario, borra automáticamente todos sus posts asociados. Sin CASCADE, el DELETE del usuario fallaría si tiene posts.
> 5. ¿Para qué sirve un índice compuesto `(author_id, created_at DESC)`?
>    - R: Optimiza queries que filtran por author_id Y ordenan por created_at. Sirve para `WHERE author_id = ? ORDER BY created_at DESC` con un solo index scan.

→ Ver [6.8 Índices](#68-índices--cómo-acelerar-queries) para tipos de índices, [6.3 Modelado de Datos Relacional](#63-modelado-de-datos-relacional) para FKs y relaciones, y [6.9 Backups y Restauración](#69-backups-y-restauración) para pg_dump.

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

**Técnicamente:** Las formas normales se basan en dependencias funcionales. 1FN elimina grupos repetitivos (valores atómicos). 2FN elimina dependencias parciales (todo atributo no-key depende de toda la PK compuesta). 3FN elimina dependencias transitivas (ningún atributo no-key depende de otro atributo no-key). BCNF (Boyce-Codd) es una forma normal más estricta que 3FN: cada determinante debe ser una clave candidata. La [documentación de DDL](https://www.postgresql.org/docs/current/ddl.html) y [foreign keys](https://www.postgresql.org/docs/current/tutorial-fk.html) de PostgreSQL cubre la implementación práctica.

> **Check de comprensión**:
> 1. ¿Por qué guardar `"tag1,tag2,tag3"` en un solo campo VARCHAR viola 1FN?
>    - R: Porque cada tag es un valor independiente que debería estar en su propia fila. Buscar tags individuales requiere string parsing en vez de queries SQL eficientes.
> 2. Si tenés una tabla `order_items` con PK compuesta `(order_id, product_id)`, ¿por qué guardar `product_name` ahí viola 2FN?
>    - R: Porque `product_name` depende solo de `product_id`, no de toda la PK compuesta. Debería estar en la tabla `products`.
> 3. ¿Cuándo es válido desnormalizar a propósito?
>    - R: Cuando el costo de JOINs frecuentes supera el costo de mantener datos duplicados, o cuando necesitás un snapshot histórico (precio al momento de compra).
> 4. ¿Cómo modelás una relación N:M entre estudiantes y cursos en SQL?
>    - R: Con una tabla intermedia (junction): `enrollments(student_id FK, course_id FK, enrolled_at)`. La PK puede ser compuesta `(student_id, course_id)`.
> 5. ¿Qué pasa si no tenés FKs y borrás un usuario que tiene posts?
>    - R: Los posts quedan huérfanos (author_id apunta a un usuario inexistente). Con FK + CASCADE se borran; con FK + SET NULL quedan con author_id NULL.

→ Ver [6.1 SQL vs NoSQL](#61-sql-vs-nosql--el-falso-dilema) para cuándo usar modelo relacional, [6.2 PostgreSQL](#62-postgresql--la-base-sql-del-stack) para tipos de datos, y [6.8 Índices](#68-índices--cómo-acelerar-queries) para optimizar joins.

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

**Técnicamente:** Las migrations usan versionamiento timestamp-based para garantizar orden de ejecución. Prisma genera SQL desde el AST del `schema.prisma` y lo almacena en `prisma/migrations/`. Cada migration tiene un `migration.sql` y una fila en `_prisma_migrations` que trackea estado. El rollback manual es necesario porque el AST no puede inferir la operación inversa automáticamente (ej: ¿qué datos ponés en una columna recién agregada?). La [documentación de Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate) y la [referencia de comandos](https://www.prisma.io/docs/reference/api-reference/command-reference) cubren todos los escenarios.

> **Check de comprensión**:
> 1. ¿Por qué nunca debés editar una migration ya aplicada en producción?
>    - R: Porque el hash de la migration cambia y Prisma detecta inconsistencia entre lo que dice `_prisma_migrations` y el archivo real. Además, otros entornos ya ejecutaron la versión vieja.
> 2. ¿Por qué Prisma no tiene rollback automático?
>    - R: Porque no puede inferir la operación inversa de forma segura: si agregás una columna sin default, ¿qué datos ponés al revertir? Si borrás datos, ¿cómo los recuperás?
> 3. ¿Cuál es la diferencia entre `prisma migrate dev` y `prisma migrate deploy`?
>    - R: `dev` crea migrations nuevas y las aplica (desarrollo). `deploy` aplica migrations pendientes sin crear nuevas (producción/CI).
> 4. ¿Por qué las migraciones destructivas deben hacerse en dos pasos?
>    - R: Si el deploy del código falla y hacés rollback, la columna ya no existe en la DB pero el código viejo la referencia → crash. El two-step deploy evita esto.
> 5. ¿Qué pasa si dos developers crean migrations con el mismo timestamp?
>    - R: Prisma detecta el conflicto y falla. Se resuelve reordenando las migrations o mergiándolas en una sola.

→ Ver [6.2 PostgreSQL](#62-postgresql--la-base-sql-del-stack) para schema SQL, [6.9 Backups y Restauración](#69-backups-y-restauración) para backup antes de migrar, y [6.8 Índices](#68-índices--cómo-acelerar-queries) para índices en migrations.

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

**Técnicamente:** MongoDB usa BSON (Binary JSON) como formato de almacenamiento, que soporta tipos adicionales como Date, ObjectId, Binary y Regex. WiredTiger es el storage engine default desde v3.2, con compression Snappy/ZSTD y checkpoint-based durability. El aggregation pipeline se ejecuta como una serie de stages que procesan documentos secuencialmente, con optimización automática que reordena `$match` y `$project` cuando es posible (query optimization). El [modelo de documentos](https://www.mongodb.com/docs/manual/core/document/) y la [documentación de aggregation](https://www.mongodb.com/docs/manual/aggregation/) cubren todos los stages disponibles.

> **Check de comprensión**:
> 1. ¿Cuándo embeber documentos y cuándo referenciar en MongoDB?
>    - R: Embeber cuando los datos se leen juntos frecuentemente y el array no crece sin límite. Referenciar cuando los datos se consultan por separado o el sub-documento puede crecer indefinidamente.
> 2. ¿Qué es BSON y por qué MongoDB lo usa en vez de JSON puro?
>    - R: BSON es JSON binario que soporta tipos adicionales (Date, ObjectId, Binary) y permite traversal más rápido porque incluye metadata de longitud de cada campo.
> 3. ¿Qué hace `$match` en el aggregation pipeline y por qué debería ir primero?
>    - R: Filtra documentos antes de procesarlos. Debería ir primero porque reduce la cantidad de documentos que pasan por stages posteriores, mejorando performance.
> 4. ¿Cuál es el límite de tamaño de un documento en MongoDB?
>    - R: 16 MB. Si un documento embebido supera ese límite, debés usar referencias en vez de embebido.
> 5. ¿Qué diferencia hay entre `find()` y `aggregate()` en MongoDB?
>    - R: `find()` es para queries simples (filtrar, proyectir, ordenar). `aggregate()` es para procesamiento complejo: grouping, joins (`$lookup`), transformación de datos en múltiples stages.

→ Ver [6.1 SQL vs NoSQL](#61-sql-vs-nosql--el-falso-dilema) para cuándo usar MongoDB, [6.6 Estrategias de Query](#66-estrategias-de-query) para el problema N+1, y [6.7 Redis](#67-redis--mucho-más-que-un-caché) para caching de documentos.

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

**Técnicamente:** El problema N+1 se resuelve con batching (DataLoader pattern de GraphQL) o con JOINs/IN queries. La paginación cursor-based usa un índice ordenado y una condición WHERE con el último valor visto, evitando el OFFSET que requiere escanear y descartar filas. En PostgreSQL, `EXPLAIN ANALYZE` muestra el plan real con tiempos de ejecución; en MongoDB, `explain("executionStats")` hace lo mismo. La [documentación de relaciones en Prisma](https://www.prisma.io/docs/concepts/components/prisma-client/relations) y las [operaciones de lectura en MongoDB](https://www.mongodb.com/docs/manual/core/read-operations/) cubren los patrones de query.

> **Check de comprensión**:
> 1. ¿Por qué el problema N+1 es tan común y tan dañino para la performance?
>    - R: Porque es fácil de escribir (un loop natural) pero escala linealmente: 100 items = 101 queries. Cada query tiene overhead de red + parsing + ejecución.
> 2. ¿Cómo resuelve Prisma el N+1 con `include`?
>    - R: Genera una query con JOIN o múltiples queries batcheadas (dependiendo del ORM), trayendo todos los datos relacionados en una sola ida a la DB.
> 3. ¿Por qué `LIMIT 20 OFFSET 10000` es lento?
>    - R: Porque la DB debe leer 10020 filas, ordenarlas, descartar las primeras 10000, y devolver 20. El OFFSET fuerza un scan completo de las filas descartadas.
> 4. ¿Cuándo usar cursor-based en vez de offset-based?
>    - R: Cursor-based para feeds infinitos, timelines, o datos que cambian frecuentemente. Offset-based para back office, paginación con saltos, o datasets estáticos.
> 5. ¿Qué es DataLoader y cómo resuelve el N+1?
>    - R: DataLoader es un patrón que batchea y cachea requests: en vez de N queries individuales, junta todas las keys y hace 1 query con `WHERE id IN (...)`.

→ Ver [6.8 Índices](#68-índices--cómo-acelerar-queries) para optimizar queries, [6.2 PostgreSQL](#62-postgresql--la-base-sql-del-stack) para EXPLAIN ANALYZE, y [6.5 MongoDB](#65-mongodb--la-base-documental) para explain en MongoDB.

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

**Técnicamente:** Redis es single-threaded en su event loop principal (desde Redis 6 hay I/O threads para network I/O, pero la ejecución de comandos sigue siendo single-threaded). Esto elimina race conditions y simplifica el modelo de concurrencia. La persistencia usa RDB (snapshots periódicos) y/o AOF (append-only file con cada write). El eviction policy por default es `noeviction` — cuando la memoria se llena, Redis rechaza writes. Los [tipos de datos de Redis](https://redis.io/docs/latest/develop/data-types/) y la [documentación de persistencia](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) cubren las opciones de configuración.

> **Check de comprensión**:
> 1. ¿Por qué Redis es single-threaded y por qué eso es una ventaja?
>    - R: Porque elimina race conditions sin necesidad de locks. Un solo thread procesa comandos secuencialmente, lo que simplifica la lógica y evita deadlocks.
> 2. ¿Qué pasa cuando Redis se queda sin memoria con la política `noeviction`?
>    - R: Rechaza los comandos de escritura con un error. Las lecturas siguen funcionando. Para evitar esto, configurá una política como `allkeys-lru`.
> 3. ¿Cuál es la diferencia entre RDB y AOF en Redis?
>    - R: RDB es un snapshot periódico (rápido, pero puede perder datos del último intervalo). AOF registra cada write (más seguro, pero más lento y ocupa más espacio).
> 4. ¿Por qué la combinación TTL + invalidación explícita es la estrategia más práctica?
>    - R: Porque el TTL garantiza que los datos stale eventualmente expiren, y la invalidación explícita garantiza coherencia inmediata cuando los datos cambian.
> 5. ¿Cómo usarías Redis para rate limiting?
>    - R: Con `INCR` + `EXPIRE`: cada request incrementa un contador `rate:{ip}` con TTL de ventana. Si el contador supera el límite, rechazás el request.

→ Ver [6.6 Estrategias de Query](#66-estrategias-de-query) para patrones de caché, [6.8 Índices](#68-índices--cómo-acelerar-queries) para performance, y [6.5 MongoDB](#65-mongodb--la-base-documental) para caching de documentos.

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

**Técnicamente:** Los índices B-tree en PostgreSQL tienen complejidad O(log n) para búsqueda, inserción y eliminación. Los índices GIN (Generalized Inverted Index) son para datos compuestos (arrays, JSONB, full-text) y funcionan como un inverted index. Un índice parcial reduce el tamaño indexando solo filas que cumplen un WHERE, ahorrando espacio y mejorando writes. El comando `EXPLAIN ANALYZE` revela si el planner usa un Index Scan, Index Only Scan, o Seq Scan. La [documentación de índices](https://www.postgresql.org/docs/current/indexes.html) y los [tipos de índices](https://www.postgresql.org/docs/current/indexes-types.html) de PostgreSQL cubren todos los detalles.

> **Check de comprensión**:
> 1. ¿Por qué un índice B-tree tiene complejidad O(log n)?
>    - R: Porque es un árbol balanceado: cada nivel del árbol reduce el espacio de búsqueda a la mitad. Con 1M de filas, solo necesitás ~20 comparaciones.
> 2. ¿Por qué el orden importa en un índice compuesto `(a, b)`?
>    - R: Porque B-tree ordena primero por `a`, luego por `b`. Para usar el índice, la query debe filtrar por `a` (o por `a` y `b`). Solo filtrar por `b` ignora el índice.
> 3. ¿Cuándo conviene un índice parcial?
>    - R: Cuando solo consultás un subconjunto de filas frecuentemente (ej: `WHERE active = true`). Ahorra espacio y es más rápido de mantener.
> 4. ¿Por qué cada índice ralentiza los writes?
>    - R: Porque cada INSERT/UPDATE/DELETE debe actualizar también el índice. Con 5 índices, un INSERT = 1 write en tabla + 5 writes en índices.
> 5. ¿Qué diferencia hay entre Index Scan e Index Only Scan?
>    - R: Index Scan usa el índice para encontrar filas pero lee la tabla para obtener columnas no indexadas. Index Only Scan obtiene TODO del índice (covering index), sin tocar la tabla.

→ Ver [6.2 PostgreSQL](#62-postgresql--la-base-sql-del-stack) para ejemplos de índices en schema, [6.6 Estrategias de Query](#66-estrategias-de-query) para EXPLAIN ANALYZE, y [6.3 Modelado de Datos Relacional](#63-modelado-de-datos-relacional) para columnas a indexar.

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

**Técnicamente:** `pg_dump` genera un backup lógico (SQL statements) que es portable entre versiones y arquitecturas. El formato custom (-Fc) permite restauración selectiva y parallel restore con `pg_restore -j`. Para backups de gran escala, WAL archiving (Write-Ahead Log) permite point-in-time recovery (PITR). MongoDB usa `mongodump` que exporta BSON, y para producción se recomienda MongoDB Atlas backups o OPS Manager con snapshot-based backups. La [documentación de backup](https://www.postgresql.org/docs/current/backup.html) de PostgreSQL y la [guía de backups](https://www.mongodb.com/docs/manual/core/backups/) de MongoDB cubren los escenarios de producción.

> **Check de comprensión**:
> 1. ¿Cuál es la diferencia entre backup lógico (`pg_dump` SQL) y backup binario (`-Fc`)?
>    - R: El lógico genera SQL puro (portable entre versiones, editable). El binario es un formato comprimido de PostgreSQL (más rápido, permite restore paralelo, pero solo para misma versión).
> 2. ¿Qué es PITR (Point-In-Time Recovery) y por qué es importante?
>    - R: Permite restaurar la base a un momento exacto en el pasado usando WAL segments. Es crucial para recovering de errores humanos (borré la tabla equivocada a las 14:32).
> 3. ¿Por qué debés hacer backup ANTES de correr una migration en producción?
>    - R: Porque si la migration falla o corrompe datos, el backup te permite restaurar al estado anterior. Sin backup, la migration destructiva es irreversible.
> 4. ¿Qué advantage tiene `pg_restore -j 4` sobre `psql < backup.sql`?
>    - R: `-j 4` usa 4 threads paralelos para restaurar, acelerando significativamente la restauración de bases grandes.
> 5. ¿Por qué los snapshots de volumen Docker no reemplazan los backups lógicos?
>    - R: Porque los snapshots capturan el estado binario del volumen (dependiente de versión de Postgres). Los backups lógicos son portables y permiten restaurar en otra versión o máquina.

→ Ver [6.4 Migrations](#64-migrations--evolución-del-schema) para backup antes de migrar, [6.2 PostgreSQL](#62-postgresql--la-base-sql-del-stack) para pg_dump, y [6.5 MongoDB](#65-mongodb--la-base-documental) para mongodump.

---

## 6.10 Herramientas de trabajo

*En criollo:* Ningún IDE es obligatorio: DBeaver es la opción universal para SQL, y las GUIs oficiales de Mongo y Redis son las más completas para explorar documentos y keys. ¿Por qué importa? Porque ver los datos directamente te da feedback inmediato sobre lo que tu código escribe en la DB.

| Herramienta | Para qué | Guía |
|-------------|----------|------|
| **DBeaver Community** | IDE SQL universal — PostgreSQL, MySQL, etc. | [setup-dbeaver](../guides/06-databases/setup-dbeaver.md) |
| **MongoDB Compass** | GUI oficial de MongoDB — ver docs, modelar, aggregation | [setup-compass](../guides/06-databases/setup-compass.md) |
| **Redis Insight** | GUI oficial de Redis — ver keys, TTL, memoria | [setup-redis-insight](../guides/06-databases/setup-redis-insight.md) |

**Técnicamente:** DBeaver usa JDBC para conectar a bases de datos, lo que le da soporte universal pero requiere drivers por cada DB. MongoDB Compass usa el driver oficial de Node.js internamente. Redis Insight se conecta directamente al protocolo RESP (Redis Serialization Protocol). Cada herramienta tiene tradeoffs: DBeaver es universal pero más pesado, Compass es específico de MongoDB pero con features avanzadas de aggregation, Insight es específico de Redis con visualización de memoria en tiempo real. Las docs de [DBeaver](https://dbeaver.io/docs/), [Compass](https://www.mongodb.com/docs/compass/) y [Redis Insight](https://redis.io/docs/latest/develop/connect/clients/redis-insight/) cubren el uso avanzado.

> **Check de comprensión**:
> 1. ¿Por qué DBeaver necesita drivers separados para cada base de datos?
>    - R: Porque usa JDBC (Java Database Connectivity), que requiere un driver específico por cada motor de DB para traducir el protocolo nativo al formato JDBC.
> 2. ¿Qué ventaja tiene Compass sobre `mongosh` para el aggregation pipeline?
>    - R: Compass permite armar el pipeline stage por stage con feedback visual inmediato, viendo el resultado de cada stage antes de agregar el siguiente.
> 3. ¿Qué información clave te muestra Redis Insight que `redis-cli` no muestra tan fácilmente?
>    - R: Visualización de memoria por key, tipos de datos con formato legible, TTL en countdown visual, y gráficos de uso de memoria en tiempo real.
> 4. ¿Cuándo preferirías `redis-cli` sobre Redis Insight?
>    - R: Para scripting automatizado, debugging rápido en servidores sin GUI, o cuando necesitás ejecutar comandos en batch (pipes, scripts de migración).
> 5. ¿Por qué es importante ver los datos directamente en la DB y no solo confiar en el ORM?
>    - R: Porque el ORM puede ocultar problemas: queries N+1 silenciosos, datos corruptos, índices no usados, o migraciones que no se aplicaron correctamente.

→ Ver [6.2 PostgreSQL](#62-postgresql--la-base-sql-del-stack) para conectar DBeaver a PostgreSQL, [6.5 MongoDB](#65-mongodb--la-base-documental) para Compass, y [6.7 Redis](#67-redis--mucho-más-que-un-caché) para Redis Insight.

---

> **Check de comprensión integrador**:
> 1. ¿Cuándo modelarías datos como documentos embebidos en MongoDB y cuándo como referencias?
>    - R: Embeber cuando los datos se leen juntos frecuentemente y el array no crece sin límite (< 16MB). Referenciar cuando los datos se consultan por separado o pueden crecer indefinidamente.
> 2. ¿Qué es el problema N+1? ¿Cómo lo resolvés con Prisma y con queries manuales?
>    - R: Es hacer N+1 queries cuando bastan 2. Con Prisma: `include`. Manualmente: traer todos los IDs y hacer 1 query con `WHERE id IN (...)`.
> 3. ¿Por qué `LIMIT 20 OFFSET 1000` es más lento que `LIMIT 20 OFFSET 0`? ¿Qué alternativa existe?
>    - R: Porque OFFSET fuerza a la DB a leer y descartar 1000 filas. Alternativa: cursor-based pagination con `WHERE id > last_seen_id LIMIT 20`.
> 4. ¿Qué pasa si borrás un usuario que tiene 50 posts con `ON DELETE CASCADE`? ¿Y sin él?
>    - R: Con CASCADE: se borran los 50 posts automáticamente. Sin CASCADE: el DELETE del usuario falla por violación de foreign key constraint.
> 5. ¿Por qué no deberías cachear el saldo bancario de un usuario con TTL de 5 minutos?
>    - R: Porque el saldo cambia en tiempo real con cada transacción. Un TTL de 5 minutos puede mostrar un saldo incorrecto, causando overdrafts o rechazos erróneos.
