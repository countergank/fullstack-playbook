# 03 — IA & Desarrollo Asistido: Orden de Ejecución

> ⚠️ **El orden importa.** Cada guía asume que la anterior fue completada.

## Paso a paso

1. **[setup-opencode.md](setup-opencode.md)** — Instalar OpenCode. Verificar Node + nvm primero.
2. **[setup-gentle-ai.md](setup-gentle-ai.md)** — Instalar Gentle AI CLI. El comando `gentle-ai` abre la TUI.
3. **[setup-profiles.md](setup-profiles.md)** — Configurar perfiles Go y Zen-Free desde la TUI de `gentle-ai`.
4. **[setup-engram.md](setup-engram.md)** — Instalar y activar Engram desde la TUI de `gentle-ai`.
5. **[setup-context7.md](setup-context7.md)** — Verificar que Context7 está activo como MCP.
6. **[setup-codegraph.md](setup-codegraph.md)** — Indexar tu repositorio para exploración estructural.

---

## Verificación final

Abrí OpenCode (`opencode`) y ejecutá estas verificaciones:

```
1. ¿Qué perfil estoy usando? (debe responder con go o zen-free)
2. Guardá en Engram: "prueba de entorno IA"
3. Buscá en Engram: "prueba de entorno"
4. Buscá en Context7 la documentación de Express para crear un servidor básico
```

**Si los 4 comandos funcionan → el entorno del tópico 3 está completo. ✅**
