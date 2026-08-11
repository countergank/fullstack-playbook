# Setup — JavaScript en el Navegador + DevTools

> **Tópico**: 7 — Frontend Core
> **Objetivo**: que tu página haga un `fetch` a una API pública y muestre datos, usando JavaScript moderno (ES6+) y las DevTools para inspeccionar el tráfico.
> **Prerequisito**: `setup-html.md` (página creada) y `01-web/setup-browser-devtools.md` (consola y Network).

---

## ¿Por qué JavaScript moderno?

JS es el único lenguaje que el navegador entiende nativamente. Desde ES6+ (2015) dejó de ser un lenguaje tramposo: `const`/`let`, arrow functions, template literals, `map`/`filter`/`reduce` y `async/await` hacen el código legible y predecible. Acá usás `.js` plano, sin TypeScript (eso llega en la próxima guía) y sin frameworks (tópico 8). Si dominás esto, ningún framework te va a resultar magia negra.

---

## Checklist

### 1. Crear `app.js` y cargarlo como módulo

Al final del `<body>` de `index.html`, antes de cerrar la etiqueta:

```html
<script type="module" src="app.js"></script>
```

`type="module"` da scope aislado (nada se filtra al global) y permite `import`/`export`. La mayoría del tiempo lo combinás con `defer`, pero con el script al final del body no hace falta.

Crealo vacío por ahora:

```bash
touch app.js
```

- [ ] `app.js` existe y está cargado con `type="module"`

### 2. ES6+: declaración, arrow functions y template literals

```js
// app.js
const API_URL = 'https://jsonplaceholder.typicode.com/users';

const saludar = (nombre) => `Hola, ${nombre}!`;
console.log(saludar('Lean'));
```

- `const` para lo que no se reasigna (la regla: `const` por defecto, `let` solo si lo vas a reasignar, `var` nunca).
- Arrow functions (`=>`) con sintaxis corta y `this` léxico.
- Template literals (backticks) para interpolar.

### 3. `map`, `filter` y `reduce` — la tríada de arrays

```js
const numeros = [1, 2, 3, 4, 5];

const dobles = numeros.map((n) => n * 2);           // [2, 4, 6, 8, 10] — transforma
const pares = numeros.filter((n) => n % 2 === 0);    // [2, 4] — selecciona
const suma = numeros.reduce((acc, n) => acc + n, 0); // 15 — acumula
```

- [ ] Los tres métodos corren en la consola y devuelven lo esperado

### 4. `async/await` con `fetch` a una API pública

La API de ejemplo es **JSONPlaceholder** (`https://jsonplaceholder.typicode.com/users`), una API pública de práctica. Todo `await` vive DENTRO de una función `async`:

```js
// app.js
const API_URL = 'https://jsonplaceholder.typicode.com/users';

async function cargarUsuarios() {
  const res = await fetch(API_URL);

  if (!res.ok) {
    throw new Error(`HTTP ${res.status}: ${res.statusText}`);
  }

  const users = await res.json();
  const nombres = users.map((user) => user.name);
  const conEmail = users.filter((user) => user.email.includes('@'));

  console.log('Nombres:', nombres);
  console.table(users);
  return users;
}

cargarUsuarios().catch((err) => console.error('Error al cargar:', err));
```

Reglas de oro:
- `res.json()` devuelve OTRA Promise → necesita su propio `await`.
- `fetch` NO lanza error en 404/500 → siempre chequeá `res.ok`.
- Los errores se atrapan con `try/catch` o con `.catch()` en la llamada.

- [ ] La consola muestra la tabla de usuarios con `console.table`

### 5. Mostrar los datos en pantalla

La página debe mostrar lo que la API devolvió, no solo loguearlo:

```html
<section id="usuarios">
  <h2>Usuarios</h2>
  <ul id="lista-usuarios"></ul>
</section>
```

```js
function renderLista(users) {
  const lista = document.querySelector('#lista-usuarios');
  const items = users.map((user) => `<li>${user.name} — ${user.email}</li>`).join('');
  lista.innerHTML = items;
}
```

Y llamalo al final de `cargarUsuarios()`.

- [ ] La página muestra la lista de usuarios en pantalla

### 6. DevTools Console — logs y `console.table`

1. Abrí DevTools con **F12** → pestaña **Console**.
2. `console.log` imprime mensajes; `console.table` imprime arrays de objetos en formato tabla (mucho más legible).
3. Usá los filtros de nivel (Info / Warnings / Errors) arriba de la consola para filtrar ruido.

### 7. Breakpoints en Sources

1. Pestaña **Sources** → en el árbol de la izquierda, abrí `app.js`.
2. Clic en el número de línea (ej: la línea del `await fetch`) → se marca en azul.
3. Recargá la página: la ejecución se pausa ahí. Usá **step over / step into** y mirá el panel **Watch**.
4. Esto es el debugging de verdad: vés tus variables justo en el momento del request.

### 8. Throttling en Network — ver la latencia

1. Pestaña **Network** → dropdown arriba que dice "No throttling" → elegí **Slow 3G** o **Fast 3G**.
2. Recargá con **F5**: el request a `jsonplaceholder.typicode.com` va a tardar visiblemente.
3. Clic en ese request → pestaña **Timing**: vas a ver el waterfall (espera de red, TTFB, etc.) y la **Response**.

- [ ] Con throttling activo, el request a la API se ve lento en el waterfall

---

## Verificación

```text
1. La página muestra la lista de usuarios (nombre + email) en pantalla.
2. F12 → Console: console.table muestra la tabla completa de usuarios.
3. F12 → Network con throttling Fast 3G: el fetch a jsonplaceholder aparece con su waterfall.
```

**Si la app muestra los usuarios en pantalla y `console.table` muestra los datos → JavaScript + DevTools listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Cannot read properties of null" al hacer querySelector | El script corre antes de que exista el elemento: movelo al final del `body` o corré el query después de definir el HTML |
| CORS error en el fetch | APIs públicas como JSONPlaceholder lo permiten; tu backend debe enviar el header `Access-Control-Allow-Origin` (lo verás en el tópico 4) |
| No aparece el request en Network | DevTools muestra solo lo que pasa DESPUÉS de abrirse: recargá la página con F5 |
| `res.json()` da error | Falta el `await`: `res.json()` devuelve una Promise, hay que esperarla |
| `console.table` me muestra una fila rara | Pasale un ARRAY de objetos, no un objeto suelto |

---

## Recursos

- [MDN — `fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [MDN — Array methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- [JSONPlaceholder](https://jsonplaceholder.typicode.com/)
- [Chrome DevTools — Console](https://developer.chrome.com/docs/devtools/console/)

## Preguntas de repaso

- **P:** ¿Por qué `const` es preferible a `let` como regla general?
  **R:** `const` previene reasignaciones accidentales y comunica intención clara. Usás `let` solo cuando necesitás reasignar (contadores, flags). `var` tiene scope de función y hoisting tramposo — nunca se usa en código moderno.

- **P:** ¿Qué devuelve `res.json()` y por qué necesita su propio `await`?
  **R:** Devuelve una Promise que resuelve al body parseado como JSON. Necesita `await` porque el parsing es asíncrono — el body puede ser grande y el main thread no debe bloquearse.

- **P:** ¿Qué diferencia hay entre `map`, `filter` y `reduce`?
  **R:** `map` transforma cada elemento y devuelve un array del mismo largo. `filter` selecciona elementos que cumplen una condición y devuelve un array más corto. `reduce` acumula todos los elementos en un solo valor (número, objeto, string).

- **P:** ¿Por qué `fetch` no lanza error en un 404?
  **R:** `fetch` solo rechaza la Promise en fallos de red (sin conexión, DNS inválido). Un 404 es una respuesta HTTP válida del servidor. Siempre hay que chequear `res.ok` manualmente.

- **P:** ¿Para qué sirve `console.table` y cuándo lo usás?
  **R:** Imprime arrays de objetos en formato tabla legible en la consola. Es ideal para inspeccionar datos de APIs (como usuarios, productos) porque muestra cada propiedad como columna.