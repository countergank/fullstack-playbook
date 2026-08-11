# Setup — TypeScript en el Frontend (sin bundler)

> **Tópico**: 7 — Frontend Core
> **Objetivo**: compilar TypeScript a JavaScript plano con `tsc`, tipar tus datos del frontend y cargar el resultado en el navegador sin bundlers.
> **Prerequisito**: Node (`setup-node.md` de 02-programming) y JavaScript (`setup-javascript.md`).
> La compilación con bundlers (Vite, etc.) es del tópico 8 — acá usamos `tsc` directo.

---

## ¿Por qué TypeScript en el frontend?

TypeScript es JavaScript con tipos que **se desvanecen** en runtime: escribís la lógica del frontend con tipos, el compilador valida los errores ANTES de que corran (un `user.name` sobre `undefined` se detecta en el editor, no en producción), y al compilar quedan borrados dejando JS puro que el navegador entiende. Tu HTML **nunca** carga el `.ts`: carga el `.js` compilado.

---

## Checklist

### 1. Inicializar el proyecto e instalar TypeScript

En `~/proyectos/frontend-core` (o donde tenés el proyecto del tópico 7):

```bash
cd ~/proyectos/frontend-core
npm init -y
npm install -D typescript
```

- [ ] `typescript` instalado como dependencia de desarrollo

### 2. Inicializar la configuración de `tsc`

```bash
npx tsc --init
```

Esto genera un `tsconfig.json` con un montón de opciones comentadas. Editálo y dejalo mínimo para el navegador:

```jsonc
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "lib": ["ES2020", "DOM"],
    "moduleResolution": "Node",
    "strict": true,
    "outDir": "dist",
    "rootDir": "src",
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

- `"target": "ES2020"` / `"module": "ES2020"` → JS moderno que el navegador entiende sin bundler, emitido como ES modules.
- `"lib": ["ES2020", "DOM"]` → sin esto, TypeScript no conoce `fetch`, `console` ni el DOM.
- `"strict": true` → el `null`/`undefined` se controla. No te duermas: es la red de seguridad completa.
- `"outDir": "./dist"` / `"rootDir": "./src"` → espejo: `src/app.ts` compila a `dist/app.js`.

- [ ] `tsconfig.json` tiene target, module, outDir y rootDir configurados

### 3. Crear `src/app.ts` con tipos

```ts
// src/app.ts
interface User {
  id: number;
  name: string;
  email: string;
}

const users: User[] = [
  { id: 1, name: 'Leandro', email: 'lean@example.com' },
  { id: 2, name: 'Ana', email: 'ana@example.com' },
];

function findByEmail(email: string): User | undefined {
  return users.find((user) => user.email === email);
}

const nombres: string[] = users.map((user) => user.name);

console.table(users);
console.log('Encontré a:', findByEmail('ana@example.com'));
console.log('Nombres:', nombres);
```

- [ ] `src/app.ts` existe con una `interface`, un array tipado y una función con tipos

### 4. Compilar a `dist/app.js`

```bash
npx tsc
```

Verificá el output — los tipos DESAPARECIERON:

```bash
cat dist/app.js
```

- [ ] `dist/app.js` existe y no contiene ningún tipo (es JS plano)

### 5. Compilar en watch mode

Agregá scripts a `package.json`:

```jsonc
{
  "scripts": {
    "build": "tsc",
    "watch": "tsc --watch"
  }
}
```

```bash
npm run watch
```

Cada vez que guardás `src/app.ts`, `tsc --watch` recompila al toque.

- [ ] `npm run watch` recompila automáticamente al guardar

### 6. Cargar el JS compilado en el HTML

El navegador carga EL PRODUCTO COMPILADO, jamás el `.ts`:

```html
<script type="module" src="dist/app.js"></script>
```

> **Por qué nunca `.ts`**: el navegador no entiende tipos. TypeScript los borra en compilación. Esa es la historia completa: tipo = red de seguridad en DEV, cero costo en runtime.

- [ ] `index.html` apunta a `dist/app.js` y la consola muestra los datos tipados

---

## Verificación

```bash
# 1. Sin errores de tipos
npx tsc --noEmit

# 2. Compilación limpia
npm run build
```

**Si `npx tsc --noEmit` no reporta errores Y la consola del navegador muestra los datos tipados (la tabla + "Encontré a: ...") → TypeScript listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Cannot find name 'fetch'/'console'" | Falta agregar `"lib": ["ES2020", "DOM"]` al tsconfig |
| `npm run watch` deja un proceso corriendo | Esa es la idea: tenés watch en una terminal y editás en otra |
| La página da 404 de `dist/app.js` | Corré `npx tsc` (o `npm run build`) ANTES de abrir el HTML: si no compilaste, `dist/` no existe |
| `tsc --init` genera sinsentido de configs | Editá manualmente el archivo y dejalo como el del paso 2 — las opciones comentadas se pueden borrar |
| Los tipos se "pierden" | No se pierden, se borran a propósito: el navegador corre JS plano |

---

## Recursos

- [TypeScript — get started](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html)
- [TypeScript — tsconfig reference](https://www.typescriptlang.org/tsconfig/)
- [MDN — ES modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

## Preguntas de repaso

- **P:** ¿Qué pasa con los tipos de TypeScript cuando el código corre en el navegador?
  **R:** Desaparecen. TypeScript hace "type erasure" en compilación: borra todas las anotaciones de tipo y emite JavaScript puro. Los tipos solo existen en tiempo de desarrollo.

- **P:** ¿Por qué necesitás `"lib": ["ES2020", "DOM"]` en el tsconfig?
  **R:** Sin `"DOM"`, TypeScript no conoce `fetch`, `document`, `console` ni las Web APIs. Sin `"ES2020"`, no reconoce métodos modernos de arrays como `flat()` o `Promise.allSettled`.

- **P:** ¿Qué hace `strict: true` en el tsconfig?
  **R:** Activa todas las opciones estrictas: `strictNullChecks` (null/undefined no son asignables a otros tipos), `noImplicitAny` (prohíbe tipos `any` implícitos), y otras. Es la red de seguridad completa.

- **P:** ¿Por qué el HTML carga `dist/app.js` y nunca `src/app.ts`?
  **R:** Porque el navegador no entiende TypeScript. `tsc` compila el `.ts` a `.js` borrando los tipos. El HTML siempre carga el producto compilado.

- **P:** ¿Qué diferencia hay entre `outDir` y `rootDir` en el tsconfig?
  **R:** `rootDir` es la carpeta de entrada donde están los `.ts` (ej: `src`). `outDir` es la carpeta de salida donde se generan los `.js` compilados (ej: `dist`). La estructura de carpetas se replica.