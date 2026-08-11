# 8. Frameworks & Herramientas Frontend

> Objetivo: elegir y usar el stack frontend real. El concept 07 te dio HTML/CSS/JS/TS puro; acá entra React como framework de UI, Vite como bundler/dev server, Vitest como testing, y las decisiones de estado, routing, estilos y renderizado (SPA vs Next.js).

---

## 8.1 React — El Framework de UI

*En criollo:* React resuelve el problema del concept 7.5: manipular el DOM a mano es tedioso, lento y lleno de bugs. Con React **declarás** qué se ve ("si el usuario está logueado, mostrá su avatar") y React se encarga de reconciliar el DOM real con lo que dijiste. Vos no tocás el DOM — React lo hace por vos.

*Técnicamente:* React mantiene un **Virtual DOM** (un árbol de objetos JS que replica la estructura del DOM real). Cuando el estado cambia, React ejecuta el algoritmo de **reconciliation** (diffing) entre el árbol anterior y el nuevo, calcula el mínimo conjunto de mutaciones necesarias, y las aplica en batch al DOM real. Desde React 18, el scheduler usa **Fiber** (linked-list de work units) para poder pausar, priorizar y reanudar renders — esto habilita `useTransition` y `useDeferredValue`. → [React — What is React?](https://react.dev/learn/describing-the-ui) | → [React — Render and Commit](https://react.dev/learn/render-and-commit)

→ Ver [Tópico 7: Frontend Core](../concepts/07-frontend-core.md#7.5-dom) para el problema que React resuelve. → Ver [Tópico 10: Testing](../concepts/10-testing.md#10.3-testing-de-componentes) para cómo testear componentes React.

### Componentes: la unidad base

Un componente es una función que recibe **props** (inputs) y devuelve **JSX** (el "HTML" de React). Componentes chicos y reutilizables es la idea central.

```tsx
interface UserCardProps {
  name: string;
  email: string;
}

function UserCard({ name, email }: UserCardProps) {
  return (
    <article className="user-card">
      <h2>{name}</h2>
      <p>{email}</p>
    </article>
  );
}
```

### Estado: cuando los datos cambian

- **`useState`** — estado local del componente. Cuando cambia, el componente se RE-RENDERIZA.
- **Rules of Hooks**: hooks solo al nivel superior del componente (nunca dentro de `if` o loops), y solo en componentes/functions que empiezan con mayúscula.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicks: {count}
    </button>
  );
}
```

### Los hooks que no pueden faltar

React tiene pocos hooks — pero ESOS pocos son la mitad de la biblioteca. `useState` ya lo viste; estos son el resto de los que usás TODOS los días:

| Hook | Qué hace | Caso típico |
|------|----------|-------------|
| **`useEffect`** | Ejecutar efectos secundarios (fetch, suscripciones, timers, sincronizar con APIs externas) | Cargar datos al montar, suscribirse a un store, limpiar timers |
| **`useRef`** | Referencia mutable que NO provoca re-render; persistir valor entre renders; apuntar a un nodo DOM | Foco de un input, guardar el valor anterior, medir un elemento |
| **`useMemo`** | Memorizar el RESULTADO de un cálculo costoso (solo recalcula si cambian las deps) | Ordenar/filtrar listas grandes, cálculos derivados |
| **`useCallback`** | Memorizar una FUNCIÓN (estable entre renders, no recrea el callback) | Pasar callbacks a hijos memoizados, evita re-renders |
| **`useReducer`** | Estado complejo con lógica tipo reducer (`state`, `action` → `newState`) | Formularios multi-campo, estado con transiciones |

#### `useEffect` — el rey de los efectos secundarios

```tsx
function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    let cancelled = false;                      // evita setState tras desmontar

    fetch(`/api/users/${userId}`)
      .then(res => { if (!res.ok) throw new Error('HTTP ' + res.status); return res.json(); })
      .then(data => { if (!cancelled) setUser(data); })
      .catch(err => console.error(err));

    return () => { cancelled = true; };         // cleanup: se corre al desmontar o al cambiar deps
  }, [userId]);                                 // deps: se re-ejecuta SOLO si userId cambia

  return <div>{user?.name ?? 'Cargando...'}</div>;
}
```

**Las 2 reglas de oro de `useEffect`:**
1. **`[]` deps = corre UNO vez al montar** (más el cleanup al desmontar). Sin deps explícitas = corre en CADA render (¡casi siempre es un bug!).
2. **Todo lo que setéeas adentro debe ser parte de un ciclo**: `setState` dentro de un `useEffect` con deps mal definidas = loop infinito. La cura casi siempre es definir las deps exactas.

#### `useRef` — memoria mutable sin re-render

```tsx
function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>Enfocar</button>
    </div>
  );
}
```

> `useRef` NO dispara re-render cuando cambia `.current`. Si querés que un cambio se REEJECUTE la UI → es `useState`, no `useRef`.

#### `useMemo` y `useCallback` — evitar trabajo y renders innecesarios

```tsx
// useMemo: el resultado filtrado NO se recalcula en cada render
const visibleItems = useMemo(
  () => items.filter(it => it.visible).sort(byDate),
  [items, sortOrder]
);

// useCallback: la MISMA identidad de función entre renders
const handleAdd = useCallback(
  (item: Item) => setItems(prev => [...prev, item]),
  []
);
```

> **Regla**: NO envuelvas todo en `useMemo`/`useCallback` — memorizar también tiene costo. Usalos cuando el cálculo es pesado o el componente hijo está memoizado (`React.memo`). Primero medí, después optimizá.

#### `useReducer` — estado complejo y predecible

```tsx
type State = { count: number };
type Action = { type: 'incr' } | { type: 'reset' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'incr':  return { count: state.count + 1 };
    case 'reset': return { count: 0 };
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0 });
// dispatch({ type: 'incr' })
```

- Toda transición de estado pasa por una función pura `(state, action) → newState` — testeable sin UI.
- Cuando el estado tiene varias piezas que cambian juntas (formulario, carrito), `useReducer` gana a `useState`.

### Hooks avanzados (existen, no son el default)

`useLayoutEffect` (efecto síncrono antes del paint), `useTransition` (actualizaciones no urgentes), `useDeferredValue`, `useId` (IDs accesibles), `useImperativeHandle` (API expuesta por `forwardRef`). Los ves cuando los necesites — los 6 esenciales (`useState`, `useEffect`, `useRef`, `useMemo`, `useCallback`, `useReducer`) son el 95% del uso real.

### La regla de oro: props fluyen hacia abajo, eventos hacia arriba

- Los datos viajan de padre a hijo por **props** (unidireccional).
- Para "subir" un cambio, el hijo recibe un **callback** como prop y lo llama.

```tsx
function TodoItem({ todo, onToggle }: { todo: Todo; onToggle: (id: number) => void }) {
  return (
    <li>
      <input type="checkbox" checked={todo.done} onChange={() => onToggle(todo.id)} />
      {todo.title}
    </li>
  );
}
```

> **Check de comprensión**
> 1. ¿Qué problema resuelve React respecto a manipular el DOM a mano?
>    - R: Evita la manipulación imperativa del DOM; declarás qué querés ver y React reconcilia el Virtual DOM con el real aplicando solo los cambios necesarios.
> 2. ¿Qué son las Rules of Hooks y por qué existen?
>    - R: Hooks solo al nivel superior del componente y solo en componentes/custom hooks. Existen porque React depende del orden de llamada para asociar cada hook con su estado interno (Fiber).
> 3. ¿Cuándo usás `useEffect` vs cuándo NO?
>    - R: Usás `useEffect` para efectos secundarios (fetch, timers, suscripciones, sincronizar con APIs externas). NO lo usás para transformar datos derivados de props/estado (eso es `useMemo` o cálculo directo).
> 4. ¿Qué diferencia hay entre `useRef` y `useState` en términos de re-render?
>    - R: `useRef` es mutable pero cambiar `.current` NO dispara re-render. `useState` sí dispara re-render cuando cambia. Usás `useRef` para valores que no necesitan actualizar la UI.
> 5. ¿Por qué `useMemo` y `useCallback` no deben usarse en todo?
>    - R: Memorizar también tiene costo (allocación, comparación de deps). Solo conviene cuando el cálculo es pesado o el hijo está memoizado con `React.memo`. Primero medí, después optimizá.
> 6. ¿Qué hace `useReducer` y cuándo gana sobre `useState`?
>    - R: Maneja estado complejo con una función pura `(state, action) → newState`. Gana cuando el estado tiene varias piezas que cambian juntas o transiciones predecibles (formularios, carritos).
> 7. ¿Qué significa que las props fluyen unidireccionalmente?
>    - R: Los datos van de padre a hijo; para comunicar cambios hacia arriba el hijo recibe un callback como prop y lo invoca. No hay "binding bidireccional" como en otros frameworks.

---

## 8.2 Manejo de Estado

*En criollo:* A medida que la app crece, el estado deja de ser local de un componente. "¿Quién está logueado?", "¿qué hay en el carrito?" — datos que MUCHOS componentes necesitan. Esos no van en un `useState` por componente: van afuera.

*Técnicamente:* React Context usa el **Context API** interno: al llamar `useContext(ThemeContext)`, el componente se suscribe al Provider más cercano en el árbol. Cuando el `value` del Provider cambia, React marca TODOS los suscriptores como dirty y los re-renderiza en el próximo ciclo. Esto es O(n) respecto a los consumidores — por eso Context es ideal para datos que cambian poco (tema, auth) y no para datos frecuentes (input text, scroll position). Zustand, en cambio, usa un store externo con **selectores finos**: cada componente se suscribe solo al slice que le interesa, evitando re-renders innecesarios. → [React — Context](https://react.dev/learn/passing-data-deeply-with-context) | → [Zustand — Introduction](https://zustand.docs.pmnd.rs/getting-started/introduction)

→ Ver [Tópico 7: Frontend Core](../concepts/07-frontend-core.md#7.4-typescript) para el tipado de props y estado. → Ver [Tópico 8.1: React Hooks](../concepts/08-frameworks-herramientas-frontend.md#8.1-react--el-framework-de-ui) para `useState` y `useReducer` como base del estado local.

| Herramienta | Estado resuelve | Cuándo usarla |
|-------------|----------------|---------------|
| **`useState`** | Estado local de UN componente (open/close, contador) | Casi siempre, es el default |
| **Props drilling** | Pasar props por varios niveles | Evitar — se vuelve ilegible rápido |
| **React Context** | Datos globales de LECTURA (tema, usuario logueado) | Cuando muchos componentes leen lo mismo |
| **Zustand / Redux Toolkit** | Estado global con lógica y acciones (carrito, sesión compleja) | Cuando Context no alcanza: mucha escritura, updates frecuentes, devtools |

### Context — para datos globales de lectura

```tsx
const ThemeContext = createContext<'light' | 'dark'>('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Header />  {/* cualquier nested component puede leerlo */}
    </ThemeContext.Provider>
  );
}

function Header() {
  const theme = useContext(ThemeContext); // "dark"
  return <header className={`header-${theme}`} />;
}
```

> **Regla de oro**: Context provoca re-render de TODOS los consumidores cuando cambia. No metas cosas que cambian constantemente (el texto de un input) en Context — para eso está el estado local.

> **Check de comprensión**
> 1. ¿Qué problema resuelve el manejo de estado global?
>    - R: Evita el prop drilling (pasar datos por muchos niveles) y centraliza datos que múltiples componentes necesitan (usuario logueado, tema, carrito).
> 2. ¿Cuándo usás React Context y cuándo Zustand?
>    - R: Context para datos globales de lectura que cambian poco (tema, idioma). Zustand para estado con lógica, acciones y updates frecuentes (carrito, sesión compleja).
> 3. ¿Por qué no conviene meter datos que cambian constantemente en Context?
>    - R: Porque cada cambio de value en el Provider re-renderiza TODOS los consumidores, lo que degrada performance.
> 4. ¿Qué ventaja tiene Zustand sobre Context en términos de re-render?
>    - R: Zustand usa selectores finos: cada componente se suscribe solo al slice del estado que necesita, evitando re-renders innecesarios.
> 5. ¿Qué es prop drilling y por qué es un problema?
>    - R: Pasar props por muchos niveles de componentes intermedios que no las usan. Hace el código ilegible y difícil de mantener.

---

## 8.3 Routing

*En criollo:* Una SPA tiene UNA página HTML. "Navegar" es cambiar qué componente se muestra según la URL. El router es quien observa la URL, decide qué renderizar, y reescribe el historial del navegador (para que back/forward funcionen).

*Técnicamente:* React Router usa la **History API** del navegador (`pushState`, `replaceState`, evento `popstate`). `BrowserRouter` escucha cambios de URL y renderiza el `<Route>` que matchea. Los `<Link>` interceptan el evento `click` del `<a>`, llaman a `event.preventDefault()`, y usan `history.pushState` para cambiar la URL sin recargar. El parámetro `:id` se parsea del path y se expone via `useParams()`. → [React Router — Getting Started](https://reactrouter.com/start/framework/installation) | → [MDN — History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)

→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.4-navegadores) para cómo el navegador maneja URLs y el historial. → Ver [Tópico 8.6: Next.js](../concepts/08-frameworks-herramientas-frontend.md#8.6-nextjs--el-framework-full-stack-de-react) para routing por archivos vs routing declarativo.

### React Router — el estándar

```tsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Inicio</Link>
        <Link to="/users">Usuarios</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users" element={<UserList />} />
        <Route path="/users/:id" element={<UserDetail />} />  {/* parámetro dinámico */}
        <Route path="*" element={<NotFound />} />             {/* 404 */}
      </Routes>
    </BrowserRouter>
  );
}
```

### Parámetros y navegación programática

```tsx
import { useParams, useNavigate } from 'react-router-dom';

function UserDetail() {
  const { id } = useParams();          // lee :id de la URL
  const navigate = useNavigate();      // navegación después de una acción

  return (
    <button onClick={() => navigate('/users')}>Volver</button>
  );
}
```

> **Detalle**: `<Link>` para links, `useNavigate` para redirecciones después de lógica (login, submit). Nunca uses `<a href>` crudo dentro de una SPA — recarga la página entera y mata el estado.

> **Check de comprensión**
> 1. ¿Cómo "navega" una SPA si solo tiene una página HTML?
>    - R: El router observa la URL, renderiza el componente correspondiente, y reescribe el historial con la History API para que back/forward funcionen.
> 2. ¿Qué diferencia hay entre `<Link>` y `<a href>` en una SPA?
>    - R: `<Link>` intercepta el clic y cambia la URL sin recargar (usa `pushState`). `<a href>` recarga toda la página y pierde el estado React.
> 3. ¿Cuándo usás `<Link>` vs `useNavigate()`?
>    - R: `<Link>` para navegación declarativa (links en la UI). `useNavigate()` para redirecciones programáticas después de lógica (login exitoso, submit de formulario).
> 4. ¿Cómo leés un parámetro dinámico como `:id` de la URL?
>    - R: Con `useParams()` que devuelve un objeto con las claves declaradas en el path (ej: `{ id: "3" }` para `/users/3`).
> 5. ¿Qué hace la ruta `path="*"` en React Router?
>    - R: Es un catch-all: matchea cualquier URL que no haya matcheado ninguna ruta anterior. Se usa para la página 404.
> 6. ¿Por qué el orden de las `<Route>` importa?
>    - R: `Routes` renderiza la PRIMERA que matchea. Si ponés `*` antes que otras rutas, siempre cae en el 404.

---

## 8.4 Vite — El Bundler y Dev Server Moderno

*En criollo:* Antes (Webpack), cada cambio en un archivo re-empaquetaba todo el proyecto: segundos (a veces minutos) de espera por cada save. Vite rompió eso: en dev NO bundlea — sirve los ES modules nativos del navegador como están. El servidor arranca al instante y el Hot Module Replacement (HMR) actualiza SOLO el módulo que cambió, sin recargar la página y sin perder el estado.

*Técnicamente:* Vite usa **ES modules nativos** del navegador en desarrollo: cada `import` se resuelve on-demand via HTTP, sin bundling previo. El dev server intercepta requests, transforma TS/JSX al vuelo con **esbuild** (escrito en Go, ~10-100x más rápido que babel/webpack), y sirve el resultado. En producción, usa **Rolldown** (Rust) para generar bundles optimizados con code-splitting y tree-shaking. HMR funciona via WebSocket: el server notifica al browser qué módulo cambió, y el browser lo re-importa sin reload. → [Vite — Why Vite](https://vite.dev/guide/why.html) | → [Vite — Features](https://vite.dev/guide/features.html)

→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.7-terminal) para el entorno donde corren los comandos de build. → Ver [Tópico 9: DevOps](../concepts/09-devops-deployment.md#9.1-docker) para cómo se deploya el `dist/` que genera Vite.

### Por qué Vite gana

| | Webpack (vieja escuela) | Vite |
|---|-------------------------|------|
| Dev server startup | Minutos en proyectos grandes | Instante (on-demand) |
| HMR | Re-bundlea el graph completo | Actualiza solo el módulo cambiado |
| Build | Su propio bundler | **Rolldown** (Rust, muy rápido) |
| Config | Archivos enormes | `vite.config.ts` mínimo |
| Template | Config a mano | `npm create vite` con presets |

### Crear un proyecto

```bash
# npm 7+: los "--" separan args para create-vite
npm create vite@latest mi-app -- --template react-ts
cd mi-app
npm install
npm run dev    # dev server con HMR en http://localhost:5173
npm run build  # build de producción a dist/ (Rolldown)
npm run preview # servir el build localmente
```

### Qué genera y por qué importa

```
mi-app/
├── index.html          # ÚNICA página — la SPA vive dentro de <div id="root">
├── src/
│   ├── main.tsx        # entry: ReactDOM.createRoot(...).render(<App />)
│   ├── App.tsx         # componente raíz
│   └── vite-env.d.ts   # tipos de Vite
└── vite.config.ts      # configuración (alias @, plugins, server.port)
```

> Ojo: `npm run build` genera `dist/` — eso es lo que deployment sirve (tópico 9). Vite en dev está pensado para el developer; el build para el usuario final.

> **Check de comprensión**
> 1. ¿Por qué Vite es tan rápido en dev comparado con Webpack?
>    - R: Vite sirve ES modules nativos on-demand sin bundlear; Webpack re-bundlea todo el graph en cada cambio. esbuild (Go) transforma al vuelo ~10-100x más rápido.
> 2. ¿Qué es HMR y por qué es mejor que recargar la página?
>    - R: Hot Module Replacement actualiza solo el módulo cambiado via WebSocket, sin recargar. Mantiene el estado de la app (formularios, navegación) intacto.
> 3. ¿Qué usa Vite para el build de producción y por qué?
>    - R: Rolldown (Rust) para bundles optimizados con tree-shaking y code-splitting. En producción necesitás un solo bundle o pocos chunks, no cientos de ES modules.
> 4. ¿Qué genera `npm create vite` y por qué importa la estructura?
>    - R: Un proyecto con `index.html` (única página), `src/main.tsx` (entry), `src/App.tsx` (componente raíz), y `vite.config.ts` (config). Es la base que usan todas las guías siguientes.
> 5. ¿Qué diferencia hay entre `npm run dev` y `npm run build`?
>    - R: `dev` levanta el servidor con HMR para desarrollo. `build` genera archivos optimizados en `dist/` para producción (servidos por un servidor estático).
> 6. ¿Para qué sirve `npm run preview`?
>    - R: Sirve localmente el build de producción (`dist/`) para verificar que funciona antes de deployar. No es el dev server — es el build real.

---

## 8.5 Vitest — Testing de Componentes

*En criollo:* Vitest es el framework de testing "nativo" de Vite: comparte la misma config, el mismo transform y la misma velocidad. Es la evolución natural de Jest — la API es la misma (`describe`, `test`, `expect`), pero corre sobre la infraestructura de Vite. Por eso el tópico 10 (Testing) lo tiene como opción principal para frontend.

*Técnicamente:* Vitest reutiliza el **pipeline de transformación de Vite**: el mismo plugin de React/TS que transforma tu app transforma tus tests. No necesita configuración extra para JSX o TS. En modo watch, usa **invalidation graphs** para re-correr solo los tests afectados por un cambio. `jsdom` simula la API del DOM en Node (sin navegador real); el browser mode corre tests en un navegador real via Playwright. → [Vitest — Getting Started](https://vitest.dev/guide/) | → [Testing Library — React](https://testing-library.com/docs/react-testing-library/intro)

→ Ver [Tópico 10: Testing](../concepts/10-testing.md#10.1-unit-tests) para los fundamentos de testing que Vitest implementa. → Ver [Tópico 8.4: Vite](../concepts/08-frameworks-herramientas-frontend.md#8.4-vite--el-bundler-y-dev-server-moderno) para la infraestructura que Vitest comparte.

### Instalar y el primer test

```bash
npm install -D vitest @testing-library/react jsdom
```

```ts
// sum.test.ts
import { expect, test } from 'vitest';
import { sum } from './sum';

test('suma 1 + 2 = 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

### Modos

```bash
npx vitest          # watch mode — corre tests al guardar (default)
npx vitest run      # una sola pasada — para CI
```

### Testear un componente React

Con Testing Library: renderizás el componente y oprimís como un usuario real (por rol y texto, NO por clase CSS).

```tsx
import { render, screen } from '@testing-library/react';
import { expect, test } from 'vitest';
import Counter from './Counter';

test('el botón incrementa el contador', async () => {
  render(<Counter />);
  expect(screen.getByText('Clicks: 0')).toBeTruthy();

  await screen.getByRole('button', { name: 'Increment' }).click();

  expect(screen.getByText('Clicks: 1')).toBeTruthy();
});
```

### Ambiente jsdom vs browser

- **jsdom**: simula el DOM en Node — rápido, para la mayoría de tests de componentes.
- **Browser mode** (`vitest` con playwright): el test corre en un navegador REAL — cuando necesitás layout/estilos reales, o interacción compleja.

> **Nota**: para correr componentes en jsdom, definís `environment: 'jsdom'` en `vitest.config.ts` (o docblock `@jest-environment jsdom` por archivo). Sin eso, un componente con DOM falla.

> **Check de comprensión**
> 1. ¿Por qué Vitest se siente "nativo" con Vite?
>    - R: Comparte el mismo pipeline de transformación, la misma config y los mismos plugins. No necesita configuración extra para JSX o TypeScript.
> 2. ¿Qué diferencia hay entre `vitest` y `vitest run`?
>    - R: `vitest` corre en watch mode (escucha cambios y re-corre). `vitest run` hace una sola pasada y termina — ideal para CI.
> 3. ¿Para qué sirve jsdom y cuándo lo necesitás?
>    - R: Simula la API del DOM en Node. Lo necesitás cuando testeás componentes React que interactúan con el DOM (render, clicks, queries).
> 4. ¿Qué principio sigue Testing Library al testear componentes?
>    - R: Testear como un usuario real: buscar por rol y texto visible, no por clases CSS internas o implementación.
> 5. ¿Qué hace `vi.fn()` en un test?
>    - R: Crea una función espía (mock) que registra cuántas veces fue llamada y con qué argumentos. Sirve para verificar que callbacks se disparan correctamente.
> 6. ¿Cuándo usarías el browser mode de Vitest en vez de jsdom?
>    - R: Cuando necesitás un navegador real: layout/estilos reales, interacción compleja, o APIs que jsdom no soporta (Canvas, WebGL).

---

## 8.6 Next.js — El Framework Full-Stack de React

*En criollo:* React solo es una librería de UI que corre en el cliente. La página llega vacía y React llena el DOM en el navegador (CSR). Next.js agrega lo que falta para apps serias: renderizado en el servidor, routing por archivos, API routes, SEO. Es el framework más usado de React — el 90% de los proyectos profesionales nuevos usan Next.

*Técnicamente:* Next.js implementa **Server Components** (RSC): componentes que corren en el servidor y envían un serializado (no HTML, no JSON) al cliente. El cliente hidrata solo los Client Components (`'use client'`). El App Router usa **file-based routing**: cada `page.tsx` en `app/` es una ruta. Next soporta CSR, SSR (render por request), SSG (render en build), e ISR (revalidate periódico). El streaming SSR envía HTML en chunks para que el usuario vea contenido antes de que todo esté listo. → [Next.js — Getting Started](https://nextjs.org/docs/app/getting-started) | → [React — Server Components](https://react.dev/reference/rsc/server-components)

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.2-apis-rest) para entender por qué los Server Components pueden leer la DB directo sin API intermedia. → Ver [Tópico 8.3: Routing](../concepts/08-frameworks-herramientas-frontend.md#8.3-routing) para la diferencia entre routing declarativo (React Router) y por archivos (Next.js).

### CSR vs SSR vs SSG — el espectro de renderizado

| | CSR (React solo) | SSR (Next dinámico) | SSG (Next estático) |
|---|---|---|---|
| Quién renderiza | Navegador | Servidor en cada request | Servidor en build |
| Tiempo hasta ver contenido | Lento (toca red → JS → render) | Rápido (HTML servido directo) | Instantáneo (archivos estáticos) |
| SEO | Malo (contenido llega tarde) | Bueno | Ideal |
| Datos dinámicos | Cualquiera (fetch en cliente) | En cada request | Solo en build (revalidate opcional) |
| Caso | Dashboards, apps internas | Apps con datos por usuario | Blogs, docs, landing |

### El modelo de archivos (App Router)

En Next 13+, el routing es por archivos en `app/`:

```
app/
├── layout.tsx        # layout compartido (navbar, footer)
├── page.tsx          # ruta "/"
└── users/
    ├── page.tsx      # ruta "/users"
    └── [id]/
        └── page.tsx  # ruta "/users/:id" (dinámico)
```

```tsx
// app/users/page.tsx — componente SERVER, puede ser async y leer la DB directo
export default async function UsersPage() {
  const users = await prisma.user.findMany();  // sin API route intermedia
  return <UserList users={users} />;
}
```

### Client vs Server Components — la decisión que define todo

- **Server Components** (default): corren en el servidor, pueden ser `async`, leen DB/API directo. NO usan hooks ni eventos.
- **Client Components** (`'use client'`): corren en el navegador — estado, `useEffect`, `onClick`.

> **Regla práctica**: default server; marcá `'use client'` SOLO cuando necesitás interactividad (estado, eventos).

> **Check de comprensión**
> 1. ¿Qué agrega Next.js que React solo no tiene?
>    - R: Renderizado en servidor (SSR/SSG), routing por archivos, API routes, SEO optimizado, y Server Components. React solo es CSR (cliente).
> 2. ¿Qué diferencia hay entre CSR, SSR y SSG?
>    - R: CSR renderiza en el navegador (lento, mal SEO). SSR renderiza en el servidor en cada request (rápido, buen SEO). SSG genera HTML estático en build (instantáneo, ideal para contenido fijo).
> 3. ¿Qué es un Server Component y qué NO puede hacer?
>    - R: Un componente que corre en el servidor, puede ser async y leer datos directo. NO puede usar hooks (`useState`, `useEffect`) ni eventos (`onClick`).
> 4. ¿Cuándo marcás un componente con `'use client'`?
>    - R: Solo cuando necesitás interactividad: estado local, efectos, o eventos de usuario. El default es server component.
> 5. ¿Cómo funciona el routing en Next.js App Router?
>    - R: Por archivos: cada `page.tsx` en una carpeta de `app/` es una URL. `layout.tsx` es compartido. `[id]` es un segmento dinámico.
> 6. ¿Qué ventaja tiene que un Server Component pueda leer la DB directo?
>    - R: Elimina la necesidad de una API route intermedia: el servidor hace el query y envía el HTML ya con los datos. Menos latencia, menos código.

---

## 8.7 Sistemas de Estilos

*En criollo:* En el concept 7.2 viste CSS puro. En una app React, el CSS necesita resolvers: cómo escopar los estilos por componente, cómo evitar conflictos de nombres, cómo no repetir. Acá están las tres estrategias que dominan el mercado — cada una es una filosofía distinta.

*Técnicamente:* **CSS Modules** transforma cada clase en un identificador único (hash) en build time: `.card` → `UserCard_card__x7k2m`. El CSS se scropea por archivo. **Tailwind** usa un parser JIT (Just-In-Time) que escanea tu código y genera solo las clases utilitarias que usás — el CSS final es mínimo porque purga las no usadas. **styled-components** genera `<style>` tags dinámicos en runtime: cada componente inyecta sus reglas con clases únicas, y las props se interpolan en template literals. → [CSS Modules — spec](https://github.com/css-modules/css-modules) | → [Tailwind — Utility-First](https://tailwindcss.com/docs/utility-first) | → [styled-components — Basics](https://styled-components.com/docs/basics)

→ Ver [Tópico 7: Frontend Core](../concepts/07-frontend-core.md#7.2-css-moderno) para los fundamentos de CSS que estos sistemas extienden. → Ver [Tópico 8.4: Vite](../concepts/08-frameworks-herramientas-frontend.md#8.4-vite--el-bundler-y-dev-server-moderno) para cómo Vite procesa CSS Modules nativamente.

### CSS Modules — el clásico silencioso

Estilos por archivo, escopados automáticamente. El nombre de clase se hashea para no chocar con otros componentes.

```css
/* UserCard.module.css */
.card { border: 1px solid #ddd; border-radius: 8px; padding: 1rem; }
```

```tsx
import styles from './UserCard.module.css';

function UserCard() {
  return <article className={styles.card}>...</article>;  // .card se vuelve único
}
```

### Tailwind — utility-first

Nada de archivos CSS separados: clases utilitarias directo en el JSX. Rápido para iterar, consistente por convención (tokens), y el build purga las clases que no usás (CSS final mínimo).

```tsx
function Badge({ label }: { label: string }) {
  return (
    <span className="inline-flex items-center rounded-full bg-blue-100 px-3 py-1 text-sm font-medium text-blue-700">
      {label}
    </span>
  );
}
```

### styled-components — CSS-in-JS

Los estilos son componentes (template literals con CSS). Tipado perfecto, estilos co-localizados con la lógica, temas con props dinámicas.

```tsx
import styled from 'styled-components';

const Button = styled.button<{ $variant: 'primary' | 'ghost' }>`
  padding: 0.5rem 1rem;
  border-radius: 6px;
  background: ${({ $variant }) => ($variant === 'primary' ? '#3b82f6' : 'transparent')};
`;

function App() {
  return <Button $variant="primary">Guardar</Button>;
}
```

### Cómo elegir

| | CSS Modules | Tailwind | styled-components |
|---|---|---|---|
| Escopeo | ✅ automático | Convención de clases | ✅ por componente |
| Velocidad de dev | Media (escribís CSS) | Alta (utility inline) | Alta (CSS en JSX) |
| Curva de aprendizaje | Baja (CSS puro) | Media (aprender clases) | Media (sintaxis CSS-in-JS) |
| Bundle final | Por uso | Purge automático | Runtime JS extra |
| Team estandarizado | Depende de cada uno | ✅ tokens únicos | Convención del equipo |

> **Regla de oro**: cualquiera de las tres es válida; lo NO negociable es consistencia dentro del equipo. Elegí UNA y mantenela.

> **Check de comprensión**
> 1. ¿Qué problema resuelven los sistemas de estilos en React?
>    - R: El CSS puro tiene nombres de clase globales que se pisan. Los sistemas de estilos resuelven el escopeo: CSS Modules con hash, Tailwind con convención utility-first, styled-components con clases únicas por componente.
> 2. ¿Cómo funciona CSS Modules para evitar conflictos de nombres?
>    - R: Hashea cada clase en build time: `.card` se convierte en algo como `Button_card__x7k2m`. Cada archivo tiene su propio namespace.
> 3. ¿Qué es Tailwind y por qué el bundle final es mínimo?
>    - R: Es un framework utility-first: usás clases predefinidas directo en el JSX. El build purga las clases que no usás, dejando solo las necesarias.
> 4. ¿Qué ventaja y desventaja tiene styled-components?
>    - R: Ventaja: estilos co-localizados con la lógica, tipado perfecto, temas con props dinámicas. Desventaja: agrega runtime JS extra al bundle.
> 5. ¿Cómo elegís entre las tres estrategias?
>    - R: Depende del equipo: CSS Modules si ya saben CSS puro, Tailwind para velocidad de dev y consistencia, styled-components para co-localización y temas dinámicos. Lo clave es elegir UNA y ser consistente.
> 6. ¿Qué es el "runtime JS extra" de styled-components?
>    - R: styled-components genera estilos en runtime (no en build time): el bundle incluye la librería que crea `<style>` tags dinámicos. Esto aumenta el JS que el navegador descarga.