# Setup — Engram (Memoria Persistente)

> **Tópico**: 3 — IA & Desarrollo Asistido (sección 3.6-3.8)
> **Objetivo**: verificar que Engram está corriendo como MCP server, entender sus comandos, y configurar tu proyecto para memoria persistente.
> **Prerequisito**: OpenCode (`setup-opencode.md`).

---

## ¿Qué es Engram?

Engram es una base de datos de memoria persistente que los agentes de IA consultan entre sesiones. Sin Engram, cada conversación con IA arranca de cero. Con Engram, la IA "recuerda" tus decisiones, bugs resueltos, patrones y contexto entre sesiones.

---

## Checklist

### 1. Verificar que Engram está activo como MCP
Engram corre como servidor MCP (Model Context Protocol) conectado a OpenCode.
```bash
# Ver si engram está en los MCP servers de opencode:
grep -o '"engram"' ~/.config/opencode/opencode.json | head -1
```
- [ ] Si devuelve `"engram"`, está configurado. Si no, seguir paso de configuración manual.

### 2. Configuración manual (si no detectaste Engram)
Agregar a `~/.config/opencode/opencode.json` en la sección `mcp`:
```jsonc
{
  "mcp": {
    "engram": {
      "type": "local",
      "command": "npx",
      "args": ["@gentleman-programming/engram-mcp-server"]
    }
  }
}
```
Reiniciar OpenCode.

### 3. Verificar que Engram responde
Dentro de OpenCode, ejecutá un comando que use memoria:
```
¿Qué proyecto estoy usando ahora?
```
- [ ] El agente debería llamar `mem_current_project` y detectar tu proyecto.

### 4. Comandos esenciales de Engram (vía agente)

| Operación | Qué hace | Cuándo se usa |
|-----------|----------|---------------|
| `mem_save` | Guardar un hecho/descubrimiento/decision | Automático — el agente lo llama proactivamente |
| `mem_search` | Buscar en todas las sesiones anteriores | Cuando preguntás "¿cómo resolvimos X?" |
| `mem_context` | Recuperar sesiones recientes | Al inicio de sesión |
| `mem_session_summary` | Guardar resumen al final de la sesión | Automático al cerrar |
| `mem_get_observation` | Leer contenido completo de un resultado | Cuando `mem_search` devuelve resultados truncados |

### 5. Entender el ciclo de memoria
```
Inicio de sesión → mem_context (últimas sesiones)
Durante el trabajo → mem_save (decisiones, bugs, descubrimientos)
Ante una pregunta → mem_search (¿ya hicimos esto antes?)
Fin de sesión → mem_session_summary (resumen para la próxima)
```

### 6. CLI de Engram (si querés interactuar directo desde terminal)
```bash
# Instalar CLI global (si no vino con gentle-ai)
npm install -g @gentleman-programming/engram

# Buscar desde terminal
gentle-ai engram search "roadmap fullstack"

# Detectar proyecto actual
gentle-ai engram current-project --cwd /home/leandrojaviercepeda/countergank/fullstack-playbook
```

---

## Verificación

Dentro de OpenCode, preguntá:
```
¿Tenés memoria de lo que hicimos antes? Dame un resumen de las últimas decisiones.
```
**Si el agente responde con información de sesiones anteriores → Engram funcionando. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Agente no llama a `mem_save` | Es normal si no hay decisiones nuevas. Forzá un save: "Guardá en Engram que el stack es Node + Express" |
| `mem_search` no encuentra nada | Primera sesión = sin memoria aún. Los saves se acumulan con el tiempo |
| "Engram MCP not connected" | Revisar que `engram` esté en `opencode.json` → `mcp`. Reiniciar OpenCode |
| Memoria entre proyectos se mezcla | Usar `project` explícito en `mem_save` y `mem_search` |

---

## Recursos

- [Engram — GitHub](https://github.com/gentleman-programming/engram)
- [Engram MCP Server](https://www.npmjs.com/package/@gentleman-programming/engram-mcp-server)