# 13 — Performance: Orden de Ejecución

> **El orden importa. Bundle Analysis → Database Indexing & N+1.** Primero medís y achicás el frontend (el treemap te muestra qué pesar de más estás mandando al navegador); después atacás el backend, que es donde viven los dos cuellos de botella más silenciosos: el N+1 y las queries sin índice. Empezar midiendo evita que optimices a ciegas, y arreglar el frontend primero te da la win rápida que valida el proceso antes de tocar la base de datos.

## Prerequisito

- Concepto 13 completo (leelo antes de ejecutar estas guías).
- Para bundle analysis: un proyecto frontend con Vite (o webpack) y `package.json` propio.
- Para indexing/N+1: una app con base de datos (Postgres o MongoDB) y un ORM (Prisma de ejemplo).
- Recomendado: concepto 6 (bases de datos) para el contexto de índices y queries.

## Paso a paso

1. **[setup-bundle-analysis.md](setup-bundle-analysis.md)** — visualizá tu bundle con `rollup-plugin-visualizer`/`webpack-bundle-analyzer`, detectá dependencias infladas y reducilas con tree-shaking y code splitting. Te deja el frontend medido y achicado.
2. **[setup-db-indexing-n1.md](setup-db-indexing-n1.md)** — eliminá el N+1 (eager loading con `include`, batching con DataLoader) y acelerá queries con índices, verificando con `EXPLAIN ANALYZE`. Te deja el backend sin las dos causas más comunes de lentitud.

---

## Verificación final

- `npm run build` genera `stats.html`/`report.html` y el treemap muestra cada dependencia con su peso gzip; tras eliminar una librería inflada, el bundle gzip bajó.
- El log de queries muestra 2 queries en vez de 101 (el N+1 desapareció).
- `EXPLAIN ANALYZE` dice "Index Scan" en vez de "Seq Scan" para tus queries de filtro.

**Si todo eso pasa → Performance listo. ✅**
