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

## Recursos

- [curl — documentación oficial](https://curl.se/docs/)
- [Postman Learning Center](https://learning.postman.com/)
- [httpbin — servicio para testear requests](https://httpbin.org/)