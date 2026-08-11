# Setup — Routing con React Router

> **Tópico**: 8 — Frameworks & Herramientas Frontend
> **Objetivo**: darle navegación a la SPA con React Router: rutas, parámetros dinámicos, 404, navegación programática y cómo se comporta el historial del navegador.
> **Prerequisito**: proyecto Vite + React (`setup-vite-react.md`) y el manejo de estado con hooks (`setup-react-hooks.md`).

---

## ¿Por qué un router?

Tu SPA tiene UNA página HTML (el `index.html` de Vite). "Navegar" es cambiar qué componente se muestra según la URL y reescribir el historial del navegador para que back/forward funcionen (concept 8.3). React Router es el estándar: observa la URL, decide qué renderizar y actualiza el historial. Sin él, tendrías que reimplementar eso a mano con `history.pushState` y un `popstate`.

---

## Checklist

### 1. Instalar React Router

```bash
npm install react-router-dom
```

> **Ojo con las versiones**: en la v7, `react-router-dom` quedó como re-export de `react-router`, así que la API clásica (`BrowserRouter`, `Routes`, `Route`, `Link`) sigue funcionando igual que en la v6. Para este curso usamos la forma declarativa clásica.

- [ ] `react-router-dom` instalado

### 2. Configurar las rutas en `App.tsx`

```tsx
// src/App.tsx
import { BrowserRouter, Link, Route, Routes } from 'react-router-dom';
import Home from './pages/Home';
import UserList from './pages/UserList';
import UserDetail from './pages/UserDetail';
import NotFound from './pages/NotFound';

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
        <Route path="/users/:id" element={<UserDetail />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

- `BrowserRouter` → sincroniza la URL con la app (usa la History API). Sin él, nada navega.
- `Routes` + `Route` → mapeo declarativo de URL a componente.
- `:id` → segmento dinámico: `/users/3` matchea esta ruta y expone `id = "3"`.
- `*` → catch-all: cualquier URL que no haya matcheado cae acá (nuestro 404).

- [ ] Las 4 rutas (`/`, `/users`, `/users/:id`, `*`) están configuradas

### 3. `Link` para la navegación entre vistas

`Link` es la forma declarativa de navegar. Renderiza un `<a>` pero intercepta el clic para NO recargar la página: solo cambia la URL y el componente que se muestra.

```tsx
// src/pages/UserList.tsx
import { useEffect, useState } from 'react';
import { Link } from 'react-router-dom';

interface User {
  id: number;
  name: string;
  email: string;
}

export default function UserList() {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/users')
      .then((res) => res.json())
      .then(setUsers);
  }, []);

  return (
    <section>
      <h2>Usuarios</h2>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            <Link to={`/users/${user.id}`}>{user.name}</Link>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

- Usá `to` con template literal para rutas dinámicas: `/users/${user.id}`.
- [ ] La lista de usuarios navega a `/users/<id>` con `Link`

### 4. `useParams` — leer el parámetro de la URL

```tsx
// src/pages/UserDetail.tsx
import { useEffect, useState } from 'react';
import { useNavigate, useParams } from 'react-router-dom';

interface User {
  id: number;
  name: string;
  email: string;
}

export default function UserDetail() {
  const { id } = useParams();        // lee el :id de la URL actual
  const navigate = useNavigate();    // navegación después de una acción
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetch(`https://jsonplaceholder.typicode.com/users/${id}`)
      .then((res) => res.json())
      .then(setUser);
  }, [id]);

  if (!user) return <p>Cargando usuario...</p>;

  return (
    <section>
      <h2>{user.name}</h2>
      <p>{user.email}</p>

      <button onClick={() => navigate('/users')}>Volver a usuarios</button>
    </section>
  );
}
```

- `useParams()` devuelve un objeto con las claves declaradas en el path (`:id` → `{ id: '3' }`).
- Como la URL cambia, el efecto usa `[id]` como deps: si navegás de `/users/3` a `/users/9`, se re-fetchea.
- [ ] `/users/3` muestra el usuario 3 y el efecto re-corre al cambiar de id

### 5. `useNavigate` — navegación programática

Lo usás cuando la navegación NO es un clic en un link sino el RESULTADO de una acción (login, submit, volver tras cargar):

```tsx
function LoginForm() {
  const navigate = useNavigate();

  function handleLogin() {
    // ...lógica de login...
    navigate('/dashboard');        // después de la acción, redirijo
  }

  return <button onClick={handleLogin}>Iniciar sesión</button>;
}
```

Regla (concept 8.3):

- `<Link>` → para links normales de navegación.
- `useNavigate()` → para redirecciones programáticas después de lógica.
- **NUNCA `<a href>`** crudo adentro de una SPA: recarga la página entera y mata el estado React (perdés todo lo que no esté persistido).

- [ ] Entendés cuándo usar `<Link>` vs `useNavigate()` vs cuándo es un bug usar `<a href>`

### 6. Página de 404 y Home

```tsx
// src/pages/Home.tsx
export default function Home() {
  return (
    <section>
      <h2>Inicio</h2>
      <p>Bienvenido a la SPA con React Router.</p>
    </section>
  );
}
```

```tsx
// src/pages/NotFound.tsx
import { Link } from 'react-router-dom';

export default function NotFound() {
  return (
    <section>
      <h2>404 — Página no encontrada</h2>
      <Link to="/">Volver al inicio</Link>
    </section>
  );
}
```

- [ ] `Home` y `NotFound` existen; una URL inventada (ej: `/xyz`) cae en el 404

---

## Verificación

```bash
cd ~/proyectos/frontend-frameworks/mi-app
npm run dev         # http://localhost:5173
```

```text
1. Desde el nav navegás Inicio → Usuarios → detalle de un usuario con los botones Back/Forward del navegador funcionando.
2. La URL `/users/3` muestra el usuario 3 y la URL `xyz` inventada muestra el 404.
3. El botón "Volver a usuarios" navega programáticamente.
4. A lo largo de todo eso la página NUNCA recarga (abrí DevTools → Network y no hay ningún document reload).
```

**Si navegás por las rutas con back/forward funcionando, `/users/3` muestra el usuario 3 y no hay recargas de página → React Router listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Back/Forward no funciona | Usás `window.location` o `<a href>`: eso recarga. Para una SPA, siempre `Link`/`useNavigate` |
| `/users/3` da 404 en dev | Los segmentos dinámicos van con `:id` en el path y se leen con `useParams()`. Revisá que el orden de `<Route>` no colisione con `*` |
| El fetch vuelve a correr al navegar | Es correcto: el efecto depende de `[id]` y la URL cambió. Si NO querés re-fetch, pensá en caching (Outro tópico) |
| La página recarga con cada clic | Algo está usando `<a href>` en vez de `<Link>`. En una SPA eso mata el estado |
| `useNavigate` fuera de un Router | `useNavigate`, `useParams` y `Link` deben estar DENTRO de `<BrowserRouter>` (los componentes de las rutas lo están) |
| El 404 se muestra en todas las rutas | El catch-all `path="*"` debe ir al final y matchear solo lo que no matcheó nada anterior |

---

## Recursos

- [React Router — declarative mode](https://reactrouter.com/start/modes)
- [React Router — Route](https://reactrouter.com/api/components/Route)
- [React Router — useParams](https://reactrouter.com/api/hooks/useParams)
- [React Router — useNavigate](https://reactrouter.com/api/hooks/useNavigate)

## Preguntas de repaso

- **P:** ¿Por qué una SPA necesita un router si solo tiene una página HTML?
  **R:** Porque "navegar" en una SPA es cambiar qué componente se muestra según la URL. El router observa la URL, decide qué renderizar, y actualiza el historial para que back/forward funcionen.

- **P:** ¿Qué diferencia hay entre `<Link>` y `<a href>` en una SPA?
  **R:** `<Link>` intercepta el clic y cambia la URL sin recargar (usa `pushState`). `<a href>` recarga toda la página y mata el estado React.

- **P:** ¿Cómo leés un parámetro dinámico como `:id` de la URL?
  **R:** Con `useParams()` que devuelve un objeto con las claves del path. Para `/users/3`, `useParams()` devuelve `{ id: "3" }`.

- **P:** ¿Cuándo usás `useNavigate()` en vez de `<Link>`?
  **R:** `useNavigate()` para redirecciones programáticas después de lógica (login exitoso, submit de formulario). `<Link>` para links normales de navegación en la UI.

- **P:** ¿Qué hace la ruta `path="*"` y por qué debe ir al final?
  **R:** Es un catch-all para URLs que no matchean ninguna ruta anterior (404). Debe ir al final porque `Routes` renderiza la PRIMERA que matchea; si va antes, captura todo.

- **P:** ¿Por qué el efecto de fetch en `UserDetail` depende de `[id]`?
  **R:** Porque cuando navegás de `/users/3` a `/users/9`, el `id` cambia y el efecto debe re-ejecutarse para fetchear el usuario correcto. Sin `[id]` en las deps, no re-fetchea.