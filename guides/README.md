# Guides — Preparación del Entorno de Trabajo

> 📌 **La diferencia entre concepts y guides**:
> - `concepts/` → **QUÉ entender** (teoría, fundamentos).
> - `guides/` → **QUÉ PREPARAR y CÓMO** (setup del entorno, herramientas instaladas y configuradas).

Las guides son **checklists accionables** para preparar tu entorno a medida que avanzás en el roadmap. Cada una termina con una **verificación** ("si podés hacer esto, está listo").

---

## Cómo usar este directorio

1. Seguí el roadmap en orden. Los tópicos listan qué concepts leer y qué guides ejecutar.
2. Cuando llegues a un tópico con guides, ejecutá las que apliquen ANTES de practicar los conceptos (el entorno debe estar listo).
3. Cada guide asume la anterior completada (ej: WSL antes que terminal, Git antes que SSH).
4. Marcá cada checklist item cuando lo cumplas. Si algo falla, resolvé antes de seguir.

---

## Índice de guides

### 01 — Fundamentos de la Web

| Guía | Estado | Depende de |
|------|--------|------------|
| `01-web/setup-browser-devtools.md` | 🟡 Puede esperar al tópico de Frontend | — |
| `01-web/setup-local-https.md` | 🟡 Puede esperar al tópico de Backend | WSL |
| `01-web/setup-dns-check-tools.md` | 🟡 Muy simple, entra junto a WSL | WSL |

### 02 — Fundamentos de Programación

> 📋 Orden de ejecución: [`02-programming/README.md`](02-programming/README.md)

| Guía | Estado | Depende de |
|------|--------|------------|
| `02-programming/setup-wsl.md` | ✅ Lista | — |
| `02-programming/setup-docker.md` | ✅ Lista | WSL |
| `02-programming/setup-vscode.md` | ✅ Lista | WSL |
| `02-programming/setup-terminal.md` | ✅ Lista | WSL |
| `02-programming/setup-git.md` | ✅ Lista | WSL, terminal |
| `02-programming/setup-ssh-github.md` | ✅ Lista | Git |
| `02-programming/setup-node.md` | ✅ Lista | WSL, terminal |
| `02-programming/setup-http-clients.md` | ✅ Lista | WSL |

### 03 — IA & Desarrollo Asistido

> 📋 Orden de ejecución: [`03-ai/README.md`](03-ai/README.md)

| Guía | Estado | Depende de |
|------|--------|------------|
| `03-ai/setup-opencode.md` | ✅ Lista | Node, Git |
| `03-ai/setup-gentle-ai.md` | ✅ Lista | OpenCode |
| `03-ai/setup-profiles.md` | ✅ Lista | OpenCode, Gentle AI |
| `03-ai/setup-codegraph.md` | ✅ Lista | OpenCode, Gentle AI |
| `03-ai/setup-engram.md` | ✅ Lista | OpenCode |
| `03-ai/setup-context7.md` | ✅ Lista | OpenCode |

### 04 — Backend Core

> 📋 Orden de ejecución: [`04-backend/README.md`](04-backend/README.md)

| Guía | Estado | Depende de |
|------|--------|------------|
| `04-backend/setup-express-project.md` | ✅ Lista | Node, Docker, PostgreSQL |
| `04-backend/setup-env-config.md` | ✅ Lista | Express project |

### 05 — Frameworks Backend

> 📋 Sin guías por ahora — el concept 05 cubre Express, NestJS, Fastify, Prisma, logging, queues.

### 06 — Bases de Datos & Persistencia

> 📋 Orden de ejecución: [`06-databases/README.md`](06-databases/README.md)

| Guía | Estado | Depende de |
|------|--------|------------|
| `06-databases/setup-postgres.md` | ✅ Lista | Docker |
| `06-databases/setup-mongodb.md` | ✅ Lista | Docker |
| `06-databases/setup-redis.md` | ✅ Lista | Docker |

---

## Verificación general del entorno

### Antes del tópico 4 (Backend)

```bash
# Desde la terminal Linux (WSL)
git --version              # git instalado
ssh -T git@github.com      # conexión SSH a GitHub funcionando
docker --version           # Docker Desktop + WSL2
node --version             # node instalado vía nvm/fnm
code .                     # VS Code abre el directorio actual
```

Si todos esos comandos funcionan → el entorno del tópico 2 está listo.

### Antes del tópico 6 (Bases de Datos)

```bash
docker compose ps                                                     # los 3 "Up"
docker compose exec db psql -U dev -d fullstack_dev -c "SELECT 1"      # PostgreSQL
docker compose exec mongo mongosh --eval "db.runCommand({ ping: 1 })"   # MongoDB
docker compose exec redis redis-cli ping                                # Redis
```

Si los 3 responden → el entorno de bases de datos está listo.