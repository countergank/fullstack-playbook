# 07 — Frontend Core: Orden de Ejecución

> ⚠️ **El orden importa. HTML → CSS → JS → TS → DOM → a11y → Web APIs.** Cada guía construye sobre la anterior: el HTML semántico es el esqueleto, CSS lo estila, y sobre esa base van llegando JavaScript, TypeScript, DOM, accesibilidad y las APIs del navegador.

## Prerequisito

Navegador moderno (Chrome recomendado) + Node instalado (`setup-node.md` de 02-programming) — Node se usa SOLO en `setup-typescript.md`. Nada de Docker acá: todo son archivos locales que abrís en el navegador con doble clic.

## Paso a paso

1. **[setup-html.md](setup-html.md)** — HTML semántico: la estructura base de TU página del tópico 7, validada con W3C.
2. **[setup-css-moderno.md](setup-css-moderno.md)** — CSS moderno: reset, variables, Flexbox, Grid y responsive mobile-first.
3. **[setup-javascript.md](setup-javascript.md)** — JavaScript ES6+ en el navegador: `fetch` a una API pública + uso de la consola y Network.
4. **[setup-typescript.md](setup-typescript.md)** — TypeScript plano con `tsc`, sin bundler, y el HTML cargando el `.js` compilado.
5. **[setup-dom.md](setup-dom.md)** — Playground del DOM: crear, agregar y borrar elementos con event delegation.
6. **[setup-accesibilidad.md](setup-accesibilidad.md)** — axe DevTools + Lighthouse: la página del tópico 7 en verde.
7. **[setup-web-apis.md](setup-web-apis.md)** — `fetch`, `localStorage`, `IntersectionObserver` y `geolocation`.

> La guía de DevTools ([`01-web/setup-browser-devtools.md`](../01-web/setup-browser-devtools.md)) se ejecuta ANTES del setup-javascript — la consola y el tab Network son prerequisito para entender qué pasa cuando tu código hace `fetch`.

---

## Verificación final

- Abrí tu `index.html` del tópico 7 en Chrome y navegala con **Tab**: todo es operable y el foco es visible.
- **Lighthouse** (F12 → Lighthouse) da score de **Accessibility ≥ 90** y **axe DevTools** reporta **0 violaciones serias**.
- `npx tsc --noEmit` (en `~/proyectos/frontend-core`) no reporta errores.

**Si todo eso pasa → Frontend Core listo. ✅**