# Setup — Perfiles SDD (Go y Zen-Free)

> **Tópico**: 3 — IA & Desarrollo Asistido (secciones 3.4-3.7)
> **Objetivo**: configurar los perfiles `go` y `zen-free` desde la TUI de Gentle AI, entender qué modelo usa cada fase SDD, y verificar que el switch de perfil funciona.
> **Prerequisito**: OpenCode (`setup-opencode.md`), Gentle AI (`setup-gentle-ai.md`).

---

## ¿Qué es un perfil SDD?

Un **perfil** es un conjunto de modelos asignados a cada fase del ciclo SDD. Al cambiar de perfil (presionando `Tab` en OpenCode), todos los sub-agentes SDD automáticamente usan los modelos de ese perfil. Esto te permite:

- **Desarrollo rápido y barato** con modelos gratuitos (zen-free)
- **Máxima calidad** con modelos pagos pero baratos (go)

---

## Checklist

### 1. Abrir la TUI de Gentle AI

```bash
gentle-ai
```
- [ ] Se abre la interfaz TUI. Desde acá gestionás perfiles, instalación de componentes y configuración.

### 2. Configurar los perfiles desde la TUI

Dentro de la TUI de `gentle-ai`:

1. Navegá a la sección de perfiles (buscá "Profiles", "SDD Profiles" o similar)
2. Deberías ver los perfiles disponibles:
   - **`go`** — modelos OpenCode Go (pago, calidad profesional)
   - **`zen-free`** — modelos gratuitos del provider `opencode`
3. Seleccioná cada perfil para revisar o ajustar los modelos asignados por fase
4. La TUI ya trae preconfigurados los modelos recomendados (tablas abajo). Si querés, podés personalizarlos

### 3. Modelos del perfil Go (`sdd-orchestrator-go`)

> **Costo**: pago pero barato. **Ideal para**: trabajo profesional, calidad consistente.

| Fase SDD | Modelo | Descripción |
|----------|--------|-------------|
| **orchestrator** | `opencode-go/deepseek-v4-pro` | Razonamiento profundo, coordinación de fases |
| **sdd-init** | `opencode-go/kimi-k2.6` | Detección de stack, análisis de arquitectura |
| **sdd-explore** | `opencode-go/deepseek-v4-flash` | Exploración rápida del repo |
| **sdd-propose** | `opencode-go/deepseek-v4-pro` | Propuestas arquitectónicas |
| **sdd-spec** | `opencode-go/qwen3.6-plus` | Output estructurado, specs detalladas |
| **sdd-design** | `opencode-go/deepseek-v4-pro` | Decisiones de arquitectura complejas |
| **sdd-tasks** | `opencode-go/qwen3.5-plus` | Desglose mecánico de tareas |
| **sdd-apply** | `opencode-go/qwen3.6-plus` | Generación de código, tool calling |
| **sdd-verify** | `opencode-go/qwen3.6-plus` | Validación contra specs |
| **sdd-archive** | `opencode-go/deepseek-v4-flash` | Archivado simple, velocidad > profundidad |

> Para usar Go necesitás una cuenta en [opencode.ai](https://opencode.ai) con créditos.

### 4. Modelos del perfil Zen-Free (`sdd-orchestrator-zen-free`)

> **Costo**: GRATUITO. **Ideal para**: práctica, prototipado, aprendizaje.

| Fase SDD | Modelo (slug opencode) | Velocidad | Nota |
|----------|----------------------|-----------|------|
| **orchestrator** | `opencode/deepseek-v4-flash-free` | ⚡ Rápido | Coordinación, no necesita razonamiento pesado |
| **sdd-init** | `opencode/nemotron-3-ultra-free` | 🐢 Lento (~45s) | Detección de stack, razonamiento profundo |
| **sdd-explore** | `opencode/deepseek-v4-flash-free` | ⚡ Rápido | Exploración multi-paso del repo |
| **sdd-propose** | `opencode/nemotron-3-ultra-free` | 🐢 Lento | Propuestas arquitectónicas |
| **sdd-spec** | `opencode/north-mini-code-free` | ⚡ Rápido | Output estructurado, specs |
| **sdd-design** | `opencode/nemotron-3-ultra-free` | 🐢 Lento | Decisiones de arquitectura |
| **sdd-tasks** | `opencode/north-mini-code-free` | ⚡ Rápido | Task breakdown (69 tok/s) |
| **sdd-apply** | `opencode/deepseek-v4-flash-free` | ⚡ Rápido | Code generation, tool calling |
| **sdd-verify** | `opencode/deepseek-v4-flash-free` | ⚡ Rápido | Test verification |
| **sdd-archive** | `opencode/north-mini-code-free` | ⚡ Rápido | Tarea trivial, velocidad > profundidad |
| **sdd-onboard** | `opencode/big-pickle` | — | Onboarding guiado |

> **Regla práctica**: `deepseek-v4-flash-free` para la mayoría de las tareas, `nemotron-3-ultra-free` solo para fases que requieren razonamiento pesado (init, propose, design), `north-mini-code-free` para tareas mecánicas (tasks, archive).

### 5. Cómo cambiar de perfil (el atajo que usás todos los días)

1. Abrí OpenCode: `opencode`
2. **Presioná `Tab`** — se abre el selector de perfiles
3. Elegí uno con las flechas y Enter:
   - `sdd-orchestrator-go` → modelos go (pago, calidad)
   - `sdd-orchestrator-zen-free` → modelos gratuitos opencode
4. Todos los sub-agentes SDD que lances ahora usarán los modelos de ese perfil

---

## Comparación rápida de perfiles

| Criterio | Go | Zen-Free |
|----------|-----|----------|
| Costo | Pago (barato) | **Gratis** |
| Calidad | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Velocidad | ⚡ Rápido | ⚡ Rápido (nemotron lento) |
| Requiere internet | Sí | Sí |
| Mejor para | Trabajo profesional | Práctica, aprendizaje |

---

## Verificación

```bash
# 1. Abrí la TUI y confirmá que ves los dos perfiles
gentle-ai

# 2. Abrí OpenCode, presioná Tab, seleccioná zen-free
opencode

# 3. Preguntale al agente:
# "¿Qué perfil estoy usando y qué modelo tenés asignado?"
```
**Si OpenCode te responde con el modelo correcto del perfil seleccionado → Perfiles listos. ✅**

---

## Recursos

- [OpenCode — Profiles](https://opencode.ai/docs/profiles)
- [Gentle AI — TUI](https://github.com/gentleman-programming/gentle-ai)