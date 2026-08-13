# Setup — Playwright para E2E

> **Tópico**: 10 — Testing (sección 10.4 E2E)
> **Objetivo**: instalar Playwright, descargar los navegadores y escribir tu primer test end-to-end contra una app corriendo en local.
> **Prerequisito**: Node instalado (`setup-node.md` de 02-programming) y una app web corriendo en local (por ejemplo, la app Vite de `setup-vite-react.md`). Se asume que ya sabés qué es un test E2E (`setup-vitest.md`).

---

## ¿Por qué?

Playwright prueba la aplicación completa en un navegador real (Chromium, Firefox y WebKit), igual que un usuario. Es la red de seguridad final: atrapa los problemas que los unit e integration tests no ven, como una integración rota entre frontend y backend. Su auto-wait elimina la mayoría de los `sleep` y los tests flaky.

---

## Checklist

### 1. Crear el proyecto (o reusar el de la app)

```bash
mkdir ~/proyectos/mi-app-e2e && cd ~/proyectos/mi-app-e2e
npm init -y
```

- [ ] `package.json` creado con `npm init -y`

### 2. Instalar Playwright

```bash
npm install -D @playwright/test
```

- [ ] `@playwright/test` aparece en `devDependencies` de `package.json`

### 3. Descargar los navegadores

```bash
npx playwright install
```

Esto descarga los binarios de Chromium, Firefox y WebKit (son grandes, la primera vez tarda). Para CI sin interfaz gráfica también necesitás las dependencias del sistema:

```bash
npx playwright install-deps
```

- [ ] `npx playwright install` termina sin errores

### 4. Agregar el script de test

```jsonc
// package.json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui"
  }
}
```

- [ ] Los scripts `test:e2e` y `test:e2e:ui` están en `package.json`

### 5. Configurar la URL base

```ts
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: 'http://localhost:5173',
  },
});
```

- [ ] `playwright.config.ts` existe con `baseURL`

### 6. Escribir el primer test E2E

```ts
// e2e/inicio.spec.ts
import { expect, test } from '@playwright/test';

test('la página de inicio muestra el título', async ({ page }) => {
  await page.goto('/');

  await expect(page).toHaveTitle(/Vite/);
  await expect(page.getByRole('heading', { level: 1 })).toBeVisible();
});
```

- `page.goto('/')` usa la `baseURL` de la config.
- `getByRole('heading', ...)` busca por rol accesible, igual que un usuario.
- `toHaveTitle(/Vite/)` verifica el `<title>` de la pestaña.

- [ ] `e2e/inicio.spec.ts` existe con al menos 1 test

### 7. Correr el test

```bash
npm run test:e2e
```

- [ ] La salida termina con `1 passed`

---

## Verificación

```bash
cd ~/proyectos/mi-app-e2e
npm run test:e2e            # corre en headless contra http://localhost:5173
```

```text
1. La app está corriendo en http://localhost:5173 (npm run dev en el proyecto Vite).
2. npm run test:e2e termina con "1 passed".
3. Cambiás el título de la página en la app y el test falla al instante.
4. npm run test:e2e:ui abre la interfaz gráfica y podés ver el test paso a paso.
```

**Si `npm run test:e2e` pasa contra la app corriendo en local → Playwright listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Executable doesn't exist" | Faltan los navegadores: corré `npx playwright install` (y `npx playwright install-deps` en CI) |
| "net::ERR_CONNECTION_REFUSED" | La app no está corriendo en la `baseURL`; levantá el dev server antes de correr los tests |
| Tests fallan por timeout | Playwright espera por defecto; si el elemento nunca aparece, verificá el selector/rol y que la app cargue |
| `getByRole` no encuentra el elemento | Asegurate de que el elemento tenga el rol correcto (ej: `button`, `heading`) y el nombre accesible esperado |
| Los tests son flaky en CI | Usá `baseURL` fija, datos de prueba aislados y evitá depender del reloj o de servicios externos |
| Error al descargar navegadores | Revisá el proxy/red corporativa; podés pasar `PLAYWRIGHT_DOWNLOAD_HOST` como espejo alternativo |

---

## Recursos

- [Playwright — Installation](https://playwright.dev/docs/intro)
- [Playwright — Writing tests](https://playwright.dev/docs/writing-tests)
- [Playwright — Locators](https://playwright.dev/docs/locators)
- [Playwright — Test configuration](https://playwright.dev/docs/test-configuration)

## Preguntas de repaso

- **P:** ¿Qué hace `npx playwright install` y por qué es necesario?
  **R:** Descarga los binarios de los navegadores (Chromium, Firefox, WebKit) que Playwright necesita para correr los tests; sin ellos, no hay navegador que manejar.

- **P:** ¿Qué es la `baseURL` y para qué sirve en la config?
  **R:** Define la URL base de la app; permite usar rutas relativas en `page.goto('/')` y cambiar el entorno (local, staging) sin tocar los tests.

- **P:** ¿Qué significa "auto-wait" y qué problema resuelve?
  **R:** Que Playwright espera a que el elemento esté presente y accionable antes de interactuar; elimina los `sleep` arbitrarios y reduce tests flaky.

- **P:** ¿Por qué buscar por rol (`getByRole`) en vez de por selector CSS?
  **R:** Porque replica cómo un usuario percibe la UI (roles accesibles y texto), haciendo los tests robustos ante cambios de implementación.

- **P:** ¿Qué diferencia hay entre `test:e2e` (headless) y `test:e2e:ui`?
  **R:** `playwright test` corre en modo headless (sin ventana); `--ui` abre una interfaz gráfica para ver los tests correr paso a paso y depurar.

- **P:** ¿Qué problemas atrapa un test E2E que un unit test no ve?
  **R:** Integraciones rotas entre frontend y backend, configuraciones de deploy, y cualquier fallo que solo aparece cuando la app completa corre junta.
