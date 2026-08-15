# 12 — Security: Orden de Ejecución

> **El orden importa. Seguridad Express → Hashing bcrypt.** Primero blindás tu API con los middlewares base (helmet, CORS, rate limiting, validación) que cierran los vectores de OWASP A01/A03/A05/A07; después endurecés la autenticación con hashing de passwords. La primera guía te deja el server "blindado por fuera", la segunda te deja el almacenamiento de credenciales "blindado por dentro".

## Prerequisito

- Concepto 12 completo (leelo antes de ejecutar estas guías).
- Una app Express corriendo con un `package.json` propio (podés usar la del tópico 4 o 8).
- Node.js y npm funcionando (`guides/02-programming/`).
- Proyecto de práctica: `~/proyectos/mi-app` (o tu app existente).

## Paso a paso

1. **[setup-seguridad-express.md](setup-seguridad-express.md)** — helmet, CORS con lista blanca, rate limiting, validación de input y security headers en tu API Express. Cierra los vectores más comunes antes de tocar auth.
2. **[setup-hashing-bcrypt.md](setup-hashing-bcrypt.md)** — hashing de passwords con bcrypt (y argon2id como alternativa): hash, verify, salt rounds y comparación timing-safe. Endurece el registro/login.

---

## Verificación final

- `curl -I http://localhost:3000/` muestra los headers de seguridad (`Content-Security-Policy`, `X-Frame-Options`, `Strict-Transport-Security`…) seteados por helmet.
- Un request desde un origen no permitido recibe un error CORS, y más de N requests por minuto reciben `429 Too Many Requests`.
- Registrar un usuario guarda un hash (nunca la password en texto plano) y `bcrypt.compare` valida el login; cambiar un caracter de la password hace fallar la comparación.

**Si todo eso pasa → Seguridad listo. ✅**
