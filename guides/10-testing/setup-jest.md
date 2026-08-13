# Setup — Jest para Unit Testing con Mocks

> **Tópico**: 10 — Testing (secciones 10.1 Unit Tests y 10.6 Mocking)
> **Objetivo**: instalar y configurar Jest en un proyecto Node, escribir un unit test y mockear una dependencia externa.
> **Prerequisito**: Node instalado (`setup-node.md` de 02-programming) y un proyecto con `package.json`. Se asume que ya sabés qué es un unit test (`setup-vitest.md`).

---

## ¿Por qué?

Jest es el runner clásico de JavaScript: maduro, con un ecosistema enorme de plugins y con mocking integrado de fábrica (`jest.fn`, `jest.mock`). Si tu equipo ya tiene infraestructura sobre Jest, o si el proyecto no usa Vite, es la opción segura. Acá lo levantamos para unit tests con mocks, el caso más común.

---

## Checklist

### 1. Crear el proyecto (si no tenés uno)

```bash
mkdir ~/proyectos/mi-app-jest && cd ~/proyectos/mi-app-jest
npm init -y
```

- [ ] `package.json` creado con `npm init -y`

### 2. Instalar Jest

```bash
npm install -D jest
```

- [ ] `jest` aparece en `devDependencies` de `package.json`

### 3. Agregar el script de test

```jsonc
// package.json
{
  "scripts": {
    "test": "jest",
    "test:run": "jest --runInBand"
  }
}
```

- `npm test` → watch mode (`jest --watch`); `npm run test:run` → una pasada para CI.
- [ ] Los scripts `test` y `test:run` están en `package.json`

### 4. Escribir un servicio con una dependencia externa

```js
// src/usuarios.js
const { db } = require('../lib/db');

async function listarActivos() {
  const todos = await db.usuarios.findMany();
  return todos.filter((u) => u.activo);
}

module.exports = { listarActivos };
```

```js
// lib/db.js
const db = {
  usuarios: { findMany: async () => [] },
};

module.exports = { db };
```

- [ ] `src/usuarios.js` y `lib/db.js` existen

### 5. Escribir el test mockeando la dependencia

```js
// src/usuarios.test.js
const { db } = require('../lib/db');
const { listarActivos } = require('./usuarios');

jest.mock('../lib/db', () => ({
  db: { usuarios: { findMany: jest.fn() } },
}));

describe('listarActivos', () => {
  beforeEach(() => jest.clearAllMocks());

  test('filtra los usuarios inactivos', async () => {
    db.usuarios.findMany.mockResolvedValue([
      { id: 1, activo: true },
      { id: 2, activo: false },
    ]);

    const activos = await listarActivos();

    expect(activos).toEqual([{ id: 1, activo: true }]);
    expect(db.usuarios.findMany).toHaveBeenCalledTimes(1);
  });
});
```

- `jest.mock` reemplaza el módulo `db` por una versión controlada.
- `mockResolvedValue` fija lo que devuelve la promesa.
- `toHaveBeenCalledTimes(1)` verifica la interacción (eso lo vuelve un mock, no un stub).

- [ ] `src/usuarios.test.js` existe y el test pasa

### 6. Correr los tests

```bash
npm run test:run
```

- [ ] La salida termina con `Tests: 1 passed`

---

## Verificación

```bash
cd ~/proyectos/mi-app-jest
npm run test:run            # una sola pasada, todo en verde
```

```text
1. npm run test:run termina con "Tests: 1 passed".
2. Cambiás mockResolvedValue a devolver usuarios inactivos y el test falla.
3. Quitás jest.mock y el test intenta usar el db real (findMany devuelve []), fallando el filtro.
```

**Si `npm run test:run` pasa y el mock controla el resultado → Jest listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `SyntaxError: Cannot use import statement outside a module` | Jest usa CommonJS por defecto; usá `require` o configurá Babel/ESM en `jest.config.js` |
| `jest.mock` no hace efecto | El mock debe declararse en el mismo archivo del test y ANTES de usarse; Jest hoistea `jest.mock` al inicio |
| "Cannot find module" al mockear | La ruta en `jest.mock` debe ser idéntica a la que usa el archivo bajo prueba, relativa a ese archivo |
| Los tests se contaminan entre sí | Agregá `jest.clearAllMocks()` en `beforeEach`, o `resetMocks: true` en la config |
| `jest: command not found` | Instalá con `npm install -D jest` y corré vía script de `package.json` |
| Tests muy lentos al correrlos todos juntos | Usá `--runInBand` para CI (evita paralelismo que compite por recursos) |

---

## Recursos

- [Jest — Getting Started](https://jestjs.io/docs/getting-started)
- [Jest — Mock Functions](https://jestjs.io/docs/mock-functions)
- [Jest — Configuring](https://jestjs.io/docs/configuration)

## Preguntas de repaso

- **P:** ¿Por qué Jest puede requerir más configuración que Vitest para TypeScript o ESM?
  **R:** Porque Jest usa CommonJS por defecto; para `import`/TS necesitás Babel o un preset como `ts-jest`, mientras Vitest los resuelve nativo vía esbuild.

- **P:** ¿Qué hace `jest.mock('../lib/db', ...)`?
  **R:** Reemplaza el módulo `db` real por una versión controlada por el test, evitando depender de la implementación real de la base.

- **P:** ¿Cuál es la diferencia entre `mockResolvedValue` y `mockRejectedValue`?
  **R:** `mockResolvedValue` hace que la promesa se resuelva con un valor; `mockRejectedValue` hace que se rechace con un error, útil para testear ramas de fallo.

- **P:** ¿Qué verifica `expect(db.usuarios.findMany).toHaveBeenCalledTimes(1)`?
  **R:** Que la dependencia se llamó exactamente una vez; esa verificación de interacción es lo que convierte al doble en un mock.

- **P:** ¿Para qué sirve `jest.clearAllMocks()` en `beforeEach`?
  **R:** Para limpiar el historial de llamadas y valores entre tests, evitando que un test contamine el resultado del siguiente.

- **P:** ¿Qué script usarías en CI y por qué?
  **R:** `npm run test:run` con `--runInBand`, que corre una sola pasada secuencial y termina con un código de salida que CI puede interpretar.
