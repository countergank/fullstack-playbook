# 4. Backend Core

> Objetivo: dominar los fundamentos del backend en Node.js — cómo funciona el runtime, cómo estructurar una API REST sólida, y los patrones que separan un backend profesional de uno de tutorial.

---

## 4.1 Node.js Runtime

*En criollo:* Node.js es JavaScript corriendo FUERA del navegador. Le das al motor V8 (el mismo de Chrome) acceso al sistema operativo: archivos, red, procesos. Eso convierte al lenguaje que solo sabía animar botones en un lenguaje de backend con todas las letras.

### Conceptos fundamentales

**Event Loop — el corazón de Node**
- *En criollo:* Node es de hilo único pero no se queda quieto esperando. Cuando pide algo lento (leer un archivo, una query a la base), NO se queda bloqueado mirando el reloj — delega la espera y sigue atendiendo el siguiente request. Cuando la operación lenta termina, avisa y se procesa el resultado. Es como un mozo que toma varios pedidos en vez de esperar a que se cocine cada plato.
- *Técnicamente:* El event loop es un ciclo que procesa microtasks y callbacks en fases (timers → pending → poll → check → close). Las operaciones I/O delegadas al thread pool de libuv (por defecto 4 threads) corren en paralelo real. El código síncrono pesado (CPU-bound, loops largos, JSON.parse gigante) BLOQUEA el loop y congela a TODOS los usuarios.
- **Regla de oro**: el código JavaScript tuyo nunca debe hacer trabajo pesado de CPU. Para eso hay worker threads o se delega a servicios.

**Callbacks → Promesas → Async/Await**
```js
// Callback (viejo, anidamiento = infierno)
fs.readFile('a.txt', (err, data) => {
  fs.readFile('b.txt', (err2, data2) => { /* callback hell */ });
});

// Promesas (mejor, pero verboso)
fs.promises.readFile('a.txt').then(data => ...);

// Async/await (moderno, legible, el estándar HOY)
const a = await fs.promises.readFile('a.txt');
const b = await fs.promises.readFile('b.txt');
```
- Todo lo que es promesa se debe `await` — el error más común de backend junior es no esperar y operar sobre datos que aún no llegaron.
- **Manejo de errores en async**: `try/catch` alrededor de `await`, o `.catch()` en cadenas de promesas. Un `await` sin try/catch y sin catch global = crash o 500 silencioso.

**Módulos: CommonJS vs ESM**
| | CommonJS (`require`) | ESM (`import`) |
|---|---|---|
| Sintaxis | `const x = require('x')` | `import x from 'x'` |
| Carga | Síncrona | Asíncrona |
| Soporte | Node desde siempre | Estándar desde Node 12+, default moderno |
| `package.json` | — | `"type": "module"` |
| Proyectos nuevos | Legado | **Recomendado** |

**V8 Engine and Memory**
- *En criollo:* V8 compila JS a código máquina (JIT). Maneja la memoria con un garbage collector que libera objetos que ya no se usan. No necesitás gestionar memoria manualmente (no hay `malloc`), pero sí entender que los objetos grandes que quedan referenciados jamas se liberan.
- **Memory leak clásico**: listeners sin remover, cachés que crecen sin límite, globales que acumulan datos.

**Cluster y PM2 (escalabilidad básica)**
- Node es single-thread → para usar los múltiples cores, se lanzan varios procesos (clusters) o se escala horizontalmente (múltiples instancias detrás de un load balancer).
- **PM2**: process manager que mantiene tu app viva, reinicia en crash, logs rotados, cluster mode. El estándar para Node en servidores sin Docker/K8s.

> [Node.js — Event Loop Timers and Next Tick](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-next-tick)
> [Node.js — Working with File System](https://nodejs.org/en/learn/manipulating-files/working-with-files)
> [libuv — Thread Pool Documentation](https://docs.libuv.org/en/v1/threadpool.html)

→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.3-asincronía) — asincronía y callbacks.
→ Ver [Tópico 5: Frameworks Backend](../concepts/05-frameworks-backend.md#5.1-express) — Express usa el event loop de Node.

> **Check de comprensión**
> 1. ¿Por qué un `while(true)` síncrono en Node congela TODAS las conexiones simultáneamente?
>    - R: Porque el event loop es single-thread en la capa JS; un loop síncrono impide que pase a la siguiente fase y ningún callback se ejecuta hasta que termine.
> 2. ¿Cuál es la diferencia entre microtasks (Promesas) y macrotasks (setTimeout) en el event loop?
>    - R: Las microtasks se ejecutan inmediatamente después de cada fase del loop antes de pasar a la siguiente; las macrotasks esperan su turno en la fase de timers. Una Promesa resuelta corre antes que un setTimeout de 0ms.
> 3. ¿Por qué `async/await` es preferible a callbacks anidados en código de producción?
>    - R: Porque elimina el callback hell, permite usar try/catch para errores, y el código se lee de forma secuencial y mantenible.
> 4. ¿Qué diferencia hay entre CommonJS y ESM en términos de carga de módulos?
>    - R: CommonJS carga síncronamente (require bloquea hasta que el módulo está listo); ESM carga asíncronamente y soporta top-level await. ESM es el estándar moderno.
> 5. ¿Qué es un memory leak en Node y nombrá 2 causas comunes?
>    - R: Es cuando objetos referenciados nunca se liberan del heap. Causas comunes: event listeners sin remover, cachés que crecen sin límite (Map/Set sin TTL), y variables globales que acumulan datos.

## 4.2 Arquitectura de una API REST

*En criollo:* Una API REST es un cajero automático de datos: le hablás con verbos claros (GET = "dame", POST = "creá", etc.) a URLs que nombran recursos, y te responde JSON. La arquitectura define CÓMO estructuramos ese cajero para que sea mantenible.

### Anatomía de un request/response

**Routing**: mapear `MÉTODO + RUTA` a un handler.
```
GET    /api/users        → listar usuarios
GET    /api/users/:id    → un usuario
POST   /api/users        → crear usuario
PUT    /api/users/:id    → reemplazar usuario
PATCH  /api/users/:id    → modificar parcial
DELETE /api/users/:id    → eliminar usuario
```

**Capas (lo que separa un backend profesional del que "funciona")**:

```
Ruta (router) → Controller → Service → Repository → Modelo/DB
     │              │           │          │
   define el      valida y     lógica de  habla con
   endpoint       orquesta    negocio    el ORM/DB
```

- **Controller**: recibe el request, valida entrada, llama al service, arma la respuesta HTTP (status code, body). NO tiene lógica de negocio.
- **Service**: contiene la lógica de negocio pura. Por ejemplo: "al crear un usuario, verificar email único, hashear password, generar token". NO sabe qué es HTTP.
- **Repository**: abstrae el acceso a datos. El service llama a `userRepo.findByEmail()` sin saber si es PostgreSQL, MongoDB o un archivo JSON.
- **Beneficio**: testear es trivial (cada capa con su responsabilidad), cambiar de base NO toca la lógica de negocio, y el código se lee de arriba hacia abajo sin sorpresas.

### Response JSON — contrato consistente

Un contrato de respuesta bien definido evita que el frontend tenga que adivinar:

```json
// Éxito
{
  "data": { "id": 42, "name": "Lean" }
}

// Error (consistente SIEMPRE)
{
  "error": {
    "message": "Email ya registrado",
    "code": "EMAIL_IN_USE",
    "status": 409
  }
}
```

- **Regla de oro**: el frontend debe poder parsear CUALQUIER error con la misma estructura. Si los errores cambian de forma según el endpoint, el frontend se vuelve un `if` gigante.

### Paginación, filtros y orden

Los endpoints que devuelven listas SIEMPRE deben estar paginados desde el día uno:

```
GET /api/users?page=2&limit=20
GET /api/users?sort=-createdAt
GET /api/users?role=admin&active=true
```

Respuesta de paginación estándar:
```json
{
  "data": [...],
  "meta": {
    "page": 2,
    "limit": 20,
    "total": 154,
    "totalPages": 8
  }
}
```

*Técnicamente:* REST se basa en 6 restricciones de Roy Fielding: cliente-servidor, stateless, cacheable, interfaz uniforme (identificación de recursos, manipulación a través de representaciones, mensajes auto-descriptivos, HATEOAS), sistema en capas y código bajo demanda (opcional). HTTP es el protocolo de transporte: métodos semánticos (GET, POST, PUT, PATCH, DELETE), códigos de estado (1xx-5xx), y headers de control (Cache-Control, Content-Type, Authorization). La paginación offset/limit escala hasta ~100K registros; después se recomienda cursor-based pagination (`?after=cursor`) para evitar `OFFSET` costosos en SQL.

> [Express — Routing](https://expressjs.com/en/guide/routing.html)
> [Node.js — HTTP Server](https://nodejs.org/en/learn/serving-http/serving-http-nodejs)
> [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)

→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.3-http) — protocolo HTTP y verbos.
→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.2-modelado-relacional) — cómo el repository traduce queries SQL.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre PUT y PATCH en términos de idempotencia?
>    - R: PUT es idempotente (reemplaza el recurso completo, mismo resultado si se repite); PATCH modifica parcialmente y NO garantiza idempotencia.
> 2. ¿Por qué separar Controller, Service y Repository mejora la testabilidad?
>    - R: Porque cada capa tiene una responsabilidad única y se puede mockear independientemente: el controller se testa con requests falsos, el service con repositorios mock, y el repository con DB de test.
> 3. ¿Qué problema resuelve la paginación y por qué debe ser obligatoria desde el día uno?
>    - R: Evita devolver miles de registros en una sola respuesta (memoria, red, tiempo). Sin paginación, un endpoint se vuelve lento y puede colapsar el servidor cuando la tabla crece.
> 4. ¿Por qué el frontend necesita un contrato de error consistente en TODOS los endpoints?
>    - R: Porque permite un manejo genérico de errores (un solo interceptor) en vez de lógica custom por endpoint. Si cada endpoint devuelve errores con forma distinta, el frontend necesita `if/else` para cada caso.
> 5. ¿Qué es cursor-based pagination y cuándo es preferible a offset/limit?
>    - R: Es paginación basada en un punto de referencia (ej: `?after=id_123`) en vez de saltar N registros. Es preferible cuando la tabla tiene >100K filas porque evita el `OFFSET` costoso que escanea y descarta rows.

## 4.3 Middlewares

*En criollo:* Un middleware es un "portero" que se ejecuta ANTES de que llegue al handler. Cada request pasa por una fila de porteros: uno verifica autenticación, otro parsea el body, otro loguea, otro mide tiempo. Podés agregar porteros sin tocar el código del endpoint.

*Técnicamente:* En Express, un middleware es una función `(req, res, next) => {}`. Si llama a `next()`, pasa al siguiente; si responde (`res.status(...).json()`), corta la cadena. El orden de registro = orden de ejecución. Los middlewares se dividen en:

**Built-in (Express)**
- `express.json()` — parsea el body JSON entrante.
- `express.urlencoded({ extended: true })` — parsea formularios.

**De terceros (ecosistema Express)**
- `cors()` — habilita/limita Cross-Origin Resource Sharing.
- `helmet()` — headers de seguridad automáticos (CSP, X-Frame-Options, etc.).
- `morgan` / `pino-http` — logging de requests.
- `express-rate-limit` — protege contra brute force/DoS limitando peticiones por IP.

**Propios (los que escribís vos)**
```js
// Autenticación: verifica el token y adjunta el usuario al request
async function auth(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: { message: 'No token' } });
  try {
    req.user = await verifyJwt(token);
    next();
  } catch {
    return res.status(401).json({ error: { message: 'Token inválido' } });
  }
}

// Manejo de errores: DEBE tener 4 parámetros para que Express lo detecte
function errorHandler(err, req, res, next) {
  console.error(err);
  const status = err.status || 500;
  res.status(status).json({ error: { message: err.message, code: err.code } });
}
```

**Error middleware**: si un middleware llama `next(err)` con un error, Express salta TODOS los middlewares normales y va directo al middleware de errores (el de 4 parámetros). Ahí centralizás logging + respuesta uniforme.

> [Express — Using Middleware](https://expressjs.com/en/guide/using-middleware.html)
> [OWASP — API Security Top 10](https://owasp.org/www-project-api-security/)

→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.8-cors) — el middleware `cors()` implementa CORS.
→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.1-sql) — cómo el repository layer se conecta desde el service.

> **Check de comprensión**
> 1. ¿Por qué el orden de registro de middlewares en Express es crítico?
>    - R: Porque Express los ejecuta en el orden exacto en que se registran con `app.use()`. Si el auth middleware va después del route handler, el handler se ejecuta sin autenticación.
> 2. ¿Qué diferencia hay entre un middleware que llama `next()` y uno que responde con `res.json()`?
>    - R: `next()` pasa al siguiente middleware en la cadena; `res.json()` envía la respuesta y corta la cadena — ningún middleware posterior se ejecuta.
> 3. ¿Por qué el error middleware necesita 4 parámetros `(err, req, res, next)` y no 3?
>    - R: Porque Express detecta middlewares de error por la cantidad de parámetros. Con 3 parámetros es un middleware normal; con 4 Express sabe que es para manejo de errores y solo lo invoca con `next(err)`.
> 4. ¿Qué hace `helmet()` y por qué es esencial en producción?
>    - R: Setea headers de seguridad HTTP como Content-Security-Policy, X-Frame-Options, X-Content-Type-Options. Previene ataques como clickjacking, XSS por MIME sniffing, y otros vectores comunes.
> 5. ¿Qué pasa si un middleware de autenticación no llama ni a `next()` ni a `res.status()`?
>    - R: El request queda colgado (hung). El cliente espera indefinidamente hasta timeout porque la cadena se detuvo sin respuesta ni continuación.

## 4.4 Manejo de Errores

*En criollo:* Un backend profesional no "deja que los errores exploten". Los captura, los entiende, y responde con el código HTTP correcto y un mensaje claro. El usuario no debería ver jamás un 500 con stack trace crudo.

*Técnicamente:* El manejo de errores en Express se centraliza con un middleware de 4 parámetros al final de la cadena. Las clases custom (`AppError`, `NotFoundError`) permiten propagar contexto (status, code, message) desde el service hasta el handler sin perder información. En producción, los errores se loguean con pino/winston en formato JSON con requestId para tracing distribuido. Los errores no manejados en Promesas se capturan con `process.on('unhandledRejection')` para evitar crashes silenciosos.

### Errores esperados vs inesperados
- **Esperados** (validación falló, email duplicado, recurso no encontrado): se detectan en el código, se responde 400/401/403/404/409 con mensaje claro.
- **Inesperados** (DB se cayó, bug real): se LOGUEAN con detalle (stack trace, request info), se responde 500 genérico sin exponer detalles internos.

### Códigos HTTP correctos (no abusar del 500)
| Situación | Status | Body |
|-----------|--------|------|
| Recurso no existe | 404 | `{ message: 'User 42 not found' }` |
| Payload inválido | 400 / 422 | Detalle del campo que falló |
| Token faltante/inválido | 401 | `{ message: 'Unauthorized' }` |
| Sin permisos (autenticado pero no autorizado) | 403 | `{ message: 'Forbidden' }` |
| Conflicto (email duplicado) | 409 | `{ message: 'Email already in use' }` |
| Algo inesperado | 500 | `{ message: 'Internal server error' }` (sin stack trace) |

### Clases de error custom (en vez de strings sueltos)
```js
class AppError extends Error {
  constructor(status, code, message) {
    super(message);
    this.status = status;
    this.code = code;
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(404, 'NOT_FOUND', `${resource} not found`);
  }
}

class ConflictError extends AppError {
  constructor(message) {
    super(409, 'CONFLICT', message);
  }
}

// Uso en el service:
throw new NotFoundError('User');
// El error middleware lo convierte en respuesta HTTP
```

### Logging estructurado
- Usá **pino** (JSON nativo, rápido) o **winston** (más plugins). No `console.log` suelto.
- Logs por nivel: `fatal`, `error`, `warn`, `info`, `debug`, `trace`.
- Cada log con contexto: `requestId`, `userId`, `route`, `durationMs`. Así un error en producción se rastrea de punta a punta.

> [Node.js — Errors](https://nodejs.org/en/learn/getting-started/error-handling)
> [Express — Error Handling](https://expressjs.com/en/guide/error-handling.html)

→ Ver [Tópico 5: Frameworks Backend](../concepts/05-frameworks-backend.md#5.1-express) — middlewares de error en Express.
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.3-http) — códigos de estado HTTP.

> **Check de comprensión**
> 1. ¿Por qué NUNCA debés exponer el stack trace en una respuesta 500 de producción?
>    - R: Porque revela la estructura interna del código, versiones de librerías, y paths del servidor. Un atacante usa esa información para identificar vulnerabilidades específicas.
> 2. ¿Cuál es la diferencia entre un error esperado (400, 404, 409) y uno inesperado (500) en términos de manejo?
>    - R: Los esperados se manejan con respuesta clara al cliente (el problema es del request); los inesperados se loguean con detalle internamente pero se responde genérico al cliente (el problema es del servidor).
> 3. ¿Por qué usar clases de error custom (`NotFoundError`, `ConflictError`) en vez de strings o números sueltos?
>    - R: Porque permiten propagar status, código y mensaje en un solo objeto, y el error middleware los convierte automáticamente en la respuesta HTTP correcta sin lógica condicional dispersa.
> 4. ¿Qué captura `process.on('unhandledRejection')` y por qué es necesario?
>    - R: Captura Promesas rechazadas que no tienen `.catch()` ni están dentro de un `try/catch`. Sin este handler, Node.js puede crashear o silenciar el error dependiendo de la versión.
> 5. ¿Qué información debe incluir un log estructurado de error para ser útil en producción?
>    - R: requestId (para trazar el request completo), userId, ruta, método HTTP, status code, mensaje de error, stack trace (solo en log interno), timestamp, y duración de la request.

## 4.5 Validación

*En criollo:* La validación es el primer portero que todo dato externo debe cruzar. NUNCA confíes en lo que el cliente te manda — el cliente puede ser un robot, un atacante, o un frontend bugueado. Cada campo que entra a tu API se valida antes de tocar la base.

*Técnicamente:* La validación con schemas (Zod, Joi) define tipos y restricciones declarativas. `safeParse()` devuelve `{ success: true, data }` o `{ success: false, error }` sin lanzar excepciones, lo que permite manejo limpio en el controller. Los schemas son composables (`z.object().extend()`) y generan tipos TypeScript automáticamente (`z.infer<typeof schema>`). La sanitización (trim, escape) previene XSS e inyección. La validación debe ocurrir en el perímetro (controller/middleware), no en el service.

### Qué validar
- **Presencia**: campos obligatorios.
- **Tipo**: string, number, email, uuid, date.
- **Formato**: regex (email, phone, URL), longitud min/max.
- **Valores**: enums (`role: ['admin', 'user']`), rangos numéricos.
- **Referencias**: que el `userId` que te pasan exista (integridad).
- **Sanitización**: trim, lowercase donde aplique, escapado.

### Herramientas modernas
| Librería | Enfoque |
|----------|---------|
| **Zod** | Schema validation con TypeScript nativo. Tipos derivados automáticamente. El estándar moderno. |
| **Yup** | Similar a Zod, popular en frontend (Formik lo usa). |
| **Joi** | Madura, great en Express clásico. |
| **Valibot** | Liviana, tree-shakeable, alternativa moderna a Zod. |

### Ejemplo con Zod
```js
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
  role: z.enum(['admin', 'user']).default('user'),
  age: z.number().int().min(0).max(120).optional(),
});

// En el controller:
const parsed = createUserSchema.safeParse(req.body);
if (!parsed.success) {
  return res.status(422).json({
    error: { message: 'Validation failed', issues: parsed.error.issues }
  });
}
```

**Regla de oro**: validás en el CONTORNO (ál linde del sistema), no en el interior. El service recibe datos ya válidos y tipados.

> [Zod — Documentation](https://zod.dev/)
> [OWASP — Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.5-tipos-de-datos) — tipos y validación de datos.
→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.1-sql) — integridad referencial en la base.

> **Check de comprensión**
> 1. ¿Por qué la validación debe ocurrir en el "contorno" del sistema y no en el service?
>    - R: Porque el service asume datos válidos; validar adentro mezcla responsabilidades y duplica lógica. El contorno (controller/middleware) es el punto de entrada único donde todo dato externo se filtra.
> 2. ¿Qué ventaja tiene `safeParse()` de Zod sobre `parse()` directo?
>    - R: `safeParse()` no lanza excepciones — devuelve un objeto con `success: true/false`. Esto permite manejar errores de validación como flujo normal (422) en vez de try/catch para errores esperados.
> 3. ¿Qué diferencia hay entre validación y sanitización?
>    - R: Validación verifica que el dato cumple las reglas (es email, tiene longitud correcta); sanitización transforma el dato para que sea seguro (trim de espacios, escape de HTML, lowercase).
> 4. ¿Por qué validar solo la extensión de un archivo subido es insuficiente?
>    - R: Porque un atacante puede renombrar un `.js` a `.jpg`. La validación real requiere verificar los magic bytes (primeros bytes del archivo) y el MIME type real, no solo la extensión.
> 5. ¿Qué es la validación de referencia y por qué es distinta de la validación de formato?
>    - R: Validar formato verifica que un `userId` sea un UUID válido; validar referencia verifica que ese UUID exista en la base de datos. La primera es sintáctica, la segunda requiere una query.

## 4.6 Autenticación y Autorización

*En criollo:* **Autenticación** responde "¿quién sos?" (verificás credenciales → emitís un pase). **Autorización** responde "¿qué se te permite hacer con ese pase?" (revisás roles/recursos). Son DOS pasos distintos y un junior que los confunde crea agujeros de seguridad.

*Técnicamente:* JWT (RFC 7519) tiene 3 partes: header (`{"alg":"HS256","typ":"JWT"}`), payload (claims como `sub`, `iat`, `exp`), y signature (`HMACSHA256(base64(header) + "." + base64(payload), secret)`). La firma garantiza integridad (si alguien modifica el payload, la firma no coincide). bcrypt aplica un salt automático y un work factor (cost) que determina iteraciones de hashing — factor 12 = ~250ms en hardware moderno. Argon2 (ganador del Password Hashing Competition 2015) es resistente a ataques GPU y side-channel.

### Modelos de autenticación

**Session-based (cookie session)**
- Servidor guarda la sesión en memoria o DB, manda un cookie de sesión al cliente.
- Simple de revocar (borrar del servidor). Stateful.
- Requiere sticky sessions o sesión compartida en multi-instancia.

**Token-based (JWT) — el estándar moderno para APIs**
- Servidor emite un JWT firmado; el cliente lo manda en `Authorization: Bearer <token>`.
- Stateless — el servidor NO guarda nada. Verifica la firma y lee el payload.
- JWT = `header.payload.signature` (base64url). La firma usa HMAC (simétrica) o RSA/Ed25519 (asimétrica).
- **Riesgo**: un JWT robado es válido hasta su expiración. Mitigar con expiración corta (15-60 min) + refresh tokens + revocation list si hace falta.

**OAuth 2.0 / OpenID Connect**
- Delegás autenticación a un tercero (Google, GitHub). OIDC agrega el `id_token` y el discovery.
- **No implementes OAuth desde cero** — usá librerías de confianza (Auth.js, Passport, o providers como Auth0/Keycloak).

### Hashing de contraseñas (NUNCA guardes passwords en texto plano)
```js
import bcrypt from 'bcryptjs';

// Al crear: hash con salt automático
const hash = await bcrypt.hash(password, 12);

// Al login: comparar
const ok = await bcrypt.compare(password, user.passwordHash);
```
- **bcrypt** (con factor 10-12) o **argon2** (aún más resistente a GPU). NUNCA MD5, SHA1, SHA256 simple (son hash rápidos = fáciles de romper).
- El hash con salt hace que dos usuarios con la misma password tengan hashes distintos.

### Autorización — RBAC
```js
// Middleware de roles
const requireRole = (...roles) => (req, res, next) => {
  if (!req.user) return res.status(401).json({ error: { message: 'Unauthorized' } });
  if (!roles.includes(req.user.role)) return res.status(403).json({ error: { message: 'Forbidden' } });
  next();
};

// Uso
router.post('/admin/users', auth, requireRole('admin'), createUser);

// Owner check (más fino que rol: el usuario puede editar SU recurso)
async function requireOwner(req, res, next) {
  const resource = await getResource(req.params.id);
  if (resource.ownerId !== req.user.id) return res.status(403).json(...);
  next();
}
```

### Refresh tokens (keep the user logged in)
- Access token: corto (15 min), se manda en cada request.
- Refresh token: largo (7-30 días), SOLO se usa para pedir un access token nuevo.
- El refresh token viaja en cookie `HttpOnly` para no exponerlo al JS del cliente.

> [Node.js — Crypto (HMAC)](https://nodejs.org/api/crypto.html)
> [JWT — Introduction](https://jwt.io/introduction)
> [OWASP — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
> [bcrypt — npm](https://www.npmjs.com/package/bcrypt)

→ Ver [Tópico 3: IA y Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.4-seguridad) — riesgos de seguridad con IA y credenciales.
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.9-https-tls) — HTTPS y TLS para proteger tokens en tránsito.

> **Check de comprensión**
> 1. ¿Por qué un JWT robado es válido hasta su expiración aunque el usuario cambie su password?
>    - R: Porque JWT es stateless — el servidor solo verifica la firma, no consulta una lista de tokens válidos. Si el token fue firmado correctamente y no expiró, se acepta.
> 2. ¿Cuál es la diferencia entre autenticación session-based y token-based en términos de escalabilidad?
>    - R: Session-based requiere storage compartido (Redis, DB) entre instancias para verificar sesiones; token-based (JWT) es stateless y escala horizontalmente sin infraestructura adicional.
> 3. ¿Por qué usar bcrypt o argon2 en vez de SHA256 para passwords?
>    - R: SHA256 es rápido (~millones de hashes/segundo en GPU), ideal para brute force. bcrypt y argon2 son intencionalmente lentos y consumen memoria, haciendo ataques de fuerza bruta inviables.
> 4. ¿Qué es RBAC y cómo se implementa en Express?
>    - R: Role-Based Access Control. Se implementa como middleware que verifica `req.user.role` contra una lista de roles permitidos para la ruta. Si el rol no está en la lista, responde 403.
> 5. ¿Por qué el refresh token viaja en cookie `HttpOnly` y no en el body o header?
>    - R: Porque `HttpOnly` impide que JavaScript del cliente acceda a la cookie, protegiendo contra XSS. Si viaja en body/header, cualquier script malicioso podría robarlo.

## 4.7 File Upload y Recursos Estáticos

*En criollo:* Subir archivos es como recibir paquetes por correo: no aceptás cualquier cosa sin revisar. Validás que sea el tipo correcto, que no sea gigante, y lo guardás en un lugar seguro (no en tu escritorio). En producción, los archivos van a un servicio especializado (S3, Cloudinary), no al mismo servidor que corre tu app.

*Técnicamente:* `multipart/form-data` es el encoding que permite enviar archivos binarios junto con campos de texto en un solo POST. `multer` parsea el stream multipart y escribe los archivos en disco o memoria. Los magic bytes son los primeros bytes de un archivo que identifican su tipo real (ej: PNG empieza con `89 50 4E 47`). Validar solo la extensión es vulnerable: un `.exe` renombrado a `.jpg` pasa la extensión pero no los magic bytes. En producción, usar streams (`multer` con `dest` o `storage`) evita cargar archivos completos en memoria RAM.

> [Express — Static Files](https://expressjs.com/en/starter/static-files.html)
> [Multer — npm](https://www.npmjs.com/package/multer)
> [OWASP — File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)

→ Ver [Tópico 5: Frameworks Backend](../concepts/05-frameworks-backend.md#5.1-express) — middleware `multer` en Express.
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.8-cors) — CORS y políticas de origen para uploads.

> **Check de comprensión**
> 1. ¿Por qué validar solo la extensión de un archivo es insuficiente para seguridad?
>    - R: Porque un atacante puede renombrar cualquier archivo para que tenga una extensión válida. Los magic bytes (primeros bytes del archivo) revelan el tipo real independientemente del nombre.
> 2. ¿Por qué no se deben guardar archivos subidos en la misma instancia del servidor en producción?
>    - R: Porque un redeploy, crash o escalado horizontal pierde los archivos locales. Object storage (S3) es persistente, escalable y separado del ciclo de vida del servidor.
> 3. ¿Qué es `multipart/form-data` y por qué se usa para file uploads?
>    - R: Es un encoding HTTP que permite enviar múltiples partes (campos de texto + archivos binarios) en un solo request. Cada parte tiene su propio Content-Type y boundary delimiter.
> 4. ¿Por qué usar streams en vez de cargar el archivo completo en memoria?
>    - R: Porque un archivo de 100MB cargaría 100MB de RAM por request. Con streams, se procesa en chunks pequeños y la memoria se mantiene constante independientemente del tamaño.
> 5. ¿Qué límite de tamaño máximo debería tener un file upload y por qué?
>    - R: Depende del caso (avatars: 2-5MB, documentos: 10-25MB). Sin límite, un atacante puede enviar GB de datos y agotar memoria/disco (DoS). El límite se configura en multer (`limits: { fileSize: 5 * 1024 * 1024 }`).

## 4.8 GraphQL (perspectiva fundamental)

*En criollo:* REST te da endpoints fijos (una URL por recurso). GraphQL te da UN solo endpoint donde el CLIENTE pide exactamente los datos que necesita en forma de query. Menos over-fetching y under-fetching, a cambio de más complejidad en el servidor (schemas, resolvers, cacheo).

| Aspecto | REST | GraphQL |
|---------|------|---------|
| Endpoints | Múltiples (`/users`, `/users/:id/posts`) | Uno solo (`/graphql`) |
| Datos | Fijos por endpoint | El cliente elige campos |
| Over-fetching | Común (recibís de más) | No existe |
| Caching HTTP | Natural (por URL) | Complejo (hay que resolver) |
| Aprendizaje | Más simple | Más complejo |
| Uso típico | APIs públicas, CRUD, mobile simple | Productos complejos, múltiples apps consumiendo |

**Recomendación**: dominá REST primero (es el 90% de las APIs del mundo). GraphQL como segundo idioma cuando el producto lo pida.

*Técnicamente:* GraphQL usa un schema definido en SDL (Schema Definition Language) con tipos, queries, mutations y subscriptions. Los resolvers son funciones que resuelven cada campo del schema. El problema N+1 ocurre cuando un resolver hace una query por cada item de una lista; se mitiga con DataLoader (batching + caching). GraphQL no usa caching HTTP nativo porque todas las requests van a `/graphql` con POST, pero Apollo Client y Relay implementan caching en el cliente con normalización de respuestas.

> [GraphQL — Learn](https://graphql.org/learn/)
> [Apollo Server — Documentation](https://www.apollographql.com/docs/apollo-server/)

→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.4-estructuras-de-datos) — el problema N+1 como patrón de acceso ineficiente.
→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.4-redis) — caching con Redis como alternativa al caching de GraphQL.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia principal entre REST y GraphQL en términos de flexibilidad del cliente?
>    - R: En REST el servidor define qué datos devuelve cada endpoint; en GraphQL el cliente especifica exactamente qué campos necesita en cada query, evitando over-fetching y under-fetching.
> 2. ¿Qué es el problema N+1 en GraphQL y cómo se resuelve?
>    - R: Ocurre cuando un resolver hace una query a la base por cada item de una lista (N queries + 1 inicial). Se resuelve con DataLoader, que batchea todas las requests en una sola query.
> 3. ¿Por qué el caching HTTP es más complejo en GraphQL que en REST?
>    - R: Porque REST usa URLs como identificadores de recursos (cacheable por URL); GraphQL usa un solo endpoint POST para todo, así que no hay URLs distintas para cachear. Se resuelve con caching en el cliente (Apollo, Relay).
> 4. ¿Qué son las subscriptions en GraphQL y para qué caso de uso sirven?
>    - R: Son queries en tiempo real que usan WebSockets para push de datos del servidor al cliente. Sirven para chat, notificaciones, dashboards en vivo — datos que cambian frecuentemente.
> 5. ¿Por qué se recomienda aprender REST antes que GraphQL?
>    - R: Porque REST es más simple, más usado (90% de APIs), y enseña los fundamentos de APIs (routing, status codes, recursos). GraphQL añade complejidad (schemas, resolvers, N+1) que se entiende mejor con base REST.