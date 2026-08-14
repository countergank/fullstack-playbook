# Setup — Seguridad en Express (helmet + CORS + rate limiting + validación)

> **Tópico**: 12 — Seguridad (secciones 12.1–12.5)
> **Objetivo**: blindar TU API Express con security headers (helmet), CORS con lista blanca, rate limiting, y validación de input — cerrando los vectores OWASP A01, A03, A05 y A07.
> **Prerequisito**: una app Express funcionando con `package.json` propio y un endpoint `GET /` o `GET /health` que responda 200.

---

## ¿Por qué?

La mayoría de los ataques a una API no son exploits sofisticados: son configuraciones ausentes y defaults inseguros (A05). Sin headers de seguridad el navegador no sabe que no debe meter tu app en un iframe (clickjacking) ni hacer MIME-sniffing. Sin CORS restringido, cualquier origen lee tus respuestas. Sin rate limiting, un brute force contra tu login no tiene freno. Sin validación, el input sucio entra directo a tus servicios. Estos middlewares son baratos de agregar y cierran la mayor superficie de ataque antes de escribir una sola línea de negocio.

---

## Checklist

### 1. Crear el proyecto de práctica

- [ ] Creá `~/proyectos/seguridad-practica` con una app Express mínima (o usá tu app existente).
- [ ] `npm install express` y verificá que `GET /` responda 200 con `node server.js`.

### 2. Instalar las dependencias

- [ ] `npm install helmet cors express-rate-limit zod`

### 3. Aplicar helmet (security headers)

- [ ] Agregá `helmet()` como primer middleware. Setea una CSP por defecto y endurecela:

```js
import helmet from 'helmet';

app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", 'https://cdn.confiable.com'],
        styleSrc: ["'self'", "'unsafe-inline'"],
      },
    },
  }),
);
```

> helmet setea por defecto `X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`, `X-XSS-Protection` y `Content-Security-Policy`. Ajustá la CSP a los orígenes reales de tu app.

### 4. Configurar CORS con lista blanca

- [ ] Aplicá CORS con orígenes permitidos explícitos (nunca `*` con credenciales):

```js
import cors from 'cors';

const allowedOrigins = ['https://miapp.com', 'http://localhost:3000'];

app.use(
  cors({
    origin(origin, callback) {
      if (!origin || allowedOrigins.includes(origin)) callback(null, true);
      else callback(new Error('Origen no permitido por CORS'));
    },
    credentials: true,
  }),
);
```

### 5. Agregar rate limiting

- [ ] Limitá los requests globales, y con un límite más estricto para el login:

```js
import rateLimit from 'express-rate-limit';

const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutos
  max: 100,                 // 100 requests por IP por ventana
  standardHeaders: true,
  legacyHeaders: false,
});

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // 5 intentos por IP por ventana → frena brute force
  message: 'Demasiados intentos, probá más tarde',
});

app.use('/api', globalLimiter);
app.post('/api/login', loginLimiter, (req, res) => {
  // …
});
```

### 6. Validar input con Zod

- [ ] Validá todo input en el perímetro (antes del service) con un schema:

```js
import { z } from 'zod';

const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(128),
});

app.post('/api/login', loginLimiter, (req, res) => {
  const parsed = LoginSchema.safeParse(req.body);
  if (!parsed.success) {
    return res.status(400).json({ error: parsed.error.flatten() });
  }
  const { email, password } = parsed.data;
  // recién acá usás los datos validados
});
```

### 7. Verificar los headers

- [ ] Levantá el server y comprobá los headers y los límites (ver sección Verificación).

---

## Verificación

```bash
curl -I http://localhost:3000/          # → Content-Security-Policy, X-Frame-Options, Strict-Transport-Security
curl -i http://localhost:3000/ -H "Origin: https://evil.com"   # → error CORS (origen no permitido)
curl -X POST http://localhost:3000/api/login -H 'Content-Type: application/json' \
  -d '{"email":"no-es-mail","password":"x"}'   # → 400 con detalle de validación
# repetí el POST al login 6 veces seguidas → el 6to devuelve 429 Too Many Requests
```

**Si los headers aparecen, el origen ajeno es rechazado, el input inválido devuelve 400 y el 6to intento de login devuelve 429 → API blindada. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| El frontend deja de cargar scripts de un CDN tras agregar helmet | Agregá el CDN a `scriptSrc` en la CSP en vez de usar solo `'self'` |
| CORS bloquea tu frontend en desarrollo | Agregá `http://localhost:3000` a `allowedOrigins`; no caigas en la tentación de `*` |
| `429` para usuarios legítimos tras agregar rate limiting | Subí `max` o usá un store compartido (Redis) en vez del store en memoria |
| Los headers de helmet no aparecen | Asegurate de que `helmet()` esté ANTES de cualquier ruta (`app.use(helmet())` al inicio) |
| La CSP rompe estilos inline | Agregá `'unsafe-inline'` a `styleSrc` (solo si no podés externalizar los estilos) |
| El rate limit cuenta todos los requests detrás de un proxy | Configurá `app.set('trust proxy', 1)` para que use la IP real del cliente |

---

## Recursos

- [helmet — npm](https://www.npmjs.com/package/helmet)
- [cors — npm](https://www.npmjs.com/package/cors)
- [express-rate-limit — npm](https://www.npmjs.com/package/express-rate-limit)
- [Zod — docs](https://zod.dev/)
- [OWASP — Security Headers](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)

## Preguntas de repaso

- **P:** ¿Qué hace helmet y por qué va como primer middleware?
  **R:** Setea security headers (CSP, X-Frame-Options, Strict-Transport-Security, etc.) que mitigan clickjacking, MIME sniffing y XSS. Va primero para que TODOS los requests y respuestas posteriores pasen por esos headers.

- **P:** ¿Por qué usás una lista blanca de orígenes en CORS en vez de `Access-Control-Allow-Origin: *`?
  **R:** Porque `*` permitiría a cualquier origen leer respuestas que pueden incluir cookies o tokens del usuario. Con credenciales hay que restringir a orígenes exactos.

- **P:** ¿Qué diferencia hay entre el rate limit global y el del login?
  **R:** El global limita el total de requests por IP (protege contra abuso general); el del login tiene un límite mucho más bajo (ej. 5/15min) porque es el objetivo de ataques de fuerza bruta.

- **P:** ¿Por qué validás el input en el perímetro (controller/middleware) y no en el service?
  **R:** Porque el perímetro es la frontera de confianza: el input no validado nunca debería llegar a la lógica de negocio. Validar temprano corta la inyección y simplifica el service.

- **P:** ¿Qué devuelve `safeParse` de Zod y por qué lo preferís a lanzar excepciones?
  **R:** Devuelve `{ success: true, data }` o `{ success: false, error }` sin lanzar. Permite manejar el error de validación limpio en el controller y devolver un 400 con detalle.

- **P:** ¿Cómo frenás un ataque de fuerza bruta contra el login?
  **R:** Con rate limiting estricto en el endpoint de login (pocos intentos por IP por ventana) y, adicionalmente, con bloqueo de cuenta tras N intentos fallidos.
