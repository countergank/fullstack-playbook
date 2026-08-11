# Setup — CodeGraph (Exploración Estructural)

> **Tópico**: 3 — IA & Desarrollo Asistido
> **Objetivo**: indexar tu repositorio para que los agentes exploren símbolos, call paths y dependencias en una sola llamada en vez de un loop de Read/Grep.
> **Prerequisito**: OpenCode (`setup-opencode.md`), Gentle AI (`setup-gentle-ai.md`).

---

## ¿Por qué CodeGraph?

Cuando un agente necesita entender cómo funciona algo en tu código, sin CodeGraph tiene que hacer un loop de Read → Grep → Read → Grep — múltiples llamadas para reconstruir mentalmente la estructura. CodeGraph resuelve esto indexando todos los símbolos (funciones, clases, imports) y sus relaciones en una base SQLite. Con `codegraph_explore`, el agente obtiene el código relevante + call paths + blast-radius en UNA sola llamada. Es la diferencia entre leer 10 archivos para entender un flujo vs preguntar y recibir la respuesta estructurada.

---

## ¿Qué es CodeGraph?

CodeGraph indexa los símbolos y relaciones de tu código (funciones, clases, imports, callers, callees) en una base SQLite (`.codegraph/`). Cuando un agente necesita entender cómo funciona algo, `codegraph_explore` devuelve el código fuente relevante + los call paths + un resumen de blast-radius en UNA sola llamada — reemplazando un loop de Read + Grep + Read.

---

## Checklist

### 1. Inicializar CodeGraph en el repositorio
```bash
cd ~/proyectos/tu-repo
gentle-ai codegraph init --cwd .
```
- [ ] Aparece el directorio `.codegraph/` en la raíz del repo
- [ ] El indexado inicial toma unos segundos

### 2. Verificar que está indexado
```bash
codegraph status --cwd .
```
- [ ] Muestra cantidad de archivos, símbolos y edges indexados

### 3. Probar desde OpenCode
Dentro de OpenCode (asegurate de estar en el directorio del repo), preguntá:
```
¿Qué archivos definen la función `createUser` y quién la llama?
```
- [ ] El agente usa `codegraph_explore` y responde con precisión sin hacer múltiples Read/Grep

### 4. Mantenimiento (el index se actualiza solo)
- CodeGraph watcher sincroniza cambios automáticamente mientras editás.
- Solo si el watcher falla: `codegraph sync --cwd .`
- **Nunca** ejecutar `codegraph index` salvo recuperación explícita de corrupción.

---

## Verificación

```bash
gentle-ai codegraph init --cwd ~/proyectos/tu-repo
ls .codegraph/   # el directorio existe con archivos de índice
```
**Si `.codegraph/` existe con contenido → CodeGraph listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `gentle-ai codegraph init` falla | Verificar que estás en la raíz de un repositorio Git. CodeGraph necesita un repo válido para indexar |
| El agente no usa CodeGraph automáticamente | Verificar que el servidor MCP de CodeGraph está configurado en `opencode.json`. El agente lo usa para preguntas estructurales |
| CodeGraph reporta archivos stale | Ejecutar `codegraph sync --cwd .` para forzar re-indexado. El watcher normalmente lo hace automático |
| El indexado inicial tarda mucho | Es normal en repos grandes (>10K archivos). El indexado inicial es la única vez que tarda; después el watcher mantiene sync |
| `codegraph index` se ejecuta solo | No lo ejecutes manualmente salvo recuperación de corrupción. El watcher auto-sync es suficiente para uso normal |

---

## Recursos

- [CodeGraph CLI (upstream)](https://github.com/rustcore-dev/codegraph)

## Preguntas de repaso

- **P:** ¿Qué problema resuelve CodeGraph en el flujo de desarrollo con IA?
  **R:** Reemplaza loops de Read/Grep múltiples con una sola llamada que devuelve código fuente relevante, call paths, y blast-radius. Acelera la comprensión estructural del código.

- **P:** ¿Cómo se inicializa CodeGraph en un repositorio?
  **R:** Con `gentle-ai codegraph init --cwd .` desde la raíz del repo. Crea el directorio `.codegraph/` con el índice SQLite.

- **P:** ¿Por qué NO debés ejecutar `codegraph index` manualmente?
  **R:** Porque el watcher de CodeGraph sincroniza cambios automáticamente mientras editás. `codegraph index` solo se usa para recuperación explícita de corrupción del índice.

- **P:** ¿Qué comando usás si el watcher falla y CodeGraph reporta archivos stale?
  **R:** `codegraph sync --cwd .` para forzar una sincronización manual del índice con el estado actual del filesystem.

- **P:** ¿Qué tipo de preguntas debería responder CodeGraph?
  **R:** Preguntas estructurales: "¿quién llama a esta función?", "¿qué archivos definen esta clase?", "¿cuál es el flujo de X a Y?", "¿qué se rompe si cambio Z?".