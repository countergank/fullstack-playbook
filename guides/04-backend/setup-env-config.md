# Setup — Variables de Entorno y Configuración

> **Tópico**: 4 — Backend Core (sección 4.4, 4.5)
> **Objetivo**: manejar variables de entorno de forma segura y tipada, con validación al arranque para que los errores de configuración fallen ANTES de recibir tráfico.
> **Prerequisito**: proyecto Express creado (`setup-express-project.md`).

---

## ¿Por qué validar variables de entorno al arranque?

Las variables de entorno son la configuración que tu app necesita para funcionar: puerto, URL de la base de datos, secretos. Si una variable falta o tiene un valor inválido, tu app va a fallar — la pregunta es CUÁNDO. Sin validación, el error aparece en el peor momento: cuando un usuario hace un request y el server crashea con un `Cannot read property of undefined`. Con validación al arranque (fail-fast), el error aparece ANTES de recibir tráfico, en la terminal del dev o en los logs del deploy. Es infinitamente mejor que el server no arranque a que arranque y falle silenciosamente.

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

---

## Problemas comunes

| Problema | Causa | Solución |
|---|---|---|
| `Cannot read property of undefined` al usar `process.env` | No estás usando el objeto `env` tipado | Importá `env` desde `./config/env.js` y usá `env.PORT` en vez de `process.env.PORT` |
| El server arranca pero `DATABASE_URL` es `undefined` | Falta la variable en `.env` o el schema de Zod no la declaró | Verificá que la variable esté en `.env` y que `envSchema` la incluya con `z.string()` |
| `JWT_SECRET` muy corto, Zod lo rechaza | `.min(32)` en el schema exige mínimo 32 caracteres | Generá un nuevo secreto con `openssl rand -hex 64` y actualizá `.env` |
| `.env` se subió al repo por accidente | `.gitignore` no incluye `.env` o se commiteó antes de agregarlo | Agregá `.env` al `.gitignore`, ejecutá `git rm --cached .env`, rotá todos los secretos |
| Error de tipo en `env.PORT` (string vs number) | `z.coerce.number()` convierte automáticamente, pero si falla tira error | Usá `z.coerce.number().int().default(3000)` para tener un fallback seguro |

---

## Preguntas de repaso

- **P:** ¿Por qué es mejor que el server FALLE al arrancar en vez de fallar cuando un usuario hace un request?
**R:** Porque un error al arrancar (fail-fast) te avisa inmediatamente en la terminal o en los logs del deploy, antes de recibir tráfico. Si falla en runtime con un request real, el usuario ve un error 500 y vos te enterás por los logs de producción, posiblemente cuando ya hay varios usuarios afectados. Es el principio de "fallá temprano, fallá fuerte".

- **P:** ¿Qué diferencia hay entre `.env` y `.env.example`? ¿Cuál se commitea y por qué?
**R:** `.env` contiene valores reales (secretos, contraseñas) y NUNCA se commitea. `.env.example` es una plantilla con valores de ejemplo que SÍ se commitea para que otros devs sepan qué variables necesita el proyecto sin exponer secretos reales.

- **P:** ¿Qué ventaja tiene usar Zod para validar variables de entorno en vez de chequear `if (!process.env.DATABASE_URL) throw...`?
**R:** Zod te da: (1) validación tipada — `DATABASE_URL` se infiere como `string` (no `string | undefined`), (2) mensajes de error claros y agrupados (`.flatten().fieldErrors`), (3) coerciones automáticas (`z.coerce.number()`), (4) valores por defecto (`.default()`), y (5) un schema declarativo que sirve como documentación viva de lo que tu app necesita.

- **P:** Si necesitás agregar una nueva variable de entorno `REDIS_URL` a un proyecto existente, ¿qué archivos tenés que tocar?
**R:** Cuatro: (1) `.env` — agregar el valor real, (2) `.env.example` — agregar un valor de ejemplo, (3) `src/config/env.ts` — agregar `REDIS_URL: z.string().url()` al schema de Zod, (4) buscar `process.env.REDIS_URL` en el código y reemplazar por `env.REDIS_URL`.

- **P:** ¿Por qué el JWT_SECRET debería tener al menos 32 caracteres? ¿Qué pasa si es más corto?
**R:** Porque la seguridad del JWT depende de la entropía del secreto. Un secreto corto (ej. "secreto123") es vulnerable a ataques de fuerza bruta: un atacante puede probar combinaciones hasta encontrar la que valida tus tokens. 32 caracteres hexadecimales (128 bits de entropía) hacen que el espacio de búsqueda sea astronómicamente grande (~3.4 × 10³⁸ combinaciones), haciendo el ataque inviable.