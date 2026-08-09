# Setup — Perfiles SDD (Go y Zen-Free)

> **Tópico**: 3 — IA & Desarrollo Asistido (secciones 3.4-3.7)
> **Objetivo**: configurar y entender los perfiles `sdd-orchestrator-go` y `sdd-orchestrator-zen-free` de OpenCode, con los modelos asignados por fase SDD.
> **Prerequisito**: OpenCode (`setup-opencode.md`), Gentle AI (`setup-gentle-ai.md`). Ollama (`setup-ollama.md`) opcional.

---

## ¿Qué es un perfil SDD?

Un **perfil** es un conjunto de modelos asignados a cada fase del ciclo SDD. Al cambiar de perfil (presionando `Tab` en OpenCode), todos los sub-agentes SDD automáticamente usan los modelos de ese perfil. Esto te permite:

- **Desarrollo rápido y barato** con modelos gratuitos (zen-free)
- **Máxima calidad** con modelos pagos pero baratos (go)
- **Privacidad total** con modelos locales (ollama-local)

---

## Perfil 1: OpenCode Go (`sdd-orchestrator-go`)

> **Costo**: pago pero barato (modelos asiáticos con precios muy competitivos).
> **Ideal para**: trabajo profesional, calidad consistente, fases que requieren razonamiento real.

### Modelos asignados por fase

| Fase SDD | Modelo | Descripción |
|----------|--------|-------------|
| **orchestrator** | `opencode-go/deepseek-v4-pro` | Razonamiento profundo, coordinación de fases, validación de artefactos |
| **sdd-init** | `opencode-go/kimi-k2.6` | Detección de stack, análisis de arquitectura del proyecto |
| **sdd-explore** | `opencode-go/deepseek-v4-flash` | Exploración rápida del repo, lectura masiva de archivos |
| **sdd-propose** | `opencode-go/deepseek-v4-pro` | Propuestas arquitectónicas, decisiones de diseño a alto nivel |
| **sdd-spec** | `opencode-go/qwen3.6-plus` | Output estructurado, especificaciones detalladas con contexto de código |
| **sdd-design** | `opencode-go/deepseek-v4-pro` | Decisiones de arquitectura complejas, tradeoffs |
| **sdd-tasks** | `opencode-go/qwen3.5-plus` | Desglose mecánico de tareas, listas atómicas |
| **sdd-apply** | `opencode-go/qwen3.6-plus` | Generación de código, tool calling, implementación |
| **sdd-verify** | `opencode-go/qwen3.6-plus` | Validación contra specs, ejecución de tests |
| **sdd-archive** | `opencode-go/deepseek-v4-flash` | Archivado simple, velocidad > profundidad |

### Configuración

El perfil go viene preconfigurado con la instalación de OpenCode. Para verificarlo:

1. Abrí OpenCode: `opencode`
2. Presioná `Tab` para ver los perfiles disponibles
3. Seleccioná `sdd-orchestrator-go`
4. Ejecutá cualquier comando SDD — los sub-agentes usarán automáticamente los modelos de esta tabla

> Para usar Go necesitás una cuenta en [opencode.ai](https://opencode.ai) con créditos.

---

## Perfil 2: OpenCode Zen-Free (`sdd-orchestrator-zen-free`)

> **Costo**: GRATUITO (modelos del provider `opencode` con tier free).
> **Ideal para**: práctica, prototipado, proyectos personales, aprendizaje.

### Modelos asignados por fase

| Fase SDD | Modelo (slug opencode) | Velocidad | Nota |
|----------|----------------------|-----------|------|
| **orchestrator** | `opencode/deepseek-v4-flash-free` | ⚡ Rápido | Coordinación, no necesita razonamiento pesado |
| **sdd-init** | `opencode/nemotron-3-ultra-free` | 🐢 Lento (~45s) | Detección de stack, necesita razonamiento profundo |
| **sdd-explore** | `opencode/deepseek-v4-flash-free` | ⚡ Rápido | Exploración multi-paso del repo |
| **sdd-propose** | `opencode/nemotron-3-ultra-free` | 🐢 Lento | Propuestas arquitectónicas, razonamiento pesado |
| **sdd-spec** | `opencode/north-mini-code-free` | ⚡ Rápido | Output estructurado, specs detalladas |
| **sdd-design** | `opencode/nemotron-3-ultra-free` | 🐢 Lento | Decisiones de arquitectura complejas |
| **sdd-tasks** | `opencode/north-mini-code-free` | ⚡ Rápido | Task breakdown mecánico (69 tok/s) |
| **sdd-apply** | `opencode/deepseek-v4-flash-free` | ⚡ Rápido | Code generation, tool calling |
| **sdd-verify** | `opencode/deepseek-v4-flash-free` | ⚡ Rápido | Test verification, assertions |
| **sdd-archive** | `opencode/north-mini-code-free` | ⚡ Rápido | Tarea trivial, velocidad > profundidad |
| **sdd-onboard** | `opencode/big-pickle` | — | Modelo principal para onboarding guiado |

### Modelos gratuitos disponibles (provider `opencode`)

Estos son los slugs verificados con tool calling funcional:

| Slug opencode | Modelo real | Tipo |
|---------------|------------|------|
| `opencode/deepseek-v4-flash-free` | DeepSeek V4 Flash | Rápido, multi-propósito |
| `opencode/nemotron-3-ultra-free` | NVIDIA Nemotron 3 Ultra 550B | Razonamiento pesado |
| `opencode/north-mini-code-free` | Cohere North Mini Code | Código rápido (69 tok/s) |
| `opencode/laguna-s-2.1-free` | Laguna S 2.1 | Alternativa rápida |
| `opencode/ling-3.0-flash-free` | Ling 3.0 Flash | Alternativa rápida |
| `opencode/mimo-v2.5-free` | MiMo V2.5 | Alternativa rápida |
| `opencode/big-pickle` | Big Pickle | Modelo principal |

> **Regla práctica**: `deepseek-v4-flash-free` para la mayoría de las tareas, `nemotron-3-ultra-free` para las que requieren razonamiento profundo (init, propose, design), `north-mini-code-free` para tareas mecánicas y rápidas (tasks, archive).

---

## Cómo cambiar de perfil (el atajo que usás todos los días)

1. Abrí OpenCode: `opencode`
2. **Presioná `Tab`** — se abre el selector de perfiles
3. Elegí uno con las flechas y Enter:
   - `sdd-orchestrator-go` → modelos go (pago, calidad)
   - `sdd-orchestrator-zen-free` → modelos gratuitos opencode
   - `sdd-orchestrator-ollama-local` → modelos locales Ollama
4. Todos los sub-agentes SDD que lances ahora usarán los modelos de ese perfil

---

## Comparación rápida de perfiles

| Criterio | Go | Zen-Free | Ollama Local |
|----------|-----|----------|--------------|
| Costo | Pago (barato) | **Gratis** | Gratis (electricidad) |
| Calidad | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ (modelos chicos) |
| Velocidad | ⚡ Rápido | ⚡ Rápido (nemotron lento) | 🐢 Depende de tu GPU |
| Privacidad | Cloud | Cloud | **Total (local)** |
| Requiere internet | Sí | Sí | No |
| Mejor para | Trabajo profesional | Práctica, prototipado | Offline, datos sensibles |

---

## Verificación

```bash
opencode
# Dentro de OpenCode:
# 1. Presioná Tab
# 2. Deberías ver los 3 perfiles (go, zen-free, ollama-local)
# 3. Seleccioná zen-free
# 4. Escribí: "Decime qué perfil estoy usando y qué modelo"
```
**Si OpenCode te responde correctamente desde el perfil seleccionado → Perfiles listos. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Perfil go no aparece | Necesitás cuenta en opencode.ai. Verificá tu suscripción |
| Perfil zen-free usa modelos que no responden | Los modelos gratuitos pueden tener rate limits. Cambiá a otro modelo free de la lista |
| `nemotron-3-ultra-free` tarda demasiado | Es normal (modelo de 550B params). Usalo solo para fases que realmente necesitan razonamiento pesado |
| Quiero crear un perfil nuevo (ej: openrouter-free) | Ver guía avanzada de perfiles. Básicamente: duplicar la sección de agentes en `opencode.json` con el nuevo `variant` |

---

## Recursos

- [OpenCode — Profiles](https://opencode.ai/docs/profiles)
- [Lista de modelos gratuitos opencode (actualizada)](https://opencode.ai/models)