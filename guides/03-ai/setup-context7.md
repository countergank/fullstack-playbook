# Setup — Context7 (Documentación Actualizada)

> **Tópico**: 3 — IA & Desarrollo Asistido (sección 3.12)
> **Objetivo**: integrar Context7 como fuente de documentación actualizada para que los agentes consulten librerías en tiempo real sin alucinar APIs inventadas.
> **Prerequisito**: OpenCode (`setup-opencode.md`).

---

## ¿Por qué Context7?

Sin Context7, el agente de IA depende exclusivamente de su conocimiento de entrenamiento, que tiene fecha de corte y puede estar desactualizado. Esto genera alucinaciones: el agente inventa APIs, funciones o comportamientos que ya no existen o nunca existieron. Context7 resuelve esto indexando documentación oficial en tiempo real y exponiéndola como un servidor MCP. Cuando el agente necesita saber cómo usar Prisma, React, o cualquier librería, consulta Context7 primero y responde con información verificada, no inventada. Es la diferencia entre un asistente que "cree saber" y uno que "verifica antes de responder".

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

## Problemas comunes

| Problema | Solución |
|----------|----------|
| El agente no llama a Context7 automáticamente | Verificá que el servidor MCP está configurado en `opencode.json`. Reiniciar OpenCode después de agregar la config |
| Context7 no encuentra una librería | No todas las librerías están indexadas. Podés pedirle a Context7 que la indexe usando `resolve-library-id` con el nombre del repo de GitHub |
| Respuesta de Context7 lenta | Es normal la primera vez porque resuelve el library ID. Las consultas subsecuentes son más rápidas gracias al caching |
| Error de conexión a `mcp.context7.com` | Verificar conexión a internet. Si el servicio está caído, el agente fallback a su conocimiento de entrenamiento (con riesgo de alucinación) |
| Context7 devuelve doc desactualizada | Reportalo en el repo de Context7. El índice se actualiza periódicamente pero puede haber delay |

---

## Recursos

- [Context7](https://context7.com/)
- [Context7 MCP](https://mcp.context7.com)

## Preguntas de repaso

- **P:** ¿Qué problema resuelve Context7 en el flujo de desarrollo con IA?
  **R:** Evita alucinaciones del agente al proveer documentación oficial actualizada de librerías y frameworks, en vez de depender del conocimiento de entrenamiento desactualizado.

- **P:** ¿Cómo se configura Context7 en OpenCode?
  **R:** Agregando un entry en la sección `mcp` de `~/.config/opencode/opencode.json` con `"type": "remote"` y `"url": "https://mcp.context7.com"`.

- **P:** ¿Qué pasa si Context7 no tiene indexada una librería que necesitás?
  **R:** Podés pedirle al agente que use `resolve-library-id` para intentar indexarla, o fallback a WebFetch para buscar la documentación manualmente.

- **P:** ¿Por qué es importante verificar que Context7 responde antes de usarlo en trabajo real?
  **R:** Porque si el servidor no está activo o la config está mal, el agente no tendrá acceso a doc actualizada y podría alucinar APIs sin que te des cuenta.

- **P:** ¿Qué formato tiene el ID de una librería en Context7?
  **R:** Sigue el formato `/org/proyecto`, por ejemplo `/vercel/next.js` para Next.js o `/prisma/docs` para Prisma.