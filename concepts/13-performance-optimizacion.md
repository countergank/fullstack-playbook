# 13. Performance & Optimización

> Objetivo: entender que la performance no es un paso final opcional, sino parte del diseño — medís con Web Vitals, cargás solo lo necesario con lazy loading y code splitting, y atacás los cuellos de botella reales (N+1, caching, índices, paginación) en vez de optimizar a ciegas.

---

## 13.1 Web Vitals

> Referencia base: [web.dev — Web Vitals](https://web.dev/articles/vitals) · [web.dev — LCP](https://web.dev/articles/lcp) · [web.dev — CLS](https://web.dev/articles/cls) · [web.dev — INP](https://web.dev/articles/inp)

*En criollo:* Web Vitals es el conjunto de métricas que Google (y el sentido común) usa para medir si una página "se siente" rápida. No te fijes solo en el tiempo de carga total: lo que importa es qué percibe el usuario. Las tres que mandan son LCP, INP y CLS. **LCP** (Largest Contentful Paint) mide cuánto tarda en aparecer el contenido principal; **INP** (Interaction to Next Paint, el sucesor de FID) mide cuánto tarda la página en responder a un click o teclado; **CLS** (Cumulative Layout Shift) mide cuánto "salta" el contenido mientras carga. Si dominás estas tres, dominás la experiencia percibida.

*Técnicamente:* Los umbrales "buenos" definidos por Google:

| Métrica | Qué mide | Umbral "bueno" |
|---------|----------|----------------|
| **LCP** | Tiempo hasta que el contenido principal es visible | ≤ 2.5s |
| **INP** | Latencia de respuesta a la interacción (reemplaza FID) | ≤ 200ms |
| **CLS** | Estabilidad visual (cuánto se mueve el layout) | ≤ 0.1 |
| **TTFB** | Tiempo hasta el primer byte del servidor | ≤ 0.8s |

```js
// Medir en el navegador con la Web Vitals API
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP((metric) => console.log('LCP:', metric.value));  // milisegundos
onINP((metric) => console.log('INP:', metric.value));
onCLS((metric) => console.log('CLS:', metric.value));  // score adimensional
```

Los culpables típicos y su ataque:

- **LCP lento** → imágenes sin `loading`/`fetchpriority`, HTML render-blocking, server lento. Solución: `fetchpriority="high"` en la imagen hero, CSS crítico inline, CDN.
- **INP alto** → main thread bloqueado por JS pesado (un listener gigante, re-renders). Solución: splitear tareas largas, `requestIdleCallback`, web workers.
- **CLS alto** → imágenes sin dimensiones, fonts que cambian el layout, contenido inyectado tarde. Solución: `width`/`height` en imágenes (o `aspect-ratio`), reservar espacio para iframes/ads.

```html
<!-- CLS: reservá el espacio de la imagen para que no salte el layout -->
<img
  src="hero.jpg"
  alt="Producto destacado"
  width="1200"
  height="630"
  fetchpriority="high" <!-- LCP: priorizá la imagen hero -->
/>
```

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre LCP, INP y CLS?
>    - R: LCP mide cuánto tarda en aparecer el contenido principal (carga); INP mide la latencia de respuesta a la interacción del usuario; CLS mide cuánto se mueve visualmente el layout durante la carga.
> 2. ¿Por qué INP reemplazó a FID?
>    - R: FID solo medía el retardo del PRIMER evento de interacción; INP mide la latencia de TODAS las interacciones durante la vida de la página, lo que da una imagen más real de la responsividad.
> 3. ¿Qué causa un CLS alto y cómo lo prevenís?
>    - R: Contenido que se inserta o cambia de tamaño sin reservar espacio (imágenes sin dimensiones, fonts, ads). Se previene dando `width`/`height` o `aspect-ratio` a los elementos y reservando espacio para contenido dinámico.
> 4. ¿Cómo ayuda `fetchpriority="high"` al LCP?
>    - R: Le dice al navegador que priorice la descarga de esa imagen (la hero, que es el contenido principal) por encima de otros recursos, reduciendo el tiempo hasta que el contenido principal es visible.
> 5. ¿Qué es TTFB y por qué importa aunque no sea una de las tres "Core Web Vitals"?
>    - R: Es el tiempo hasta el primer byte del servidor; un TTFB alto arrastra al LCP (todo tarda en empezar). Importa porque señala problemas de servidor/red que las métricas de frontend no explican.

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.1-react) — cómo el renderizado de React impacta en el LCP y el CLS.
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.6-navegadores) — el pipeline de renderizado del navegador detrás de estas métricas.

---

## 13.2 Lazy Loading

> Referencia base: [MDN — loading attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#loading) · [web.dev — Lazy loading images](https://web.dev/articles/lazy-loading-images) · [React — lazy](https://react.dev/reference/react/lazy)

*En criollo:* Lazy loading es la regla de oro del rendimiento: **no cargues nada hasta que el usuario lo necesite**. Si tu página tiene 50 imágenes y el usuario solo ve las 3 de arriba, ¿para qué descargar las 47 restantes? Cargás lo visible, y el resto se trae recién cuando el usuario scrollea hacia abajo (o cuando entra a la ruta). Lo mismo aplica a componentes y rutas: una sección de "configuración" que el usuario rara vez toca no tiene por qué estar en el bundle inicial.

*Técnicamente:* Hay dos capas de lazy loading: el **nativo del navegador** y el **a nivel de código** (JS):

```html
<!-- Lazy loading nativo: el navegador decide cuándo cargar -->
<img src="foto.jpg" alt="..." loading="lazy" />
<iframe src="mapa.html" loading="lazy"></iframe>
```

```jsx
// Lazy loading a nivel de componente (React): el chunk se descarga al montar
import { lazy, Suspense } from 'react';

const PanelConfiguracion = lazy(() => import('./PanelConfiguracion'));

function App() {
  return (
    <Suspense fallback={<p>Cargando…</p>}>
      <PanelConfiguracion />
    </Suspense>
  );
}
```

```jsx
// Next.js: dynamic import desactiva SSR para un componente pesado (charts, mapas)
import dynamic from 'next/dynamic';

const Mapa = dynamic(() => import('./Mapa'), { ssr: false });
```

Reglas de oro:

- **Imágenes**: `loading="lazy"` para todo lo que esté fuera del viewport inicial; NUNCA para la imagen hero (eso retrasa el LCP). Combiná con `decoding="async"`.
- **`content-visibility: auto`** en CSS para secciones largas: el navegador salta el render de lo que no se ve.
- **No abuses**: lazy loading en elementos que se ven de inmediato agrega un delay perceptible y empeora las cosas.

```css
/* CSS: el navegador no renderiza secciones fuera de pantalla hasta acercarse */
.seccion-larga {
  content-visibility: auto;
  contain-intrinsic-size: 0 600px; /* evita saltos de scroll */
}
```

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre el lazy loading nativo (`loading="lazy"`) y el de React (`lazy`)?
>    - R: El nativo es para recursos como imágenes/iframes, y lo maneja el navegador. El de React (junto a `dynamic import`) divide el JS en chunks y carga un COMPONENTE completo bajo demanda.
> 2. ¿Por qué NO deberías poner `loading="lazy"` a la imagen hero?
>    - R: Porque la imagen hero es el contenido principal (LCP); si la marcás lazy, el navegador la descarga más tarde y retrasás el LCP, empeorando la métrica más importante.
> 3. ¿Qué hace `content-visibility: auto` y por qué acelera la página?
>    - R: Le dice al navegador que no renderice el contenido de una sección que está fuera del viewport, ahorrando trabajo de layout y paint hasta que el usuario se acerca.
> 4. ¿Qué rol cumple `Suspense` con un componente lazy?
>    - R: Define qué mostrar mientras el chunk del componente se descarga; sin un fallback, el usuario vería un hueco en blanco.
> 5. ¿Cuándo conviene lazy loading y cuándo NO?
>    - R: Conviene para contenido fuera del viewport inicial o rutas poco usadas. No conviene para lo que se ve de inmediato, porque agrega un delay de red extra perceptible.

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.6-nextjs) — `next/dynamic` y `next/image` como lazy loading integrado en Next.js.
→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.1-react) — `React.lazy` y el patrón de carga de componentes bajo demanda.

---

## 13.3 Code Splitting

> Referencia base: [MDN — Dynamic import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) · [React — Code Splitting](https://react.dev/learn/code-splitting) · [Vite — Build](https://vite.dev/guide/build)

*En criollo:* Si lazy loading es la idea, code splitting es la herramienta que la hace posible en el JS. Sin splitting, todo tu código termina en UN bundle gigante que el navegador descarga de una. Con splitting, el bundler parte tu app en pedazos (chunks): uno inicial con lo esencial y el resto que se carga cuando hace falta. El resultado: el primer paint es mucho más rápido porque se descarga menos JavaScript, y el JS es lo más caro de descargar y ejecutar.

*Técnicamente:* El mecanismo clave es el **`import()` dinámico**, que le dice al bundler "esto va en un chunk aparte":

```js
// Static import: entra al bundle principal (siempre)
import { format } from 'date-fns';

// Dynamic import: se parte en un chunk separado, descargado bajo demanda
const { format } = await import('date-fns');
```

En React, `lazy` + `Suspense` hace el splitting por **ruta** (lo más común) o por **componente**:

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Panel = lazy(() => import('./pages/Panel'));
const Reportes = lazy(() => import('./pages/Reportes'));

<BrowserRouter>
  <Suspense fallback={<div>Cargando…</div>}>
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/panel" element={<Panel />} />
      <Route path="/reportes" element={<Reportes />} />
    </Routes>
  </Suspense>
</BrowserRouter>
```

**Splitting por librería**: bibliotecas grandes que solo se usan en un lugar (un editor de texto, un graficador) se cargan solo cuando se necesitan:

```js
// El parser pesado solo se descarga cuando el usuario sube un archivo
async function procesarArchivo(file) {
  const { parse } = await import('papaparse'); // ~100kb que no van al bundle inicial
  return parse(await file.text());
}
```

Reglas de oro:

- **Partí por ruta primero** (es lo más simple y de mayor impacto), después por componente pesado, después por librería.
- **No partas en pedazos demasiado chicos**: cada chunk tiene overhead de request; miles de chunks son peores que un bundle mediano.
- Usá el **análisis de bundle** (ver 13.4) para ver qué estás metiendo en el chunk inicial y qué conviene mover.

> **Check de comprensión**
> 1. ¿Cuál es la relación entre code splitting y lazy loading?
>    - R: Lazy loading es el objetivo (cargar bajo demanda); code splitting es el mecanismo técnico que parte el JS en chunks para que el lazy loading sea posible.
> 2. ¿Qué diferencia hay entre `import` estático y `import()` dinámico?
>    - R: El estático incluye el módulo en el bundle principal siempre; el dinámico crea un chunk separado que se descarga solo cuando se ejecuta esa línea.
> 3. ¿Por qué partir por ruta es la estrategia más común y de mayor impacto?
>    - R: Porque el usuario casi nunca visita todas las rutas en una sesión; al partir por ruta, cada pantalla descarga solo su código, y el bundle inicial queda chico.
> 4. ¿Por qué partir en chunks demasiado chicos puede empeorar la performance?
>    - R: Porque cada chunk es un request HTTP extra con overhead; miles de requests pequeñas son más lentas que un bundle razonablemente agrupado.
> 5. ¿Qué es el "bundle inicial" y por qué conviene mantenerlo mínimo?
>    - R: Es el JS que se descarga y ejecuta antes de mostrar la primera pantalla; es lo más caro en tiempo de arranque, así que conviene que solo incluya lo esencial del primer viewport.

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.4-vite) — cómo Vite/Rollup genera y nombra los chunks automáticamente.
→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.1-react) — `lazy` + `Suspense` como API de code splitting en React.

---

## 13.4 Bundle Analysis

> Referencia base: [webpack-bundle-analyzer — npm](https://www.npmjs.com/package/webpack-bundle-analyzer) · [rollup-plugin-visualizer — npm](https://www.npmjs.com/package/rollup-plugin-visualizer) · [Vite — Build options](https://vite.dev/config/build-options)

*En criollo:* No podés optimizar lo que no podés ver. El bundle analysis es la radiografía de tu JS: una herramienta que te muestra, en un mapa visual, cuánto pesa cada dependencia y qué está ocupando espacio en tu bundle. Casi siempre la sorpresa es la misma: trajiste una librería de 300kb para usar una sola función, o importaste `lodash` entero en vez de la función puntual. Es el primer paso ANTES de optimizar — de ahí salen las decisiones de code splitting (13.3) y de eliminar dependencias.

*Técnicamente:* El visualizador estándar es `webpack-bundle-analyzer`; para Vite (que usa Rollup por debajo) se usa `rollup-plugin-visualizer`:

```bash
# Vite: agregás el plugin y corrés el build
npm install -D rollup-plugin-visualizer
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      open: true,        // abre el reporte en el navegador al terminar el build
      gzipSize: true,    // mostrá el tamaño real que viaja por la red (gzip)
      brotliSize: true,
    }),
  ],
});
```

```bash
npm run build
# → genera stats.html: un mapa de treemap donde cada rectángulo es un módulo
```

Qué buscar en el mapa:

- **Rectángulos gigantes** → dependencias pesadas que tal vez no usás del todo.
- **`lodash` / `date-fns` / `moment` enteros** → importá la función puntual (`import debounce from 'lodash/debounce'`) o usá la alternativa tree-shakeable.
- **Moment.js** → es enorme y legacy; `date-fns` o el `Intl` nativo pesan una fracción.

```js
// MAL: trae TODO lodash (~70kb gzip) por una función
import { debounce } from 'lodash';

// BIEN: solo esa función (tree-shaking)
import debounce from 'lodash/debounce';
```

- **Tu propio código duplicado** → dos versiones de una dependencia (`react` dos veces por versión incompatible) → alineá las versiones.
- **Chunk inicial inflado** → mové lo que no sea del primer viewport a un chunk aparte (13.3).

```bash
# También podés listar tamaños sin abrir el navegador
npx vite build --report
```

> **Check de comprensión**
> 1. ¿Para qué sirve el bundle analysis y por qué va ANTES de optimizar?
>    - R: Te muestra visualmente cuánto pesa cada dependencia en el bundle, así decidís qué achicar con datos en vez de intuición. Sin medir, optimizás a ciegas.
> 2. ¿Qué herramienta usás con Vite y qué con webpack?
>    - R: Con Vite (Rollup) usás `rollup-plugin-visualizer`; con webpack usás `webpack-bundle-analyzer`. Ambas generan un treemap del bundle.
> 3. ¿Por qué `gzipSize: true` importa en el reporte?
>    - R: Porque el navegador recibe los archivos comprimidos con gzip/brotli; el tamaño "gzip" es el que realmente viaja por la red, no el tamaño en disco.
> 4. ¿Qué problema detecta un rectángulo de `lodash` gigante y cómo lo arreglás?
>    - R: Que importaste la librería entera cuando usás una función. Se arregla importando solo la función (`lodash/debounce`) para que el tree-shaking elimine el resto.
> 5. ¿Qué significa ver dos versiones de `react` en el treemap?
>    - R: Que hay una dependencia incompatible que trajo su propia copia de React, duplicando tamaño y pudiendo causar bugs. Se resuelve alineando las versiones a una sola.

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.4-vite) — el pipeline de build de Vite donde encaja el análisis de bundle.

---

## 13.5 N+1 Problem

> Referencia base: [Prisma — Relations](https://www.prisma.io/docs/orm/prisma-schema/data-model/relations) · [DataLoader — GitHub](https://github.com/graphql/dataloader) · [TypeORM — Eager and Lazy Relations](https://typeorm.io/eager-and-lazy-relations)

*En criollo:* El problema N+1 es el clásico error de performance en backend, y es tan común como dañino: en vez de hacer UNA query para traer los datos, hacés una query para traer N cosas y después **una query más por cada una**. Si tenés 100 usuarios y para cada uno preguntás sus posts, son 101 queries (1 + 100). Funciona igual en chico, pero explota en grande: es O(n) consultas extra que podrían ser 2. La peor parte es que casi siempre es **silencioso**: el ORM lo esconde y el código se ve limpio.

*Técnicamente:* El patrón clásico:

```js
// ❌ N+1: 1 query de usuarios + N queries de posts
const usuarios = await db.usuarios.findMany(); // 1 query

for (const usuario of usuarios) {
  usuario.posts = await db.posts.findMany({
    where: { autorId: usuario.id }, // ← una query POR usuario (N queries)
  });
}
// 100 usuarios → 101 queries
```

Cómo se arregla, de mejor a peor:

1. **Eager loading con JOIN o `include`** (Prisma): el ORM trae todo en una query con JOIN.

```js
// ✅ Prisma: include resuelve el N+1 con un JOIN en una sola query
const usuarios = await db.usuarios.findMany({
  include: { posts: true },
});
```

2. **Batching (DataLoader)**: agrupás N claves y las resolvés en una query `IN`. Es EL patrón en GraphQL.

```js
import DataLoader from 'dataloader';

const postLoader = new DataLoader(async (autorIds) => {
  const posts = await db.posts.findMany({
    where: { autorId: { in: autorIds } }, // 1 query para TODOS los ids
  });
  return autorIds.map((id) => posts.filter((p) => p.autorId === id));
});

// En cada resolver, usás el loader en vez de una query directa
const posts = await postLoader.load(usuario.id);
```

3. **Query manual con `IN`**: traés todos los ids y hacés una segunda query.

```js
// ✅ Manual: 2 queries en total
const usuarios = await db.usuarios.findMany();              // 1
const ids = usuarios.map((u) => u.id);
const posts = await db.posts.findMany({
  where: { autorId: { in: ids } },                         // 2
});
// agrupás posts por autorId en memoria
```

**Cómo detectarlo**: en desarrollo, logueá las queries (Prisma tiene `log: ['query']`; en Postgres `EXPLAIN ANALYZE`); en producción, un dashboard de DB muestra picos de queries idénticas. La regla mental: si ves un `for` con una query adentro, es N+1.

> **Check de comprensión**
> 1. ¿Qué es exactamente el problema N+1?
>    - R: Es hacer 1 query para traer N registros y después 1 query extra por cada registro para traer sus datos relacionados, cuando 2 queries (o un JOIN) bastarían.
> 2. ¿Por qué es "silencioso" con un ORM?
>    - R: Porque el ORM genera las queries por vos y el código se ve limpio; el N+1 no tira un error, solo degrada la performance a medida que crecen los datos.
> 3. ¿Cómo resuelve Prisma el N+1 con `include`?
>    - R: Genera una sola query con JOIN (o queries optimizadas) que trae los registros y sus relaciones de una, en vez de una query por cada relación.
> 4. ¿Qué es DataLoader y cómo resuelve el N+1?
>    - R: Es una librería de batching que acumula claves de requests individuales y las resuelve en UNA query con `IN`, y cachea los resultados dentro del request. Es la solución estándar en GraphQL.
> 5. ¿Cómo detectás un N+1 en tu código antes de que llegue a producción?
>    - R: Activando el log de queries del ORM y buscando el patrón "una query repetida N veces", o revisando cualquier `for`/`map` con una query adentro.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.6-estrategias-de-query) — el N+1 y otras estrategias de query en detalle.
→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.8-indices) — cómo los índices hacen que esas queries sean rápidas una vez resueltas.

---

## 13.6 Caching Strategies

> Referencia base: [MDN — Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) · [web.dev — HTTP caching](https://web.dev/articles/http-cache) · [Redis — Documentation](https://redis.io/docs/latest/)

*En criollo:* Caching es la optimización con mejor relación costo/beneficio: **no recalculés ni retransmitas lo que ya tenés**. Hay dos mundos. El **caching HTTP** (en el navegador y en CDNs) evita volver a descargar assets estáticos que no cambian. El **caching de datos** (Redis, memoria) evita pegarle a la base de datos para cada request idéntico. La trampa del caching es siempre la misma: la **invalidación** — ¿cómo sabés que lo cacheado sigue siendo válido? Los dos errores clásicos son cachear lo que cambia seguido (servís datos viejos) o no cachear nada (pegarle a la DB en cada request).

*Técnicamente:* Estrategias de caché, de la más simple a la más sofisticada:

**1. Cache-Control (HTTP)** — el navegador y los CDN cachean respuestas:

```http
# Inmutable: assets con hash de contenido, cachear para siempre
Cache-Control: public, max-age=31536000, immutable

# Revalidar: el servidor confirma si cambió con ETag/Last-Modified
Cache-Control: no-cache

# Nunca cachear: datos personales o que cambian
Cache-Control: no-store
```

**2. Cache-aside (el patrón más común)** — revisás el caché antes de la DB:

```js
import { createClient } from 'redis';

const redis = createClient();

async function getUsuario(id) {
  const clave = `usuario:${id}`;

  // 1. Mirá el caché primero
  const cacheado = await redis.get(clave);
  if (cacheado) return JSON.parse(cacheado);

  // 2. Si no está, pegale a la DB y guardá en caché
  const usuario = await db.usuarios.findUnique({ where: { id } });
  await redis.set(clave, JSON.stringify(usuario), { EX: 300 }); // TTL 5 min

  return usuario;
}
```

**3. Invalidación** — el corazón del problema. Patrones:

| Patrón | Cómo funciona | Cuándo usarlo |
|--------|---------------|---------------|
| **TTL** | El dato expira solo después de X segundos | Datos tolerantes a leves demoras (listas, catálogo) |
| **Invalidación por escritura** | Al actualizar la DB, borrás/actualizás la clave | Datos que deben ser exactos tras un write |
| **Cache stampede** | Muchos misses simultáneos pegan a la DB a la vez | Mitigás con bloqueo, request coalescing o early-expiration |

```js
// Invalidación por escritura: al actualizar, borrá la clave
async function actualizarUsuario(id, datos) {
  await db.usuarios.update({ where: { id }, data: datos });
  await redis.del(`usuario:${id}`); // ← la próxima lectura revalida
}
```

**Regla de oro**: cacheá en la capa correcta. Assets estáticos → HTTP/CDN. Datos de DB calientes → Redis. No metas lógica de negocio en el caché; cacheá RESULTADOS, no proceso.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre `no-cache` y `no-store` en Cache-Control?
>    - R: `no-cache` permite cachear pero obliga a revalidar con el servidor antes de usar; `no-store` prohíbe cachear del todo. `no-cache` es para datos que cambian pero se pueden revalidar; `no-store` para datos sensibles.
> 2. ¿Qué es el patrón cache-aside?
>    - R: Mirar el caché primero; si está (hit) devolverlo; si no (miss), leer de la DB, guardarlo en caché con un TTL y devolverlo. Es el patrón más común.
> 3. ¿Qué es la invalidación y por qué es "el problema difícil" del caching?
>    - R: Es decidir cuándo un dato cacheado dejó de ser válido. Es difícil porque si cacheás de más servís datos viejos, y si invalidás de menos, el caché no sirve.
> 4. ¿Qué es un cache stampede y cómo lo mitigás?
>    - R: Es cuando una clave popular expira y cientos de requests simultáneos hacen miss y pegan a la DB a la vez. Se mitiga con bloqueo (un solo request rellena), request coalescing o expiración temprana.
> 5. ¿Por qué los assets con hash de contenido se cachean `immutable`?
>    - R: Porque el hash cambia cuando cambia el contenido, así que una URL con hash apunta siempre al mismo bytes; puede cachearse para siempre sin riesgo de servir una versión vieja.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.7-redis) — Redis como capa de caching de datos.
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.3-protocolo-http) — los headers HTTP donde se declara el caching.

---

## 13.7 Database Indexing

> Referencia base: [PostgreSQL — Indexes](https://www.postgresql.org/docs/current/indexes.html) · [MongoDB — Indexes](https://www.mongodb.com/docs/manual/indexes/) · [Use The Index, Luke](https://use-the-index-luke.com/)

*En criollo:* Un índice es a una tabla lo que el índice de un libro es a sus capítulos: sin él, para encontrar algo recorrés página por página (full scan); con él, vas directo. Cuando una query filtra por `WHERE email = ...`, sin índice la base revisa TODAS las filas una por una. Con un índice en `email`, salta directo al registro. El costo: cada índice ocupa espacio y hace que los `INSERT`/`UPDATE` sean un poco más lentos (porque hay que mantenerlo actualizado). Por eso no indexás todo: indexás las columnas por las que **filtrás, ordenás y joineás**.

*Técnicamente:* Tipos de índice y cuándo usarlos:

| Tipo | Qué acelera | Cuándo |
|------|-------------|--------|
| **B-tree** (default) | Igualdad, rangos, orden (`=`, `>`, `<`, `ORDER BY`) | La mayoría de los casos |
| **Hash** | Solo igualdad exacta (`=`) | Lookups por clave exacta |
| **GIN** | Full-text search, arrays, JSONB | Búsqueda en documentos |
| **Compuesto** | Consultas que filtran por varias columnas | `WHERE a = 1 AND b = 2` |
| **Unique** | Integridad + velocidad | `email`, `username` |

```sql
-- Crear un índice en la columna que filtrás
CREATE INDEX idx_usuarios_email ON usuarios (email);

-- Índice compuesto: el orden importa (columna más selectiva primero)
CREATE INDEX idx_posts_autor_fecha ON posts (autor_id, creado_en DESC);

-- Ver si la query usa el índice (o hace full scan)
EXPLAIN ANALYZE SELECT * FROM usuarios WHERE email = 'leandro@ejemplo.com';
-- Buscá "Index Scan" (bueno) vs "Seq Scan" (full scan, malo)
```

```js
// Prisma: declarás el índice en el schema
model Usuario {
  id    Int    @id @default(autoincrement())
  email String @unique // ← índice unique automático

  @@index([nombre]) // ← índice simple
}
```

Reglas de oro:

- **El orden de las columnas en un índice compuesto importa**: el índice `(a, b)` no ayuda a filtrar solo por `b` (el prefijo izquierdo manda).
- **`EXPLAIN ANALYZE` antes de indexar**: no adivines; mirá el plan de ejecución real.
- **No indexes en exceso**: cada índice ralentiza escrituras y gasta disco.
- **Cuidado con los leading wildcards**: `LIKE '%texto'` no usa índice; `LIKE 'texto%'` sí.

```sql
-- ❌ No usa índice (leading wildcard)
SELECT * FROM posts WHERE titulo LIKE '%performance';

-- ✅ Usa índice
SELECT * FROM posts WHERE titulo LIKE 'performance%';
```

> **Check de comprensión**
> 1. ¿Qué es un índice y qué problema resuelve?
>    - R: Es una estructura que permite a la base encontrar filas sin recorrer toda la tabla (full scan). Resuelve la lentitud de queries que filtran, ordenan o joinean por una columna no indexada.
> 2. ¿Cuál es el costo de un índice y por qué no indexás todo?
>    - R: Ocupa espacio en disco y hace más lentos los `INSERT`/`UPDATE` (hay que mantenerlo actualizado). Por eso indexás solo las columnas que realmente se usan en filtros, orden y joins.
> 3. ¿Qué significa que un índice compuesto es "por prefijo izquierdo"?
>    - R: Que un índice `(a, b)` sirve para filtrar por `a` y por `a + b`, pero NO por `b` solo. El orden de las columnas define qué combinaciones de filtro se benefician.
> 4. ¿Para qué usás `EXPLAIN ANALYZE`?
>    - R: Para ver el plan de ejecución real de una query: si usa "Index Scan" (bien) o "Seq Scan" (full scan). Es la forma de confirmar que un índice se está usando antes/después de crearlo.
> 5. ¿Por qué `LIKE '%texto'` no usa índice pero `LIKE 'texto%'` sí?
>    - R: Porque el índice B-tree está ordenado de izquierda a derecha; con un prefijo conocido puede saltar directo, pero con un wildcard al inicio no sabe dónde empezar y debe escanear todo.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.8-indices) — índices en PostgreSQL y MongoDB en profundidad.
→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.6-estrategias-de-query) — cómo indexar se combina con el N+1 y la paginación.

---

## 13.8 Paginación

> Referencia base: [PostgreSQL — LIMIT and OFFSET](https://www.postgresql.org/docs/current/queries-limit.html) · [Prisma — Pagination](https://www.prisma.io/docs/orm/prisma-client/queries/pagination) · [Apollo — Pagination](https://www.apollographql.com/docs/react/pagination/overview/)

*En criollo:* Nadie lista mil registros de una. La paginación es partir un conjunto de resultados en páginas chicas, y es OBLIGATORIA: si devolvés todo de una, la API tarda, la DB sufre y el cliente se cuelga renderizando. Pero no todas las paginaciones son iguales. El `OFFSET` clásico es fácil de entender pero se degrada con datos grandes (para llegar a la página 1000, la base salta 99.000 filas). La **cursor-based** (la que usan las APIs modernas) es más robusta: en vez de "saltá N", decís "dame los que vienen después de ESTE elemento".

*Técnicamente:* Los dos enfoques:

**1. Offset/Limit (simple, pero se degrada):**

```js
// Página 3, 20 por página
const posts = await db.posts.findMany({
  skip: 40,   // OFFSET 40
  take: 20,   // LIMIT 20
  orderBy: { creadoEn: 'desc' },
});
// Problema: para skip 40 la base igual recorre 40 filas; con skip 1.000.000 es lento
```

```sql
SELECT * FROM posts ORDER BY creado_en DESC LIMIT 20 OFFSET 40;
```

**2. Cursor-based (keyset pagination, la robusta):**

```js
// Primera página: sin cursor
const { items, nextCursor } = await getPosts({ take: 20 });

// Siguientes: pasás el cursor (el último elemento que viste)
const { items, nextCursor } = await getPosts({ take: 20, cursor: nextCursor });
```

```js
// Implementación con keyset: WHERE con el último valor visto, no OFFSET
async function getPosts({ take, cursor }) {
  const posts = await db.posts.findMany({
    take,
    orderBy: { creadoEn: 'desc' },
    ...(cursor && { cursor: { id: cursor }, skip: 1 }), // Prisma cursor pagination
  });
  const nextCursor = posts.length === take ? posts[posts.length - 1].id : null;
  return { items: posts, nextCursor };
}
```

**Por qué cursor-based gana en datos grandes:**

| | Offset/Limit | Cursor-based |
|---|---|---|
| Costo en página 1000 | Escanea 99.999 filas para saltar | Salta directo con un índice |
| Estabilidad con datos que cambian | Salta/duplica items si insertás en el medio | Estable (sigue al cursor) |
| Complejidad | Simple | Un poco más |
| ¿Saltar a página N directo? | Sí | No (solo siguiente/anterior) |

```js
// Un índice en la columna de orden hace el cursor O(log n)
// CREATE INDEX idx_posts_creado ON posts (creado_en DESC);
```

**Reglas de oro:**

- **`LIMIT` SIEMPRE** en toda query que devuelva listas; un `findMany()` sin límite es una bomba de tiempo.
- El **cursor debe ser estable y ordenable** (un `id` monotónico o `(creado_en, id)` compuesto para evitar empates).
- El cliente devuelve el cursor; **nunca confíes en el offset** como identidad (los datos cambian entre requests).

> **Check de comprensión**
> 1. ¿Por qué la paginación es obligatoria en APIs que devuelven listas?
>    - R: Porque devolver todos los registros de una satura la base, la red y el cliente. Partir en páginas chicas mantiene cada respuesta acotada y predecible.
> 2. ¿Cuál es el problema de `OFFSET` con datos grandes?
>    - R: Para llegar a un offset alto, la base debe escanear y descartar todas las filas anteriores; el costo crece linealmente, así que la página 1000 es mucho más lenta que la 1.
> 3. ¿Cómo funciona la paginación cursor-based?
>    - R: En vez de "saltá N", usás la clave del último elemento visto como punto de partida: "dame los que vienen después de este cursor". Con un índice, salta directo en O(log n).
> 4. ¿Por qué el cursor debe ser estable y ordenable?
>    - R: Porque define el punto exacto de continuación; si no es único (hay empates) o cambia con el tiempo, podés saltear o duplicar elementos entre páginas.
> 5. ¿Cuándo elegís offset/limit en vez de cursor?
>    - R: Para datasets chicos, o cuando el cliente necesita saltar a una página arbitraria (página 5 de 10). Para feeds infinitos o datos grandes, cursor-based es la opción robusta.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.6-estrategias-de-query) — paginación cursor-based y condiciones WHERE con el último valor visto.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.2-apis-rest) — cómo exponer la paginación en el contrato de una API REST.
