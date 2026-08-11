# Setup — Clientes HTTP (Postman/Insomnia + curl)

> **Tópico**: 2 — Fundamentos de Programación (sección 2.5, debugging)
> **Objetivo**: poder inspeccionar y probar APIs REST desde el inicio, herramienta clave para testing manual y debugging.
> **Prerequisito**: WSL instalado (`setup-wsl.md`).

---

## ¿Qué herramienta elegir?

| Herramienta | Tipo | Ideal para |
|-------------|------|------------|
| **Postman** | GUI | Exploración visual de APIs, colecciones, entornos, equipos |
| **Insomnia** | GUI | Más liviano, buen soporte GraphQL, restful |
| **Thunder Client** | Extensión VS Code | No salís del editor, rápido para pruebas rápidas |
| **REST Client** | Extensión VS Code | Requests como archivos `.http` versionables en el repo |
| **curl** | Terminal | Scripting, debugging profundo, pipelines |

> Recomendación: instalá **Postman** (o Insomnia) PARA GUI + aprendé **curl** porque lo usás en cualquier servidor sin GUI.

---

## ¿Por qué clientes HTTP?

Cuando desarrollás APIs, necesitás probarlas SIN depender del frontend. Un cliente HTTP te permite enviar requests manuales, inspeccionar responses, verificar status codes, headers, y body. curl es la navaja suiza de la terminal — está en todo servidor Linux, incluyendo producción. Postman/Insomnia te dan una GUI para explorar, organizar colecciones, y compartir requests con el equipo. Sin estas herramientas, debuggear APIs es a ciegas.

---

## Checklist

### 1. curl (viene con WSL/Ubuntu)
```bash
# Probar que curl existe
curl --version

# Básico: pedir una API pública (ejemplo clásico)
curl https://api.github.com/users/octocat

# Con headers
curl -H "Accept: application/json" https://api.github.com/users/octocat

# Ver todo el intercambio (headers + body)
curl -v https://api.github.com/users/octocat

# POST con body JSON
curl -X POST https://api.ejemplo.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Leandro", "email": "leandro@ejemplo.com"}'
```
- [ ] `curl` responde con JSON de GitHub

### 2. Postman o Insomnia (GUI)
- [ ] Descargar [Postman](https://www.postman.com/downloads/) o [Insomnia](https://insomnia.rest/download) para Linux (en WSL, muchas veces es más cómodo instalarlas en Windows y usarlas por fuera — la API corre en WSL y se accede vía `localhost`)
- [ ] Crear tu primera colección "Pruebas locales"
- [ ] Probar una request GET a `http://localhost:3000/` (cuando tengas un server corriendo)
- [ ] Aprender: variables de entorno (`{{baseUrl}}`), headers, body, auth (Bearer token)

### 3. curl avanzado que vas a usar siempre
```bash
# Timeout (evitar colgarse)
curl --max-time 10 https://...

# Seguir redirecciones
curl -L https://...

# Guardar respuesta en archivo
curl -o respuesta.json https://api.ejemplo.com/users

# Solo status code
curl -o /dev/null -w "%{http_code}" https://api.ejemplo.com/users

# Con token de auth
curl -H "Authorization: Bearer <token>" https://api.ejemplo.com/users

# Enviar datos de formulario
curl -F "image=@foto.png" https://api.ejemplo.com/upload

# Probar endpoint POST que espera JSON (más legible con --json en curl 7.82+)
curl --json '{"name": "Leandro"}' https://api.ejemplo.com/users
```

### 4. Extensiones VS Code (opcional, recomendado)
- [ ] **Thunder Client** — para pruebas rápidas sin salir del editor
- [ ] **REST Client** — crear archivo `requests.http` versionable:
```http
### Listar usuarios
GET http://localhost:3000/users

### Crear usuario
POST http://localhost:3000/users
Content-Type: application/json

{
  "name": "Leandro",
  "email": "leandro@ejemplo.com"
}
```

---

## Verificación

```bash
# Después de estas pruebas, deberías poder:
curl https://api.github.com/users/octocat        # JSON válido
curl -o /dev/null -w "%{http_code}" https://google.com   # 200
curl -X POST --json '{"a": 1}' https://httpbin.org/post  # 200 con tu JSON de vuelta
```

**Si probaste al menos una API pública y viste el JSON → cliente HTTP listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `curl: (7) Failed to connect` | El servidor no está corriendo o el puerto es incorrecto. Verificá con `docker compose ps` o `netstat -tlnp` |
| `curl: (60) SSL certificate problem` | El certificado SSL no es válido. Usá `curl -k` para ignorar (solo en desarrollo) o `curl --cacert` para usar un CA específico |
| Postman no conecta a `localhost` en WSL | WSL2 expone puertos en localhost automáticamente. Si falla, reiniciá WSL: `wsl --shutdown` |
| Response JSON no se formatea en Postman | Click en "Pretty" en la parte inferior del body. Si el Content-Type no es `application/json`, Postman no lo detecta |
| `curl --json` no funciona | Tu versión de curl es anterior a 7.82. Usá `-H "Content-Type: application/json" -d '{"a":1}'` en su lugar |
| Request con auth falla | Verificá que el token no haya expirado. Usá `curl -v` para ver el exchange completo de headers |

---

## Recursos

- [curl — documentación oficial](https://curl.se/docs/)
- [Postman Learning Center](https://learning.postman.com/)
- [httpbin — servicio para testear requests](https://httpbin.org/)

## Preguntas de repaso

- **P:** ¿Cuál es la diferencia entre `curl` y Postman/Insomnia?
  **R:** curl es una herramienta de terminal, scripteable, disponible en todo servidor Linux (incluyendo producción). Postman/Insomnia son GUIs con features visuales como colecciones, entornos, y testing automatizado. Ambos son complementarios.

- **P:** ¿Qué hace el flag `-v` en curl y por qué es útil para debugging?
  **R:** Muestra el intercambio completo de HTTP: headers de request, headers de response, handshake TLS, y body. Es la forma más rápida de ver exactamente qué está pasando entre cliente y servidor.

- **P:** ¿Cómo enviás un POST con JSON usando curl?
  **R:** Con `curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' URL` o, en curl 7.82+, `curl --json '{"key":"value"}' URL`.

- **P:** ¿Para qué sirve `curl -o /dev/null -w "%{http_code}" URL`?
  **R:** Descarta el body (`-o /dev/null`) y muestra solo el status code HTTP. Es útil para health checks rápidos o scripts de monitoreo.

- **P:** ¿Qué ventaja tiene REST Client de VS Code sobre Postman?
  **R:** Los requests se guardan como archivos `.http` versionables en el repo. Todo el equipo puede ver, ejecutar, y modificar los requests sin salir del editor y sin depender de una cuenta de Postman.

- **P:** ¿Cómo probás un endpoint que requiere autenticación Bearer?
  **R:** Con `curl -H "Authorization: Bearer <token>" URL`. El token va en el header Authorization con el prefijo "Bearer".