# Setup — HTTPS Local con mkcert

> **Tópico**: 1 — Fundamentos Web
> **Objetivo**: Configurar HTTPS en desarrollo local con certificados válidos para evitar warnings del navegador y probar funcionalidades que requieren contexto seguro
> **Prerequisito**: Tener Node.js y npm instalados (ver [Setup — Node.js](../../02-programming/setup-node.md)) y WSL o Linux nativo

---

## ¿Por qué HTTPS local?

El navegador trata `http://localhost` como un contexto seguro parcial, pero hay funcionalidades que **exigen HTTPS real**: cookies con el flag `Secure`, Service Workers, APIs como `getUserMedia` (cámara/micrófono), y la Policy de mismo origen (Same-Origin Policy) en ciertos escenarios. Además, si tu app en producción usa HTTPS, desarrollar en HTTP te oculta bugs que solo aparecen cuando el protocolo cambia.

`mkcert` resuelve esto generando certificados **válidos para tu máquina local** — no el típico certificado autofirmado que el navegador rechaza con una pantalla roja de advertencia. Crea una CA (Certificate Authority) local que tu sistema confía, y con ella firma certificados para `localhost` o dominios que mapees en `/etc/hosts`.

---

## Checklist

### 1. Instalar mkcert

**En Ubuntu/WSL:**
```bash
sudo apt install -y libnss3-tools
```

Descargá el binario desde [GitHub](https://github.com/FiloSottile/mkcert):
```bash
# Para Linux amd64:
curl -JLO "https://dl.filippo.io/mkcert/latest?for=linux/amd64"
chmod +x mkcert-v*-linux-amd64
sudo mv mkcert-v*-linux-amd64 /usr/local/bin/mkcert
```

**En macOS:**
```bash
brew install mkcert
```

- [ ] `mkcert --version` responde con un número de versión

### 2. Crear la CA (Certificate Authority) local

```bash
mkcert -install
```

Este comando crea una CA local y la agrega al almacén de confianza de tu sistema (y de Firefox si está instalado). **Solo se hace una vez.**

- [ ] El comando termina sin errores
- [ ] La CA quedó en `$(mkcert -CAROOT)` (generalmente `~/.local/share/mkcert`)

### 3. Generar certificado para localhost

Creá un directorio para los certificados en tu proyecto:

```bash
mkdir -p ~/proyectos/mi-app/certs
cd ~/proyectos/mi-app/certs
mkcert localhost 127.0.0.1 ::1
```

Esto genera dos archivos:
- `localhost.pem` — el certificado público
- `localhost-key.pem` — la clave privada

- [ ] Los archivos `localhost.pem` y `localhost-key.pem` existen en el directorio `certs/`

### 4. Configurar Express con HTTPS

En tu proyecto Node.js, creá un servidor que use los certificados:

```js
const https = require('https');
const fs = require('fs');
const path = require('path');

const options = {
  key: fs.readFileSync(path.join(__dirname, 'certs', 'localhost-key.pem')),
  cert: fs.readFileSync(path.join(__dirname, 'certs', 'localhost.pem'))
};

const server = https.createServer(options, (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('HTTPS funcionando!\n');
});

server.listen(3000, () => {
  console.log('Servidor HTTPS corriendo en https://localhost:3000');
});
```

- [ ] El servidor arranca sin errores con `node server.js`

### 5. Verificar en el navegador

Abrí `https://localhost:3000` en tu navegador.

- [ ] El navegador muestra el **candado verde** (o el ícono de conexión segura) en la barra de direcciones
- [ ] Al hacer clic en el candado → "La conexión es segura" o "Certificate is valid"
- [ ] La página muestra "HTTPS funcionando!"

---

## Verificación

```bash
# 1. El servidor responde por HTTPS:
curl -k https://localhost:3000
# Debería responder: "HTTPS funcionando!"

# 2. Verificar el certificado:
echo | openssl s_client -connect localhost:3000 2>/dev/null | openssl x509 -noout -subject -dates
# Debería mostrar subject= y fechas válidas
```

**Si el navegador muestra candado verde en `https://localhost:3000` → HTTPS local listo. ✅**

---

## Problemas comunes

| Problema | Causa | Solución |
|----------|-------|----------|
| El navegador muestra "Conexión no segura" | La CA no se instaló en el sistema | Ejecutá `mkcert -install` de nuevo y reiniciá el navegador |
| `mkcert` no encontrado | El binario no está en el PATH | Verificá con `which mkcert`; si no responde, reinstalá o agregá al PATH |
| ERR_SSL_VERSION_INTERFERENCE | El servidor no está usando los certificados correctos | Verificá que las rutas a `localhost.pem` y `localhost-key.pem` sean correctas |
| El certificado expiró | Los certificados de mkcert duran ~2 años por defecto | Regenerá: borrá los `.pem` viejos y volvé a ejecutar `mkcert localhost` |
| Firefox sigue mostrando warning | Firefox usa su propio almacén de certificados | Ejecutá `mkcert -install` con Firefox cerrado; si persiste, configurá `security.enterprise_roots.enabled` en `about:config` |
| `EACCES: permission denied` al leer los certs | Los permisos del archivo son restrictivos | `chmod 644 localhost.pem` y `chmod 600 localhost-key.pem` |

---

## Recursos

- [mkcert — GitHub](https://github.com/FiloSottile/mkcert)
- [Node.js HTTPS API](https://nodejs.org/api/https.html)
- [MDN — Secure Contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts)

---

## Preguntas de repaso

- **P:** ¿Por qué no basta con usar `http://localhost` para desarrollar?
  **R:** Porque funcionalidades como cookies `Secure`, Service Workers, y APIs como `getUserMedia` exigen HTTPS. Además, desarrollar en HTTP oculta bugs que aparecen en producción cuando el protocolo cambia.

- **P:** ¿Qué ventaja tiene `mkcert` sobre un certificado autofirmado?
  **R:** `mkcert` crea una CA local que tu sistema confía, así que los certificados que genera son válidos para el navegador. Un certificado autofirmado muestra warnings de seguridad que interrumpen el desarrollo.

- **P:** ¿Cada cuánto hay que regenerar los certificados?
  **R:** Los certificados de mkcert tienen una validez de aproximadamente 2 años. Cuando expiren, borrá los archivos `.pem` y volvé a ejecutar `mkcert localhost`.

- **P:** ¿Para qué sirve `mkcert -install` y cuándo hay que ejecutarlo?
  **R:** Crea la CA local y la agrega al almacén de confianza del sistema. Se ejecuta **una sola vez** por máquina. Si cambiás de computadora, hay que repetirlo.

- **P:** ¿Qué archivos genera `mkcert localhost` y cuál es la diferencia entre ellos?
  **R:** Genera `localhost.pem` (certificado público, se comparte con el servidor y el cliente) y `localhost-key.pem` (clave privada, nunca se comparte y debe protegerse). El servidor usa ambos para establecer la conexión HTTPS.

- **P:** ¿Cómo verificás que el certificado es válido desde la terminal?
  **R:** Con `echo | openssl s_client -connect localhost:3000 2>/dev/null | openssl x509 -noout -subject -dates`, que muestra el subject y las fechas de validez del certificado.
