# 1. Fundamentos de la Web

> Objetivo: comprender cómo se comunican cliente y servidor, qué pasa entre que escribís una URL y ves la página.

## 1.1 Cómo funciona Internet

> Referencia base: [MDN — How does the Internet work?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/How_does_the_Internet_work)

### Redes simples
- Dos computadoras se conectan por cable (Ethernet) o de forma inalámbrica (Wi-Fi, Bluetooth).
- Al agregar más computadoras, el cableado se vuelve inviable → se introduce un **switch**: una computadora especial que recibe mensajes y los reenvía solo al destinatario correcto.

### Redes de redes
- Un switch no escala a miles de millones de dispositivos → conectamos switches entre sí con **routers**.
- **Router**: computadora que sabe reenviar mensajes entre redes distintas. Como una oficina de correo: lee la dirección de destino y deriva al destinatario correcto.
- Para conectar nuestra red a la infraestructura de telefonía existente, necesitamos un **módem** (convierte la señal de nuestra red a la de la línea telefónica y viceversa).
- El router hogareño típico es en realidad **switch + router + módem** en un solo dispositivo.
- **ISP (Internet Service Provider)**: empresa que gestiona routers especiales conectados entre sí y con otros ISPs. Internet es esa infraestructura completa de redes interconectadas.

### Encontrar computadoras: IP y DNS
- Cada dispositivo conectado tiene una **dirección IP** única (ej: `192.0.2.172`).
- Los humanos no recordamos números fácilmente → **Domain Name System (DNS)** traduce nombres de dominio (ej: `google.com`) a direcciones IP.
- [MDN — What is a Domain Name?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_domain_name)

### Internet ≠ Web
- **Internet**: infraestructura física/técnica que conecta computadoras.
- **Web**: servicio construido sobre Internet. Otros servicios sobre Internet: email, IRC, FTP.
- **Intranet**: red privada restringida a miembros de una organización. **Extranet**: intranet que abre parte de su red a colaboradores externos.
- [MDN — What is a web server?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_web_server)
- [MDN — What is a URL?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_URL)

---

## 1.2 Arquitectura Cliente-Servidor

- **Ciclo request-response**: el cliente inicia, el servidor responde. El servidor NUNCA inicia comunicación (salvo WebSockets y Server-Sent Events).
- **User-agent**: típicamente el navegador, pero puede ser curl, un script, una app mobile. Siempre es quien inicia el request.
- **Proxies**: intermediarios entre cliente y servidor. Pueden cachear, filtrar, balancear carga, autenticar, loguear.
- **Thin client vs thick client**: dónde vive la lógica — en el servidor (SSR tradicional) o en el cliente (SPA).

---

## 1.3 Protocolo HTTP

> Referencia base: [MDN — Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)

### Características clave
- **Simple y legible**: mensajes HTTP/1.1 son texto plano que un humano puede leer.
- **Extensible**: los headers permiten agregar funcionalidad sin romper compatibilidad.
- **Stateless, no sessionless**: HTTP no guarda estado entre requests, pero las cookies permiten crear sesiones.
- **Capa de aplicación**: HTTP viaja sobre TCP (confiable, orientado a conexión), o sobre TCP+TLS (HTTPS). HTTP/3 usa QUIC sobre UDP.

### HTTP Flow
1. Cliente abre (o reusa) una conexión TCP con el servidor.
2. Cliente envía un HTTP message (request).
3. Servidor procesa y devuelve un HTTP message (response).
4. Se cierra o reusa la conexión.

### Métodos HTTP
| Método  | Semántica          | Idempotente | Body  |
|---------|--------------------|-------------|-------|
| GET     | Leer recurso       | Sí          | No    |
| POST    | Crear recurso      | No          | Sí    |
| PUT     | Reemplazar recurso | Sí          | Sí    |
| PATCH   | Modificar parcial  | No*         | Sí    |
| DELETE  | Eliminar recurso   | Sí          | No*   |
| OPTIONS | Métodos disponibles| Sí          | No    |
| HEAD    | Solo headers       | Sí          | No    |

> *PATCH puede ser idempotente si usás condiciones atómicas. DELETE rara vez lleva body.

### Status Codes
- **2xx** — Éxito: `200 OK`, `201 Created`, `204 No Content`.
- **3xx** — Redirección: `301 Moved Permanently`, `302 Found`, `304 Not Modified`.
- **4xx** — Error del cliente: `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`, `429 Too Many Requests`.
- **5xx** — Error del servidor: `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`.

### Headers esenciales
- **Request**: `Host`, `User-Agent`, `Accept`, `Content-Type`, `Authorization`, `Cookie`, `Cache-Control`, `Origin`.
- **Response**: `Content-Type`, `Set-Cookie`, `Cache-Control`, `ETag`, `Location`, `Access-Control-*`.

### Formatos de Body
- `application/json` — estándar para APIs.
- `multipart/form-data` — upload de archivos.
- `application/x-www-form-urlencoded` — formularios HTML clásicos.

> [MDN — HTTP Messages](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Messages)
> [MDN — Evolution of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Evolution_of_HTTP)

---

## 1.4 HTTPS y TLS

- **TLS Handshake**: intercambio de claves, verificación de certificado, cifrado simétrico de la sesión.
- **Certificados**: emitidos por CAs (Certificate Authorities). Cadena de confianza.
- **HSTS**: fuerza HTTPS para todo el dominio vía `Strict-Transport-Security`.
- **HTTP/2**: multiplexación, server push, compresión de headers, binario.
- **HTTP/3**: usa QUIC (sobre UDP), elimina head-of-line blocking, 0-RTT handshake.

---

## 1.5 DNS en profundidad

> [MDN — What is a Domain Name?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_domain_name)

- **Resolución**: root servers → TLD servers → authoritative nameservers.
- **Caching**: browser, OS, router, ISP (cada nivel cachea por TTL).
- **Registros**: `A` (IPv4), `AAAA` (IPv6), `CNAME` (alias), `MX` (mail), `TXT` (SPF, DKIM, verificación).

---

## 1.6 Navegadores

> [MDN — What are browser developer tools?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Tools_and_setup/What_are_browser_developer_tools)

- **Critical Rendering Path**: DOM → CSSOM → Render Tree → Layout → Paint → Composite.
- **DevTools**: Network, Performance, Console, Application (storage, service workers).
- **Storage del navegador**:
  - `localStorage` — persistente, ~5 MB, solo strings.
  - `sessionStorage` — por pestaña, se borra al cerrar.
  - `cookies` — se envían en cada request, ~4 KB, flags `HttpOnly`, `Secure`, `SameSite`.
  - `IndexedDB` — estructurado, asíncrono, para grandes volúmenes de datos.

---

## 1.7 REST (Representational State Transfer)

- **Recursos**: todo es un recurso identificado por URL (`/users/42`, `/orders/101`).
- **Verbos HTTP como acciones**: GET lee, POST crea, PUT reemplaza, PATCH modifica parcialmente, DELETE borra.
- **Stateless**: cada request contiene toda la información necesaria. El servidor no guarda estado entre requests.
- **HATEOAS**: la respuesta incluye links a acciones relacionadas (ideal, rara vez implementado puro).
- **REST vs GraphQL vs gRPC**: REST es resource-oriented, GraphQL es query-oriented, gRPC es procedure-oriented.

---

## 1.8 CORS (Cross-Origin Resource Sharing)

> [MDN — CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)

- **Same-Origin Policy**: un script de `ejemplo.com` no puede leer la respuesta de `api.otro.com`.
- **Preflight**: el navegador manda `OPTIONS` antes del request real para verificar permisos CORS.
- **Headers CORS**:
  - `Access-Control-Allow-Origin` — qué orígenes pueden acceder.
  - `Access-Control-Allow-Methods` — qué métodos permite el servidor.
  - `Access-Control-Allow-Headers` — qué headers custom acepta.
  - `Access-Control-Allow-Credentials` — permite cookies/auth headers cross-origin.
- Con `credentials: 'include'`, el servidor DEBE devolver `Access-Control-Allow-Credentials: true` Y un origen específico (no `*`).

---

## 1.9 Conceptos complementarios

- **URL vs URI vs URN**: URL es un tipo de URI que identifica un recurso por su ubicación.
- **Query params vs path params vs body**: path identifica el recurso, query filtra/ordena/pagina, body transporta datos de creación/actualización.
- **Idempotencia**: mismo request → mismo resultado, sin importar cuántas veces se ejecute. Fundamental para APIs robustas.
- **Encoding**: URL encoding (`%20`), Base64, UTF-8.

---

## Referencias MDN

| Tema | Link |
|------|------|
| How does the Internet work? | [MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/How_does_the_Internet_work) |
| What is a Domain Name? | [MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_domain_name) |
| What is a URL? | [MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_URL) |
| What is a web server? | [MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_web_server) |
| Overview of HTTP | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview) |
| HTTP Messages | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Messages) |
| Evolution of HTTP | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Evolution_of_HTTP) |
| CORS | [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) |
| Browser DevTools | [MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Tools_and_setup/What_are_browser_developer_tools) |

---

> **Check de comprensión**: ¿Podés explicar qué pasa desde que escribís `https://api.ejemplo.com/users/42` hasta que ves el JSON de respuesta, cubriendo: DNS resolution, TCP/TLS handshake, HTTP request, server routing y HTTP response?
