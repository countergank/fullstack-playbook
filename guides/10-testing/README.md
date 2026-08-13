# 10 — Testing: Orden de Ejecución

> **El orden importa. Vitest → Jest → Playwright.** Primero entendés el unit test (la base de todo) con Vitest; después ves el runner alternativo con mocks integrados (Jest); y recién al final subís a la app completa con Playwright. Cada guía apoya a la siguiente.

## Prerequisito

Node instalado (`setup-node.md` de 02-programming) + **concepto 10 completo** (los 9 niveles de testing). No hace falta saber todo de antemano: la primera guía arranca desde cero con una función pura. Proyecto de práctica: `~/proyectos/mi-app`.

## Paso a paso

1. **[setup-vitest.md](setup-vitest.md)** — Unit testing con Vitest: instalás el runner, escribís tu primer test de una función pura (`cartTotal`) y generás cobertura con `--coverage`. Es el corazón del tópico.
2. **[setup-jest.md](setup-jest.md)** — Jest para unit tests con mocks: el runner clásico, con `jest.mock` para aislar una dependencia y verificar interacciones. Misma lógica que Vitest, distinto ecosistema.
3. **[setup-playwright-e2e.md](setup-playwright-e2e.md)** — E2E con Playwright: descargás navegadores, configurás `baseURL` y escribís un test que maneja la app completa como un usuario.

---

## Verificación final

- `npm run test:run` en el proyecto Vitest pasa 2 tests y `npm run coverage` muestra `cart.ts` al 100%.
- `npm run test:run` en el proyecto Jest pasa el test que mockea la dependencia `db`.
- `npm run test:e2e` con Playwright pasa contra la app corriendo en `http://localhost:5173`.

**Si los tres pasan → Testing listo. ✅**
