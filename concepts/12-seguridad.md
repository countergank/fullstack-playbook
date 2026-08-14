# 12. Seguridad

> Objetivo: entender los vectores de ataque más comunes en aplicaciones web y las defensas concretas para cerrarlos — no por paranoia, sino porque la seguridad es parte del diseño, no un parche que se pega al final.

---

## 12.1 OWASP Top 10

> Referencia base: [OWASP Top 10](https://owasp.org/Top10/) · [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)

*En criollo:* OWASP (Open Worldwide Application Security Project) es la organización de referencia en seguridad web. Cada pocos años publica una lista con los **diez riesgos más críticos** que afectan a las aplicaciones web reales. No es teoría: está armada con datos de vulnerabilidades reportadas por empresas de todo el mundo. La usás como checklist de "no me olvidé de nada" cuando diseñás una API, un form, o cualquier endpoint que toque datos de usuarios.

*Técnicamente:* La edición 2021 del Top 10 (la vigente al escribir esto) es:

| # | Riesgo | Qué es (en una línea) |
|---|--------|------------------------|
| A01 | Broken Access Control | Endpoints que no validan permisos → accedés a datos ajenos (IDOR) |
| A02 | Cryptographic Failures | Datos sensibles sin cifrar en tránsito o en reposo |
| A03 | Injection | SQL, NoSQL, OS command injection por concatenar input |
| A04 | Insecure Design | Falta de threat modeling desde el diseño |
| A05 | Security Misconfiguration | Defaults inseguros, headers ausentes, debug activado |
| A06 | Vulnerable Components | Dependencias con CVEs conocidos sin actualizar |
| A07 | Auth Failures | Sesiones/credenciales débiles, brute force sin límite |
| A08 | Software & Data Integrity Failures | Deserialización insegura, CI/CD sin verificar |
| A09 | Logging & Monitoring Failures | Sin detección → los ataques pasan desapercibidos |
| A10 | SSRF | El servidor hace requests a URLs controladas por el usuario |

Las secciones siguientes de este tópico profundizan los más relevantes para un dev full stack (CORS, CSRF, XSS, SQLi, JWT, hashing, secretos y supply chain), que caen en A01, A02, A03, A05, A06, A07 y A08.

> **Check de comprensión**
> 1. ¿Qué es OWASP y para qué sirve su Top 10?
>    - R: Es la organización de referencia en seguridad web; su Top 10 lista los diez riesgos más críticos de las aplicaciones web reales, y sirve como checklist de diseño y auditoría.
> 2. ¿Qué riesgo describe "Broken Access Control" (A01)?
>    - R: Endpoints que no validan permisos correctamente, permitiendo acceder o modificar recursos de otros usuarios (por ejemplo, IDOR cambiando un `id` en la URL).
> 3. ¿Cuál es la diferencia entre A02 (Cryptographic Failures) y A03 (Injection)?
>    - R: A02 es exponer datos sensibles sin cifrar (en tránsito o en reposo); A03 es inyectar comandos/código malicioso concatenando input del usuario sin sanitizar.
> 4. ¿Por qué "Vulnerable Components" (A06) se relaciona con el supply chain?
>    - R: Porque se refiere a dependencias de terceros con CVEs conocidos; si no actualizás librerías, heredás vulnerabilidades que no escribiste vos.
> 5. ¿Qué te permite decidir A09 (Logging & Monitoring Failures) en un incidente?
>    - R: Sin logs ni monitoreo no detectás ataques a tiempo; con logging adecuado podés identificar qué pasó, cuándo y qué datos se comprometieron.

→ Ver [Tópico 9: DevOps & Deployment](../concepts/09-devops-deployment.md#9.6-logging-centralizado) — el logging centralizado que A09 exige.

---

## 12.2 CORS

> Referencia base: [MDN — CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) · [OWASP — CORS](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing)

*En criollo:* CORS ya lo viste en el tópico 1: es el guardia del navegador que decide si un script de un dominio puede leer la respuesta de otro. Acá vamos un paso más allá: **la parte que rompe todo**. CORS no es una defensa del servidor — es una restricción del navegador. El error de CORS que te aparece en la consola no es que "el servidor se protegió": el request LLEGÓ al servidor y este lo respondió; el navegador bloqueó que tu script leyera la respuesta. Confundir eso te lleva a "arreglar" CORS con `Access-Control-Allow-Origin: *` en producción, que es exactamente el agujero que no querés.

*Técnicamente:* El navegador agrega el header `Origin` a los requests cross-origin. El servidor responde con headers que deciden si el navegador entrega la respuesta al script:

| Header | Qué controla |
|--------|--------------|
| `Access-Control-Allow-Origin` | Qué orígenes pueden leer la respuesta (`*` = todos) |
| `Access-Control-Allow-Methods` | Qué métodos HTTP se permiten (GET, POST, PUT…) |
| `Access-Control-Allow-Headers` | Qué headers custom se permiten en el request |
| `Access-Control-Allow-Credentials` | Si se permiten cookies/`Authorization` con origen específico |

Reglas que se rompen todo el tiempo:

- **No podés usar `*` con credenciales.** Si `Allow-Credentials: true`, `Allow-Origin` debe ser el origen exacto (no `*`).
- **Preflight**: requests "no simples" (método distinto a GET/HEAD/POST, o header custom como `Authorization` o `Content-Type: application/json`) disparan un request `OPTIONS` previo.
- La config se hace en el **backend**, no en el frontend.

```js
// Express: lista blanca de orígenes, nunca "*" con credenciales
import cors from 'cors';

const allowedOrigins = ['https://miapp.com', 'https://admin.miapp.com'];

app.use(
  cors({
    origin(origin, callback) {
      if (!origin || allowedOrigins.includes(origin)) {
        callback(null, true);
      } else {
        callback(new Error('Origen no permitido por CORS'));
      }
    },
    credentials: true,
  }),
);
```

```nginx
# O el header directo: restringe al origen exacto
add_header Access-Control-Allow-Origin https://miapp.com;
add_header Access-Control-Allow-Credentials true;
```

> **Check de comprensión**
> 1. ¿CORS es una defensa del servidor o una restricción del navegador? ¿Qué implica para tu API?
>    - R: Es una restricción del navegador: el request llega y se procesa, pero el navegador bloquea la LECTURA de la respuesta. Implica que CORS no protege tu API de requests maliciosos fuera del navegador (curl, bots); protege al USUARIO en su navegador.
> 2. ¿Por qué `Access-Control-Allow-Origin: *` es peligroso con credenciales?
>    - R: Porque permitiría a CUALQUIER origen leer respuestas que incluyen cookies o tokens del usuario. Con credenciales hay que restringir a orígenes exactos.
> 3. ¿Qué es un preflight y cuándo se dispara?
>    - R: Es un request `OPTIONS` que el navegador manda antes del real para verificar permisos. Se dispara con requests "no simples": métodos distintos a GET/HEAD/POST o headers custom como `Authorization` o `Content-Type: application/json`.
> 4. ¿Dónde se configura CORS: en el frontend o en el backend?
>    - R: En el backend, que es quien emite los headers `Access-Control-*`. El frontend solo envía el `Origin` y lee la respuesta si se lo permiten.
> 5. ¿Qué devuelve el callback de error si el `Origin` no está en tu lista blanca?
>    - R: Un error que hace que el request falle; el navegador no entrega la respuesta al script, y en el server podés loguear el intento de acceso desde un origen no permitido.

→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.8-cors-cross-origin-resource-sharing) — la base de same-origin policy y el funcionamiento del preflight.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.3-middlewares) — dónde encaja el middleware `cors()` en la cadena de middlewares.

---

## 12.3 CSRF

> Referencia base: [OWASP — CSRF](https://owasp.org/www-community/attacks/csrf) · [OWASP — CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) · [MDN — Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

*En criollo:* CSRF (Cross-Site Request Forgery) es engañar al navegador para que haga un request que el usuario NO quiso, aprovechando que está autenticado. Imaginate: estás logueado en tu banco, y en otra pestaña abrís un sitio malicioso. Ese sitio tiene un `<img src="https://banco.com/transferir?monto=1000&a=atacante">`. Tu navegador, que tiene la cookie de sesión del banco, hace el request como si fueras vos. El banco ve la cookie válida y transfiere. El atacante nunca vio tu cookie — solo la usó.

*Técnicamente:* El ataque explota que el navegador **adjunta cookies automáticamente** a cada request al dominio, sin importar de dónde vino. Las defensas:

1. **SameSite cookies** (la defensa moderna y más simple):

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

- `SameSite=Strict` / `Lax` → el navegador NO envía la cookie en requests cross-site (como el `<img>` del sitio malicioso).
- `Lax` es el default razonable: permite navegación top-level (click en un link) pero bloquea sub-requests y POST cross-site.

2. **CSRF token**: un token secreto que el servidor genera y el frontend reenvía en cada request de estado (no cookie), que el atacante no puede leer por same-origin policy:

```html
<form method="POST" action="/transferir">
  <input type="hidden" name="_csrf" value="TOKEN_GENERADO_POR_EL_SERVIDOR" />
</form>
```

3. **Custom header**: los requests que requieren un header custom (como `X-Requested-With` o `Authorization: Bearer`) disparan preflight CORS, y un form de otro sitio no puede setear headers custom.

**Nota clave**: si usás **JWT en header `Authorization`** (no en cookie), ya sos inmune a CSRF por defecto, porque el token no viaja en cookie y no se adjunta automáticamente.

> **Check de comprensión**
> 1. ¿Qué es CSRF y qué mecánica del navegador explota?
>    - R: Es un ataque que fuerza al navegador a hacer un request no deseado a un sitio donde el usuario está autenticado. Explota que el navegador adjunta cookies automáticamente a todo request al dominio, sin importar su origen.
> 2. ¿Cómo `SameSite=Strict` previene CSRF?
>    - R: El navegador no envía la cookie en requests cross-site, así que el `<img>` o el form del sitio malicioso viaja sin la cookie de sesión y el servidor no reconoce al usuario.
> 3. ¿Por qué un token CSRF en un campo oculto funciona?
>    - R: Porque el atacante no puede leer el token (same-origin policy le impide leer la respuesta del servidor), y sin el token el request POST se rechaza.
> 4. ¿Por qué usar JWT en header `Authorization` te hace inmune a CSRF?
>    - R: Porque el token no viaja en cookie, así que el navegador no lo adjunta automáticamente; un sitio malicioso no puede setear el header `Authorization` en un form o `<img>`.
> 5. ¿Cuándo `SameSite=Lax` es una mejor elección que `Strict`?
>    - R: Cuando querés que el usuario mantenga sesión al navegar por links externos hacia tu sitio (navegación top-level), pero aún bloquear requests cross-site como POST o sub-requests.

→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.6-navegadores) — flags de cookies (`HttpOnly`, `Secure`, `SameSite`).

---

## 12.4 XSS

> Referencia base: [OWASP — XSS](https://owasp.org/www-community/attacks/xss/) · [OWASP — XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) · [MDN — Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

*En criollo:* XSS (Cross-Site Scripting) es inyectar JavaScript malicioso que se ejecuta en el navegador de OTRA persona. Si tu app muestra contenido de usuarios sin escapar (`<h1>{comentario}</h1>` y el comentario es `<script>…</script>`), el script corre con los permisos de la víctima: roba cookies, tokens, redirige, modifica la página. Hay tres variantes: **reflected** (el payload viene en la URL y se refleja), **stored** (el payload queda guardado en la DB y afecta a todos los que lo ven) y **DOM-based** (el payload se inyecta vía JavaScript del lado del cliente sin tocar el servidor).

*Técnicamente:* La defensa principal es **escapar todo output** según el contexto (HTML, atributo, JS, URL), y **no usar `innerHTML` con input de usuario**:

```jsx
// MAL: inyecta HTML sin sanitizar
function Comentario({ texto }) {
  return <div dangerouslySetInnerHTML={{ __html: texto }} />; // ⚠️ XSS
}

// BIEN: React escapa por defecto
function Comentario({ texto }) {
  return <div>{texto}</div>; // React escapa `texto` automáticamente
}
```

```js
// Backend: sanitizar input con una librería dedicada
import DOMPurify from 'dompurify';

const limpio = DOMPurify.sanitize(comentario); // elimina <script> y atributos peligrosos
```

Y una **Content Security Policy** como defensa en profundidad: declara de qué orígenes se permite cargar scripts, así aunque algo se inyecte, el navegador no lo ejecuta:

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.confiable.com
```

Otro refuerzo: cookies con `HttpOnly` para que el JS no pueda leerlas, y `SameSite` para limitar el alcance.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre XSS reflected, stored y DOM-based?
>    - R: Reflected: el payload viene en la URL y se refleja en la respuesta inmediata. Stored: queda guardado en la DB y afecta a todos los que lo ven. DOM-based: se inyecta por JS del cliente manipulando el DOM, sin pasar por el servidor.
> 2. ¿Por qué usar `innerHTML` / `dangerouslySetInnerHTML` con input de usuario es peligroso?
>    - R: Porque inyecta HTML sin escapar; si el input contiene `<script>`, se ejecuta con los permisos de la víctima.
> 3. ¿Qué hace React para prevenir XSS por defecto?
>    - R: Escapa automáticamente el contenido que renderizás con `{variable}`, convirtiendo caracteres como `<` y `>` en entidades para que se muestren como texto y no como HTML.
> 4. ¿Qué es una CSP y por qué es "defensa en profundidad"?
>    - R: Es un header que declara de qué orígenes se permite cargar scripts. Es defensa en profundidad porque si falla el escape, el navegador igual no ejecuta scripts de orígenes no permitidos.
> 5. ¿Cómo ayuda la flag `HttpOnly` en una cookie contra XSS?
>    - R: Impide que JavaScript del cliente lea la cookie; aunque un atacante inyecte script, no puede robar el valor de la cookie de sesión.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.5-validacion) — validación y sanitización de input en el perímetro.

---

## 12.5 SQL Injection

> Referencia base: [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) · [OWASP — SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

*En criollo:* SQL Injection es la abuela de las vulnerabilidades web — tiene décadas y sigue en el Top 3 de OWASP. Pasa cuando armás una consulta SQL **concatenando strings** con input del usuario. Si el usuario manda `' OR '1'='1` donde vos esperabas un nombre, la consulta se convierte en algo que devuelve TODO (o peor: borra tablas). La regla es simple: **el input del usuario es data, nunca código**. Nunca lo concatenes a una query.

*Técnicamente:* La consulta vulnerable vs. la segura:

```sql
-- VULNERABLE: concatenación de strings
SELECT * FROM usuarios WHERE email = '${email}' AND password = '${password}';
-- con email = ' OR '1'='1' --  → devuelve todos los usuarios
```

```js
// VULNERABLE en Node (nunca hagas esto)
await db.query(`SELECT * FROM usuarios WHERE email = '${email}'`);
```

```js
// SEGURO: consultas parametrizadas (prepared statements)
await db.query('SELECT * FROM usuarios WHERE email = $1', [email]);
// el driver manda el valor como PARÁMETRO, nunca como parte del SQL
```

La defensa es siempre un **OR**:

1. **Consultas parametrizadas / prepared statements** (la defensa número uno, en todo lenguaje).
2. **ORM/query builder** (Prisma, Drizzle, Knex) que parametriza por defecto.
3. **Stored procedures** con parámetros tipados.
4. **Whitelist de input** para valores que no se pueden parametrizar (nombres de columnas, `ORDER BY`, `LIMIT`).
5. **Principio de menor privilegio**: el usuario de la DB con permisos mínimos (sin `DROP`, sin acceso a otras tablas).

```js
// Drizzle (query builder): parametriza automáticamente
const usuarios = await db.select().from(usuarios).where(eq(usuarios.email, email));

// El único caso donde NO podés parametrizar: ORDER BY — validá con whitelist
const columnasPermitidas = ['nombre', 'email', 'fecha'];
if (!columnasPermitidas.includes(ordenarPor)) throw new Error('Columna inválida');
```

> **Check de comprensión**
> 1. ¿Qué es SQL Injection y cuál es su causa raíz?
>    - R: Es inyectar SQL malicioso concatenando input del usuario en una consulta. Causa raíz: tratar el input como código en vez de como data.
> 2. ¿Cómo funcionan las consultas parametrizadas para prevenirla?
>    - R: El driver envía el valor como un parámetro separado de la estructura SQL; la base sabe qué es código y qué es data, así el input nunca altera la consulta.
> 3. ¿Por qué un ORM no te salva automáticamente en todos los casos?
>    - R: Porque podés usar raw queries o string interpolation dentro del ORM; y para valores no parametrizables (ORDER BY, nombres de columna) el ORM no protege — ahí necesitás whitelist.
> 4. ¿Qué defensa usás para un `ORDER BY` donde no podés parametrizar?
>    - R: Una whitelist de columnas permitidas; si el valor no está en la lista, se rechaza.
> 5. ¿Por qué el principio de menor privilegio mitiga el impacto de una SQLi?
>    - R: Porque si el usuario de la DB tiene permisos mínimos, un atacante que inyecta SQL no puede borrar tablas ni leer otras bases — limita el daño aunque la inyección ocurra.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.2-postgresql) — el motor y el acceso a datos donde aplicás estas defensas.

---

## 12.6 JWT

> Referencia base: [JWT — Introduction](https://jwt.io/introduction) · [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) · [Auth0 — JWT Best Practices](https://auth0.com/blog/a-look-at-the-latest-draft-for-jwt-best-current-practices/)

*En criollo:* JWT (JSON Web Token) ya lo tocaste en el tópico 4 como mecanismo de auth. Acá lo vemos desde el lado de la SEGURIDAD: qué puede salir mal y cómo evitarlo. Un JWT es un token firmado, **no cifrado** — cualquiera que lo tenga puede LEER su contenido en base64. Eso está bien: no guardes datos sensibles adentro. La firma solo garantiza que nadie LO MODIFICÓ. Los errores clásicos: guardar el token en `localStorage` (lo roba cualquier XSS), expiraciones eternas, y algoritmos mal configurados.

*Técnicamente:* Estructura y verificación:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMifQ.signature
      ↑ header           ↑ payload         ↑ firma
```

- **Header**: `{"alg":"HS256","typ":"JWT"}`.
- **Payload**: claims como `sub` (sujeto), `iat` (emitido), `exp` (expiración). Se lee, no se confía ciegamente.
- **Firma**: `HMACSHA256(base64(header) + "." + base64(payload), secret)` — garantiza integridad.

```js
import jwt from 'jsonwebtoken';

// Emitir: expiración corta + issuer + audience
const token = jwt.sign(
  { sub: usuario.id, role: usuario.role },
  process.env.JWT_SECRET,
  { expiresIn: '15m', issuer: 'miapp', audience: 'miapp-web' },
);

// Verificar: nunca aceptar token sin validar alg y exp
try {
  const payload = jwt.verify(token, process.env.JWT_SECRET, {
    issuer: 'miapp',
    audience: 'miapp-web',
    algorithms: ['HS256'], // ← fija el algoritmo, evita algorithm confusion
  });
} catch (err) {
  // token inválido, expirado o mal firmado
}
```

**Buenas prácticas de seguridad**:

- **Expiración corta** (15–60 min) + **refresh token** separado y rotativo.
- **`algorithms` fijo**: sin esto, un atacante puede intentar `alg: none` o algorithm confusion (firmar con clave pública como HMAC).
- **Secreto fuerte** (≥256 bits, aleatorio, en variable de entorno, nunca en el repo).
- **Almacenamiento**: preferí cookie `HttpOnly` + `Secure` + `SameSite` (o header `Authorization` en APIs sin browser). **Nunca `localStorage`**, porque cualquier XSS lo lee.
- **No pongas datos sensibles** en el payload: es legible por cualquiera que tenga el token.

> **Check de comprensión**
> 1. ¿Un JWT está cifrado o firmado? ¿Qué implica para el payload?
>    - R: Firmado, no cifrado. Cualquiera puede leer el payload en base64; la firma solo garantiza que no fue modificado. Por eso no se guardan datos sensibles adentro.
> 2. ¿Por qué `localStorage` es un mal lugar para guardar un JWT?
>    - R: Porque cualquier XSS puede leer `localStorage` y robar el token. En cambio, una cookie `HttpOnly` no es accesible desde JavaScript.
> 3. ¿Qué es "algorithm confusion" y cómo lo prevenís?
>    - R: Es un ataque donde el atacante cambia el algoritmo declarado en el header (por ejemplo `none` o HS256 con clave pública). Se previene fijando `algorithms: ['HS256']` en la verificación.
> 4. ¿Por qué un JWT robado sigue siendo válido hasta que expira?
>    - R: Porque JWT es stateless: el servidor verifica la firma y la expiración, no consulta una lista de tokens revocados. Mitigás con expiración corta, refresh tokens rotativos y, si hace falta, una deny-list.
> 5. ¿Qué rol cumplen `issuer` y `audience` en un JWT?
>    - R: Declaran quién emitió el token y para quién es. Verificarlos evita que un token emitido para otro sistema o por otro emisor sea aceptado por tu API.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.6-autenticacion-y-autorizacion) — el mecanismo de auth donde JWT se emite y valida.
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.4-https-y-tls) — HTTPS para que el token no viaje en texto plano.

---

## 12.7 Hashing (bcrypt, argon2)

> Referencia base: [OWASP — Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) · [bcrypt — npm](https://www.npmjs.com/package/bcrypt) · [argon2 — npm](https://www.npmjs.com/package/argon2)

*En criollo:* Guardar passwords en texto plano es regalar las cuentas de todos tus usuarios si te roban la DB. Hashear es transformar la password en una cadena irreparable de un solo sentido: de un hash no volvés a la password. Pero NO sirve cualquier hash. SHA256 o MD5 son **rápidos**: un atacante con una GPU prueba miles de millones por segundo. bcrypt y argon2 son **deliberadamente lentos y caros** (en CPU y memoria), así que un ataque de fuerza bruta se vuelve inviable. Esa lentitud es la feature, no el bug.

*Técnicamente:*

| Función | Característica | ¿Cuándo? |
|---------|----------------|----------|
| bcrypt | Work factor (cost) = iteraciones 2^cost; salt automático | Default sólido, muy usado |
| argon2id | Ganador del Password Hashing Competition 2015; parámetros de tiempo Y memoria; resistente a GPU | Recomendado por OWASP para proyectos nuevos |
| PBKDF2 | Configurable, estándar NIST | Cuando necesitás cumplimiento específico |
| SHA256/MD5 | Rápidos | **NUNCA para passwords** |

```js
// bcrypt: salt automático + work factor
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12; // 2^12 iteraciones ≈ ~250ms por hash

// Al registrar
const hash = await bcrypt.hash(password, SALT_ROUNDS);
await db.usuarios.create({ email, passwordHash: hash });

// Al loguear — NUNCA compares con `===`, usá la comparación timing-safe
const ok = await bcrypt.compare(password, usuario.passwordHash);
if (!ok) throw new Error('Credenciales inválidas');
```

```js
// argon2id: parámetros de tiempo, memoria y paralelismo
import argon2 from 'argon2';

const hash = await argon2.hash(password, {
  type: argon2.argon2id, // la variante resistente a side-channel
  memoryCost: 19456,      // 19 MiB
  timeCost: 2,
  parallelism: 1,
});

const ok = await argon2.verify(hash, password);
```

**Reglas de oro**:

- **Salt único por password** (bcrypt/argon2 lo hacen automáticamente): dos usuarios con la misma password tienen hashes distintos.
- **`bcrypt.compare` en vez de `===`**: evita timing attacks comparando en tiempo constante.
- **Work factor alto**: el suficiente para que hash tardar ~250ms-1s; reajustalo cuando el hardware mejore.
- **Nunca loguees ni guardes la password en texto plano**; ni siquiera en logs.

> **Check de comprensión**
> 1. ¿Por qué SHA256 o MD5 son malos para guardar passwords aunque sean hashes?
>    - R: Porque son rápidos; un atacante con GPU prueba miles de millones de combinaciones por segundo. Para passwords se necesita un hash deliberadamente lento.
> 2. ¿Qué es un salt y por qué bcrypt lo agrega automáticamente?
>    - R: Es un valor aleatorio único por password que se mezcla antes de hashear. Hace que dos usuarios con la misma password tengan hashes distintos y evita ataques con tablas precomputadas (rainbow tables).
> 3. ¿Por qué comparás con `bcrypt.compare` y no con `===`?
>    - R: Porque `bcrypt.compare` compara en tiempo constante (timing-safe), evitando que un atacante deduzca caracteres correctos midiendo el tiempo de respuesta.
> 4. ¿Cuál es la diferencia clave entre bcrypt y argon2id?
>    - R: bcrypt usa un work factor de CPU; argon2id agrega además un parámetro de memoria, lo que lo hace resistente a ataques con GPU y es el recomendado por OWASP para proyectos nuevos.
> 5. ¿Qué significa que un hash sea "de un solo sentido"?
>    - R: Que no hay forma práctica de revertir el hash a la password original; la única forma es probar combinaciones (fuerza bruta), que la lentitud hace inviable.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.6-autenticacion-y-autorizacion) — el flujo de registro/login donde hasheás y verificás.

---

## 12.8 Secret Management

> Referencia base: [OWASP — Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) · [12 Factor App — Config](https://12factor.net/config)

*En criollo:* Un secreto es cualquier credencial que no debe quedar expuesta: claves de API, passwords de DB, `JWT_SECRET`, tokens de servicios. El error número uno de un junior es **commitear el `.env`** (o hardcodear secretos en el código). Una vez que un secreto está en el historial de git, está comprometido PARA SIEMPRE: aunque lo borres en un commit siguiente, sigue en el historial. Rotarlo es la única solución real. Los secretos van en variables de entorno inyectadas en runtime, nunca en el repo.

*Técnicamente:*

```bash
# .env — solo local, JAMÁS al repo
JWT_SECRET=...
DATABASE_URL=...
STRIPE_API_KEY=...

# .gitignore — SIEMPRE
.env
.env.*
*.pem
```

```js
// Acceso en runtime — sin valores por defecto inseguros en producción
const secret = process.env.JWT_SECRET;
if (!secret) throw new Error('Falta JWT_SECRET en el entorno');
```

**Escalera de madurez**:

| Nivel | Práctica |
|-------|----------|
| Básico | `.env` en `.gitignore`, secretos en variables de entorno |
| Medio | Secrets por entorno (dev/staging/prod), sin compartir `.env` por chat |
| Avanzado | Secret manager (AWS Secrets Manager, Vault, Doppler), inyección en el deploy |
| Pro | Rotación automática, secretos efímeros, scanning de secretos en CI |

**Defensas extra**:

- **Scanning de secretos**: herramientas (gitleaks, trufflehog, GitHub secret scanning) detectan secretos commiteados.
- **Rotación**: asumí que un secreto puede haberse filtrado; rotalo en cualquier sospecha.
- **Separación**: el secreto de producción NO es el de desarrollo.
- **Menor privilegio**: cada servicio recibe solo los secretos que necesita.

> **Check de comprensión**
> 1. ¿Por qué borrar un secreto en un commit siguiente NO lo arregla?
>    - R: Porque el secreto sigue en el historial de git. La única solución real es rotar el secreto (generar uno nuevo y revocar el viejo).
> 2. ¿Qué archivo nunca debe commitearse y cómo lo evitás?
>    - R: El `.env` (o cualquier archivo con secretos). Se evita agregándolo a `.gitignore`.
> 3. ¿Por qué los secretos se inyectan como variables de entorno en runtime?
>    - R: Para separar el código de la configuración: el mismo build corre en cualquier entorno con secretos distintos, y los secretos no quedan en el código fuente ni en el repo.
> 4. ¿Qué es un secret manager y qué ventaja da sobre un `.env`?
>    - R: Es un servicio (AWS Secrets Manager, Vault, Doppler) que almacena, rota y audita secretos de forma centralizada, con control de acceso fino y rotación automática — el `.env` es estático y manual.
> 5. ¿Qué es el "secret scanning" y para qué sirve en CI?
>    - R: Herramientas (gitleaks, trufflehog) que detectan secretos commiteados; corren en CI para bloquear un push/merge si se filtró una clave antes de que llegue a producción.

→ Ver [Tópico 9: DevOps & Deployment](../concepts/09-devops-deployment.md#9.7-secrets-management) — secretos en el pipeline de deploy y en producción.

---

## 12.9 Supply Chain

> Referencia base: [OWASP — Software Supply Chain Security](https://owasp.org/www-project-software-supply-chain-security/) · [npm — Audit](https://docs.npmjs.com/cli/v10/commands/npm-audit) · [SLSA — Supply-chain Levels](https://slsa.dev/)

*En criollo:* El supply chain es todo lo que entra a tu app que vos no escribiste: dependencias de npm, imágenes base de Docker, herramientas de CI. Un ataque de supply chain es cuando alguien compromete UNA de esas piezas y vos la heredás al hacer `npm install`. El caso clásico es un paquete popular con un mantenedor comprometido, o un typo-squatting (un paquete con nombre parecido al real, como `loadsh` en vez de `lodash`). La lección: **cada dependencia es código de terceros corriendo con tus privilegios**.

*Técnicamente:*

```bash
# Auditar dependencias por vulnerabilidades conocidas (CVEs)
npm audit
pnpm audit
yarn audit

# Ver el árbol completo de dependencias
npm ls --all

# Lockfile: garantiza versiones exactas y reproducibles
# package-lock.json / pnpm-lock.yaml / yarn.lock → COMMITEALO SIEMPRE
```

```jsonc
// package.json: fijar versiones y alcance
{
  "scripts": {
    "audit": "npm audit --audit-level=high" // falla en CI si hay high/critical
  }
}
```

**Defensas**:

1. **Lockfile commiteado**: builds reproducibles, sin sorpresas por rangos `^`/`~`.
2. **`npm audit` en CI**: bloquea merges con vulnerabilidades `high`/`critical`.
3. **Dependabot / Renovate**: PRs automáticos que actualizan dependencias vulnerables.
4. **Menos dependencias**: cada una es una superficie de ataque; preferí la stdlib o una dependencia bien mantenida.
5. **Verificar integridad**: checksums y, para CI/CD, firma de artefactos (SLSA) y pines de versión en imágenes (`node:20-alpine` con digest SHA).

```dockerfile
# Pineá la imagen por digest, no por tag mutable
FROM node:20-alpine@sha256:abcdef1234567890...
```

> **Check de comprensión**
> 1. ¿Qué es un ataque de supply chain?
>    - R: Es comprometer una pieza de terceros (una dependencia de npm, una imagen Docker, una herramienta de CI) para que su código malicioso entre a tu app cuando la instalás o construís.
> 2. ¿Qué es el typo-squatting?
>    - R: Publicar un paquete con un nombre casi idéntico a uno popular (por ejemplo `loadsh` en vez de `lodash`) esperando que alguien lo instale por error de tipeo y ejecute código malicioso.
> 3. ¿Por qué commitear el lockfile es una defensa de supply chain?
>    - R: Porque fija las versiones exactas instaladas; sin lockfile, los rangos `^`/`~` pueden traer una versión nueva y comprometida en el próximo install.
> 4. ¿Qué hace `npm audit` y por qué lo corrés en CI?
>    - R: Revisa el árbol de dependencias contra bases de CVEs conocidas y reporta vulnerabilidades. En CI, con `--audit-level=high`, bloquea merges que introduzcan vulnerabilidades altas o críticas.
> 5. ¿Por qué pinear una imagen Docker por digest SHA en vez de por tag?
>    - R: Porque un tag como `node:20-alpine` es mutable y puede ser reemplazado por una imagen comprometida; el digest SHA identifica un contenido exacto e inmutable.

→ Ver [Tópico 9: DevOps & Deployment](../concepts/09-devops-deployment.md#9.1-docker) — imágenes Docker y su papel en el supply chain.
