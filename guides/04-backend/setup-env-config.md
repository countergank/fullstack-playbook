# Setup — Variables de Entorno y Configuración

> **Tópico**: 4 — Backend Core (sección 4.4, 4.5)
> **Objetivo**: manejar variables de entorno de forma segura y tipada, con validación al arranque para que los errores de configuración fallen ANTES de recibir tráfico.
> **Prerequisito**: proyecto Express creado (`setup-express-project.md`).

---

## Checklist

### 1. Crear `.env` y `.env.example`

```bash
# .env — NUNCA se commitea (tiene secretos reales)
PORT=3000
DATABASE_URL="postgresql://dev:dev@localhost:5432/fullstack_dev"
JWT_SECRET="cambiar-por-algo-seguro-en-produccion"

# .env.example — SÍ se commitea (sirve de plantilla para otros devs)
PORT=3000
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"
JWT_SECRET="generar-con-openssl-rand-hex-64"
```

- [ ] `.env` está en `.gitignore` (ya debería estarlo con el `.gitignore_global` de `setup-git.md`)

### 2. Instalar dotenv

```bash
npm install dotenv
```

### 3. Validar configuración con Zod (falla al arranque, no en runtime)

```bash
npm install zod
```

```ts
// src/config/env.ts
import 'dotenv/config';
import { z } from 'zod';

const envSchema = z.object({
  PORT: z.coerce.number().int().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32, 'JWT_SECRET must be at least 32 characters'),
});

const parsed = envSchema.safeParse(process.env);

if (!parsed.success) {
  console.error('❌ Invalid environment variables:');
  console.error(parsed.error.flatten().fieldErrors);
  process.exit(1);
}

export const env = parsed.data;
```

### 4. Usar `env` en vez de `process.env` en todo el código

```ts
// src/app.ts
import { env } from './config/env.js';

const app = express();
// ...
app.listen(env.PORT, () => console.log(`Server on http://localhost:${env.PORT}`));
```

- [ ] Buscar y reemplazar todos los `process.env.X` por `env.X`

### 5. Generar un JWT_SECRET seguro

```bash
openssl rand -hex 64
```

- [ ] Copiá el output a `.env` como `JWT_SECRET`

---

## Verificación

```bash
# 1. Con .env correcto
npm run dev   # arranca sin errores

# 2. Borrá JWT_SECRET del .env y volvé a arrancar
npm run dev   # debe mostrar "Invalid environment variables" y NO arrancar
```

**Si el server arranca con `.env` válido y FALLA con `.env` inválido → configuración lista. ✅**

---

## Reglas de oro

- `.env` NUNCA se commitea. Secretos en producción van en variables de entorno del sistema o un secret manager.
- `.env.example` SÍ se commitea, con valores de ejemplo (nunca reales).
- La validación al arranque (fail-fast) es INFINITAMENTE mejor que un "Cannot read property of undefined" en producción a las 3 AM.
- Usar `env` tipado en todo el código elimina la clase entera de bugs por typos en nombres de variables de entorno.

---

## Recursos

- [dotenv](https://github.com/motdotla/dotenv)
- [Zod](https://zod.dev/)