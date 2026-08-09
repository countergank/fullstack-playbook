# Setup — Testing de Componentes con Vitest

> **Tópico**: 8 — Frameworks & Herramientas Frontend
> **Objetivo**: instalar y configurar Vitest + Testing Library en el proyecto Vite y escribir tu primer test de un componente React real.
> **Prerequisito**: proyecto Vite + React (`setup-vite-react.md`) y un componente simple para testear (el `TaskItem` del `setup-react-hooks.md`).

---

## ¿Por qué Vitest?

Vitest es el framework de testing "nativo" de Vite (concept 8.5): comparte config, transformador y velocidad. La API es la de Jest (`describe`, `test`, `expect`) pero corre sobre la infraestructura de Vite, así que no hay que configurar nada raro para JSX o TS. Y con Testing Library testeás como un usuario real: por rol y texto visible, no por clases CSS internas.

---

## Checklist

### 1. Instalar las dependencias de testing

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

- `vitest` → el runner de tests.
- `@testing-library/react` → `render`, `screen`, `fireEvent` para componentes React.
- `@testing-library/jest-dom` → matchers legibles (`toBeInTheDocument`, `toHaveBeenCalledWith`).
- `jsdom` → simula el DOM en Node (necesario para renderizar componentes).

- [ ] Las 4 dependencias instaladas como dev

### 2. Configurar `vitest.config.ts`

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
  },
});
```

- `environment: 'jsdom'` → los tests corren con un DOM simulado (sin navegador real). Se puede poner global acá, o por archivo con el docblock `// @jest-environment jsdom` arriba del test como alternativa.
- `globals: true` → `describe`, `test`, `expect` disponibles sin importarlos.
- `setupFiles` → se ejecuta antes de cada test file.
- [ ] `vitest.config.ts` existe con `environment`, `globals` y `setupFiles`

### 3. Crear el setup file

```ts
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
```

La variante `/vitest` registra los matchers de jest-dom directamente sobre `expect` de Vitest (con tipos incluidos). Sin el sufijo también funciona en runtime, pero esta forma es la type-safe.

- [ ] `src/test/setup.ts` importa los matchers de jest-dom

### 4. Agregar los scripts al `package.json`

```jsonc
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run"
  }
}
```

- `npm test` (o `npx vitest`) → **watch mode**: corre los tests y re-corré al guardar. Es el default, para desarrollo.
- `npm run test:run` (`vitest run`) → **una sola pasada** y el proceso termina. Es para CI.

- [ ] Los scripts `test` y `test:run` están en `package.json`

### 5. Elegir el componente a testear

Extraé el `TaskItem` del `setup-react-hooks.md` a su propio archivo (ya quedó memoizado y es ideal para este test):

```tsx
// src/components/TaskItem.tsx
import type { Task } from '../types';

export default function TaskItem({
  task,
  onToggle,
}: {
  task: Task;
  onToggle: (id: number) => void;
}) {
  return (
    <li>
      <input
        type="checkbox"
        checked={task.done}
        onChange={() => onToggle(task.id)}
      />
      {task.title}
    </li>
  );
}
```

- [ ] `TaskItem.tsx` existe y recibe `task` + `onToggle` como props

### 6. Escribir el test con Testing Library

```tsx
// src/components/TaskItem.test.tsx
import { fireEvent, render, screen } from '@testing-library/react';
import { expect, test, vi } from 'vitest';
import TaskItem from './TaskItem';

test('muestra el título de la tarea', () => {
  render(
    <TaskItem
      task={{ id: 1, title: 'Aprender Vitest', done: false }}
      onToggle={() => {}}
    />,
  );

  expect(screen.getByText('Aprender Vitest')).toBeInTheDocument();
});

test('dispara onToggle con el id al hacer clic en el checkbox', () => {
  const onToggle = vi.fn();

  render(
    <TaskItem
      task={{ id: 3, title: 'Comprar pan', done: false }}
      onToggle={onToggle}
    />,
  );

  fireEvent.click(screen.getByRole('checkbox'));

  expect(onToggle).toHaveBeenCalledTimes(1);
  expect(onToggle).toHaveBeenCalledWith(3);
});
```

- `render(<TaskItem ... />)` → monta el componente en el DOM simulado.
- `screen.getByText(...)` → busca por texto visible (cómo lo ve un usuario).
- `screen.getByRole('checkbox')` → busca por rol ARIA, no por clase CSS.
- `fireEvent.click(...)` → simula el clic del usuario.
- `vi.fn()` → función espía: registra cuántas veces fue llamada y con qué args.
- El `task` con `done: false` en el render es el "arrangement" del test: cada test monta su propia versión del componente.

Corré el test en watch:

```bash
npm test
```

- [ ] Los 2 tests pasan y `npm test` queda escuchando cambios (watch)

### 7. Correr una sola pasada (modo CI)

```bash
npm run test:run
```

- [ ] `npm run test:run` pasa todo de una y el proceso termina con `Test Files 1 passed`

---

## Verificación

```bash
cd ~/proyectos/frontend-frameworks/mi-app
npm run test:run            # una sola pasada, resultado único
npm test                    # queda en watch, re-corre al guardar
```

```text
1. npm run test:run termina con los tests en verde (Test Files 1 passed, 2 passed).
2. npm test queda en watch: tocás TaskItem.test.tsx y re-corre solo.
3. Edito el título en el test a algo que no existe y falla con un mensaje claro (para probar que el setup funciona).
```

**Si `npm run test:run` pasa en una sola pasada Y `npm test` queda en watch → Vitest listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Failed to resolve import '@testing-library/jest-dom'" | El import del setup debe ser `'@testing-library/jest-dom/vitest'` y el setup definido en `setupFiles` |
| Tests fallan con "document is not defined" | Falta `environment: 'jsdom'` (o el docblock `// @jest-environment jsdom` en el archivo) |
| `describe`/`test`/`expect` no existen en el editor | Están habilitados por `globals: true`; si TS se queja, importalos directo desde `vitest` como en el ejemplo |
| `fireEvent` no dispara nada | Verificá que el `onChange` del checkbox esté conectado y que el test no esté apuntando a otro elemento |
| El test pasa pero la app rompe en runtime | El test verifica UN componente aislado; si el resto de la app falla, el problema está afuera del test |
| Vitest importa archivos que no quiero | Usá `exclude` en `test.exclude` para archivos no-test, o configura `include` de los `.test.tsx` |

---

## Recursos

- [Vitest — Getting started](https://vitest.dev/guide/)
- [Vitest — Config reference](https://vitest.dev/config/)
- [Vitest — `vitest run` vs watch](https://vitest.dev/guide/cli)
- [Testing Library — React](https://testing-library.com/docs/react-testing-library/intro)
- [Testing Library — Queries por rol](https://testing-library.com/docs/queries/about)