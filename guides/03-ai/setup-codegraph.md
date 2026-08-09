# Setup — CodeGraph (Exploración Estructural)

> **Tópico**: 3 — IA & Desarrollo Asistido
> **Objetivo**: indexar tu repositorio para que los agentes exploren símbolos, call paths y dependencias en una sola llamada en vez de un loop de Read/Grep.
> **Prerequisito**: OpenCode (`setup-opencode.md`), Gentle AI (`setup-gentle-ai.md`).

---

## ¿Qué es CodeGraph?

CodeGraph indexa los símbolos y relaciones de tu código (funciones, clases, imports, callers, callees) en una base SQLite (`.codegraph/`). Cuando un agente necesita entender cómo funciona algo, `codegraph_explore` devuelve el código fuente relevante + los call paths + un resumen de blast-radius en UNA sola llamada — reemplazando un loop de Read + Grep + Read.

---

## Checklist

### 1. Inicializar CodeGraph en el repositorio
```bash
cd ~/proyectos/fullstack-playbook
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
gentle-ai codegraph init --cwd ~/proyectos/fullstack-playbook
ls .codegraph/   # el directorio existe con archivos de índice
```
**Si `.codegraph/` existe con contenido → CodeGraph listo. ✅**

---

## Recursos

- [CodeGraph CLI (upstream)](https://github.com/rustcore-dev/codegraph)