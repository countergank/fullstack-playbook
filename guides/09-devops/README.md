# 09 — DevOps & Deployment: Orden de Ejecución

> **El orden importa. Docker → CI/CD → Monitoreo/Logging.** Primero empaquetás tu app en una imagen de producción (Docker); después automatizás build/test/deploy de esa imagen (CI/CD); recién al final le agregás observabilidad para operarla (monitoreo y logging). Cada guía asume la anterior completada.

## Prerequisito

- Concepto 09 completo (leelo antes de ejecutar estas guías).
- Docker Desktop funcionando (`guides/02-programming/setup-docker.md`).
- Una app con build definido (un `npm run build` que genere `dist/` o `.next/`), por ejemplo la del tópico 8.
- Proyecto de práctica: `~/proyectos/mi-app` (o tu app existente).

## Paso a paso

1. **[setup-docker.md](setup-docker.md)** — Dockerfile multi-stage de producción + `docker-compose.yml` + optimización de imagen (`.dockerignore`, base alpine, cache por capas). Empaqueta tu app y correla local con Docker.
2. **[setup-ci-cd-github-actions.md](setup-ci-cd-github-actions.md)** — Pipeline de GitHub Actions: build → test → deploy, con secrets cifrados y environment protegido. Requiere la imagen y el repo ya funcionando.
3. **[setup-monitoring-logging.md](setup-monitoring-logging.md)** — Health checks + logging estructurado con correlation ID + métricas básicas. Le da observabilidad a lo que deployaste en las dos guías anteriores.

---

## Verificación final

- `docker build` termina con una imagen final de menos de ~200MB (multi-stage + alpine) y `docker compose up` levanta la app con su health check respondiendo.
- Un push a `main` dispara el workflow de GitHub Actions y los jobs `build` y `test` pasan en verde.
- `GET /health` responde 200; los logs son JSON estructurado con `requestId`; y podés filtrar un request puntual por su correlation ID.

**Si todo eso pasa → DevOps & Deployment listo. ✅**
