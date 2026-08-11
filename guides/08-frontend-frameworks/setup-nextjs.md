# Setup — Next.js (App Router)

> **Tópico**: 8 — Frameworks & Herramientas Frontend
> **Objetivo**: crear una app Next.js con App Router, entender la diferencia entre Server y Client Components, y ver por qué Next renderiza en el servidor.
> **Prerequisito**: Node (`setup-node.md` de 02-programming) y React (`setup-vite-react.md` + hooks). Corré esta guía en un directorio propio, NO dentro de `mi-app` de Vite.

---

## ¿Por qué Next.js?

React solo es una librería de UI que corre en el cliente: la página llega vacía y React llena el DOM (CSR). Next.js suma renderizado en el servidor, routing por archivos y SEO — concept 8.6. La app que armás acá es distinta de la de Vite: acá el navegador recibe HTML ya renderizado desde el servidor y los Server Components pueden leer datos directo, sin API route intermedia.

---

## Checklist

### 1. Crear el proyecto con `create-next-app`

```bash
cd ~/proyectos/frontend-frameworks
npx create-next-app@latest mi-app-next --ts --app --tailwind --eslint --no-src-dir
```

Qué hace cada flag:

- `--ts` → TypeScript (es el default, pero lo explicitamos).
- `--app` → App Router (routing por archivos en `app/`), no Pages Router.
- `--tailwind` → arma Tailwind ya configurado (en la v4 no hace falta `tailwind.config.js`).
- `--eslint` → configura ESLint.
- `--no-src-dir` → todo vive en `app/` en la raíz del proyecto (sin carpeta `src/`).

Al pasar flags, el CLI omite los prompts interactivos y usa defaults sensatos para el resto. Si preferís elegir en vivo, corré solo `npx create-next-app@latest mi-app-next` y respondé los prompts.

```bash
cd mi-app-next
npm run dev
```

- [ ] El proyecto se creó sin errores y `npm run dev` responde en `http://localhost:3000`

### 2. Entender la estructura del App Router

```
mi-app-next/
├── app/
│   ├── layout.tsx        # layout COMPARTIDO (nav, footer) — envuelve todas las rutas
│   ├── page.tsx          # ruta "/"
│   ├── globals.css       # estilos globales (Tailwind v4: @import "tailwindcss")
│   └── users/
│       ├── page.tsx      # ruta "/users"
│       └── [id]/
│           └── page.tsx  # ruta "/users/1" (segmento dinámico)
├── public/               # assets estáticos
├── next.config.ts
├── package.json
└── tsconfig.json
```

En App Router, la ruta la define el ARCHIVO y la carpeta. `page.tsx` en una carpeta = la página de la URL de esa carpeta. Todo dentro de `app/` es Server Component por default (salvo que diga `'use client'`).

- [ ] Reconocés que `layout.tsx` es compartido y que cada `page.tsx` es una URL

### 3. Server Component async que lee datos

```tsx
// app/users/page.tsx
import Link from 'next/link';

interface User {
  id: number;
  name: string;
  email: string;
}

// SIN 'use client': esto es un Server Component. Puede ser async y hacer fetch en el servidor.
export default async function UsersPage() {
  const res = await fetch('https://jsonplaceholder.typicode.com/users');
  const users: User[] = await res.json();

  return (
    <section>
      <h1>Usuarios</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            <Link href={`/users/${user.id}`}>{user.name}</Link>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

- Un Server Component puede ser `async`: el `await fetch(...)` corre **en el servidor**, y el HTML llega ya con los datos al navegador (SSR).
- No corre en el cliente: no puede usar `useState`, `useEffect` ni `onClick`. Por eso el fetch va acá, no en un efecto.
- Beneficio de seguridad: las credenciales de la API quedan en el servidor, nunca viajan al navegador.
- `next/link` → `<Link>` de Next para navegación client-side entre páginas.

- [ ] `/users` renderiza usuarios desde el servidor

### 4. Ruta dinámica con `[id]`

```tsx
// app/users/[id]/page.tsx
interface User {
  id: number;
  name: string;
  email: string;
}

interface PageProps {
  params: Promise<{ id: string }>;
}

export default async function UserPage({ params }: PageProps) {
  const { id } = await params;   // en Next 15+ params es una Promise y se espera con await

  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
  const user: User = await res.json();

  return (
    <section>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <Link href="/users">Volver a usuarios</Link>
    </section>
  );
}
```

- La carpeta `[id]` expone el segmento de la URL como parámetro. `/users/1` → `id = "1"`.
- En Next 15+ `params` es una `Promise` y se resuelve con `await` dentro del componente. (En Next 14 era un objeto plano.)

- [ ] `/users/1` muestra el usuario 1

### 5. Client Component mínimo con `'use client'`

Los Server Components no tienen interactividad. Para estado y eventos marcamos un componente con `'use client'`:

```tsx
// app/counter.tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicks: {count}
    </button>
  );
}
```

```tsx
// app/page.tsx
import Counter from './counter';

export default function HomePage() {
  return (
    <main>
      <h1>Next.js App Router</h1>
      <p>Server Component que renderiza un Client Component adentro.</p>
      <Counter />
    </main>
  );
}
```

- Un Server Component puede renderizar un Client Component (el `<Counter />` va dentro de la página server).
- `'use client'` NO hace que todo sea cliente: marca el límite desde el cual los hijos corren en el navegador.
- Regla práctica (concept 8.6): **default server; `'use client'` SOLO cuando necesitás interactividad** (estado, eventos, efectos).

- [ ] El contador en `/` funciona con clicks (actualiza en vivo, sin recarga)

---

## Verificación

```bash
cd ~/proyectos/frontend-frameworks/mi-app-next
npm run dev         # http://localhost:3000
```

```text
1. http://localhost:3000 muestra el Home con el contador client-side funcionando.
2. http://localhost:3000/users lista usuarios renderizados EN EL SERVIDOR.
3. http://localhost:3000/users/1 muestra el usuario 1.
4. Abrís DevTools → Network → deshabilitás JS y recargás: el HTML de /users ya trae los usuarios (prueba de SSR).
```

**Si `/users` y `/users/1` muestran datos desde el servidor y el contador client-side funciona → Next.js App Router listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "You're importing a component that needs useState. It only works in a Client Component" | Faltó `'use client'` al tope del archivo que usa hooks |
| `params` es `undefined` o no se desestructura | En Next 15+ `params` es una Promise: usá `const { id } = await params;` |
| El puerto 3000 está ocupado | Next toma el siguiente libre o usá `npm run dev -- -p 3001` |
| Fetch en Server Component pega al server cada request | En Next 15 el fetch se memoiza por request de forma automática; para caching explícito usá `cache`/`revalidate` |
| ¿Dónde quedó `pages/`? | Eso es el Pages Router viejo. Con `--app` usás `app/`, que es el modelo actual |
| No veo Tailwind funcionando | Verificá que `globals.css` tenga `@import "tailwindcss";` (v4) y que no lo hayas borrado |

---

## Recursos

- [Next.js — Create a new project](https://nextjs.org/docs/app/getting-started/installation)
- [Next.js — App Router (layouts y pages)](https://nextjs.org/docs/app/building-your-application/routing)
- [Next.js — Server y Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)
- [Next.js — `create-next-app` CLI flags](https://nextjs.org/docs/app/api-reference/cli/create-next-app)

## Preguntas de repaso

- **P:** ¿Qué diferencia hay entre un Server Component y un Client Component en Next.js?
  **R:** Un Server Component corre en el servidor, puede ser async y leer datos directo. NO usa hooks ni eventos. Un Client Component (`'use client'`) corre en el navegador con estado, efectos y eventos.

- **P:** ¿Por qué en Next 15+ `params` es una Promise y cómo se resuelve?
  **R:** Porque Next 15 cambió la API para que `params` sea async. Se resuelve con `const { id } = await params;` dentro del componente server.

- **P:** ¿Qué hace el flag `--app` en `create-next-app`?
  **R:** Configura el App Router (routing por archivos en `app/`) en vez del Pages Router viejo (`pages/`). Es el modelo actual de Next.js.

- **P:** ¿Cómo verificás que un Server Component realmente renderiza en el servidor?
  **R:** Abrís DevTools → Network → deshabilitás JavaScript y recargás. Si el HTML ya trae los datos renderizados, es SSR (el servidor los incluyó).

- **P:** ¿Puede un Server Component renderizar un Client Component?
  **R:** Sí. Un Server Component puede incluir un Client Component como hijo. El `'use client'` marca el límite: desde ahí hacia abajo, todo corre en el navegador.

- **P:** ¿Qué ventaja de seguridad tiene hacer fetch en un Server Component?
  **R:** Las credenciales de la API quedan en el servidor, nunca viajan al navegador del usuario. En un Client Component, las credenciales serían visibles en el código del cliente.