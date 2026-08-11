# Setup — Vite + React + TypeScript

> **Tópico**: 8 — Frameworks & Herramientas Frontend
> **Objetivo**: levantar tu primer proyecto React con Vite: el dev server con HMR, la estructura que genera el template y el alias `@` para imports.
> **Prerequisito**: Node (`setup-node.md` de 02-programming) y TypeScript (`setup-typescript.md` de 07-frontend-core). El TS de este tópico sale del template de Vite — no lo configurás a mano.

---

## ¿Por qué Vite + React?

En el tópico 7 compilabas TS con `tsc` y cargabas el resultado con un `<script>`. Eso es correcto pero no escala. Vite te da el paquete adulto: dev server con recarga en caliente (HMR), bundles de producción, y un template con React + TypeScript listo. React es el framework de UI del concept 8.1; Vite su bundler de 8.4. Este proyecto base es el que van a usar las guías siguientes (hooks, estado, router, tests, estilos).

---

## Checklist

### 1. Crear el proyecto con el template `react-ts`

```bash
cd ~/proyectos/frontend-frameworks
npm create vite@latest mi-app -- --template react-ts
```

Con npm 7+ los `--` separan los argumentos: lo de antes de `--` es para npm, lo de después va directo a `create-vite`. Sin el `--`, el `--template react-ts` se pierde (o se interpreta mal). Si te pregunta, respondé `No` a instalar deps y sigamos vos sóles los pasos.

```bash
cd mi-app
npm install
```

- [ ] `npm install` termina sin errores en `~/proyectos/frontend-frameworks/mi-app`

### 2. Conocer la estructura que genera

```
mi-app/
├── index.html          # ÚNICA página — la SPA vive dentro de <div id="root">
├── src/
│   ├── main.tsx        # entry: createRoot(...).render(<App />)
│   ├── App.tsx         # componente raíz (trae el demo con logos)
│   ├── App.css         # estilos del demo (los vas a borrar)
│   ├── index.css       # estilos globales cargados por main.tsx
│   └── vite-env.d.ts   # tipos de Vite (no lo toques)
├── vite.config.ts      # config de Vite
├── tsconfig.json       # apunta a tsconfig.app.json y tsconfig.node.json
└── package.json
```

- `index.html` es la única página HTML que existe. Adentro solo hay un `<div id="root"></div>` vacío: toda la app React se monta ahí.
- `main.tsx` es el entry: le dice a React "montá la app en el elemento `root`".
- [ ] Reconocés cada archivo de la estructura y sabés para qué sirve

### 3. Levantar el dev server por primera vez

```bash
npm run dev
```

- [ ] El dev server arranca y Vite muestra `Local: http://localhost:5173/`
- [ ] Abrís el navegador en `http://localhost:5173` y ves la página de bienvenida con el logo de React y el contador del demo

### 4. Reemplazar el demo por tu propio componente

El template viene lleno de logos y estilos de ejemplo. Borralos y escribí tu componente:

```bash
rm src/App.css
```

```tsx
// src/App.tsx
function App() {
  return (
    <main>
      <h1>Mi primera app con Vite + React</h1>
      <p>Editá este archivo: el cambio aparece sin recargar.</p>
    </main>
  );
}

export default App;
```

- [ ] Los logos y el CSS del demo desaparecieron
- [ ] `src/App.tsx` devuelve un componente propio simple

### 5. Entender cómo `main.tsx` monta la app

Mirá `src/main.tsx` — es fijo y casi nunca se toca:

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';
import './index.css';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

- `createRoot(...).render(<App />)` → React crea el árbol y lo mete dentro del `<div id="root">` del `index.html`.
- `StrictMode` → en desarrollo duplica efectos de montaje para avisarte de bugs; en producción no hace nada.
- [ ] Sabés que el `<div id="root">` es el ancla entre HTML estático y la app React

### 6. Probar el Hot Module Replacement (HMR)

Con `npm run dev` corriendo, editá el `<p>` de `App.tsx` y guardá.

- [ ] El cambio aparece en el navegador SIN recargar la página (la URL no se resetea y el estado de la app se mantiene)
- [ ] En la terminal, Vite mostró `hmr update /src/App.tsx` o similar

### 7. (Opcional) Alias `@` para imports limpios

Vite entiende un alias `@` → `src` para evitar `../../`. Agregalo al `vite.config.ts`:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': '/src',
    },
  },
});
```

TypeScript no sabe interpretar el alias solo — hay que declarar el `paths` también. En el template actual las opciones de la app viven en `tsconfig.app.json`:

```jsonc
// tsconfig.app.json
{
  "compilerOptions": {
    // ...el resto que ya existe
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Ahora `import App from '@/App'` funciona igual que `./App`, y sin `..` anidados:

```tsx
// src/main.tsx
import App from '@/App';
```

- [ ] El alias `@` funciona en Vite y TypeScript no marca error de tipos

---

## Verificación

```bash
cd ~/proyectos/frontend-frameworks/mi-app
npm run dev          # 1. dev server levantado en http://localhost:5173
```

```text
1. Edito App.tsx y el cambio aparece en el navegador SIN recargar (HMR funcionando).
2. Un componente simple propio reemplaza al demo del template.
3. (Si hiciste el paso 7) los imports usan '@/...' y tsc no se queja.
```

**Si el dev server corre en 5173, editás `App.tsx` y el cambio aparece al toque sin recargar → Vite + React + TS listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `npm create vite` crea el proyecto pero no instala nada | Corré `npm install` dentro de `mi-app/` — create-vite no instala por defecto |
| El template no me deja usar `-- --template react-ts` | Con npm 7+ los `--` separan los args de npm de los de create-vite; sin ellos el flag se ignora |
| `npm run dev` abre en otro puerto | Vite sube el puerto si 5173 está ocupado: mirá la línea `Local:` de la terminal |
| El cambio no aparece al guardar | Verificá que el dev server siga corriendo (no cerraste la terminal) y que editaste `src/App.tsx`, no otro archivo |
| TypeScript no conoce el alias `@` | El alias solo en `vite.config.ts` no basta: declará también `paths` en `tsconfig.app.json` |
| La página sale en blanco | Mirá la consola del navegador: si es error de import, chequeá que `main.tsx` monte la app en el `root` que existe en `index.html` |

---

## Recursos

- [Vite — Getting Started](https://vite.dev/guide/)
- [Vite — Crear proyectos con templates](https://vite.dev/guide/#scaffolding-your-first-vite-project)
- [Vite — Alias de paths](https://vite.dev/config/shared-options#resolve-alias)
- [TypeScript — Module resolution paths](https://www.typescriptlang.org/tsconfig/#paths)

## Preguntas de repaso

- **P:** ¿Por qué se usan `--` antes de `--template react-ts` en `npm create vite`?
  **R:** Con npm 7+, los `--` separan los argumentos de npm de los de create-vite. Sin ellos, el flag `--template` se pierde o se interpreta mal.

- **P:** ¿Qué es `index.html` en una SPA con Vite y por qué es la única página?
  **R:** Es el punto de entrada del navegador. Contiene un `<div id="root">` vacío donde React monta toda la app. No hay otras páginas HTML porque la SPA "navega" cambiando componentes, no recargando.

- **P:** ¿Qué hace `StrictMode` en desarrollo y por qué duplica efectos?
  **R:** En desarrollo, `StrictMode` monta, desmonta y vuelve a montar los componentes para detectar bugs en efectos y lifecycle. En producción no hace nada.

- **P:** ¿Qué es HMR y cómo verificás que funciona?
  **R:** Hot Module Replacement: actualiza solo el módulo cambiado sin recargar la página. Se verifica editando `App.tsx` y guardando: el cambio aparece en el navegador sin que la URL se resetee.

- **P:** ¿Por qué el alias `@` necesita configuración en DOS lugares?
  **R:** Vite necesita el alias en `vite.config.ts` para resolver imports en runtime. TypeScript necesita `paths` en `tsconfig.app.json` para el type checking. Sin ambos, uno de los dos falla.

- **P:** ¿Qué hace `createRoot(document.getElementById('root')!).render(<App />)`?
  **R:** Crea el root de React y monta el componente `<App />` dentro del `<div id="root">` del `index.html`. Es el puente entre HTML estático y la app React.