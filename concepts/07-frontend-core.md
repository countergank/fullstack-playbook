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

> [MDN — Semantic HTML](https://developer.mozilla.org/en-US/docs/Glossary/Semantics)

*Técnicamente:* El parser HTML del navegador construye el DOM tree token por token. Cada etiqueta semántica mapea a un `HTMLElement` con un `role` ARIA implícito: `<nav>` → `role="navigation"`, `<main>` → `role="main"`, `<article>` → `role="article"`. Los screen readers (NVDA, VoiceOver) consultan la Accessibility Tree — derivada del DOM — y anuncian estos roles al usuario. Un `<div>` tiene `role="generic"`: no comunica nada. El validador W3C Nu también rechaza estructuras inválidas como `<section>` sin heading o múltiples `<main>`.

> **Check de comprensión**
> 1. ¿Por qué `<article>` es mejor que `<div class="post">`? Mencioná al menos dos consumidores que se benefician.
>    - R: `<article>` comunica explícitamente que el contenido es autocontenido. Beneficia a: (1) buscadores que indexan contenido independiente, (2) lectores de pantalla que anuncian la estructura semántica, (3) otros developers que leen el markup sin CSS.
> 2. ¿Cuántos `<h1>` debe tener una página y por qué?
>    - R: Exactamente uno. El `<h1>` representa el tema principal del documento; múltiples h1 confunden la jerarquía y penalizan SEO y accesibilidad.
> 3. ¿Cuál es la diferencia entre `<section>` y `<div>`?
>    - R: `<section>` agrupa contenido temático con su propio heading (h2-h6), mientras que `<div>` es un contenedor genérico sin significado semántico.
> 4. ¿Por qué `<label for="email">` es mejor que poner el texto al lado del input sin label?
>    - R: El `for` vincula el label al input por su `id`: (1) clic en el label enfoca el input, (2) lectores de pantalla anuncian "Email, edit text", (3) es requisito WCAG 2.1.
> 5. ¿Qué elemento usarías para una imagen con leyenda descriptiva?
>    - R: `<figure>` envuelve la imagen y `<figcaption>` provee la leyenda. Es semánticamente correcto y asociado.
> 6. Si un contenido es complementario pero no esencial (como un sidebar con links relacionados), ¿qué etiqueta usás?
>    - R: `<aside>` — indica contenido tangencial al contenido principal.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.1-modelado-relacional) — los formularios que validás en el frontend envían datos a modelos relacionales en el backend.
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.7-http) — el HTML viaja como body de respuestas HTTP desde el servidor al navegador.

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

> [MDN — Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)
> [MDN — Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout)
> [MDN — clamp()](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp)

*Técnicamente:* Flexbox usa un algoritmo de layout de una dimensión definido en la spec CSS Flexible Box Layout Module Level 1. El navegador calcula el `main-size` de cada flex item resolviendo `flex-grow`, `flex-shrink` y `flex-basis` en un proceso de "flexing" iterativo. Grid opera con un algoritmo de dos dimensiones (CSS Grid Layout Level 2) donde el track sizing resuelve `minmax()`, `fr` units y `auto` en tres passes: intrinsic sizing, max-content contribution, y fr distribution. Los media queries se evalúan en el CSS cascade: el navegador recalcula el style tree cuando el viewport cruza un breakpoint, disparando reflow. `clamp(min, preferred, max)` es equivalente a `max(min, min(max, preferred, max), min)` — una función CSS nativa resuelta en layout time, no en JS.

> **Check de comprensión**
> 1. ¿Cuándo usás Flexbox y cuándo Grid? Dá un ejemplo concreto de cada uno.
>    - R: Flexbox para alinear items en una dimensión (navbar, botones en fila). Grid para layouts de dos dimensiones (página con sidebar + contenido + footer). Ejemplo: navbar con `display: flex; justify-content: space-between`; layout de página con `display: grid; grid-template-columns: 200px 1fr`.
> 2. ¿Qué significa `1fr` en `grid-template-columns: 200px 1fr 200px`?
>    - R: "Una fracción del espacio restante". Después de asignar 200px a cada sidebar, el `1fr` ocupa todo lo que sobra.
> 3. ¿Por qué mobile-first es mejor que desktop-first?
>    - R: Porque escribís los estilos base para el dispositivo más limitado y con `min-width` vas agregando complejidad. Es más fácil mantener y los móviles cargan menos CSS innecesario.
> 4. ¿Qué hace `clamp(1rem, 2vw, 1.5rem)` exactamente?
>    - R: Establece un font-size con mínimo 1rem, ideal 2vw del viewport, y máximo 1.5rem. El navegador elige el valor intermedio acotado por los extremos.
> 5. ¿Qué diferencia hay entre `justify-content` y `align-items` en Flexbox?
>    - R: `justify-content` distribuye items en el eje principal (horizontal en row), `align-items` los alinea en el eje cruzado (vertical en row).
> 6. ¿Qué pasa si usás `px` para tamaños de texto en lugar de `rem`?
>    - R: El texto no escala con las preferencias del usuario (zoom del navegador, configuración de font-size base). `rem` respeta el font-size del root y es accesible.

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.2-componentes) — los frameworks usan CSS-in-JS o utility classes pero el layout sigue siendo Flexbox/Grid por debajo.
→ Ver [Tópico 7.6: Accesibilidad](../concepts/07-frontend-core.md#7.6-accesibilidad-a11y) — el contraste de colores y el focus visible se resuelven con CSS.

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

> [MDN — ES6+](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
> [MDN — async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
> [MDN — Array methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

*Técnicamente:* El Event Loop de JS procesa tareas en un solo thread: el call stack ejecuta código síncrono, y las operaciones asíncronas (fetch, setTimeout, I/O) se delegan a Web APIs del navegador. Cuando completan, sus callbacks se encolan en la task queue (macrotasks) o microtask queue (Promises). El event loop vacía primero la microtask queue — por eso las Promises tienen prioridad sobre `setTimeout(fn, 0)`. `async/await` es syntax sugar sobre Promises: el compilador V8 transforma `await expr` en una cadena `.then()` con estado suspendido. Los métodos de array (`map`, `filter`, `reduce`) son funciones de orden superior que reciben callbacks y retornan nuevos arrays sin mutar el original — programación funcional pura.

> **Check de comprensión**
> 1. ¿Por qué `const` es preferible a `let` como regla general?
>    - R: `const` previene reasignaciones accidentales y comunica intención: ese binding no cambia. Usás `let` solo cuando necesitás reasignar (contadores, flags). `var` tiene scope de función y hoisting tramposo — nunca se usa en código moderno.
> 2. ¿Qué devuelve `res.json()` y por qué necesita `await`?
>    - R: Devuelve una Promise que resuelve al body parseado como JSON. Necesita `await` porque el parsing es asíncrono — el body puede ser grande y el main thread no debe bloquearse.
> 3. ¿Cuándo usás `Promise.all` en lugar de múltiples `await` secuenciales?
>    - R: Cuando las llamadas son independientes entre sí. `Promise.all([fetchA(), fetchB()])` ejecuta ambas en paralelo. Dos `await` secuenciales suman latencia: A espera, luego B espera.
> 4. ¿Qué hace `reduce` y en qué se diferencia de `map`?
>    - R: `reduce` acumula un array en un solo valor (número, objeto, string). `map` transforma cada elemento y devuelve un array del mismo largo. Ejemplo: `reduce` suma edades, `map` extrae nombres.
> 5. ¿Qué es el "callback hell" y cómo lo resolvés con async/await?
>    - R: Es anidar callbacks dentro de callbacks (getUser → getPosts → render), creando pirámides ilegibles. `async/await` lo convierte en código lineal: `const user = await getUser(); const posts = await getPosts(user.id);`.
> 6. ¿Por qué los arrow functions no tienen su propio `this`?
>    - R: Capturan el `this` léxico del contexto donde se definen. Esto evita el bug clásico de `this` perdiéndose dentro de callbacks o event handlers.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.3-node-js) — Node.js usa el mismo Event Loop y las mismas Promises que el navegador.
→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.1-react) — React abstrae la manipulación del DOM pero usa JS moderno (hooks, async) por debajo.

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

> [TypeScript — Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
> [TypeScript — Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)

*Técnicamente:* TypeScript es un superset tipado de JavaScript que usa inferencia de tipos Hindley-Milner simplificada. El compilador `tsc` realiza type checking estático sin emitir código runtime: los tipos se "borran" (type erasure) y el output es JS puro. El structural typing significa que `{ name: string; age: number }` es compatible con cualquier objeto que tenga esas propiedades, sin importar si viene de una `class`, un `interface` o un literal. El flag `strict: true` activa `strictNullChecks` (null/undefined no son asignables a otros tipos), `noImplicitAny` (prohíbe tipos implícitos `any`), y `strictFunctionTypes` (contravarianza en parámetros). Los generics permiten parametrizar tipos: `Array<T>` es un contenedor cuyo tipo se resuelve en el uso (`Array<User>` → `T = User`).

> **Check de comprensión**
> 1. ¿Qué pasa con los tipos de TypeScript cuando el código corre en el navegador?
>    - R: Desaparecen. TypeScript hace "type erasure" en compilación: borra todas las anotaciones de tipo y emite JavaScript puro. Los tipos solo existen en tiempo de desarrollo.
> 2. ¿Cuándo usás `interface` y cuándo `type`?
>    - R: `interface` para shapes de objetos y entidades (es declarativo y extensible con `extends`). `type` para unions (`string | number`), tuplas, y alias compuestos. Regla: interface por defecto para objetos, type para el resto.
> 3. ¿Qué significa que TypeScript use "tipado estructural"?
>    - R: Que la compatibilidad depende de la FORMA del objeto (sus propiedades y tipos), no de su nombre o clase. Si un objeto tiene `{ id: number; name: string }`, es compatible con cualquier interface que espere esa forma.
> 4. ¿Qué hace `strict: true` en el tsconfig?
>    - R: Activa todas las opciones estrictas: `strictNullChecks` (null/undefined separados), `noImplicitAny` (prohíbe any implícito), `strictFunctionTypes`, entre otras. Es la red de seguridad completa.
> 5. ¿Qué es un generic y para qué sirve?
>    - R: Un tipo parametrizable con una variable (`T`) que se resuelve en el uso. Ejemplo: `function first<T>(items: T[]): T` — si le pasás `User[]`, `T` se resuelve como `User` y el retorno es `User | undefined`.
> 6. ¿Por qué `readonly email: string` no es lo mismo que `const email: string`?
>    - R: `readonly` en una interface prohíbe reasignar esa propiedad en el objeto (`user.email = "x"` tira error). `const` prohíbe reasignar la variable (`email = "x"`). Son niveles distintos de inmutabilidad.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.3-node-js) — TypeScript se usa tanto en frontend como en backend; el mismo tsconfig puede compartirse.
→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.1-react) — React con TypeScript tipa props, state y hooks.

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

> [MDN — DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)
> [MDN — Events](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events)

*Técnicamente:* El DOM es una API en forma de árbol donde cada nodo hereda de `Node` → `Element` → `HTMLElement`. `querySelector` usa el motor Selector API del navegador (Sizzle-like, nativo en C++). `addEventListener` registra un callback en el event dispatcher del elemento; el evento viaja en dos fases: capture (de window al target) y bubble (del target a window). `event.target` es el nodo donde ocurrió el evento real; `event.currentTarget` es el nodo donde está registrado el listener. `closest()` sube el árbol desde el elemento actual buscando un selector. Cada manipulación del DOM (`append`, `remove`, `innerHTML`) puede disparar reflow (recalcular geometría) y repaint (redibujar píxeles) — operaciones O(n) en el tamaño del subtree afectado.

> **Check de comprensión**
> 1. ¿Qué es el event bubbling y por qué es útil?
>    - R: Es el fenómeno donde un evento sube del elemento donde ocurrió (target) hacia sus ancestros hasta `window`. Es útil porque permite event delegation: un solo listener en el padre maneja eventos de todos los hijos, presentes y futuros.
> 2. ¿Por qué `textContent` es más seguro que `innerHTML`?
>    - R: `textContent` inserta texto plano sin interpretar HTML. `innerHTML` parsea y renderiza HTML, lo que permite inyección de scripts maliciosos (XSS) si el contenido viene del usuario.
> 3. ¿Qué diferencia hay entre `querySelector` y `getElementById`?
>    - R: `getElementById` solo busca por ID y es más rápido. `querySelector` acepta cualquier selector CSS (clases, atributos, pseudo-clases) pero es más lento. Ambos devuelven un solo elemento.
> 4. ¿Qué es reflow y por qué es caro?
>    - R: Reflow es el recálculo de la geometría del layout (posiciones y tamaños de elementos). Es caro porque el navegador debe recorrer el DOM, calcular estilos, y reorganizar el render tree. Múltiples cambios al DOM disparan múltiples reflows.
> 5. ¿Qué hace `event.target.closest('.item')`?
>    - R: Sube el árbol del DOM desde `event.target` buscando el primer ancestro (o el mismo elemento) que matchee `.item`. Devuelve `null` si no encuentra ninguno.
> 6. ¿Por qué los frameworks existen si podés manipular el DOM a mano?
>    - R: Porque manipular el DOM manualmente es verboso, propenso a errores, y no escala. Los frameworks abstraen las operaciones del DOM con un modelo declarativo (estado → UI) y optimizan los updates (virtual DOM, signals, fine-grained reactivity).

→ Ver [Tópico 5: WebSockets](../concepts/05-frameworks-backend.md#5.8-websockets) — los WebSockets actualizan el DOM en tiempo real sin recargar.
→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.3-renderizado) — React/Vue abstraen el DOM manual con un modelo declarativo.

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

> [W3C — WCAG 2.2](https://www.w3.org/TR/WCAG22/)
> [MDN — Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility)

*Técnicamente:* WCAG 2.2 define tres niveles de conformidad (A, AA, AAA) con criterios medibles. El nivel AA (el estándar legal en la mayoría de países) exige: contraste ≥ 4.5:1 para texto normal, ≥ 3:1 para texto grande; todo contenido operable por teclado; labels asociados a inputs; y jerarquía de headings lógica. La Accessibility Tree es un subárbol del DOM que los screen readers consumen: cada nodo tiene un `role` (implícito por el elemento HTML o explícito via ARIA), un `name` (texto anunciado), y propiedades como `checked`, `expanded`, `value`. ARIA (Accessible Rich Internet Applications) extiende la semántica HTML con `role`, `aria-*` attributes, y estados. Regla de oro de ARIA: si un elemento HTML nativo resuelve el caso (`<button>`, `<nav>`, `<main>`), NO uses ARIA — el HTML ya comunica el role correcto.

> **Check de comprensión**
> 1. ¿Por qué un `<div onclick="...">` es inaccesible comparado con un `<button>`?
>    - R: El `<button>` nativo es: (1) tabulable con teclado, (2) activable con Enter y Espacio, (3) anunciado como "button" por lectores de pantalla, (4) tiene focus visible. Un `<div>` con onclick no tiene ninguna de estas propiedades.
> 2. ¿Cuál es el ratio mínimo de contraste WCAG AA para texto normal?
>    - R: 4.5:1. Para texto grande (18pt+ o 14pt bold) el mínimo es 3:1.
> 3. ¿Cuándo deberías usar ARIA?
>    - R: Solo cuando HTML nativo no alcanza. Ejemplo: un toggle custom necesita `role="switch"` y `aria-checked`. Si un `<button>` o `<input type="checkbox">` resuelve el caso, no uses ARIA.
> 4. ¿Qué hace `alt=""` (alt vacío) en una imagen?
>    - R: Indica que la imagen es decorativa y los screen readers la ignoran. Si la imagen tiene información, el `alt` debe describirla brevemente.
> 5. ¿Por qué no debés usar `outline: none` sin reemplazo?
>    - R: Porque elimina el indicador visual de foco, haciendo imposible saber qué elemento está seleccionado al navegar con Tab. Si querés customizarlo, usá `:focus-visible` con un estilo alternativo.
> 6. ¿Qué es la Accessibility Tree?
>    - R: Es un subárbol derivado del DOM que los screen readers consumen. Cada nodo tiene un role, un nombre accesible, y propiedades (checked, expanded, etc.). Se construye a partir de la semántica HTML + atributos ARIA.

→ Ver [Tópico 7.1: HTML Semántico](../concepts/07-frontend-core.md#7.1-html-semántico) — la semántica es el pilar #1 de accesibilidad.
→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.4-testing) — las herramientas de testing incluyen auditorías de accesibilidad automatizadas.

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

> [MDN — Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
> [MDN — Web Storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
> [MDN — IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)

*Técnicamente:* `fetch` usa la Fetch API del navegador, que devuelve una `Response` object con propiedades como `ok` (boolean, true si status 200-299), `status`, `statusText`, y métodos como `.json()`, `.text()`, `.blob()` — todos asíncronos. `localStorage` y `sessionStorage` implementan la Web Storage API con un límite de ~5MB por origin, sincrona y string-only (objetos requieren `JSON.stringify`). `IntersectionObserver` registra un callback que se dispara cuando un target cruza el threshold de visibilidad respecto al root (default: viewport). Es asíncrono y no bloquea el main thread — a diferencia de escuchar `scroll` events, que disparan reflow en cada frame. `navigator.geolocation` usa GPS/WiFi/cell triangulation del dispositivo y requiere permiso explícito del usuario (HTTPS obligatorio).

> **Check de comprensión**
> 1. ¿Por qué `fetch` no lanza error en un 404 y cómo lo manejás?
>    - R: `fetch` solo rechaza la Promise en fallos de red (sin conexión, DNS inválido). Un 404 es una respuesta HTTP válida. Se maneja chequeando `res.ok` o `res.status` después del await.
> 2. ¿Qué tipo de datos guarda `localStorage` y cómo persistís un objeto?
>    - R: Solo strings. Para objetos: `localStorage.setItem('user', JSON.stringify(user))` al guardar, y `JSON.parse(localStorage.getItem('user'))` al leer.
> 3. ¿Qué ventaja tiene `IntersectionObserver` sobre escuchar el evento `scroll`?
>    - R: `IntersectionObserver` es asíncrono y no bloquea el main thread. Escuchar `scroll` dispara un handler en cada frame del scroll, lo que causa reflow y jank. El observer solo notifica cuando el elemento cruza el threshold.
> 4. ¿Por qué `navigator.geolocation` requiere HTTPS?
>    - R: Porque la ubicación es información sensible. Los navegadores bloquean APIs de geolocalización en contextos no seguros (http:// o file://) para proteger la privacidad del usuario.
> 5. ¿Qué devuelve `res.json()` y por qué es asíncrono?
>    - R: Devuelve una Promise que resuelve al body parseado como JSON. Es asíncrono porque el body puede ser grande y el parsing no debe bloquear el main thread.
> 6. ¿Cuál es la diferencia entre `localStorage` y `sessionStorage`?
>    - R: `localStorage` persiste entre sesiones (sobrevive al cerrar el navegador). `sessionStorage` se borra al cerrar la pestaña. Ambos tienen el mismo API y límite de ~5MB.

→ Ver [Tópico 1: CORS](../concepts/01-fundamentos-web.md#1.8-cors) — fetch a otra origin requiere que el servidor envíe headers CORS.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.2-apis-rest) — el fetch del frontend consume las APIs REST del backend.

---