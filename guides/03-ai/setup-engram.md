# Setup — Engram (Memoria Persistente)

> **Tópico**: 3 — IA & Desarrollo Asistido (sección 3.6-3.8)
> **Objetivo**: instalar y activar Engram desde la TUI de Gentle AI, y verificar que la memoria persistente funciona.
> **Prerequisito**: OpenCode (`setup-opencode.md`), Gentle AI (`setup-gentle-ai.md`).

---

## ¿Qué es Engram?

Engram es una base de datos de memoria persistente que los agentes de IA consultan entre sesiones. Sin Engram, cada conversación con IA arranca de cero. Con Engram, la IA "recuerda" tus decisiones, bugs resueltos, patrones y contexto entre sesiones.

---

## Checklist

### 1. Verificar que el plugin TUI de Engram está activo

El plugin `opencode-sdd-engram-manage` es el que gestiona la instalación y activación de Engram desde la UI de OpenCode.

```bash
cat ~/.config/opencode/tui.json
```
- [ ] Debe incluir `"opencode-sdd-engram-manage"` en la lista de plugins.

Si no está, agregalo:
```json
{
  "$schema": "https://opencode.ai/tui.json",
  "plugin": [
    "opencode-subagent-statusline",
    "opencode-sdd-engram-manage"
  ],
  "theme": "gentleman-kanagawa"
}
```
Reiniciá OpenCode.

### 2. Instalar/activar Engram desde la TUI

1. Abrí OpenCode: `opencode`
2. Buscá el panel o menú de plugins TUI (depende de tu tema, generalmente en la barra lateral o con un atajo como `Ctrl+E` o `Ctrl+P` → "Engram")
3. Seleccioná la opción para **instalar / activar Engram** que provee el plugin `opencode-sdd-engram-manage`
4. El plugin se encarga de registrar Engram como servidor MCP en `opencode.json` y arrancarlo

> Si el plugin ofrece opciones como "Enable Engram", "Start Engram", o "Connect Engram" — seleccionalas en orden.

### 3. Verificar que Engram está activo

Una vez instalado, el plugin muestra un indicador de estado. Verificá de dos formas:

**Desde la TUI:** el statusline (provisto por `opencode-subagent-statusline`) debería mostrar un ícono o texto indicando que Engram está conectado.

**Desde el agente (la verificación definitiva):** dentro de OpenCode, preguntá algo que requiera memoria:
```
¿Qué proyecto estoy usando ahora?
```
- [ ] El agente debe llamar a `mem_current_project` y responder con el nombre del proyecto

### 4. Primer guardado de prueba

Dentro de OpenCode, forzá un guardado para confirmar que la escritura también funciona:
```
Guardá en Engram que el stack actual usa Node.js con Express y PostgreSQL. El proyecto se llama fullstack-playbook.
```
- [ ] El agente confirma que la memoria fue guardada (`mem_save` exitoso)

### 5. Comandos de memoria que los agentes usan automáticamente

| Operación | Qué hace | Cuándo |
|-----------|----------|--------|
| `mem_save` | Guardar un hecho, decisión o descubrimiento | Automático — el agente lo llama proactivamente tras decisiones |
| `mem_search` | Buscar en sesiones anteriores | Cuando preguntás "¿cómo resolvimos X?" |
| `mem_context` | Recuperar contexto de sesiones recientes | Al inicio de cada sesión |
| `mem_session_summary` | Guardar resumen estructurado | Al cerrar una sesión (automático) |
| `mem_get_observation` | Leer contenido completo | Cuando `mem_search` devuelve resultados truncados |

---

## Verificación

Dentro de OpenCode, ejecutá estas dos pruebas:

```
1. Guardá en Engram: "prueba de verificación de instalación"
2. Ahora buscá en Engram la palabra "prueba de verificación"
```
- [ ] Primer comando confirma guardado
- [ ] Segundo comando encuentra la memoria que acabás de guardar

**Si Engram guarda y encuentra → memoria persistente activa. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| El plugin `opencode-sdd-engram-manage` no aparece | Verificar `tui.json`. Si no existe, crearlo con el contenido del paso 1 |
| El agente no llama a `mem_save` | Es normal si no detecta decisiones nuevas. Forzá un save manual como en el paso 4 |
| Engram aparece como "disconnected" en el statusline | Reinstalar desde el plugin TUI. Si persiste, verificar que `engram` esté en la sección `mcp` de `opencode.json` |
| `mem_search` no encuentra nada | Primera sesión = sin historial. Los saves se acumulan con el uso |
| Memoria entre proyectos se mezcla | Los agentes usan `project` para filtrar. Si cambiás de proyecto, usá `mem_current_project` para verificar el contexto |

---

## Recursos

- [Engram — GitHub](https://github.com/gentleman-programming/engram)
- [Gentle AI — TUI Plugins](https://github.com/gentleman-programming/gentle-ai)