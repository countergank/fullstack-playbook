# 08 — Frameworks & Herramientas Frontend: Orden de Ejecución

> **El orden importa. Vite+React → Hooks → Estado → Router → Tests → Next.js → Estilos.** La primera guía arma el proyecto Vite sobre el que se apoyan el resto (hooks, estado, router, tests y estilos); Vitest testea el componente que creaste en hooks; Next.js y los sistemas de estilos son apps/ejercicios posteriores sobre la misma base.

## Prerequisito

Node instalado (`setup-node.md` de 02-programming) + **concepto 07 completo** (HTML/CSS/JS/TS/DOM). El TypeScript de este tópico sale del template de Vite — no se configura a mano. Proyecto de práctica: `~/proyectos/frontend-frameworks`.

Podés clonar dentro del mismo directorio el proyecto del tópico 7 o crear el base de Vite desde cero con la primera guía.

## Paso a paso

1. **[setup-vite-react.md](setup-vite-react.md)** — Proyecto base Vite + React + TS: dev server con HMR en `http://localhost:5173`, estructura generada y alias `@`.
2. **[setup-react-hooks.md](setup-react-hooks.md)** — Los 6 hooks esenciales (`useState`, `useEffect`, `useRef`, `useMemo`, `useCallback`, `useReducer`) en UNA mini app de tareas real y funcional. Es el corazón del tópico.
3. **[setup-estado-global.md](setup-estado-global.md)** — Estado global: ThemeContext con Context y store de autenticación con Zustand (y cuándo usar cada uno).
4. **[setup-react-router.md](setup-react-router.md)** — Routing con React Router: `Routes`, `Route`, `:id`, 404, `<Link>` y `useNavigate`.
5. **[setup-vitest.md](setup-vitest.md)** — Testing de componentes con Vitest + Testing Library sobre el `TaskItem` del setup de hooks.
6. **[setup-nextjs.md](setup-nextjs.md)** — Next.js App Router en `~/proyectos/frontend-frameworks/mi-app-next`: Server Components async, `'use client'` y la diferencia entre ambos.
7. **[setup-sistemas-estilos.md](setup-sistemas-estilos.md)** — CSS Modules + Tailwind v4 + styled-components conviviendo en una misma página.

---

## Verificación final

- El proyecto Vite + React de `~/proyectos/frontend-frameworks/mi-app` corre con **HMR**: editás `App.tsx` y el cambio aparece sin recargar.
- La app de tareas carga datos remotos, agrega/completa/borra, tiene foco inicial y el contador de completadas se actualiza.
- `npm run test:run` pasa los tests de Vitest en una sola pasada.
- Next.js (`~/proyectos/frontend-frameworks/mi-app-next`) renderiza `/users` y `/users/1` desde el servidor y el contador client-side funciona.
- Las tres estrategias de estilos se ven correctas en la misma página.

**Si todo eso pasa → Frameworks & Herramientas Frontend listo. ✅**