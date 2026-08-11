# Setup — Gentle AI

> **Tópico**: 3 — IA & Desarrollo Asistido (sección 3.5-3.7)
> **Objetivo**: instalar la CLI de Gentle AI, configurar el agente por defecto, y conocer los comandos esenciales del ecosistema.
> **Prerequisito**: OpenCode (`setup-opencode.md`), Git y SSH (`setup-git.md`, `setup-ssh-github.md`).

---

## ¿Por qué Gentle AI?

Gentle AI es la capa de orquestación que hace que todo el ecosistema funcione junto. Sin Gentle AI, OpenCode es solo un editor con IA — no tiene SDD, no tiene code review estructurado, no tiene gestión de memoria desde CLI. Gentle AI provee los comandos (`sdd-status`, `sdd-continue`, `review status`, `engram search`) que te permiten interactuar con el ecosistema desde la terminal, sin depender exclusivamente de la UI. Es el puente entre el runtime (OpenCode) y tu flujo de trabajo profesional.

---

## ¿Qué es Gentle AI?

Gentle AI es la CLI que orquesta flujos SDD, code review, y gestión de agentes sobre OpenCode. Provee comandos como `gentle-ai sdd-continue`, `gentle-ai review status`, y `gentle-ai engram` para interactuar con el ecosistema desde la terminal sin abrir la UI.

---

## Checklist

### 1. Instalar Gentle AI
```bash
# Opción A: npm global
npm install -g gentle-ai@latest

# Opción B: Homebrew (Linux/macOS)
brew install gentleman-programming/gentle-ai/gentle-ai

# Verificar
gentle-ai --version        # debería responder 2.x.x
```

### 2. Configurar el agente por defecto
El archivo `~/.config/opencode/.gentle-ai-default-agent.json` le dice a la CLI qué agente usar cuando lanzás comandos fuera del TUI.
```bash
cat ~/.config/opencode/.gentle-ai-default-agent.json
```
- [ ] Debe mostrar `"state": "managed"`. Si no existe, crealo:
```json
{
  "schema": "gentle-ai.opencode-default-agent",
  "version": 1,
  "state": "managed"
}
```

### 3. Comandos esenciales que vas a usar
| Comando | Para qué |
|---------|----------|
| `gentle-ai --version` | Verificar instalación |
| `gentle-ai sdd-status <change>` | Estado del cambio SDD activo |
| `gentle-ai sdd-continue <change>` | Avanzar a la siguiente fase SDD |
| `gentle-ai review status --cwd <repo>` | Estado del ciclo de revisión |
| `gentle-ai engram search "termino"` | Buscar en memoria persistente |
| `gentle-ai engram current-project` | Detectar proyecto actual |
| `gentle-ai codegraph init --cwd <repo>` | Inicializar CodeGraph en un repo |

### 4. Verificar la conexión Gentle AI ↔ OpenCode
```bash
gentle-ai --version && opencode --version
```
- [ ] Ambas CLI responden sin error

### 5. Verificar el plugin TUI de Gentle AI
```bash
cat ~/.config/opencode/tui.json
```
- [ ] Deberías ver `"opencode-sdd-engram-manage"` y `"opencode-subagent-statusline"` listados como plugins activos.

---

## Verificación

```bash
gentle-ai --version           # 2.x.x
gentle-ai engram current-project --cwd ~/proyectos/tu-repo
```
**Si `gentle-ai --version` responde y `engram current-project` detecta tu proyecto → Gentle AI listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `gentle-ai: command not found` | Revisar instalación: `npm list -g gentle-ai` |
| `gentle-ai sdd-status` dice "no OpenSpec change found" | Estás en modo Engram (sin archivos). Usá `mem_search` en su lugar |
| Versión vieja (1.x) | `npm install -g gentle-ai@latest` o `brew upgrade gentle-ai` |
| Plugin no aparece en TUI | Verificar `tui.json` tiene los plugins. Reiniciar OpenCode |

---

## Recursos

- [Gentle AI — GitHub](https://github.com/gentleman-programming/gentle-ai)
- [Gentle AI — CLI Reference](https://github.com/gentleman-programming/gentle-ai/blob/main/docs/cli.md)

## Preguntas de repaso

- **P:** ¿Qué es Gentle AI y qué rol cumple en el ecosistema?
  **R:** Es la CLI que orquesta flujos SDD, code review, y gestión de agentes sobre OpenCode. Provee comandos de terminal para interactuar con el ecosistema sin abrir la UI.

- **P:** ¿Cómo instalás Gentle AI?
  **R:** Con `npm install -g gentle-ai@latest` o con Homebrew: `brew install gentleman-programming/gentle-ai/gentle-ai`.

- **P:** ¿Para qué sirve el archivo `.gentle-ai-default-agent.json`?
  **R:** Le dice a la CLI de Gentle AI qué agente usar cuando lanzás comandos fuera del TUI. Debe tener `"state": "managed"`.

- **P:** ¿Qué comando usás para ver el estado de un cambio SDD activo?
  **R:** `gentle-ai sdd-status <change>` desde la terminal, dentro del directorio del proyecto.

- **P:** ¿Qué plugins deberían aparecer en `tui.json` para que Gentle AI funcione correctamente?
  **R:** `opencode-sdd-engram-manage` y `opencode-subagent-statusline` como plugins activos.