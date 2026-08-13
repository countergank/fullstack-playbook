# 10. Testing

> Objetivo: entender los distintos niveles de testing, cuándo usar cada uno y con qué herramientas, para escribir una suite de tests que te dé confianza al refactorizar y te avise apenas algo se rompe.

## 10.1 Unit Tests

> Referencia base: [Vitest — Why Vitest](https://vitest.dev/guide/why) · [Jest — Getting Started](https://jestjs.io/docs/getting-started)

### Qué es una unidad

- Una **unidad** es el pedazo más chico de código que podés probar de forma aislada: una función, un método o un módulo puro.
- Un unit test NO toca la red, ni la base de datos, ni el filesystem. Si lo hace, dejó de ser unitario.
- Características: rápido (milisegundos), determinístico (mismo input → mismo output) y aislado.

*En criollo:* Un unit test es como probar una tuerca suelta antes de armar la máquina. Agarrás una función sola, le das entradas que vos controlás y mirás si la salida es la esperada. Si falla, sabés EXACTAMENTE dónde está el problema: no hay red, ni base de datos, ni otro componente que ensucie el diagnóstico.

*Técnicamente:* Una función pura y su test en Vitest:

```ts
// src/lib/cart.ts
export type CartItem = { id: number; price: number; qty: number };

export function cartTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.qty, 0);
}
```

```ts
// src/lib/cart.test.ts
import { describe, expect, it } from 'vitest';
import { cartTotal } from './cart';

describe('cartTotal', () => {
  it('suma precio por cantidad de cada item', () => {
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

Un buen unit test sigue la estructura **AAA**: *Arrange* (preparás), *Act* (ejecutás) y *Assert* (verificás). La idea es que la intención del test se lea de un vistazo.

> **Check de comprensión**
> 1. ¿Qué define a una "unidad" y cuándo un test deja de ser unitario?
>    - R: La unidad es el pedazo más chico de código que se puede probar aislado (una función, método o módulo puro). Deja de ser unitario cuando toca red, base de datos o filesystem.
> 2. ¿Por qué un unit test debe ser determinístico?
>    - R: Porque el mismo input debe producir siempre el mismo output; si depende de la hora, la red o el azar, no podés confiar en el resultado.
> 3. ¿Qué significan las tres A de la estructura AAA?
>    - R: Arrange (preparar datos y estado), Act (ejecutar la función bajo prueba) y Assert (verificar el resultado). Separa las fases para que el test sea legible.
> 4. ¿Qué devuelve `cartTotal([])` y por qué?
>    - R: Devuelve `0`, porque `reduce` sobre un array vacío retorna el valor inicial `0` sin ejecutar el callback ninguna vez.
> 5. ¿Por qué un unit test es más rápido que uno de integración?
>    - R: Porque no levanta servidores, ni conexiones a base de datos, ni navegadores; solo ejecuta una función en memoria.
> 6. ¿Qué matcher usarías para comparar el resultado con un valor exacto?
>    - R: `expect(resultado).toBe(valor)` para primitivos (números, strings, booleanos); `toEqual` para objetos y arrays (compara por estructura).

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.5-vitest--testing-de-componentes) para el runner que corre estos tests en un proyecto Vite.

---

## 10.2 Integration Tests

> Referencia base: [Vitest — Testing types](https://vitest.dev/guide/testing-types) · [Supertest — npm](https://www.npmjs.com/package/supertest)

### Dónde termina la unidad y empieza la integración

- Un test de integración verifica que **varias unidades trabajan juntas**: un handler con su ruta, un servicio con su repositorio, un middleware con el framework.
- A diferencia del unit test, acá sí puede haber una base de datos real (de testing), un server en memoria o el filesystem.
- Siguen siendo más lentos que los unitarios, pero atrapan bugs que los tests aislados no ven: contratos rotos entre módulos.

*En criollo:* El unit test prueba la tuerca; el de integración prueba que la tuerca entra en el tornillo y que la máquina anda. Es el test que te dice "tu función funciona, pero cuando la conectás con el router, se rompe todo". Ahí es donde aparecen los bugs de verdad: nombres de campos que no coinciden, tipos que cambian, orden de argumentos.

*Técnicamente:* Una API Express testeada de punta a punta (ruta + storage) con Supertest:

```ts
// src/server.ts
import express from 'express';

export function createApp() {
  const app = express();
  app.use(express.json());

  const users = new Map<number, { id: number; name: string }>();

  app.get('/users/:id', (req, res) => {
    const user = users.get(Number(req.params.id));
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  });

  app.post('/users', (req, res) => {
    const { id, name } = req.body;
    users.set(id, { id, name });
    res.status(201).json({ id, name });
  });

  return app;
}
```

```ts
// src/server.test.ts
import request from 'supertest';
import { describe, expect, it } from 'vitest';
import { createApp } from './server';

describe('flujo de usuarios', () => {
  it('crea un usuario y después lo devuelve', async () => {
    const app = createApp();

    const created = await request(app).post('/users').send({ id: 1, name: 'Juan' });
    expect(created.status).toBe(201);

    const fetched = await request(app).get('/users/1');
    expect(fetched.status).toBe(200);
    expect(fetched.body).toEqual({ id: 1, name: 'Juan' });
  });

  it('devuelve 404 si el usuario no existe', async () => {
    const app = createApp();
    const res = await request(app).get('/users/999');
    expect(res.status).toBe(404);
  });
});
```

Acá el test atraviesa el router, el handler y el storage a la vez. En proyectos reales, el `Map` se reemplaza por Prisma u otro ORM apuntando a una base de datos de testing.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia central entre un unit test y un test de integración?
>    - R: El unit test prueba una unidad aislada; el de integración verifica que varias unidades trabajan juntas (ruta + handler + storage, por ejemplo).
> 2. ¿Por qué un test de integración atrapa bugs que los unitarios no ven?
>    - R: Porque ejercita los contratos entre módulos: nombres de campos, tipos, orden de argumentos y estado compartido que solo fallan cuando las piezas se conectan.
> 3. ¿Qué rol cumple `createApp()` en el ejemplo y por qué se exporta?
>    - R: Fabrica una instancia de la app aislada por test, sin escuchar en un puerto real. Exportarla permite montar una app fresca en cada test.
> 4. ¿Por qué usarías una base de datos de testing en vez de la de producción?
>    - R: Porque los tests deben ser reproducibles y no contaminar datos reales; una base efímera se resetea entre corridas y garantiza resultados estables.
> 5. ¿Qué verifica Supertest en `request(app).get('/users/1')` que un unit test no podría?
>    - R: Que el request HTTP completo funciona: parsing de la URL, el router, el handler y la serialización de la respuesta.
> 6. ¿Qué status code y body esperás de `GET /users/999` en el ejemplo?
>    - R: `404` con el body `{ "error": "Not found" }`, porque el `Map` no contiene la clave `999`.

→ Ver [Tópico 5: Frameworks Backend](../concepts/05-frameworks-backend.md#5.1-express) para la base de Express que acá se monta en memoria.

---

## 10.3 Testing de Componentes

> Referencia base: [Testing Library — React](https://testing-library.com/docs/react-testing-library/intro) · [Testing Library — Queries](https://testing-library.com/docs/queries/about)

### Probar lo que el usuario ve

- Un test de componente monta un componente de UI en un DOM simulado (`jsdom`) y verifica su comportamiento desde el punto de vista del usuario.
- Se busca por **rol y texto visible**, no por clases CSS internas ni por estructura del árbol.
- El objetivo es que un refactor de la implementación (mientras el comportamiento visible no cambie) NO rompa el test.

*En criollo:* Acá testeás como si fueras el usuario: "hay un botón que dice Guardar, lo aprieto y se dispara una acción". No te importa cómo está construido por dentro, te importa lo que se ve y lo que hace. Si mañana cambiás un `div` por un `button` pero el usuario sigue viendo lo mismo, el test sigue en verde.

*Técnicamente:* Un componente y su test con Vitest + Testing Library:

```tsx
// src/components/Boton.tsx
export function Boton({
  onClick,
  children,
}: {
  onClick: () => void;
  children: React.ReactNode;
}) {
  return (
    <button type="button" onClick={onClick}>
      {children}
    </button>
  );
}
```

```tsx
// src/components/Boton.test.tsx
import { fireEvent, render, screen } from '@testing-library/react';
import { describe, expect, it, vi } from 'vitest';
import { Boton } from './Boton';

describe('Boton', () => {
  it('muestra el texto y dispara onClick al hacer clic', () => {
    const onClick = vi.fn();

    render(<Boton onClick={onClick}>Guardar</Boton>);

    fireEvent.click(screen.getByRole('button', { name: 'Guardar' }));

    expect(onClick).toHaveBeenCalledTimes(1);
  });
});
```

`getByRole('button', { name: 'Guardar' })` busca el elemento como lo haría un usuario (por su rol accesible y su texto visible), no por un selector CSS.

> **Check de comprensión**
> 1. ¿Por qué un test de componente busca por rol y texto en vez de por clase CSS?
>    - R: Porque replica lo que ve un usuario real: roles accesibles y texto visible. Es robusto ante cambios de implementación que no alteran la UI.
> 2. ¿Qué es `jsdom` y qué rol cumple en estos tests?
>    - R: Es una implementación del DOM para Node. Simula `document`, `window` y el render de componentes sin un navegador real.
> 3. ¿Qué hace `vi.fn()` en el ejemplo y qué verificás sobre él?
>    - R: Crea una función espía que registra cuántas veces fue llamada; verificás que `onClick` se disparó exactamente una vez tras el clic.
> 4. ¿En qué se diferencia el testing de componentes del unit test?
>    - R: El unit test prueba una función pura; el de componentes monta un componente de UI y verifica su render y sus interacciones en un DOM simulado.
> 5. ¿Por qué un refactor interno no debería romper un buen test de componente?
>    - R: Porque el test se ancla al comportamiento visible (rol, texto, interacción), no a la estructura interna del árbol de componentes.
> 6. ¿Qué problema revela que el test pase pero la app falle en runtime?
>    - R: Que el componente aislado funciona, pero el fallo está afuera de él (props, estado global, contexto o integración con otros componentes).

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.5-vitest--testing-de-componentes) para el setup completo de Vitest + Testing Library en un proyecto Vite.

---

## 10.4 E2E (End-to-End)

> Referencia base: [Playwright — Writing tests](https://playwright.dev/docs/writing-tests)

### La app completa, como un usuario

- Un test E2E maneja un **navegador real** contra la aplicación completa: frontend + backend + base de datos corriendo de verdad.
- Verifica flujos de usuario completos: "entrar, buscar, agregar al carrito, pagar".
- Son los más lentos y los más frágiles, por eso se reservan para los caminos críticos (happy paths), no para cubrir todos los casos borde.

*En criollo:* Es como contratar a alguien que se siente en la silla y use tu app de verdad: abre el navegador, hace clic, escribe, y vos mirás si todo funciona junto. Es la prueba final de que la máquina entera anda. Por ser lenta y frágil, no la corrés en cada guardado; la corrés en CI o antes de un deploy.

*Técnicamente:* Un test E2E con Playwright:

```ts
// e2e/login.spec.ts
import { expect, test } from '@playwright/test';

test('el usuario puede iniciar sesión', async ({ page }) => {
  await page.goto('http://localhost:5173/login');

  await page.getByLabel('Email').fill('juan@ejemplo.com');
  await page.getByLabel('Contraseña').fill('secreto123');
  await page.getByRole('button', { name: 'Entrar' }).click();

  await expect(page).toHaveURL(/dashboard/);
  await expect(page.getByText('Bienvenido, Juan')).toBeVisible();
});
```

Playwright corre el test en un Chromium real (o Firefox/WebKit), con **auto-wait**: espera a que el elemento esté accionable antes de interactuar, lo que elimina la mayoría de los `sleep` y los flaky tests.

> **Check de comprensión**
> 1. ¿Qué diferencia a un test E2E de un test de componente?
>    - R: El E2E corre en un navegador real contra la app completa (frontend + backend + base); el de componentes monta un componente aislado en un DOM simulado.
> 2. ¿Por qué los tests E2E se reservan para los caminos críticos?
>    - R: Porque son lentos y frágiles (dependen de toda la infraestructura); cubrir todos los casos borde con E2E haría la suite impracticable.
> 3. ¿Qué significa "auto-wait" en Playwright y qué problema resuelve?
>    - R: Que Playwright espera a que el elemento esté presente, visible y accionable antes de actuar. Elimina los `sleep` arbitrarios y reduce los tests flaky.
> 4. ¿Qué matcher usarías para verificar que la navegación cambió de URL?
>    - R: `await expect(page).toHaveURL(/dashboard/)`, que chequea la URL actual contra un patrón.
> 5. ¿En qué entorno se suele correr la suite E2E?
>    - R: En CI o antes de un deploy, no en cada guardado local, por su costo en tiempo y recursos.
> 6. ¿Qué navegadores puede manejar Playwright por defecto?
>    - R: Chromium, Firefox y WebKit (el motor de Safari), lo que permite cubrir los tres motores de render reales.

→ Ver [Tópico 9: DevOps](../concepts/09-devops-deployment.md#9.2-cicd) para correr esta suite en el pipeline de CI.

---

## 10.5 TDD (Test-Driven Development)

> Referencia base: [Jest — Test-driven development](https://jestjs.io/docs/getting-started) · [TestDouble — What is TDD?](https://martinfowler.com/bliki/TestDrivenDevelopment.html)

### El ciclo Red → Green → Refactor

- **Red**: escribís el test ANTES que la función; lo corrés y falla (porque la función no existe o no hace lo pedido).
- **Green**: escribís la implementación mínima que hace pasar el test.
- **Refactor**: mejorás el código sin cambiar el comportamiento, con los tests en verde como red de seguridad.
- TDD no es "escribir tests", es **usar los tests para diseñar** la interfaz de tu código.

*En criollo:* En vez de escribir la función y después el test, lo hacés al revés: primero pensás "¿cómo quiero usar esto?" y lo escribís como test. Eso te obliga a definir la API antes de implementarla. Después hacés lo mínimo para que pase, y al final limpiás. Es como el arquitecto que dibuja el plano antes de tirar ladrillos.

*Técnicamente:* El ciclo completo con una función de descuento:

```ts
// 1. RED — el test primero, falla porque la función no existe
// src/lib/descuento.test.ts
import { describe, expect, it } from 'vitest';
import { aplicarDescuento } from './descuento';

describe('aplicarDescuento', () => {
  it('aplica el porcentaje de descuento al precio', () => {
    expect(aplicarDescuento(100, 0.2)).toBe(80);
  });
});
```

```ts
// 2. GREEN — la implementación mínima que lo hace pasar
// src/lib/descuento.ts
export function aplicarDescuento(precio: number, desc: number): number {
  return precio * (1 - desc);
}
```

```ts
// 3. REFACTOR — mejorás sin romper el test (redondeo a 2 decimales)
export function aplicarDescuento(precio: number, desc: number): number {
  return Math.round(precio * (1 - desc) * 100) / 100;
}
```

En el paso Red el test falla por el motivo correcto (función inexistente o resultado incorrecto), nunca por un error de configuración.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre "escribir tests" y practicar TDD?
>    - R: TDD usa el test como herramienta de diseño: se escribe ANTES de implementar para definir la interfaz deseada, no como verificación posterior.
> 2. ¿Qué significan los tres pasos Red, Green y Refactor?
>    - R: Red (escribir un test que falla), Green (implementación mínima que lo pasa) y Refactor (mejorar sin cambiar comportamiento).
> 3. ¿Por qué es importante ver el test fallar en el paso Red?
>    - R: Para confirmar que el test realmente detecta la ausencia/comportamiento incorrecto y que no está roto por un error de configuración.
> 4. ¿Qué garantiza el paso Refactor?
>    - R: Que podés limpiar y mejorar el código con confianza, porque los tests en verde te avisan si rompés algo.
> 5. ¿Cómo te obliga TDD a definir la API antes de implementarla?
>    - R: Porque el test escrito primero fija nombres, parámetros y valores de retorno, lo que constituye la firma de la función antes de escribirla.
> 6. ¿Qué se busca con la "implementación mínima" en el paso Green?
>    - R: Hacer pasar el test con el menor código posible, sin anticipar requisitos futuros que todavía no tienen test.

→ Ver [Tópico 3: IA & Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.4-sdd--spec-driven-development) para ver cómo TDD se integra con el desarrollo dirigido por especificación.

---

## 10.6 Mocking

> Referencia base: [Vitest — Mocking](https://vitest.dev/guide/mocking) · [Jest — Mock functions](https://jestjs.io/docs/mock-functions)

### Reemplazar dependencias para aislar

- Un **mock** es un objeto o función que reemplaza una dependencia real (un módulo, una API, un servicio) para que el test no dependa de ella.
- Los mocks registran **interacciones**: cuántas veces se llamó, con qué argumentos y qué devolvió.
- Se usan para aislar la unidad bajo prueba de cosas lentas, externas o no determinísticas (red, base de datos, APIs de terceros).

*En criollo:* Imaginate que tu función habla con un servicio de pagos externo. Para testearla no querés cobrar de verdad ni depender de que ese servicio esté online. Entonces ponés un doble: un "actor suplente" que se comporta como el servicio real pero que vos controlás. El mock además te cuenta "me llamaron 2 veces, con estos datos".

*Técnicamente:* Mockear el módulo `db` del que depende un servicio:

```ts
// src/services/usuarios.ts
import { db } from '../lib/db';

export async function listarActivos() {
  const todos = await db.usuarios.findMany();
  return todos.filter((u) => u.activo);
}
```

```ts
// src/services/usuarios.test.ts
import { beforeEach, describe, expect, it, vi } from 'vitest';
import { db } from '../lib/db';
import { listarActivos } from './usuarios';

vi.mock('../lib/db', () => ({
  db: {
    usuarios: { findMany: vi.fn() },
  },
}));

describe('listarActivos', () => {
  beforeEach(() => vi.clearAllMocks());

  it('filtra los usuarios inactivos', async () => {
    vi.mocked(db.usuarios.findMany).mockResolvedValue([
      { id: 1, activo: true },
      { id: 2, activo: false },
    ]);

    const activos = await listarActivos();

    expect(activos).toEqual([{ id: 1, activo: true }]);
    expect(db.usuarios.findMany).toHaveBeenCalledTimes(1);
  });
});
```

`vi.mock` reemplaza el módulo entero en tiempo de compilación del test; `mockResolvedValue` define qué devuelve la promesa.

> **Check de comprensión**
> 1. ¿Qué es un mock y cuándo lo usás?
>    - R: Un doble que reemplaza una dependencia real y registra interacciones. Se usa para aislar la unidad de cosas externas, lentas o no determinísticas.
> 2. ¿Qué registra un mock que un stub no?
>    - R: Las interacciones: cuántas veces se llamó y con qué argumentos, lo que permite verificar el contrato además del resultado.
> 3. ¿Qué hace `vi.mock('../lib/db', ...)` en el ejemplo?
>    - R: Sustituye el módulo `db` real por una versión controlada donde `findMany` es una función espía definida por el test.
> 4. ¿Para qué sirve `mockResolvedValue`?
>    - R: Define el valor con el que se resuelve la promesa retornada por la función mockeada, simulando una respuesta exitosa.
> 5. ¿Por qué se llama a `vi.clearAllMocks()` en `beforeEach`?
>    - R: Para limpiar el estado de llamadas y valores entre tests, evitando que un test contamine al siguiente.
> 6. ¿Qué riesgo introduce un mock mal diseñado?
>    - R: Que el test pase contra un comportamiento inventado que no refleja la dependencia real, dando falsa seguridad (falso verde).

→ Ver [Tópico 10.7: Stubbing](#10.7-stubbing) para la variante más simple que solo fija valores de retorno sin verificar interacciones.

---

## 10.7 Stubbing

> Referencia base: [Sinon — Stubs](https://sinonjs.org/releases/latest/stubs/)

### Un doble que solo fija respuestas

- Un **stub** es un doble que devuelve un valor predefinido cuando se lo llama, sin lógica real y (a diferencia del mock) sin que el test verifique sus interacciones.
- Se usa para **controlar el entorno** del test: forzar que una función devuelva tal cosa, o que tire un error, para probar una rama específica.
- La distinción práctica: el mock responde "¿se usó como esperaba?", el stub responde "devolvé esto y no me importa el resto".

*En criollo:* Si el mock es un actor suplente que además te pasa el parte de lo que pasó, el stub es un cartel con la respuesta escrita: le preguntás y te dice siempre lo mismo. Lo usás cuando solo te interesa que la dependencia devuelva algo concreto para poder avanzar, sin fijarte en cuántas veces la llamaron.

*Técnicamente:* Un stub con Vitest y otro con Sinon:

```ts
// Stub con Vitest: una función que siempre devuelve lo mismo
const reloj = vi.fn().mockReturnValue('2026-08-12T00:00:00Z');
reloj(); // '2026-08-12T00:00:00Z'
reloj(); // '2026-08-12T00:00:00Z' (siempre igual)
```

```ts
// Stub con Sinon: reemplazar un método de un objeto
import sinon from 'sinon';
import * as repo from './repo';

sinon.stub(repo, 'getPrecio').returns(99);

// La función que consume repo.getPrecio recibe siempre 99
// ... al terminar, restaura:
// repo.getPrecio.restore();
```

Un caso clásico es stubear un error para testear la rama de falla: `.mockRejectedValue(new Error('DB caída'))`.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia esencial entre un mock y un stub?
>    - R: El mock verifica interacciones (llamadas y argumentos); el stub solo fija valores de retorno y no se usa para verificar cómo fue llamado.
> 2. ¿Para qué escenario es ideal un stub?
>    - R: Para controlar el entorno y forzar una rama específica (un valor concreto, o un error) sin preocuparte por el historial de llamadas.
> 3. ¿Qué hace `vi.fn().mockReturnValue(...)`?
>    - R: Crea una función que devuelve siempre el valor indicado, sin lógica real.
> 4. ¿Cómo stubearías un fallo de red para testear el manejo de errores?
>    - R: Con `.mockRejectedValue(new Error('...'))` sobre la función que haría la llamada, para que la promesa se rechace.
> 5. ¿Por qué es importante restaurar un stub de Sinon al terminar?
>    - R: Porque `sinon.stub` reemplaza el método en el objeto real; sin `restore()` (o en un `afterEach`) contaminás otros tests.
> 6. ¿Podés usar un stub para verificar que una función fue llamada dos veces?
>    - R: Técnicamente la función espía lo registra, pero por definición esa verificación de interacción corresponde a un mock, no al propósito de un stub.

→ Ver [Tópico 10.6: Mocking](#10.6-mocking) para el doble que, además de fijar respuestas, verifica cómo fue usado.

---

## 10.8 Coverage

> Referencia base: [Vitest — Coverage](https://vitest.dev/guide/coverage) · [Istanbul (nyc)](https://istanbul.js.org/)

### Medir cuánto cubren tus tests

- **Coverage** mide qué porcentaje del código ejecutan tus tests. Las métricas: **statements** (sentencias), **branches** (ramas de `if`/`switch`/ternarios), **functions** y **lines**.
- Un número alto no garantiza tests buenos: podés tener 100% de líneas con asserts débiles.
- La cobertura sirve para **encontrar código muerto y caminos sin probar**, no como meta por sí misma.

*En criollo:* La cobertura es como la cámara que te dice qué rincones de la casa limpiaste. Podés tener "todo el piso barrido" pero con las esquinas sucias — o barrer todo rápido sin sacar la mugre. El número te señala DÓNDE mirar; la calidad de los tests la ponés vos.

*Técnicamente:* Configurar coverage con umbrales en Vitest:

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
      thresholds: {
        lines: 80,
        branches: 75,
        functions: 80,
        statements: 80,
      },
    },
  },
});
```

```bash
npm run coverage
```

```text
File              | % Stmts | % Branch | % Funcs | % Lines
src/lib/cart.ts   |   100   |   100    |   100   |   100
src/lib/util.ts   |    60   |    40    |    66   |    58
------------------|---------|----------|---------|--------
All files         |    82   |    71    |    80   |    80
```

`thresholds` hace que la suite FALLE si la cobertura baja del umbral, útil en CI para no perder terreno. Si un valor no alcanza el umbral, `vitest run --coverage` termina con error.

> **Check de comprensión**
> 1. ¿Qué miden las cuatro métricas de cobertura (statements, branches, functions, lines)?
>    - R: Porcentaje de sentencias ejecutadas, de ramas (`if`/`switch`/ternarios) recorridas, de funciones llamadas y de líneas ejecutadas.
> 2. ¿Por qué una cobertura del 100% no garantiza tests de calidad?
>    - R: Porque podés ejecutar cada línea con asserts débiles o inexistentes; la cobertura mide ejecución, no la fuerza de las verificaciones.
> 3. ¿Qué hace la clave `thresholds` en la configuración?
>    - R: Define umbrales mínimos; si la cobertura cae por debajo, la corrida falla, evitando que el proyecto pierda cobertura en CI.
> 4. ¿Cuál es el uso correcto de la cobertura como herramienta?
>    - R: Detectar código muerto y caminos sin probar, señalando dónde faltan tests, en lugar de perseguir el número como meta.
> 5. ¿Qué diferencia hay entre `% Branch` y `% Lines`?
>    - R: Lines cuenta líneas ejecutadas; Branch cuenta las ramas de decisión recorridas (ambas mitades de un `if`, por ejemplo), que es más exigente.
> 6. ¿Qué reporter elegirías para inspeccionar el detalle en un navegador?
>    - R: El reporter `html`, que genera un reporte navegable donde ves línea por línea qué se cubrió y qué no.

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.4-vite--el-bundler-y-dev-server-moderno) para el pipeline de Vite sobre el que corre el coverage de Vitest.

---

## 10.9 Testing Libraries (Vitest, Jest, Playwright)

> Referencia base: [Vitest — Guide](https://vitest.dev/guide/) · [Jest — Docs](https://jestjs.io/) · [Playwright — Docs](https://playwright.dev/docs/intro)

### Tres herramientas, tres niveles

- **Vitest**: runner de unit/integración/componentes **nativo de Vite**. API compatible con Jest, ESM nativo, HMR de tests, rápido por defecto. Aunque está optimizado para proyectos frontend con Vite, **también corre código de backend** (entorno `node` por defecto).
- **Jest**: el runner clásico de JavaScript. Corre en **Node.js** y sirve **tanto para frontend** (con `jsdom`) **como para backend** (con `node`). Ecosistema enorme y maduro, pero requiere más configuración para ESM/TypeScript y no integra con Vite.
- **Playwright**: framework de E2E que maneja navegadores reales. No compite con Vitest/Jest: opera en un nivel distinto (la app completa).

*En criollo:* Pensalo como herramientas de distinto piso. Vitest y Jest son para el taller: probar piezas y ensamblajes. Playwright es para la pista: probar el auto entero en movimiento. Ojo con una idea común: **Vitest no es "solo frontend"** — por defecto corre en Node y puede probar backend; lo que tiene es una integración nativa con Vite que lo hace cómodo para frontend moderno. Jest tampoco es "solo frontend": es un runner de Node que usa jsdom para simular el navegador.

*Técnicamente:* Ambos comparten el mismo modelo de entornos: `node` es el **default** en Vitest y en Jest, y `jsdom` (o `happy-dom` en Vitest) es el que activás para emular el navegador cuando probás componentes. Tabla comparativa:

| Herramienta | Nivel | Entornos | Default | Fuerte en |
|-------------|-------|----------|---------|-----------|
| Vitest | Unit, Integración, Componentes, Backend | `node`, `jsdom`, `happy-dom`, `edge-runtime` | `node` | Velocidad, ESM, watch con HMR, integración Vite |
| Jest | Unit, Integración, Componentes, Backend | `node`, `jsdom` | `node` | Madurez, ecosistema, snapshots |
| Playwright | E2E | Navegador real | — | Auto-wait, multi-browser, tracing |

La API de Vitest es deliberadamente casi idéntica a la de Jest, así que migrar de uno a otro es mayormente cambiar la config, no los tests:

```ts
// El mismo test corre casi sin cambios en Vitest y en Jest
import { describe, expect, it } from 'vitest'; // o 'jest'

describe('suma', () => {
  it('suma dos números', () => {
    expect(1 + 2).toBe(3);
  });
});
```

Regla práctica: en un proyecto Vite usá **Vitest** para unit/componentes; usá **Playwright** para los flujos E2E; reservá **Jest** para proyectos/equipos ya atados a su ecosistema.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia fundamental entre Vitest/Jest y Playwright?
>    - R: Vitest/Jest prueban unidades, integraciones y componentes en un entorno simulado; Playwright prueba la app completa en un navegador real.
> 2. ¿Por qué Vitest es "nativo" de Vite y qué ventaja trae?
>    - R: Comparte el pipeline de transformación y config de Vite: ESM nativo, HMR de tests y velocidad sin configuración extra para JSX/TS.
> 3. ¿Cuándo elegirías Jest por sobre Vitest?
>    - R: En proyectos que no usan Vite o equipos con infraestructura y plugins ya montados sobre el ecosistema de Jest.
> 4. ¿Qué tan costoso es migrar de Jest a Vitest?
>    - R: Bajo en código, porque la API (`describe`, `test`, `expect`) es casi idéntica; el trabajo está en la configuración y en plugins específicos de Jest.
> 5. ¿En qué nivel de la pirámide de testing opera Playwright?
>    - R: En la cima, los tests E2E, que son los más lentos y frágiles y se reservan para los flujos críticos.
> 6. ¿Qué significa "auto-wait" como ventaja distintiva de Playwright?
>    - R: Que espera automáticamente a que los elementos estén accionables antes de interactuar, reduciendo tests flaky y la necesidad de sleeps.
> 7. ¿Vitest sirve para probar código de backend o es solo frontend?
>    - R: Sirve para ambos. Su entorno por defecto es `node`, así que puede probar lógica de backend; su integración nativa con Vite es lo que lo hace cómodo para frontend, no una limitación.
> 8. ¿Cuál es el entorno por defecto de Jest y cómo lo cambiás para probar frontend?
>    - R: El default es `node`. Para frontend configurás `testEnvironment: 'jsdom'` (o el docblock `@jest-environment jsdom`), que simula el navegador.

→ Ver [Tópico 10.1: Unit Tests](#10.1-unit-tests) para el nivel que cubren Vitest y Jest.
→ Ver [Tópico 10.4: E2E](#10.4-e2e-end-to-end) para el nivel que cubre Playwright.

---
