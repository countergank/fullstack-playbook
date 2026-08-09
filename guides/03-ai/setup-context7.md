# Setup — Context7 (Documentación Actualizada)

> **Tópico**: 3 — IA & Desarrollo Asistido (sección 3.12)
> **Objetivo**: integrar Context7 como fuente de documentación actualizada para que los agentes consulten librerías en tiempo real sin alucinar APIs inventadas.
> **Prerequisito**: OpenCode (`setup-opencode.md`).

---

## ¿Qué es Context7?

Context7 es un servicio MCP que indexa documentación oficial de librerías y frameworks (React, Next.js, Prisma, Tailwind, Express, etc.) y la expone para que los agentes de IA la consulten. Sin Context7, el agente depende de su conocimiento de entrenamiento (que puede estar desactualizado). Con Context7, busca la doc VIVA y actualizada antes de responder.

---

## Checklist

### 1. Verificar que Context7 está en tu configuración
```bash
grep -o '"context7"' ~/.config/opencode/opencode.json | head -1
```
- [ ] Si devuelve `"context7"`, ya está configurado. Si no, seguir paso 2.

### 2. Configuración manual (si no existe)
Agregar a `~/.config/opencode/opencode.json` en la sección `mcp`:
```jsonc
{
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com"
    }
  }
}
```
Reiniciar OpenCode.

### 3. Verificar que Context7 responde
Dentro de OpenCode, preguntá algo que requiera doc actualizada:
```
¿Cuál es la API de Prisma para crear un registro con relaciones anidadas?
```
- [ ] El agente debería llamar a `resolve-library-id` o `query-docs` de Context7

### 4. Librerías que más vas a consultar
| Librería | Context7 ID |
|----------|------------|
| React | `/reactjs/react.dev` |
| Next.js | `/vercel/next.js` |
| Prisma | `/prisma/docs` |
| Tailwind CSS | `/tailwindlabs/tailwindcss` |
| Express | `/expressjs/express` |
| Node.js | `/nodejs/node` |
| PostgreSQL | `/postgres/docs` |
| MDN Web Docs | `/mdn/content` |

---

## Verificación

Dentro de OpenCode:
```
Buscá en Context7 la documentación de Prisma para `create` con `connect`.
```
**Si el agente responde con información de la documentación oficial de Prisma (no inventada) → Context7 listo. ✅**

---

## Recursos

- [Context7](https://context7.com/)
- [Context7 MCP](https://mcp.context7.com)