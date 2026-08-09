# Setup — OpenCode

> **Tópico**: 3 — IA & Desarrollo Asistido (sección 3.7-3.9)
> **Objetivo**: instalar OpenCode, configurarlo con tu proveedor preferido, y verificar que podés lanzar agentes.
> **Prerequisito**: Node.js (`setup-node.md`), Git (`setup-git.md`).

---

## ¿Qué es OpenCode?

OpenCode es el runtime donde corren los agentes de Gentle AI. Provee herramientas (Read, Write, Bash, Task, etc.), integración con modelos (OpenAI, Anthropic, OpenRouter, Ollama, opencode), MCP (Model Context Protocol) para memoria y documentación, y perfiles para cambiar entre distintas configuraciones de modelos con un atajo de teclado.

---

## Antes de empezar — ¿Tenés Node.js?

OpenCode es un paquete npm, así que necesitás Node.js. Pero NO cualquier instalación: necesitás un **gestor de versiones** para poder cambiar de Node según el proyecto sin romper nada.

**nvm (Node Version Manager)** es la herramienta estándar para esto. Te permite instalar múltiples versiones de Node en paralelo y switchear entre ellas con un solo comando. Si aún no lo tenés, seguí primero la guía completa:

> 👉 **`guides/02-programming/setup-node.md`** — instalación de nvm + Node LTS + pnpm.

Una vez que tengas Node, verificá:

```bash
node --version   # deberías ver v22.x.x (LTS)
npm --version    # viene con Node
```

**Si `node --version` responde con una versión LTS → seguí adelante.** Si no, volvé a `setup-node.md`.

---

## Checklist

### 1. Instalar OpenCode
```bash
# Instalar CLI globalmente
npm install -g @opencode-ai/cli@latest

# Verificar
opencode --version
```

### 2. Verificar proveedores y modelos
```bash
# Ver la versión instalada (1.18+ incluye modelos gratuitos)
opencode --version
```
- [ ] Si la versión es 1.18 o superior, tenés acceso a `opencode/deepseek-v4-flash-free` y otros modelos gratuitos.

### 3. Lanzar OpenCode en modo terminal
```bash
opencode
```
- [ ] Se abre la UI de TUI. El prompt debería estar listo para escribir.
- [ ] `Ctrl+Tab` o `Tab` para cambiar de contexto (archivos, agentes, perfiles).

### 4. Configurar el proveedor `opencode` (modelos gratuitos y go)
OpenCode ya registra el proveedor `opencode` como nativo — no necesitas API key para los modelos gratuitos. Para los modelos **go** (pago pero barato), necesitás tu cuenta de OpenCode Go.
- [ ] Verificar tu cuenta Go (si querés usarla): https://opencode.ai

### 5. Atajo clave: cambiar de perfil
- [ ] Dentro de OpenCode, presioná **Tab** para abrir el selector de perfil
- [ ] Deberías ver los perfiles disponibles: `sdd-orchestrator-go` y `sdd-orchestrator-zen-free`

### 6. Primer comando de prueba
```
Escribí un "Hola mundo" en Node.js
```
- [ ] OpenCode responde con código funcional y puede ejecutarlo vía Bash.

---

## Verificación

```bash
node --version       # LTS (v22.x)
opencode --version   # 1.x.x
```

**Si `node` y `opencode` responden con sus versiones → OpenCode listo. ✅**

---

## Configuración recomendada (`~/.config/opencode/opencode.json`)

El archivo se crea automáticamente la primera vez que lanzás OpenCode. No necesitás tocarlo salvo para agregar providers custom. La guía `setup-profiles.md` cubre la configuración de perfiles (go, zen-free).

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `opencode: command not found` | Revisar `npm list -g @opencode-ai/cli`. Si lo instalaste con nvm, asegurate de que nvm esté activo |
| Modelos free no aparecen | `opencode` 1.18+ incluye los modelos gratuitos del provider `opencode`. Actualizá: `npm install -g @opencode-ai/cli@latest` |
| `Permission denied` al hacer Bash | OpenCode tiene un sistema de permisos. Permitir herramientas desde la UI o configurar permisos en `opencode.json` |
| La UI TUI no arranca | Verificar terminal: necesitás un terminal moderno (Windows Terminal, iTerm2, Kitty) con soporte de colores truecolor |

---

## Recursos

- [OpenCode](https://opencode.ai)
- [OpenCode CLI npm](https://www.npmjs.com/package/@opencode-ai/cli)