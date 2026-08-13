# Setup — Vitest para Unit Testing

> **Tópico**: 10 — Testing (sección 10.1 Unit Tests)
> **Objetivo**: instalar Vitest en un proyecto Node/Vite, configurar el runner y escribir tu primer unit test con cobertura.
> **Prerequisito**: Node instalado (`setup-node.md` de 02-programming) y un proyecto con `package.json`. No hace falta Vite: Vitest corre unit tests en Node puro.

---

## ¿Por qué?

Vitest es el runner de testing con API de Jest (`describe`, `test`, `expect`) pero con ESM nativo, watch mode con HMR y cero configuración para TypeScript. Para unit tests de funciones puras, es la opción más rápida de levantar: instalás, creás un archivo `.test.ts` y corrés. Y trae cobertura integrada con un solo flag.

---

## Checklist

### 1. Crear el proyecto (si no tenés uno)

```bash
mkdir ~/proyectos/mi-app && cd ~/proyectos/mi-app
npm init -y
```

- [ ] `package.json` creado con `npm init -y`

### 2. Instalar Vitest como dependencia de desarrollo

```bash
npm install -D vitest
```

- [ ] `vitest` aparece en `devDependencies` de `package.json`

### 3. Agregar los scripts de test

```jsonc
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "coverage": "vitest run --coverage"
  }
}
```

- `npm test` → watch mode: queda escuchando y re-corre al guardar.
- `npm run test:run` → una sola pasada, para CI.
- `npm run coverage` → una pasada con reporte de cobertura.

- [ ] Los tres scripts (`test`, `test:run`, `coverage`) están en `package.json`

### 4. Escribir una función pura a testear

```ts
// src/lib/cart.ts
export type CartItem = { id: number; price: number; qty: number };

export function cartTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.qty, 0);
}
```

- [ ] `src/lib/cart.ts` existe con `cartTotal`

### 5. Escribir el primer unit test

```ts
// src/lib/cart.test.ts
import { describe, expect, it } from 'vitest';
import { cartTotal } from './cart';

describe('cartTotal', () => {
  it('suma precio por cantidad', () => {
    const items = [
      { id: 1, price: 100, qty: 2 },
      { id: 2, price: 50, qty: 1 },
    ];

    expect(cartTotal(items)).toBe(250);
  });

  it('devuelve 0 para un carrito vacío', () => {
    expect(cartTotal([])).toBe(0);
  });
});
```

- [ ] `src/lib/cart.test.ts` existe con al menos 2 tests

### 6. Correr los tests

```bash
npm run test:run
```

- [ ] La salida termina con `Test Files 1 passed` y `Tests 2 passed`

### 7. Correr con cobertura

```bash
npm run coverage
```

- [ ] Aparece una tabla con `% Stmts`, `% Branch`, `% Funcs` y `% Lines`, y el archivo `cart.ts` al 100%

---

## Verificación

```bash
cd ~/proyectos/mi-app
npm run test:run            # una sola pasada, todo en verde
npm run coverage            # reporte con cart.ts al 100%
npm test                    # queda en watch, re-corre al guardar
```

```text
1. npm run test:run termina con "Test Files 1 passed (1)" y "Tests 2 passed (2)".
2. npm run coverage muestra la tabla de cobertura y cart.ts al 100%.
3. Tocás cart.ts y npm test (watch) re-corre automáticamente.
4. Rompés el test a propósito (cambiás el 250 por 999) y falla con un mensaje claro.
```

**Si `npm run test:run` pasa en una sola pasada Y `npm run coverage` muestra la tabla → Vitest listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `vitest: command not found` | Instalá con `npm install -D vitest` y corré vía script de `package.json` (`npm test`), no directo en la shell |
| "No test files found" | Asegurate de que el archivo termine en `.test.ts` o `.spec.ts` y esté dentro del proyecto |
| `describe`/`test`/`expect` no se resuelven | Importalos desde `vitest` como en el ejemplo, o activá `globals: true` en la config |
| Coverage no muestra nada | Necesitás el proveedor: agregá `coverage: { provider: 'v8' }` en `vitest.config.ts` o corré con `--coverage` |
| Tests con TypeScript no compilan | Vitest transpila TS por defecto; si el editor se queja, verificá que `tsconfig.json` incluya `types: ["vitest/globals"]` solo si usás globals |
| El watch no detecta cambios | En WSL, trabajá en `~/` (no en `/mnt/c/`); el bridge Windows-Linux rompe el file-watching |

---

## Recursos

- [Vitest — Getting Started](https://vitest.dev/guide/)
- [Vitest — Coverage](https://vitest.dev/guide/coverage)
- [Vitest — CLI (`run` vs watch)](https://vitest.dev/guide/cli)
- [Vitest — Config reference](https://vitest.dev/config/)

## Preguntas de repaso

- **P:** ¿Por qué Vitest no necesita configuración extra para TypeScript o ESM?
  **R:** Porque usa esbuild y el pipeline de Vite por debajo: transpila TS y resuelve ESM de forma nativa sin Babel ni config adicional.

- **P:** ¿Qué diferencia hay entre `npm test` y `npm run test:run`?
  **R:** `npm test` (`vitest`) corre en watch mode y re-corre al guardar; `npm run test:run` (`vitest run`) hace una sola pasada y termina, ideal para CI.

- **P:** ¿Qué hace `npm run coverage` y qué métricas muestra?
  **R:** Corre los tests y genera un reporte de cobertura con `% Stmts`, `% Branch`, `% Funcs` y `% Lines`, indicando qué partes del código no ejecutaron los tests.

- **P:** ¿Por qué `cartTotal([])` devuelve `0`?
  **R:** Porque `reduce` sobre un array vacío retorna el valor inicial `0` sin ejecutar el callback.

- **P:** ¿Qué estructura debe seguir un buen unit test?
  **R:** La estructura AAA: Arrange (preparar datos), Act (ejecutar la función) y Assert (verificar el resultado), para que la intención sea clara.

- **P:** ¿Cuándo deja de ser unitario un test?
  **R:** Cuando la función bajo prueba toca la red, la base de datos o el filesystem; en ese punto ya es un test de integración.
