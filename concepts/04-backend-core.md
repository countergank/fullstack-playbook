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

---

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

---

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

---

## 4.4 Manejo de Errores

*En criollo:* Un backend profesional no "deja que los errores exploten". Los captura, los entiende, y responde con el código HTTP correcto y un mensaje claro. El usuario no debería ver jamás un 500 con stack trace crudo.

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

---

## 4.5 Validación

*En criollo:* La validación es el primer portero que todo dato externo debe cruzar. NUNCA confíes en lo que el cliente te manda — el cliente puede ser un robot, un atacante, o un frontend bugueado. Cada campo que entra a tu API se valida antes de tocar la base.

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

---

## 4.6 Autenticación y Autorización

*En criollo:* **Autenticación** responde "¿quién sos?" (verificás credenciales → emitís un pase). **Autorización** responde "¿qué se te permite hacer con ese pase?" (revisás roles/recursos). Son DOS pasos distintos y un junior que los confunde crea agujeros de seguridad.

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

---

## 4.7 File Upload y Recursos Estáticos

- **`multer`** (Express) para manejar `multipart/form-data`.
- Límites obligatorios: tamaño máximo, tipos permitidos (MIME), validación de extensión Y magic bytes (no solo la extensión).
- **Nunca guardar archivos en la misma instancia del servidor** en producción — guardar en object storage (S3, Cloudinary) o al menos en un volumen separado. Un proceso que se re-deploya pierde los archivos locales.
- File upload grande → streams, no cargar todo en memoria.

---

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

---

> **Check de comprensión**:
> 1. ¿Por qué un loop síncrono pesado en Node congela a TODOS los usuarios simultáneamente?
> 2. ¿Cuál es la diferencia entre un Controller y un Service? ¿Dónde vive la regla "no se puede registrar dos emails iguales"?
> 3. ¿Qué pasaría si un endpoint de error devuelve `{ error: "Email existe" }` y otro devuelve `{ message: "pass incorrecta" }`? ¿Qué problemas le causa al frontend?
> 4. ¿Por qué un JWT robado es peligroso aunque no exponga el password? ¿Cómo se mitiga?
> 5. ¿Qué capa debería validar que el `email` del body es un email válido? ¿Y qué capa decide si el recurso existe?