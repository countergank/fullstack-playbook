# Setup — Fastify + TypeScript

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.3)
> **Objetivo**: crear un proyecto Fastify con TypeScript, validación de schemas nativa, y endpoint de prueba.
> **Prerequisito**: Node (`setup-node.md`).

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

```bash
npm run dev
```

```bash
curl http://localhost:3000/health                    # {"status":"ok",...}
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Lean", "email": "lean@ejemplo.com"}'  # { id, name, email }

# Validación: si mandás mal el body, Fastify responde 400 automáticamente
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": ""}'                                    # 400 con error descriptivo
```

---

## Verificación

```bash
curl http://localhost:3000/health | grep "ok"                                       # ok
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" \
  -d '{"name":"","email":"mal"}' -w "\n%{http_code}" | tail -1                      # 400
```

**Si el health responde y la validación devuelve 400 automáticamente → Fastify listo. ✅**

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