# Setup — Estado Global: Context + Zustand

> **Tópico**: 8 — Frameworks & Herramientas Frontend
> **Objetivo**: manejar datos que comparten varios componentes sin prop drilling: React Context para lectura global y Zustand para estado con lógica y escritura frecuente.
> **Prerequisito**: proyecto Vite + React (`setup-vite-react.md`) y los hooks (`setup-react-hooks.md`).

---

## ¿Por qué estas dos herramientas?

A medida que la app crece, el estado deja de ser local de un componente: "¿quién está logueado?", "¿qué tema usa la app?". Si cada componente tuviera su `useState`, no se enterarían de lo que pasó su vecino, y pasarlo por props por cinco niveles es ilegible (prop drilling, concept 8.2). Context resuelve lectura global; Zustand suma estado con lógica y updates frecuentes. Esta guía te muestra cuándo usar cuál.

---

## Checklist

### 1. Parte 1 — ThemeContext con `createContext` + `useContext`

Creamos un contexto para el tema (claro/oscuro) y un Provider que lo provee a toda la app:

```tsx
// src/context/ThemeContext.tsx
import { createContext, useContext, useState, type ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextValue {
  theme: Theme;
  toggleTheme: () => void;
}

// el contexto en sí: type significa "qué forma tienen los datos"
const ThemeContext = createContext<ThemeContextValue | null>(null);

// el Provider: envuelve a la app y guarda el estado del tema
export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');

  function toggleTheme() {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  }

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// hook propio para leer el contexto (es usar `useContext` adentro)
export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme debe usarse dentro de <ThemeProvider>');
  return ctx;
}
```

- `createContext` define el "tubo" de datos; `useContext` lo lee desde cualquier componente hijo.
- El error del hook es a propósito: si te olvidás el Provider, lo sabés al toque y no en producción.
- [ ] `ThemeContext.tsx` existe con `createContext`, `ThemeProvider` y `useTheme`

### 2. Montar el Provider y aplicar el tema al `body`

```tsx
// src/App.tsx
import { useEffect } from 'react';
import { ThemeProvider, useTheme } from './context/ThemeContext';

function Layout() {
  const { theme, toggleTheme } = useTheme();

  useEffect(() => {
    document.body.className = theme;        // 'light' o 'dark' como clase del <body>
  }, [theme]);

  return (
    <main>
      <h1>Theme con Context</h1>
      <button onClick={toggleTheme}>
        Cambiar a {theme === 'light' ? 'dark' : 'light'}
      </button>
      <p>Este texto cambia de color con el tema.</p>
    </main>
  );
}

function App() {
  return (
    <ThemeProvider>
      <Layout />
    </ThemeProvider>
  );
}

export default App;
```

El estilo en `src/index.css`:

```css
body.light { background: #fafafa; color: #111; }
body.dark  { background: #111;   color: #f5f5f5; }
```

> **Advertencia (concept 8.2)**: cuando el valor del contexto cambia, TODOS los consumidores se re-renderizan. Por eso el tema (o el usuario logueado) son buena candidata — cambian poco y se leen en todas partes — pero NO metas el texto de un input o un cronómetro en Context.

- [ ] El toggle cambia `light`/`dark` en todo `body` y cualquier componente puede leerlo con `useTheme()`

### 3. Parte 2 — Instalar Zustand

```bash
npm install zustand
```

- [ ] `zustand` instalado

### 4. Store de autenticación con `create`

Zustand crea un store como un hook. Las ACCIONES viven en el store (no en el componente):

```tsx
// src/store/auth.ts
import { create } from 'zustand';

interface AuthState {
  user: string | null;
  isLoading: boolean;
  login: (email: string) => void;
  logout: () => void;
}

export const useAuth = create<AuthState>((set) => ({
  user: null,
  isLoading: false,
  login: (email) => set({ user: email, isLoading: false }),
  logout: () => set({ user: null }),
}));
```

- `create` recibe una función con `set`. Llamar `set({...})` actualiza el estado de forma inmutable (merge).
- El store es un hook (`useAuth`): lo leés desde cualquier componente sin Provider ni prop drilling.
- [ ] `auth.ts` define `user`, `login`, `logout`, `isLoading` con sus acciones dentro del store

### 5. Leer el store con selectores desde dos componentes

Dos componentes independientes leen el MISMO store con selectores individuales:

```tsx
// src/components/LoginStatus.tsx
import { useAuth } from '../store/auth';

export function LoginStatus() {
  const user = useAuth((s) => s.user);      // selector: me suscribo SOLO a user
  const login = useAuth((s) => s.login);
  const logout = useAuth((s) => s.logout);

  if (!user) {
    return <button onClick={() => login('lean@example.com')}>Iniciar sesión</button>;
  }

  return (
    <p>
      Logueado como {user}{' '}
      <button onClick={logout}>Salir</button>
    </p>
  );
}
```

```tsx
// src/components/Header.tsx
import { useAuth } from '../store/auth';

export function Header() {
  const user = useAuth((s) => s.user);      // el mismo store, ningún prop
  return <header>App — {user ? `Hola, ${user}` : 'Invitado'}</header>;
}
```

```tsx
// src/App.tsx (ahora con los componentes de auth)
import { LoginStatus } from './components/LoginStatus';
import { Header } from './components/Header';

function App() {
  return (
    <main>
      <Header />
      <LoginStatus />
    </main>
  );
}

export default App;
```

- El selector `(s) => s.user` elige del estado SOLO lo que este componente necesita → cuando `user` no cambia, ese componente no re-renderiza.
- Las acciones se definen en el store, no en el componente → la lógica queda testeable y centralizada.
- [ ] `Header` y `LoginStatus` leen el mismo store sin prop drilling

### 6. Cuándo usar cada uno (tl;dr)

| Situación | Usá |
|-----------|-----|
| Datos globales de LECTURA que cambian poco (tema, idioma, usuario logueado) | **Context** |
| Estado global con LÓGICA y escritura frecuente (carrito, sesión compleja, updates por segundo) | **Zustand** |
| Estado local de un solo componente (open/close, texto de input) | **useState**, no hace falta nada global |

- Context es del framework (cero deps), pero re-renderiza a todos los consumidores. Zustand evita eso con selectores y expone devtools.
- [ ] Sabés argumentar qué herramienta elegir para un caso concreto

---

## Verificación

```bash
cd ~/proyectos/frontend-frameworks/mi-app
npm run dev         # http://localhost:5173
```

```text
1. Con Context: tocás el toggle y el tema cambia en TODA la app (body completo), no solo en el botón.
2. Con Zustand: desde LoginStatus "Iniciar sesión" y Header muestra "Hola, lean@example.com" al instante.
3. Tocás "Salir" y AMBOS componentes vuelven a estado invitado, sin prop drilling ni recarga.
```

**Si el tema se disa con Context en toda la app y login/logout de Zustand es consistente entre dos componentes distintos → estado global listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `useTheme` tira "must be used within ThemeProvider" | El componente que lo llama está fuera del `<>` del Provider o el Provider no envuelve la app |
| Todo se re-renderiza con cada cambio de estado | Context re-renderiza a todos los consumidores por diseño: para updates frecuentes usá Zustand con selectores fines |
| El componente re-renderiza igual estando logueado | El selector es grueso (`(s) => s` devuelve todo): usá `(s) => s.user` para suscribirte a un campo solo |
| No veo devtools de Zustand | Instalá la extensión de Redux DevTools en Chrome: Zustand la usa nativamente |
| `set` no re-renderiza | Verificaste que el store se llame como hook (`useAuth`) y que el componente esté montado dentro del tree de la app |

---

## Recursos

- [React — createContext](https://react.dev/reference/react/createContext)
- [React — useContext](https://react.dev/reference/react/useContext)
- [Zustand — Getting started](https://zustand.docs.pmnd.rs/getting-started/introduction)
- [Zustand — create](https://zustand.docs.pmnd.rs/apis/create)
- [React — Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)