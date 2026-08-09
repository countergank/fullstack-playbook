# 8. Frameworks & Herramientas Frontend

> Objetivo: elegir y usar el stack frontend real. El concept 07 te dio HTML/CSS/JS/TS puro; acá entra React como framework de UI, Vite como bundler/dev server, Vitest como testing, y las decisiones de estado, routing, estilos y renderizado (SPA vs Next.js).

---

## 8.1 React — El Framework de UI

*En criollo:* React resuelve el problema del concept 7.5: manipular el DOM a mano es tedioso, lento y lleno de bugs. Con React **declarás** qué se ve ("si el usuario está logueado, mostrá su avatar") y React se encarga de reconciliar el DOM real con lo que dijiste. Vos no tocás el DOM — React lo hace por vos.

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

---

## 8.2 Manejo de Estado

*En criollo:* A medida que la app crece, el estado deja de ser local de un componente. "¿Quién está logueado?", "¿qué hay en el carrito?" — datos que MUCHOS componentes necesitan. Esos no van en un `useState` por componente: van afuera.

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

---

## 8.3 Routing

*En criollo:* Una SPA tiene UNA página HTML. "Navegar" es cambiar qué componente se muestra según la URL. El router es quien observa la URL, decide qué renderizar, y reescribe el historial del navegador (para que back/forward funcionen).

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

---

## 8.4 Vite — El Bundler y Dev Server Moderno

*En criollo:* Antes (Webpack), cada cambio en un archivo re-empaquetaba todo el proyecto: segundos (a veces minutos) de espera por cada save. Vite rompió eso: en dev NO bundlea — sirve los ES modules nativos del navegador como están. El servidor arranca al instante y el Hot Module Replacement (HMR) actualiza SOLO el módulo que cambió, sin recargar la página y sin perder el estado.

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

---

## 8.5 Vitest — Testing de Componentes

*En criollo:* Vitest es el framework de testing "nativo" de Vite: comparte la misma config, el mismo transform y la misma velocidad. Es la evolución natural de Jest — la API es la misma (`describe`, `test`, `expect`), pero corre sobre la infraestructura de Vite. Por eso el tópico 10 (Testing) lo tiene como opción principal para frontend.

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

---

## 8.6 Next.js — El Framework Full-Stack de React

*En criollo:* React solo es una librería de UI que corre en el cliente. La página llega vacía y React llena el DOM en el navegador (CSR). Next.js agrega lo que falta para apps serias: renderizado en el servidor, routing por archivos, API routes, SEO. Es el framework más usado de React — el 90% de los proyectos profesionales nuevos usan Next.

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

---

## 8.7 Sistemas de Estilos

*En criollo:* En el concept 7.2 viste CSS puro. En una app React, el CSS necesita resolvers: cómo escopar los estilos por componente, cómo evitar conflictos de nombres, cómo no repetir. Acá están las tres estrategias que dominan el mercado — cada una es una filosofía distinta.

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

---

> **Check de comprensión**:
> 1. ¿Por qué Vite es tan rápido en dev mientras Webpack no? ¿Qué pasa exactamente cuando editás un archivo?
> 2. ¿Cuál es la diferencia entre `useState` y React Context? ¿Cuándo usarías Zustand además?
> 3. Una SPA tiene una sola página HTML — ¿cómo "navega" entonces? ¿Qué hace el router?
> 4. ¿En qué se diferencia un Server Component de un Client Component en Next.js? ¿Cómo decidís cuál usar?
> 5. ¿Por qué Vitest se siente nativo con Vite? ¿Qué hace `vitest run` distinto de `vitest`?
> 6. ¿Qué problema resuelven los tres sistemas de estilos? ¿Cuál elegirías para un proyecto nuevo y por qué?
> 7. ¿Qué pasa si ponés `useEffect` sin array de deps? ¿Y con `[]` pero usando un valor que cambia? ¿En qué se diferencia `useRef` de `useState`?
> 8. Tenés una lista de 10.000 items que se filtra en cada render — ¿qué hook usás y por qué?