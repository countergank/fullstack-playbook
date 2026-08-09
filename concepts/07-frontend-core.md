# 7. Frontend Core

> Objetivo: dominar los fundamentos del frente — el trío HTML/CSS/JS, TypeScript como capa de control, el DOM y las Web APIs — ANTES de tocar cualquier framework. React, Vue o Svelte son capas que asumen que ya sabés esto.

> Nota: el concept 01 cubrió HTTP, DNS y navegadores. Acá no se repite — esto es el lado cliente puro: marcado, estilos, lógica, tipos, DOM y APIs del navegador.

---

## 7.1 HTML Semántico

*En criollo:* HTML es el esqueleto de la página. Pero no cualquier esqueleto: un HTML semántico es como un esqueleto bien etiquetado que un buscador, un lector de pantalla y otro developer entienden sin leer una línea de CSS o JS. `<div>` es una caja genérica; `<header>`, `<nav>`, `<article>` dicen qué ES cada cosa.

### La diferencia que cambia todo

| Elemento | Qué es | vs el genérico |
|----------|--------|----------------|
| `<header>` | Cabecera de página/sección | `<div class="cabecera">` |
| `<nav>` | Navegación principal | `<div class="menu">` |
| `<main>` | Contenido principal (único por página) | `<div class="contenido">` |
| `<article>` | Contenido autocontenido (post, noticia) | `<div class="post">` |
| `<section>` | Agrupación temática con heading propio | `<div class="seccion">` |
| `<aside>` | Contenido complementario (sidebar) | `<div class="lateral">` |
| `<footer>` | Pie de página | `<div class="pie">` |
| `<figure>` / `<figcaption>` | Imagen/ilustración + su leyenda | `<div><img></div>` |
| `<time>` | Fecha/hora legible por máquina | `<span>12/05/2026</span>` |
| `<address>` | Info de contacto | `<div>email...</div>` |

### Reglas de oro

- **Una sola etiqueta `<h1>` por página** — y la jerarquía de headings (`h1` → `h2` → `h3`) no debe saltear niveles sin razón.
- **Los headings no son para el tamaño de fuente**: son la estructura del documento. El tamaño lo decide CSS.
- **Si hay que usar `<div>` para todo, algo está mal pensado.** El div sobra cuando existe un elemento semántico que describe el contenido.
- **Formularios**: `<label>` SIEMPRE asociado a su input (`for` + `id`), y `<fieldset>`/`<legend>` para agrupar.

```html
<!-- ❌ Genérico: nadie sabe qué es esto sin CSS -->
<div class="top">
  <div class="menu"><a href="/">Inicio</a></div>
</div>

<!-- ✅ Semántico: el navegador, el lector y el dev lo entienden -->
<header>
  <nav><a href="/">Inicio</a></nav>
</header>
```

---

## 7.2 CSS Moderno

*En criollo:* CSS ya no es "poner colores". Con Flexbox y Grid podés hacer layouts que antes requerían hacks con floats y position. La regla mental: **Flexbox alinea cosas en una línea (1 dimensión), Grid arma la grilla de la página (2 dimensiones)**.

### Flexbox — el layout de una dimensión

Usalo cuando un contenedor tiene items que deben alinearse/distribuirse en **una fila o una columna**.

```css
.nav {
  display: flex;
  justify-content: space-between; /* main axis: separar items */
  align-items: center;            /* cross axis: centrar verticalmente */
  gap: 1rem;                      /* separación sin margins */
}
```

| Propiedad | Qué controla |
|-----------|--------------|
| `justify-content` | Distribución en el eje principal (row/column) |
| `align-items` | Alineación en el eje cruzado |
| `flex-direction` | ¿Fila o columna? (`row` default, `column`) |
| `flex-wrap` | ¿Se envuelven o desbordan? |
| `gap` | Espaciado uniforme entre items (¡sin margins!) |
| `flex-grow/flex-shrink` | Cómo crecen/encogen los items |

### Grid — el layout de dos dimensiones

Usalo cuando la página o una sección tiene **filas Y columnas** simultáneas.

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr 200px; /* sidebar + contenido flexible + sidebar */
  grid-template-rows: auto 1fr auto;      /* header + contenido + footer */
  gap: 1rem;
}
```

| Propiedad | Qué controla |
|-----------|--------------|
| `grid-template-columns` | Las columnas (`1fr`, `repeat(3, 1fr)`, `minmax(200px, 1fr)`) |
| `grid-template-rows` | Las filas |
| `grid-template-areas` | Nombrar zonas del layout (`"header header" "sidebar main"`) |
| `gap` | Espaciado entre celdas |
| `fr` | La unidad "fracción del espacio restante" |

### Responsive — un layout que se adapta

- **Mobile-first**: escribís los estilos base para el teléfono y con `min-width` vas aumentando. Más fácil de mantener que empezar por el desktop.
- **Unidades relativas**: `rem` para tipografía (sigue el font-size del root), `%` y `fr` para contenedores, `vh/vw` con cuidado. Evitá `px` para tamaños de texto.
- **Media queries**: el punto de quiebre donde cambia el layout.

```css
.contenedor {
  display: grid;
  grid-template-columns: 1fr;              /* móvil: una columna */
}

@media (min-width: 768px) {
  .contenedor {
    grid-template-columns: 1fr 1fr;        /* tablet: dos */
  }
}

@media (min-width: 1024px) {
  .contenedor {
    grid-template-columns: 1fr 1fr 1fr;    /* desktop: tres */
  }
}
```

- **`clamp()`** para tipografía fluida sin media queries: `font-size: clamp(1rem, 2vw, 1.5rem);` (mínimo, ideal, máximo).
- **`min()` / `max()`**: `width: min(100%, 1200px);` — el ancho no pasa de 1200px pero nunca desborda.
- **Flexbox vs Grid**: si tenés dudas, Grid para la estructura de la página, Flexbox para alinear items dentro de un componente. Ambos se usan juntos todo el tiempo.

---

## 7.3 JavaScript (ECMAScript moderno)

*En criollo:* JS es el único lenguaje que el navegador entiende nativamente (por ahora). Durante años fue un lenguaje "viejito" con trampas; desde ES6+ (2015) es un lenguaje moderno. Hoy se escribe casi siempre con TS (7.4), pero el JS moderno es la base de todo.

### Fundamentos que importan

| Concepto | Cómo era (var/function) | Cómo es hoy |
|----------|------------------------|-------------|
| Declaración | `var x` (scope de función, hoisting tramposo) | `let` (mutable) / `const` (inmutable por binding) |
| Funciones | `function suma(a, b) { return a + b; }` | `const suma = (a, b) => a + b;` |
| Strings | `'hola ' + nombre` | `` `hola ${nombre}` `` (template literals) |
| Objetos | repetir `obj.prop` mil veces | destructuring `const { name, age } = user;` |
| Arrays | loops manuales | `map`, `filter`, `reduce`, `find`, `some`, `every` |
| Módulos | `<script>` globales | `import` / `export` (ESM) |
| Asincronía | callbacks (callback hell) | `async/await` sobre Promises |

### Asincronía — el tema que rompe más cabezas

JS es **single-threaded** pero no bloqueante: mientras espera una respuesta de red, sigue ejecutando. Las Promises y `async/await` son la forma moderna de encadenar trabajo asíncrono.

```js
// ❌ Callback hell
getUser(id, (user) => {
  getPosts(user.id, (posts) => {
    render(user, posts);
  });
});

// ✅ async/await — lineal, legible, igual a código síncrono
async function renderUser(id) {
  const user = await getUser(id);
  const posts = await getPosts(user.id);
  render(user, posts);
}
```

**Reglas de oro de asincronía:**
- `await` SOLO dentro de una función `async`.
- Los errores de una Promise se atrapan con `try/catch` alrededor del `await`.
- `Promise.all([...])` cuando las llamadas son independientes (paralelas), no secuenciales.
- Nunca hagas `await` en un loop cuando podés hacer `Promise.all` — cada `await` secuencial suma latencia.

### Arrays — los métodos que usás todos los días

```js
const users = [{ name: 'Lean', age: 34 }, { name: 'Ana', age: 41 }];

const names = users.map(u => u.name);                 // transformar → ["Lean", "Ana"]
const adults = users.filter(u => u.age >= 35);        // filtrar    → [Ana]
const totalAge = users.reduce((acc, u) => acc + u.age, 0); // acumular → 75
const found = users.find(u => u.name === 'Ana');      // encontrar 1 → objeto
const hasYoung = users.some(u => u.age < 30);         // alguno cumple → false
```

---

## 7.4 TypeScript

*En criollo:* TypeScript es JavaScript con tipos que se desvanecen. Escribís JS con anotaciones de tipos; un compilador (tsc) los valida ANTES de que el código corra; y al compilar se borran los tipos y queda JS puro que el navegador/Node entiende. Tipo = red de seguridad en tiempo de desarrollo, cero costo en runtime.

### Qué te compra

- **Errores antes de ejecutar**: un `user.name` donde `user` es `null` se detecta en el editor, no en producción.
- **Autocompletado real**: el editor conoce las propiedades de tus objetos y funciones.
- **Refactor seguro**: cambiás un tipo y el compilador te dice TODO lo que se rompe.
- **Documentación viva**: la firma de la función es su contrato.

### Tipos esenciales

```ts
// Tipos básicos
let id: number = 1;
let name: string = 'Lean';
let active: boolean = true;

// Arrays
const tags: string[] = ['node', 'ts'];

// Union — "este valor puede ser A o B"
type Id = string | number;

// Objecto tipado (interface)
interface User {
  id: Id;
  name: string;
  age?: number;        // opcional
  readonly email: string; // no reasignable
}

// Generics — tipos parametrizables (el T se rellena en el uso)
function first<T>(items: T[]): T | undefined {
  return items[0];
}

const firstUser = first<User>([user]);  // firstUser: User | undefined
```

### `interface` vs `type` — cuándo cada uno

| | `interface` | `type` |
|---|-------------|--------|
| Objetos / shape | ✅ ideal, declarativo | ✅ |
| Union / intersection | ❌ no | ✅ el único que puede |
| Extender | `extends` (natural) | `&` (intersección) |
| Cuál usar | Default para objetos/APIs | Unions, tuplas, alias compuestos |

> **Regla práctica**: `interface` para el shape de objetos (las entidades), `type` para unions y alias. Los dos coexisten — no es religión, es herramienta.

### TypeScript es TIPADO ESTRUCTURAL

En TS no importa el NOMBRE de la clase, importa la FORMA: si un objeto tiene las propiedades requeridas, es compatible. Esto se llama **duck typing estructural** — y es la razón por la que tipás tu API en el backend con Prisma y los mismos shapes fluyen al frontend sin fricción.

---

## 7.5 DOM (Document Object Model)

*En criollo:* Pagina cargada = el navegador arma un árbol en memoria con cada etiqueta HTML como un nodo. Ese árbol es el DOM. Tu JS no "ve" el HTML — ve el DOM, y lo modifica para cambiar la página sin recargar.

### Operaciones esenciales

```js
// 1. Seleccionar
const button = document.querySelector('#btn');      // el primero que matchea
const buttons = document.querySelectorAll('.btn');  // NodeList (iterable)
const input = document.getElementById('email');

// 2. Crear y agregar
const li = document.createElement('li');
li.textContent = 'Nuevo item';
list.append(li);          // append acepta nodos Y strings

// 3. Escuchar eventos
button.addEventListener('click', (event) => {
  console.log('Click!', event.target);
});

// 4. Leer/escribir contenido
input.value;                 // leer valor de un input
element.innerHTML = html;    // ⚠️ peligro: renderiza HTML (XSS si es data del usuario)
element.textContent = texto; // ✅ seguro: solo texto
```

### Event bubbling — por qué los eventos "suben"

Cuando hacés clic en un `<li>`, el evento sube: `li` → `ul` → `div` → `body`. Aprovecharlo = **event delegation**: un solo listener en el contenedor maneja todos los hijos, presentes y futuros.

```js
// ❌ Un listener por cada item (y los nuevos no tienen listener)
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handler);
});

// ✅ Delegación: un listener en el padre maneja TODO
list.addEventListener('click', (event) => {
  const item = event.target.closest('.item');  // ¿el clic fue en un item?
  if (item) handler(item);
});
```

### Performance del DOM — el costo invisible

- Cada cambio al DOM puede disparar **reflow** (recalcular layout) y **repaint** (redibujar). Son caros.
- **Batch de operaciones**: acumulá cambios y aplicálos de una — el navegador refluye una vez.
- **`textContent` > `innerHTML`** cuando no necesitás HTML.
- Los frameworks (tema 8) existen en gran parte para resolver estas operaciones de forma eficiente y declarativa — pero no sabés por qué, hasta que viste el costo manual.

---

## 7.6 Accesibilidad (a11y)

*En criollo:* Accesibilidad es que tu web la pueda usar TODO el mundo, incluida ~una de cada seis personas con discapacidad (visual, motriz, auditiva, cognitiva). No es un checkbox final: es parte del HTML semántico y de cómo escribís componentes. Además: es ley en muchos países y Google lo penaliza en SEO.

### Los pilares

| Pilar | Qué hacer |
|-------|-----------|
| **Semántica** | Usar `<button>` para botones, `<nav>` para nav, `<h1>`-`<h6>` para estructura (del 7.1) — los lectores de pantalla se apoyan en esto |
| **Texto alternativo** | `alt` descriptivo en imágenes: `alt="Gráfico de ventas Q1"` (vacío `alt=""` si es decorativa) |
| **Navegación por teclado** | Todo elemento interactivo operable con `Tab`/`Enter`/`Espacio` (los `<button>` reales lo hacen gratis; un `div clickeable` NO) |
| **Contraste** | Texto sobre fondo con ratio ≥ 4.5:1 para texto normal (WCAG AA) |
| **Focus visible** | El contorno de focus NO debe quitarse (`outline: none` sin reemplazo es un error) |
| **ARIA** | Solo cuando HTML no alcanza: `aria-label`, `role`, `aria-live`. Regla: **no ARIA si un elemento HTML nativo lo resuelve** |

### El error más común

```html
<!-- ❌ Un "botón" que no es botón: el teclado no lo puede operar -->
<div class="btn" onclick="enviar()">Enviar</div>

<!-- ✅ Un botón real: tabulable, activator con Enter/Espacio, anunciado por lectores -->
<button onclick="enviar()">Enviar</button>
```

### Test rápido de accesibilidad

1. ¿Navegás toda la página con `Tab` y ves qué estás enfocando?
2. ¿Hay un único `<h1>` y jerarquía de headings coherente?
3. ¿Todo input tiene `<label>`?
4. ¿Las imágenes informativas tienen `alt` descriptivo?
5. ¿La app funciona con un lector de pantalla? (probá el [screen reader emulator de Chrome](https://developer.chrome.com/docs/devtools/accessibility/reference/) en DevTools)

---

## 7.7 Web APIs

*En criollo:* El navegador expone decenas de APIs listas para usar — sin instalar nada. Son la interfaz entre tu JS y las capacidades del navegador: red, almacenamiento, geolocalización, canvas, observadores, notificaciones.

### Las que todo fullstack debería conocer

| API | Qué hace | Caso real |
|-----|----------|-----------|
| **`fetch`** | Hacer requests HTTP desde el cliente | Consumir tu backend: `await fetch('/api/users')` |
| **`localStorage` / `sessionStorage`** | Persistir key-values en el navegador | Token de sesión, preferencias de tema (ojo: no guardes secretos) |
| **`IntersectionObserver`** | Detectar cuándo un elemento entra en el viewport | Lazy loading de imágenes, "infinite scroll" |
| **`Canvas`** | Dibujar gráficos pixel a pixel | Gráficos, juegos, edits de imagen |
| **`Geolocation`** | Posición del usuario (pide permiso) | Mapas, delivery, check-ins |
| **`Notification`** | Notificaciones del sistema (pide permiso) | Alertas de mensajes |
| **`WebSocket`** | Conexión bidireccional persistente | Chats, dashboards en vivo (visto en 5.8) |

### `fetch` — el patrón que usás todos los días

```js
// GET
const res = await fetch('/api/users');
const users = await res.json();            // ⚠️ res.json() devuelve OTRA promise

// POST con JSON
const res = await fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Lean' }),
});

// Manejo de errores — fetch NO tira throw por status 4xx/5xx
if (!res.ok) {
  throw new Error(`HTTP ${res.status}: ${res.statusText}`);
}
```

**Gotchas de `fetch`:**
- **No lanza error en 404/500** — solo en fallo de red. Siempre revisá `res.ok`.
- **`res.json()` es asíncrono** — devuelve una Promise. Olvidar el `await` es un bug clásico.
- **CORS aplica**: si consumís otra origin, el servidor debe permitirlo (concept 1.8).

---

> **Check de comprensión**:
> 1. ¿Por qué `<button>` es mejor que un `<div>` con `onclick`? Mencioná por lo menos dos razones.
> 2. Tenés una navbar con logo, links y un botón, todos en una fila. ¿Flexbox o Grid? ¿Y para el layout general de la página con sidebar?
> 3. ¿Qué diferencia hay entre `map`, `filter` y `reduce`? Escribí un ejemplo de cada uno de memoria.
> 4. ¿Qué pasa con los tipos de TypeScript en producción? ¿Corren en el navegador?
> 5. ¿Qué es el event bubbling y cómo lo aprovecha la event delegation?
> 6. ¿Por qué `res.json()` dentro de un `fetch` necesita su propio `await`?