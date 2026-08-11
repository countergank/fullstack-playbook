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
- > [MDN — What is a Domain Name?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_domain_name)

### Internet ≠ Web
- **Internet**: infraestructura física/técnica que conecta computadoras.
- **Web**: servicio construido sobre Internet. Otros servicios sobre Internet: email, IRC, FTP.
- **Intranet**: red privada restringida a miembros de una organización. **Extranet**: intranet que abre parte de su red a colaboradores externos.
- > [MDN — What is a web server?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_web_server)
- > [MDN — What is a URL?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_URL)

*En criollo:* Imaginate que Internet es la red de rutas, autopistas y calles de un país. La Web es solo uno de los servicios que usa esas rutas: como que las autopistas sirven para autos, camiones y motos, Internet sirve para la Web, el email, el streaming y más. El DNS es como la agenda de contactos de tu celular: en vez de recordar el número (IP) de cada persona, guardás su nombre (dominio) y el sistema lo traduce.

*Técnicamente:* Cuando tu computadora envía un paquete de datos, este viaja en capas según el modelo TCP/IP:

| Capa | Protocolo | Función |
|------|-----------|---------|
| Aplicación | HTTP, DNS, SMTP | Datos que usa el usuario |
| Transporte | TCP, UDP | Entrega confiable (TCP) o rápida (UDP) |
| Internet | IP | Direccionamiento y ruteo |
| Acceso a red | Ethernet, Wi-Fi | Transmisión física |

Podés ver tu ruta de red con `traceroute` (Linux/Mac) o `tracert` (Windows):

```bash
tracerette google.com
# Cada línea = un router (hop) por el que pasa tu paquete
# 1  192.168.1.1       ← tu router
# 2  10.0.0.1          ← router del ISP
# 3  200.45.180.1      ← backbone del ISP
# ...
# 12 142.250.80.46     ← servidor de Google
```

> **Check de comprensión**
> 1. ¿Cuál es la diferencia fundamental entre Internet y la Web?
>    - R: Internet es la infraestructura física de redes interconectadas; la Web es un servicio que funciona sobre Internet usando HTTP/HTTPS.
> 2. ¿Qué función cumple un router y en qué se diferencia de un switch?
>    - R: El switch reenvía datos dentro de una misma red; el router conecta redes distintas y decide la mejor ruta para los paquetes.
> 3. ¿Por qué existe el DNS y qué problema resuelve?
>    - R: Los humanos no recordamos direcciones IP numéricas fácilmente; el DNS traduce nombres legibles (google.com) a IPs (142.250.80.46).
> 4. ¿Qué es un ISP y cuál es su rol en Internet?
>    - R: Un ISP (Internet Service Provider) gestiona routers conectados entre sí y con otros ISPs, formando la infraestructura que te da acceso a Internet.
> 5. ¿Qué es una intranet y cómo se diferencia de Internet?
>    - R: Una intranet es una red privada dentro de una organización; Internet es la red pública global. La intranet usa la misma tecnología pero con acceso restringido.
> 6. ¿Qué capas del modelo TCP/IP intervienen cuando cargás una página web?
>    - R: Las 4 capas: Acceso a red (Wi-Fi/Ethernet), Internet (IP para direccionamiento), Transporte (TCP para entrega confiable), Aplicación (HTTP para el contenido).

→ Ver [Tópico 5: DNS en profundidad](#1.5-dns-en-profundidad) para entender cómo se resuelve un dominio.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.2-apis-rest) para ver cómo los servidores usan esta infraestructura.

---

## 1.2 Arquitectura Cliente-Servidor

*En criollo:* Pensalo como un restaurante. Vos (el cliente) mirás el menú, elegís y le pedís al mozo. La cocina (el servidor) prepara tu pedido y te lo trae. La cocina NUNCA te manda comida sin que la pidas primero. Si querés más, tenés que pedir de nuevo. Así funciona la web: vos pedís, el servidor responde.

*Técnicamente:* El patrón cliente-servidor se implementa con sockets TCP. El servidor escucha en un puerto específico y el cliente se conecta:

```js
// Servidor (Node.js) — escucha en puerto 3000
const net = require('net');
const server = net.createServer((socket) => {
  socket.write('HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\n\r\nHola!');
  socket.end();
});
server.listen(3000);

// Cliente (Node.js) — se conecta y pide
const client = net.createConnection({ port: 3000 }, () => {
  client.write('GET / HTTP/1.1\r\nHost: localhost\r\n\r\n');
});
client.on('data', (data) => console.log(data.toString()));
```

| Patrón | Quién inicia | Ejemplo |
|--------|-------------|---------|
| Request-Response | Cliente | HTTP, REST APIs |
| Push | Servidor | WebSockets, Server-Sent Events |
| Pub/Sub | Ambos (vía broker) | Redis Pub/Sub, MQTT |

> **Check de comprensión**
> 1. ¿Por qué se dice que en HTTP "el servidor nunca inicia la comunicación"?
>    - R: Porque HTTP sigue el patrón request-response: el cliente siempre envía el primer mensaje y el servidor solo responde a ese request.
> 2. ¿Qué es un user-agent y puede ser algo que no sea un navegador?
>    - R: El user-agent es quien inicia el request HTTP; puede ser curl, un script de Python, una app mobile o cualquier programa que haga requests.
> 3. ¿Para qué sirven los proxies y qué tipos de funciones pueden cumplir?
>    - R: Los proxies son intermediarios que pueden cachear respuestas, filtrar contenido, balancear carga entre servidores, autenticar usuarios o loguear tráfico.
> 4. ¿Qué diferencia hay entre thin client y thick client?
>    - R: En thin client la lógica vive en el servidor (SSR tradicional); en thick client la lógica vive en el cliente (SPA con React, Angular, etc.).
> 5. ¿Qué patrón usarías si necesitás que el servidor envíe datos al cliente sin que este los pida?
>    - R: WebSockets (comunicación bidireccional persistente) o Server-Sent Events (stream unidireccional servidor→cliente).

→ Ver [Tópico 3: Protocolo HTTP](#1.3-protocolo-http) para entender el protocolo que usa esta arquitectura.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.1-node-js-runtime) para ver cómo se implementa un servidor.

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

*En criollo:* HTTP es como el idioma que usan tu navegador y el servidor para entenderse. Cuando pedís una página, el navegador dice "dame esto" (request) y el servidor responde "acá tenés" (response) o "no existe" (404). Es como hablar por walkie-talkie: uno habla, el otro responde, y no se acuerdan de la conversación anterior (stateless).

*Técnicamente:* Un request HTTP real se ve así:

```http
GET /api/users/42 HTTP/1.1
Host: api.ejemplo.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

Y la respuesta:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600
ETag: "abc123"

{"id": 42, "name": "Juan", "role": "admin"}
```

Evolución de versiones:

| Versión | Transporte | Característica clave |
|---------|-----------|---------------------|
| HTTP/1.0 | TCP | Una conexión por request |
| HTTP/1.1 | TCP | Keep-alive, pipelining, host header |
| HTTP/2   | TCP+TLS | Multiplexación, server push, binario |
| HTTP/3   | QUIC/UDP | Sin head-of-line blocking, 0-RTT |

> **Check de comprensión**
> 1. ¿Qué significa que HTTP sea "stateless" y cómo se maneja el estado entonces?
>    - R: HTTP no recuerda requests anteriores; el estado se maneja con cookies, tokens JWT o sesiones del lado del servidor.
> 2. ¿Cuál es la diferencia entre PUT y PATCH en términos de idempotencia?
>    - R: PUT siempre reemplaza el recurso completo (idempotente); PATCH modifica parcialmente y solo es idempotente si la operación es atómica.
> 3. ¿Qué status code devolverías si un usuario intenta acceder sin autenticar?
>    - R: `401 Unauthorized` — indica que falta autenticación. `403 Forbidden` sería si está autenticado pero no tiene permisos.
> 4. ¿Por qué HTTP/2 es más eficiente que HTTP/1.1 para páginas con muchos recursos?
>    - R: HTTP/2 multiplexa múltiples requests en una sola conexión TCP, eliminando el head-of-line blocking de HTTP/1.1.
> 5. ¿Qué header usarías para cachear una respuesta por 1 hora?
>    - R: `Cache-Control: max-age=3600` (3600 segundos = 1 hora).
> 6. ¿Qué diferencia hay entre `Content-Type` en el request y en el response?
>    - R: En el request indica el formato del body que envía el cliente; en el response indica el formato del body que devuelve el servidor.

→ Ver [Tópico 4: HTTPS y TLS](#1.4-https-y-tls) para entender cómo se cifra HTTP.
→ Ver [Tópico 7: REST](#1.7-rest-representational-state-transfer) para ver cómo HTTP se usa en APIs.

---

## 1.4 HTTPS y TLS

*En criollo:* HTTPS es HTTP metido dentro de un sobre sellado. Si HTTP normal es como mandar una postal (cualquier correo puede leerla), HTTPS es como mandar una carta en un sobre cifrado: aunque alguien la intercepte, no puede leer el contenido. El "sobre" lo crea TLS mediante un handshake que verifica la identidad del servidor con un certificado.

*Técnicamente:* El TLS handshake sigue estos pasos:

```
Cliente                          Servidor
  |--- ClientHello (versiones, ciphers) --->|
  |<-- ServerHello + Certificado -----------|
  |    (el cliente verifica la CA)          |
  |--- Key Exchange (Diffie-Hellman) ------>|
  |<-- ServerHello Done -------------------|
  |    (ambos derivan la clave simétrica)   |
  |--- Finished (cifrado activado) ------->|
  |<-- Finished ---------------------------|
  |==== Datos cifrados con AES/GCM =======|
```

| Concepto | Descripción |
|----------|-------------|
| Certificado | Archivo `.pem`/`.crt` emitido por una CA (Let's Encrypt, DigiCert) |
| CA (Certificate Authority) | Entidad confiable que firma certificados |
| HSTS | Header `Strict-Transport-Security` que fuerza HTTPS |
| Cipher Suite | Algoritmos de cifrado negociados (ej: `TLS_AES_256_GCM_SHA384`) |

Para desarrollo local, usá `mkcert` para generar certificados auto-firmados confiables:

```bash
mkcert -install                    # Crea CA local confiable
mkcert localhost                   # Genera cert para localhost
# Resultado: localhost.pem + localhost-key.pem
```

> **Check de comprensión**
> 1. ¿Qué problema resuelve TLS sobre HTTP plano?
>    - R: Cifra la comunicación para que terceros no puedan leer ni modificar los datos en tránsito (confidencialidad e integridad).
> 2. ¿Qué es una Certificate Authority (CA) y por qué es importante?
>    - R: Una CA es una entidad confiable que firma certificados digitales, verificando que un servidor es quien dice ser. Sin CA, cualquiera podría hacerse pasar por otro sitio.
> 3. ¿Qué hace el header HSTS y por qué es útil?
>    - R: `Strict-Transport-Security` le dice al navegador que SIEMPRE use HTTPS para ese dominio, evitando ataques de downgrade a HTTP plano.
> 4. ¿Cuál es la diferencia principal entre HTTP/2 y HTTP/3 a nivel de transporte?
>    - R: HTTP/2 usa TCP (sufre head-of-line blocking si hay pérdida de paquetes); HTTP/3 usa QUIC sobre UDP (cada stream es independiente).
> 5. ¿Por qué los certificados auto-firmados dan warnings en el navegador?
>    - R: Porque no están firmados por una CA confiable; el navegador no puede verificar la identidad del servidor.
> 6. ¿Qué es el "0-RTT handshake" de HTTP/3 y qué riesgo tiene?
>    - R: Permite enviar datos en el primer round-trip sin esperar el handshake completo; el riesgo es que puede ser vulnerable a ataques de replay.

→ Ver [Tópico 3: Protocolo HTTP](#1.3-protocolo-http) para entender la base que HTTPS cifra.
→ Ver [Tópico 8: CORS](#1.8-cors-cross-origin-resource-sharing) para ver cómo HTTPS interactúa con la same-origin policy.

---

## 1.5 DNS en profundidad

> [MDN — What is a Domain Name?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_domain_name)

### Resolución
- **Jerarquía**: root servers (`.`) → TLD servers (`.com`, `.org`) → authoritative nameservers (los del dominio).
- **Caching**: browser → OS → router → ISP (cada nivel cachea por TTL).

### Registros DNS
| Tipo | Función | Ejemplo |
|------|---------|---------|
| `A` | IPv4 | `192.0.2.1` |
| `AAAA` | IPv6 | `2001:db8::1` |
| `CNAME` | Alias | `www → ejemplo.com` |
| `MX` | Mail | `mail.ejemplo.com` |
| `TXT` | Texto libre | SPF, DKIM, verificación |
| `NS` | Nameserver | `ns1.cloudflare.com` |

*En criollo:* DNS es como la guía telefónica de Internet. Cuando escribís `google.com`, tu computadora no sabe dónde está — pregunta al DNS "¿cuál es la IP de google.com?" y el DNS le responde con la dirección numérica. Cada nivel (tu navegador, tu compu, tu router, tu ISP) guarda la respuesta un rato (TTL) para no preguntar de nuevo.

*Técnicamente:* Podés diagnosticar DNS con herramientas CLI:

```bash
# Resolver un dominio (A record)
dig google.com A
# Respuesta: google.com. 300 IN A 142.250.80.46

# Tracear toda la cadena de delegación
dig +trace google.com
# Muestra: root → .com TLD → authoritative nameservers

# Reverse lookup (IP → dominio)
host 142.250.80.46
# Respuesta: 46.80.250.142.in-addr.arpa domain name pointer gru07s01-in-f14.1e100.net

# Whois (info del dominio)
whois google.com
# Registrador, fecha de creación, nameservers, etc.
```

Flujo de resolución completo:

```
tu navegador → cache del browser → cache del OS → cache del router
    → DNS del ISP → root server (.) → TLD server (.com)
    → authoritative server (ns1.google.com) → IP: 142.250.80.46
```

> **Check de comprensión**
> 1. ¿Qué es el TTL en DNS y por qué importa?
>    - R: Time-To-Live indica cuánto tiempo se cachea un registro DNS; un TTL bajo permite cambios rápidos pero aumenta la carga de queries.
> 2. ¿Cuál es la diferencia entre un registro A y un CNAME?
>    - R: A mapea un dominio a una IP; CNAME mapea un dominio a OTRO dominio (alias).
> 3. ¿Qué herramienta usarías para ver toda la cadena de delegación DNS?
>    - R: `dig +trace dominio.com` muestra cada paso desde los root servers hasta el authoritative nameserver.
> 4. ¿Para qué sirve un registro TXT y qué casos de uso tiene?
>    - R: Almacena texto libre; se usa para SPF (anti-spam), DKIM (firma de emails), y verificación de dominio (Google, AWS).
> 5. ¿Qué pasa si el DNS de tu ISP está caído?
>    - R: No podés resolver dominios; solución: cambiar a DNS públicos como `8.8.8.8` (Google) o `1.1.1.1` (Cloudflare).
> 6. ¿Por qué el browser cachea DNS antes que el sistema operativo?
>    - R: Para evitar la latencia de una llamada al OS; si ya resolvió el dominio recientemente, usa su propia cache primero.

→ Ver [Tópico 1: Cómo funciona Internet](#1.1-cómo-funciona-internet) para el contexto general de redes.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.1-node-js-runtime) para ver cómo un servidor escucha en la IP que DNS resuelve.

---

## 1.6 Navegadores

> [MDN — What are browser developer tools?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Tools_and_setup/What_are_browser_developer_tools)

### Critical Rendering Path
- **DOM**: árbol de elementos HTML que el parser construye.
- **CSSOM**: árbol de estilos calculados.
- **Render Tree**: DOM + CSSOM combinados (solo nodos visibles).
- **Layout**: posición y tamaño de cada nodo.
- **Paint**: píxeles en pantalla.
- **Composite**: capas GPU para animaciones suaves.

### Storage del navegador
| Mecanismo | Persistencia | Capacidad | Tipo de datos |
|-----------|-------------|-----------|---------------|
| `localStorage` | Persistente | ~5 MB | Solo strings |
| `sessionStorage` | Por pestaña | ~5 MB | Solo strings |
| `cookies` | Configurable | ~4 KB | Strings, se envían en cada request |
| `IndexedDB` | Persistente | ~50+ MB | Objetos estructurados, asíncrono |

*En criollo:* El navegador es como una fábrica que transforma tu código HTML/CSS/JS en la página que ves. Primero lee el HTML y arma el esqueleto (DOM), después le pone la ropa (CSSOM), decide dónde va cada cosa (Layout), pinta los colores (Paint) y optimiza las animaciones (Composite). Todo esto pasa en milisegundos cada vez que cargás una página.

*Técnicamente:* Podés inspeccionar el Critical Rendering Path en DevTools:

```js
// Medir tiempo de renderizado
performance.getEntriesByType('paint');
// [{name: 'first-contentful-paint', startTime: 1234.5, ...}]

// Ver el DOM tree desde la consola
document.querySelectorAll('*').length;  // cantidad de nodos

// Comparar storage
localStorage.setItem('key', 'value');    // sincrónico, bloquea
IndexedDB.open('myDB', 1);               // asíncrono, no bloquea
```

Cookies con flags de seguridad:

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Path=/
```

| Flag | Protección |
|------|-----------|
| `HttpOnly` | JavaScript NO puede leer la cookie (previene XSS) |
| `Secure` | Solo se envía por HTTPS |
| `SameSite` | Controla envío cross-origin (Strict/Lax/None) |

> **Check de comprensión**
> 1. ¿Qué es el Critical Rendering Path y por qué importa para la performance?
>    - R: Es la secuencia DOM → CSSOM → Render Tree → Layout → Paint → Composite; optimizarlo reduce el tiempo hasta que el usuario ve la página.
> 2. ¿Cuándo usarías `localStorage` vs `sessionStorage` vs `cookies`?
>    - R: `localStorage` para datos persistentes (preferencias); `sessionStorage` para datos temporales de una pestaña; `cookies` para datos que el servidor necesita recibir en cada request.
> 3. ¿Qué flag de cookie previene ataques XSS y cómo funciona?
>    - R: `HttpOnly` — impide que JavaScript acceda a la cookie vía `document.cookie`, así un script malicioso no puede robarla.
> 4. ¿Por qué IndexedDB es mejor que localStorage para grandes volúmenes de datos?
>    - R: IndexedDB es asíncrono (no bloquea el main thread), soporta objetos estructurados, índices y queries, y tiene capacidad mucho mayor (~50+ MB).
> 5. ¿Qué diferencia hay entre Layout y Paint en el rendering?
>    - R: Layout calcula posición y tamaño de cada elemento; Paint dibuja los píxeles. Cambios de color solo requieren repaint; cambios de tamaño requieren relayout (más costoso).
> 6. ¿Qué hace el flag `SameSite=Strict` en una cookie?
>    - R: Impide que la cookie se envíe en requests cross-origin, protegiendo contra CSRF.

→ Ver [Tópico 3: Protocolo HTTP](#1.3-protocolo-http) para entender los headers de cookies que el navegador maneja.
→ Ver [Tópico 7: Frontend Core](../concepts/07-frontend-core.md#7.1-html-semántico) para el DOM que el navegador construye.

---

## 1.7 REST (Representational State Transfer)

*En criollo:* REST es un conjunto de reglas para diseñar APIs. La idea central es que todo es un "recurso" (usuarios, productos, órdenes) identificado por una URL, y usás los métodos HTTP como verbos: GET para leer, POST para crear, PUT para reemplazar, DELETE para borrar. Es como un sistema de archivos: cada archivo tiene una ruta y operaciones estándar para manipularlo.

*Técnicamente:* Ejemplo de API RESTful:

```http
# CRUD de usuarios
GET    /api/users          → Lista todos los usuarios
POST   /api/users          → Crea un nuevo usuario
GET    /api/users/42       → Obtiene usuario ID 42
PUT    /api/users/42       → Reemplaza usuario ID 42 completo
PATCH  /api/users/42       → Modifica parcialmente usuario ID 42
DELETE /api/users/42       → Elimina usuario ID 42

# Sub-recursos
GET    /api/users/42/orders    → Órdenes del usuario 42
POST   /api/users/42/orders    → Nueva orden para usuario 42
```

Comparación de paradigmas de API:

| Paradigma | Orientación | Transporte | Tipado |
|-----------|------------|------------|--------|
| REST | Recursos | HTTP | OpenAPI/Swagger |
| GraphQL | Queries | HTTP (POST) | Schema SDL |
| gRPC | Procedimientos | HTTP/2 + Protobuf | .proto files |

Principios REST:

1. **Client-Server**: separación de concerns.
2. **Stateless**: cada request es independiente.
3. **Cacheable**: las responses pueden cachearse.
4. **Uniform Interface**: métodos HTTP + URLs + status codes estándar.
5. **Layered System**: proxies, gateways, CDNs entre cliente y servidor.
6. **Code on Demand** (opcional): el servidor puede enviar código ejecutable (JS).

> **Check de comprensión**
> 1. ¿Qué significa que REST sea "stateless" y qué implica para el servidor?
>    - R: Cada request debe contener toda la información necesaria; el servidor no guarda estado entre requests, lo que facilita la escalabilidad horizontal.
> 2. ¿Cuándo usarías PUT vs PATCH?
>    - R: PUT cuando reemplazás el recurso completo (todos los campos); PATCH cuando solo modificás algunos campos.
> 3. ¿Qué es HATEOAS y por qué rara vez se implementa puro?
>    - R: Hypermedia As The Engine Of Application State: las responses incluyen links a acciones relacionadas. Es complejo de implementar y la mayoría de clientes ya saben las URLs.
> 4. ¿Cuál es la ventaja principal de REST sobre GraphQL?
>    - R: REST es más simple de cachear (HTTP caching nativo), más fácil de depurar (URLs directas) y tiene mejor soporte de herramientas.
> 5. ¿Qué status code devolverías al crear un recurso exitosamente?
>    - R: `201 Created`, con el header `Location` apuntando al nuevo recurso.
> 6. ¿Por qué las URLs de REST deberían usar sustantivos y no verbos?
>    - R: Porque los verbos ya están en los métodos HTTP; `/api/users` + POST es más limpio que `/api/createUser`.

→ Ver [Tópico 3: Protocolo HTTP](#1.3-protocolo-http) para los métodos y status codes que REST usa.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.2-apis-rest) para implementar una API REST en Node.js.

---

## 1.8 CORS (Cross-Origin Resource Sharing)

> [MDN — CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)

### Same-Origin Policy
- Un script de `ejemplo.com` no puede leer la respuesta de `api.otro.com`.
- **Origen** = protocolo + dominio + puerto. `http://localhost:3000` ≠ `http://localhost:3001`.

### Preflight
- El navegador manda `OPTIONS` antes del request real para verificar permisos CORS.
- Se dispara con requests "no simples": métodos distintos de GET/HEAD/POST, headers custom, `Content-Type: application/json`.

### Headers CORS
- `Access-Control-Allow-Origin` — qué orígenes pueden acceder.
- `Access-Control-Allow-Methods` — qué métodos permite el servidor.
- `Access-Control-Allow-Headers` — qué headers custom acepta.
- `Access-Control-Allow-Credentials` — permite cookies/auth headers cross-origin.
- Con `credentials: 'include'`, el servidor DEBE devolver `Access-Control-Allow-Credentials: true` Y un origen específico (no `*`).

*En criollo:* CORS es como la seguridad de un edificio: tu navegador (el guardia) no deja que un script de un dominio ajeno lea datos de otro dominio sin permiso. Si querés acceder a datos de otro dominio, el servidor de ese dominio tiene que decir explícitamente "sí, permito que este origen me lea". El "preflight" es como llamar por adelantado para preguntar si podés entrar.

*Técnicamente:* Ejemplo de request con preflight:

```js
// Frontend en http://localhost:3000
fetch('http://api.ejemplo.com/data', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  credentials: 'include',  // Envía cookies
  body: JSON.stringify({ query: 'users' })
});
```

El navegador primero envía:

```http
OPTIONS /data HTTP/1.1
Host: api.ejemplo.com
Origin: http://localhost:3000
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type
```

El servidor responde:

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: Content-Type
Access-Control-Allow-Credentials: true
```

Si el servidor no responde con los headers correctos, el navegador bloquea el request.

| Request simple | ¿Requiere preflight? |
|---------------|---------------------|
| GET sin headers custom | No |
| POST con `application/x-www-form-urlencoded` | No |
| POST con `application/json` | Sí |
| DELETE | Sí |
| GET con header `Authorization` | Sí |

> **Check de comprensión**
> 1. ¿Qué es la Same-Origin Policy y qué protege?
>    - R: Es una regla del navegador que impide que scripts de un origen lean datos de otro origen; protege contra robo de datos sensibles.
> 2. ¿Cuándo se dispara un preflight CORS y qué método HTTP usa?
>    - R: Se dispara con requests "no simples" (métodos custom, headers custom, JSON); usa el método OPTIONS.
> 3. ¿Por qué `Access-Control-Allow-Origin: *` no funciona con `credentials: 'include'`?
>    - R: Por seguridad: si permitís cookies cross-origin, debés especificar exactamente qué origen las puede enviar; el wildcard es demasiado permisivo.
> 4. ¿Qué diferencia hay entre un error CORS y un error de red?
>    - R: CORS es un bloqueo del navegador (el request llega al servidor pero la respuesta se bloquea); un error de red es que el servidor no responde.
> 5. ¿Cómo configurarías CORS en desarrollo local para un frontend en `localhost:3000` y backend en `localhost:4000`?
>    - R: En el backend: `Access-Control-Allow-Origin: http://localhost:3000` y permitir los métodos/headers necesarios.
> 6. ¿Qué es un "simple request" en CORS?
>    - R: Un request GET/HEAD/POST sin headers custom y con Content-Type `text/plain`, `multipart/form-data` o `application/x-www-form-urlencoded`.

→ Ver [Tópico 3: Protocolo HTTP](#1.3-protocolo-http) para entender los headers que CORS usa.
→ Ver [Tópico 4: HTTPS y TLS](#1.4-https-y-tls) porque el protocolo es parte del origen en la Same-Origin Policy.

---

## 1.9 Conceptos complementarios

*En criollo:* Son los detalles que marcan la diferencia entre una API que funciona y una API que funciona BIEN. Saber la diferencia entre URL y URI, cuándo usar query params vs path params, y qué significa idempotencia te va a ahorrar horas de debugging y decisiones de diseño incorrectas.

*Técnicamente:*

```
URI (Uniform Resource Identifier)
├── URL (Uniform Resource Locator) — identifica por ubicación
│   └── https://api.ejemplo.com/users/42?include=orders&page=2
│       ├── esquema: https
│       ├── host: api.ejemplo.com
│       ├── path: /users/42          ← identifica el recurso
│       └── query: include=orders&page=2  ← filtra/modifica la vista
└── URN (Uniform Resource Name) — identifica por nombre
    └── urn:isbn:0451450523
```

| Parámetro | Uso | Ejemplo | Idempotente |
|-----------|-----|---------|-------------|
| Path params | Identificar recurso | `/users/42` | GET, PUT, DELETE: sí |
| Query params | Filtrar/ordenar/paginar | `/users?role=admin&page=2` | GET: sí |
| Body | Crear/actualizar datos | `{ "name": "Juan" }` | POST: no, PUT: sí |

URL encoding de caracteres especiales:

```
Espacio     → %20
Ampersand   → %26
Slash       → %2F
Signo +     → %2B
Emoji 🚀    → %F0%9F%9A%80
```

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre URL y URI?
>    - R: URI es el término general (identificador de recurso); URL es un tipo de URI que identifica un recurso por su ubicación (dirección). Toda URL es URI, pero no toda URI es URL.
> 2. ¿Cuándo usarías path params vs query params?
>    - R: Path params para identificar un recurso específico (`/users/42`); query params para filtrar, ordenar o paginar (`/users?role=admin&page=2`).
> 3. ¿Qué significa que una operación HTTP sea idempotente?
>    - R: Que ejecutarla una vez o múltiples veces produce el mismo resultado en el servidor. GET, PUT y DELETE son idempotentes; POST no.
> 4. ¿Por qué es importante el URL encoding?
>    - R: Porque caracteres como `&`, `=`, `/` tienen significado especial en URLs; sin encoding, se confunden con la estructura de la URL.
> 5. ¿Qué es Base64 y cuándo se usa?
>    - R: Es un esquema de encoding que convierte datos binarios a texto ASCII; se usa en Data URIs, JWTs, y para embedir imágenes en HTML/CSS.
> 6. ¿Por qué POST no es idempotente?
>    - R: Porque cada POST crea un nuevo recurso; enviar el mismo POST 3 veces crea 3 recursos distintos, no 1.

→ Ver [Tópico 3: Protocolo HTTP](#1.3-protocolo-http) para la tabla de idempotencia de métodos.
→ Ver [Tópico 7: REST](#1.7-rest-representational-state-transfer) para cómo REST usa path y query params.

---
