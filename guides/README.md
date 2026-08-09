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

| Guía | Estado | Depende de |
|------|--------|------------|
| `02-programming/setup-wsl.md` | ✅ Lista | — |
| `02-programming/setup-terminal.md` | ✅ Lista | WSL |
| `02-programming/setup-git.md` | ✅ Lista | WSL, terminal |
| `02-programming/setup-ssh-github.md` | ✅ Lista | Git |
| `02-programming/setup-node.md` | ✅ Lista | WSL, terminal |
| `02-programming/setup-vscode.md` | ✅ Lista | WSL |
| `02-programming/setup-http-clients.md` | ✅ Lista | WSL |

### 03 — IA & Desarrollo Asistido

| Guía | Estado | Depende de |
|------|--------|------------|
| *(pendiente de definir con el tópico 3)* | ⬜ | — |

---

## Verificación general del entorno

Antes de avanzar al tópico 4 (Backend), tu entorno debería permitir:

```bash
# Desde la terminal Linux (WSL)
git --version          # git instalado
ssh -T git@github.com  # conexión SSH a GitHub funcionando
node --version         # node instalado vía nvm/fnm
code .                 # VS Code abre el directorio actual
```

Si todos esos comandos funcionan → el entorno del tópico 2 está listo.