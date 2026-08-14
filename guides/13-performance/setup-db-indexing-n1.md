# Setup — Database Indexing y Fix del N+1

> **Tópico**: 13 — Performance & Optimización (secciones 13.5 y 13.7)
> **Objetivo**: detectar y eliminar el problema N+1 (eager loading y batching) y acelerar queries con índices, verificando el plan de ejecución con `EXPLAIN ANALYZE`.
> **Prerequisito**: una app con acceso a base de datos (Postgres o MongoDB) y un ORM (Prisma es el ejemplo). Concepto 13 completo (13.5 y 13.7) y, recomendado, el tópico 6.

---

## ¿Por qué?

El N+1 y la falta de índices son las dos causas más comunes de lentitud en backend, y las dos son silenciosas: el código se ve limpio y no tira errores, solo degrada la performance a medida que crecen los datos. El N+1 convierte una consulta en N+1 consultas (un `for` con una query adentro), y sin índices cada `WHERE` obliga a la base a recorrer toda la tabla. Atacarlos tiene el mejor retorno de cualquier optimización de backend: pasás de cientos de queries a dos, y de full scans a búsquedas O(log n), sin tocar una línea de lógica de negocio.

---

## Checklist

### 1. Detectar el N+1

- [ ] Activá el log de queries del ORM para VER lo que pasa:

```js
// Prisma: log de queries en desarrollo
const prisma = new PrismaClient({ log: ['query'] });
```

- [ ] Buscá el patrón `for`/`map` con una query adentro — es N+1 casi seguro:

```js
// ❌ N+1: 1 query de usuarios + N queries de posts
const usuarios = await db.usuarios.findMany();

for (const usuario of usuarios) {
  usuario.posts = await db.posts.findMany({
    where: { autorId: usuario.id }, // ← una query POR usuario
  });
}
// 100 usuarios → 101 queries
```

### 2. Arreglar el N+1 con eager loading (Prisma)

- [ ] Usá `include` para traer la relación en una sola query con JOIN:

```js
// ✅ Prisma: include resuelve el N+1 con un JOIN
const usuarios = await db.usuarios.findMany({
  include: { posts: true },
});
```

### 3. Arreglar el N+1 con batching (manual o GraphQL)

- [ ] Sin ORM o en GraphQL, agrupá las claves en una query `IN`:

```js
// ✅ Manual: 2 queries en total
const usuarios = await db.usuarios.findMany();              // 1
const ids = usuarios.map((u) => u.id);
const posts = await db.posts.findMany({
  where: { autorId: { in: ids } },                         // 2
});
// agrupás posts por autorId en memoria
```

- [ ] En GraphQL, usá `DataLoader` para el batching + caching por request:

```bash
npm install dataloader
```

```js
import DataLoader from 'dataloader';

const postLoader = new DataLoader(async (autorIds) => {
  const posts = await db.posts.findMany({
    where: { autorId: { in: autorIds } }, // 1 query para TODOS los ids
  });
  return autorIds.map((id) => posts.filter((p) => p.autorId === id));
});

const posts = await postLoader.load(usuario.id);
```

### 4. Identificar queries lentas sin índice

- [ ] En Postgres, mirá el plan de ejecución de una query de filtro:

```sql
EXPLAIN ANALYZE SELECT * FROM usuarios WHERE email = 'persona@ejemplo.com';
-- Buscá "Seq Scan" (full scan, malo). Querés "Index Scan".
```

### 5. Crear índices en las columnas que filtrás

- [ ] Creá un índice en la columna de filtro/orden/join:

```sql
-- Índice simple en la columna que filtrás
CREATE INDEX idx_usuarios_email ON usuarios (email);

-- Índice compuesto: columna más selectiva primero, el orden importa
CREATE INDEX idx_posts_autor_fecha ON posts (autor_id, creado_en DESC);
```

- [ ] En Prisma, declará los índices en el schema:

```prisma
model Usuario {
  id    Int    @id @default(autoincrement())
  email String @unique // ← índice unique automático

  @@index([nombre]) // ← índice simple
}
```

### 6. Verificar el resultado

- [ ] Confirmá que las queries pasaron de `Seq Scan` a `Index Scan`, y que el N+1 desapareció (ver sección Verificación).

---

## Verificación

```bash
# 1. El N+1 desapareció: el log muestra 2 queries en vez de 101
#    (activá log: ['query'] y mirá la consola)

# 2. El índice se usa: de Seq Scan a Index Scan
node -e "
const { Client } = require('pg');
const c = new Client({ connectionString: process.env.DATABASE_URL });
c.connect().then(async () => {
  const r = await c.query(\"EXPLAIN ANALYZE SELECT * FROM usuarios WHERE email = 'persona@ejemplo.com'\");
  console.log(r.rows.map(x => x['QUERY PLAN']).join('\n'));
  await c.end();
});
"
# Buscá "Index Scan using idx_usuarios_email" (bien) vs "Seq Scan" (malo)

# 3. Listá los índices de una tabla para confirmar que existen
#    (psql) \d usuarios
```

**Si el log muestra 2 queries en vez de 101 (N+1 resuelto) y `EXPLAIN ANALYZE` dice "Index Scan" en vez de "Seq Scan" → indexing + N+1 correctos. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Creaste el índice pero la query sigue haciendo `Seq Scan` | La columna no coincide con el filtro, o el `LIKE` tiene leading wildcard (`'%texto'`). Verificá con `EXPLAIN ANALYZE` la columna real que filtra |
| El índice compuesto `(a, b)` no acelera filtrar solo por `b` | El índice es "por prefijo izquierdo": sirve para `a` y `a+b`, no para `b` solo. Creá un índice aparte para `b` si hace falta |
| `include` de Prisma hace muchas queries igual | Estás incluyendo una relación anidada profunda; revisá que el `include` no genere otro N+1 adentro (relaciones dentro de la relación) |
| DataLoader cachea datos viejos entre requests | El caché de DataLoader es por-request; si lo compartís globalmente, creá un loader nuevo por request (patrón `perRequest`) |
| Los `INSERT` se volvieron lentos después de indexar | Cada índice se mantiene en cada escritura. Si indexaste de más, remové los índices que no usan tus queries |
| `EXPLAIN ANALYZE` muestra `Index Scan` pero sigue lento | El índice está mal diseñado (poca selectividad) o la query devuelve muchas filas igual; revisá el `rows` estimado vs real y el orden de columnas |

---

## Recursos

- [PostgreSQL — Indexes](https://www.postgresql.org/docs/current/indexes.html)
- [Prisma — Relations](https://www.prisma.io/docs/orm/prisma-schema/data-model/relations)
- [Prisma — Pagination](https://www.prisma.io/docs/orm/prisma-client/queries/pagination)
- [DataLoader — GitHub](https://github.com/graphql/dataloader)
- [Use The Index, Luke](https://use-the-index-luke.com/)

## Preguntas de repaso

- **P:** ¿Qué es el problema N+1 y cómo lo reconocés en tu código?
  **R:** Es hacer 1 query para traer N registros y una query extra por cada uno para sus relaciones. Se reconoce por un `for`/`map` con una query adentro, o en el log como la misma query repetida N veces.

- **P:** ¿Cómo resuelve Prisma el N+1 con `include`?
  **R:** Genera una sola query con JOIN (o queries optimizadas) que trae los registros y sus relaciones de una vez, en vez de una query por cada relación.

- **P:** ¿Qué es DataLoader y por qué es la solución estándar en GraphQL?
  **R:** Es una librería de batching que acumula claves de requests individuales y las resuelve en una query con `IN`, cacheando dentro del request. Encaja naturalmente con los resolvers de GraphQL.

- **P:** ¿Qué es un índice y qué problema resuelve?
  **R:** Una estructura que permite a la base encontrar filas sin recorrer toda la tabla. Resuelve la lentitud de queries que filtran, ordenan o joinean por una columna no indexada.

- **P:** ¿Por qué un índice compuesto `(a, b)` no acelera filtrar solo por `b`?
  **R:** Porque el índice es por prefijo izquierdo: está ordenado por `a` primero, así que solo sirve para filtros que empiezan por `a`. Para filtrar solo por `b` necesitás otro índice.

- **P:** ¿Para qué usás `EXPLAIN ANALYZE` y qué resultados buscás?
  **R:** Para ver el plan de ejecución real de una query. Buscás "Index Scan" (usa índice, bien) y evitás "Seq Scan" (full scan, malo); también confirmás cuántas filas toca realmente.
