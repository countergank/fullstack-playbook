# Guides — 01 Fundamentos de la Web

> 📌 Estas guías ya fueron identificadas, pero **pueden esperar** a los tópicos donde se necesitan (Frontend o Backend). Ejecutalas cuando llegues ahí, salvo que quieras prepararlas antes.

| Guía | Estado | Cuándo ejecutar |
|------|--------|-----------------|
| `setup-browser-devtools.md` | ⬜ Pendiente | Antes del tópico 7 (Frontend Core) |
| `setup-local-https.md` | ⬜ Pendiente | Antes del tópico 4 (Backend Core) |
| `setup-dns-check-tools.md` | ⬜ Pendiente | Junto a WSL, si querés practicar DNS temprano |

---

## Plan breve de cada guía

### `setup-browser-devtools.md`
- Qué: dominar Network tab (requests, timing, throttling), Console (filtros, debug), Application (cookies, storage, service workers), Sources (breakpoints).
- Prerequisito: navegador + tópico de Frontend.

### `setup-local-https.md`
- Qué: correr un servidor local con HTTPS (mkcert), editar `/etc/hosts`, acceso a tu app por nombre de dominio local.
- Prerequisito: WSL + un proyecto que sirva HTTP (Node/Express).

### `setup-dns-check-tools.md`
- Qué: instalar `dig`, `nslookup`, `whois` para practicar resolución DNS y verificar dominios.
- Prerequisito: WSL (los paquetes están en `dnsutils` / `whois`).

---

> ⚠️ **Nota**: NO crear los archivos todavía — esperá a cada tópico para ejecutar la guía con su objetivo claro. Este README existe para recordarte qué viene.