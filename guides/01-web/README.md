# Guides — 01 Fundamentos de la Web

> 📌 Estás a punto de ver guías de la Web con tiempos distintos: `setup-browser-devtools.md` ya está lista y se ejecuta antes del tópico 7. Las otras dos pueden esperar a los tópicos donde se necesitan (Frontend o Backend). Ejecutalas cuando llegues ahí, salvo que quieras prepararlas antes.

| Guía | Estado | Cuándo ejecutar |
|------|--------|-----------------|
| `setup-browser-devtools.md` | ✅ Lista | Ya — ejecutala antes del tópico 7 (Frontend Core) |
| `setup-local-https.md` | ⬜ Pendiente | Antes del tópico 4 (Backend Core) |
| `setup-dns-check-tools.md` | ⬜ Pendiente | Junto a WSL, si querés practicar DNS temprano |

---

## Plan breve de cada guía

### `setup-browser-devtools.md`
- ✅ Lista: ya existe como guía completa ([`setup-browser-devtools.md`](setup-browser-devtools.md)). Ejecutala ANTES del tópico 7 (Frontend Core): cubre Network tab, Console, Application y Sources.
- Prerequisito: navegador moderno.

### `setup-local-https.md`
- Qué: correr un servidor local con HTTPS (mkcert), editar `/etc/hosts`, acceso a tu app por nombre de dominio local.
- Prerequisito: WSL + un proyecto que sirva HTTP (Node/Express).

### `setup-dns-check-tools.md`
- Qué: instalar `dig`, `nslookup`, `whois` para practicar resolución DNS y verificar dominios.
- Prerequisito: WSL (los paquetes están en `dnsutils` / `whois`).

---

> ⚠️ **Nota**: solo quedan pendientes `setup-local-https.md` (backlog, antes del tópico de Backend) y `setup-dns-check-tools.md` (opcional, junto a WSL).