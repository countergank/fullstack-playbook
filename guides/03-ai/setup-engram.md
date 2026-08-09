# Setup — Engram (Memoria Persistente)

> **Tópico**: 3 — IA & Desarrollo Asistido (sección 3.6-3.8)
> **Objetivo**: instalar y activar Engram desde la TUI de Gentle AI, y verificar que la memoria persistente funciona.
> **Prerequisito**: OpenCode (`setup-opencode.md`), Gentle AI (`setup-gentle-ai.md`).

---

## ¿Qué es Engram?

Engram es una base de datos de memoria persistente que los agentes de IA consultan entre sesiones. Sin Engram, cada conversación con IA arranca de cero. Con Engram, la IA "recuerda" tus decisiones, bugs resueltos, patrones y contexto entre sesiones.

---

## Checklist

### 1. Abrir la TUI de Gentle AI

```bash
gentle-ai
```
- [ ] Se abre la interfaz TUI de Gentle AI. Desde acá podés instalar tanto Gentle AI como Engram.

### 2. Instalar Engram desde la TUI

Dentro de la TUI de `gentle-ai`:
1. Navegá a la sección de instalación/gestión de componentes (buscá opciones como "Install", "Setup" o "Engram")
2. Seleccioná la opción para **instalar / activar Engram**
3. La TUI se encarga de:
   - Registrar Engram como servidor MCP en `~/.config/opencode/opencode.json`
   - Activar el plugin `opencode-sdd-engram-manage` en `~/.config/opencode/tui.json`
   - Arrancar el servidor Engram

> Si la TUI ofrece opciones como "Enable Engram", "Start Engram", o "Connect Engram" — seleccionalas en orden.

### 3. Verificar que Engram está activo

Una vez instalado desde la TUI, verificá de dos formas:

**Desde la TUI:** el statusline debería mostrar un indicador de que Engram está conectado.

**Desde OpenCode (la verificación definitiva):** abrí OpenCode y preguntá algo que requiera memoria:

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
| `gentle-ai` no abre la TUI | Verificar que `gentle-ai` está instalado (`setup-gentle-ai.md`). Ejecutar `gentle-ai --version` |
| La TUI no muestra opción de Engram | Puede que ya esté instalado. Verificar: `grep -o '"engram"' ~/.config/opencode/opencode.json` |
| El agente no llama a `mem_save` | Es normal si no detecta decisiones nuevas. Forzá un save manual como en el paso 4 |
| `mem_search` no encuentra nada | Primera sesión = sin historial. Los saves se acumulan con el uso |
| Memoria entre proyectos se mezcla | Los agentes usan `project` para filtrar. Si cambiás de proyecto, usá `mem_current_project` para verificar el contexto |

---

## Recursos

- [Engram — GitHub](https://github.com/gentleman-programming/engram)
- [Gentle AI — CLI + TUI](https://github.com/gentleman-programming/gentle-ai)