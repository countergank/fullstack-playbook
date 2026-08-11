# Setup — Fastify + TypeScript

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.3)
> **Objetivo**: crear un proyecto Fastify con TypeScript, validación de schemas nativa, y endpoint de prueba.
> **Prerequisito**: Node (`setup-node.md`).

---

## ¿Por qué Fastify?

Express es el framework más usado, pero no el más rápido. Fastify fue diseñado desde cero con foco en performance: su router basado en radix tree es 2-3x más rápido, la validación con JSON Schema rechaza requests inválidos antes de tocar tu código, y el plugin system con encapsulamiento evita colisiones accidentales. Si empezás un proyecto nuevo en 2026 y valorás velocidad + DX moderno, Fastify es la alternativa más sólida a Express.

---

## Checklist

### 1. Crear el proyecto

```bash
mkdir mi-backend-fastify && cd mi-backend-fastify
npm init -y
```

### 2. Instalar dependencias

```bash
npm install fastify
npm install -D typescript @types/node tsx
```

### 3. Configurar TypeScript

```bash
npx tsc --init
```

```jsonc
// tsconfig.json — mismo que Express
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

### 4. Agregar scripts

```jsonc
// package.json
{
  "scripts": {
    "dev": "tsx watch src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js"
  }
}
```

### 5. Crear el entry point

```ts
// src/app.ts
import Fastify from 'fastify';

const app = Fastify({ logger: true });

// Endpoint de prueba
app.get('/health', async () => {
  return { status: 'ok', timestamp: new Date().toISOString() };
});

// Ruta con validación de schema nativa (el superpoder de Fastify)
app.post('/users', {
  schema: {
    body: {
      type: 'object',
      required: ['name', 'email'],
      properties: {
        name: { type: 'string', minLength: 1 },
        email: { type: 'string', format: 'email' }
      }
    }
  }
}, async (request, reply) => {
  const { name, email } = request.body as { name: string; email: string };
  return { id: '1', name, email };
});

const start = async () => {
  try {
    await app.listen({ port: 3000 });
    console.log('Server running on http://localhost:3000');
  } catch (err) {
    app.log.error(err);
    process.exit(1);
  }
};

start();
```

### 6. Probar

- [ ] `npm run dev` arranca sin errores
- [ ] `curl http://localhost:3000/health` responde `{"status":"ok",...}`
- [ ] `curl -X POST http://localhost:3000/users` con body válido crea un usuario
- [ ] `curl -X POST http://localhost:3000/users` con `{"name": ""}` devuelve 400 automáticamente

---

## Verificación

```bash
curl http://localhost:3000/health | grep "ok"                                       # ok
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" \
  -d '{"name":"","email":"mal"}' -w "\n%{http_code}" | tail -1                      # 400
```

**Si el health responde y la validación devuelve 400 automáticamente → Fastify listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `FastifyError: Method 'GET' already declared` | Revisás que no haya rutas duplicadas. Fastify es estricto: no permite dos handlers para el mismo method+path. |
| Validación no rechaza el request | Verificá que el schema esté dentro del objeto de opciones de la ruta (`app.post('/x', { schema: {...} }, handler)`). Si el schema va fuera, Fastify lo ignora. |
| `Cannot find module 'fastify'` después de `npm install` | Asegurate de estar en el directorio correcto y que `package.json` tenga `fastify` en `dependencies`. Ejecutá `npm ls fastify` para verificar. |
| Logger no muestra output en desarrollo | Fastify usa `logger: true` para activar Pino integrado. Si usás tu propio logger, pasalo como `logger: myLoggerInstance`. |

---

## Preguntas de repaso

- **P:** ¿Cuál es la ventaja principal de Fastify sobre Express?
  **R:** Performance 2-3x superior gracias al radix tree router, validación nativa con JSON Schema, y handlers async sin boilerplate.

- **P:** ¿Cómo funciona la validación de schemas en Fastify?
  **R:** Se define JSON Schema inline en la ruta. Fastify compila el schema con Ajv y rechaza automáticamente requests inválidos con 400 antes de ejecutar el handler.

- **P:** ¿Qué es el encapsulamiento de plugins y por qué importa?
  **R:** Cada plugin tiene su propio scope aislado. Los decorators y hooks no "escapan" al scope global, evitando colisiones. Se puede romper con `fastify-plugin` si se necesita compartir.

- **P:** ¿Fastify reemplaza completamente a Express?
  **R:** No. Fastify es una alternativa. Si tu equipo ya conoce Express o necesitás middlewares específicos de Express, puede convenir quedarse. Fastify brilla en proyectos nuevos.

- **P:** ¿Cómo se manejan los errores en Fastify?
  **R:** Los errores async van automáticamente al error handler. Para errores síncronos, se usa `setErrorHandler` para registrar un handler global que formatea la respuesta.

---

## Fastify vs Express: diferencias visibles en el código

| Aspecto | Express | Fastify |
|---------|---------|---------|
| Crear app | `express()` | `Fastify({ logger: true })` |
| Rutas | `app.get(path, handler)` | `app.get(path, schema?, handler)` |
| Validación | Necesitás Zod aparte | JSON Schema nativo, rechaza automático |
| Logging | Necesitás Pino aparte | Pino integrado (`logger: true`) |
| Async handlers | Manual (try/catch o wrapper) | Nativos — errores van al handler automático |
| Serialización | `res.json()` | Retorno directo, Fastify serializa más rápido |

---

## Recursos

- [Fastify — Getting Started](https://fastify.dev/docs/latest/Guides/Getting-Started/)
- [Fastify — Validation & Serialization](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/)