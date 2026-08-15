# Setup — Docker en Producción (Multi-stage + Compose)

> **Tópico**: 9 — DevOps & Deployment (sección 9.1)
> **Objetivo**: empaquetar TU app en una imagen de producción con Dockerfile multi-stage, orquestarla con `docker-compose.yml`, y dejarla liviana con optimización de imagen (`.dockerignore`, base alpine, cache por capas).
> **Prerequisito**: Docker funcionando (`setup-docker.md` de 02-programming) + una app con `npm run build` que genere artefactos (`dist/` o `.next/`).

---

## ¿Por qué?

En el tópico 2 Docker te levantaba bases de datos ajenas. Acá el objetivo es distinto: convertir TU app en una imagen autocontenida que corra idéntica en tu máquina, en staging y en producción. El multi-stage build separa el entorno de compilación (pesado, con compiladores) del de ejecución (liviano, solo artefactos), y las optimizaciones de imagen reducen tamaño, aceleran builds y achican la superficie de ataque.

---

## Checklist

### 1. Crear el proyecto de práctica

- [ ] Creá `~/proyectos/mi-app` y adentro una app Node con un `npm run build` (podés copiar la app del tópico 8).
- [ ] Verificá que `npm run build` genere una carpeta de artefactos (`dist/` o `.next/`).

### 2. Escribir el `.dockerignore`

- [ ] Creá `.dockerignore` en la raíz (junto al `Dockerfile`):

```dockerignore
node_modules
.git
.env
.env.*
dist
.next
npm-debug.log
Dockerfile
.dockerignore
```

> Sin `.dockerignore`, `node_modules` local entra al build context y hace el build lento y pesado. Además, `.env` con secretos quedaría DENTRO de la imagen.

### 3. Escribir el `Dockerfile` multi-stage

- [ ] Creá el `Dockerfile` con dos stages: `builder` (compila) y `runner` (solo artefactos):

```dockerfile
# ---- Stage 1: builder ----
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# ---- Stage 2: runner ----
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/package.json /app/package-lock.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
USER node
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

- [ ] Ajustá `dist/server.js` al entrypoint real de tu app.

### 4. Construir la imagen

- [ ] `docker build -t mi-app:prod .`
- [ ] Verificá que termine sin errores y anotá el tamaño: `docker images mi-app`

### 5. Agregar health check a la app

- [ ] Asegurate de que tu app responda a `GET /health` con 200 (si no lo tiene, agregalo). El `HEALTHCHECK` del Dockerfile depende de ese endpoint.

### 6. Escribir `docker-compose.yml` de producción

- [ ] Creá `docker-compose.yml` con la app + base de datos + límites y política de reinicio:

```yaml
services:
  app:
    build: .
    restart: unless-stopped
    environment:
      DATABASE_URL: ${DATABASE_URL}
    ports:
      - "3000:3000"
    depends_on:
      db:
        condition: service_healthy
    deploy:
      resources:
        limits:
          memory: 512M

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
```

- [ ] Creá un `.env` con `DATABASE_URL` y `POSTGRES_PASSWORD` (solo para práctica local).

### 7. Levantar y probar

- [ ] `docker compose up -d --build`
- [ ] `docker compose ps` → app y db con estado healthy.

---

## Verificación

```bash
docker images mi-app                    # tamaño de la imagen (debería ser < ~200MB)
docker compose ps                       # app y db "Up" y healthy
docker compose exec app wget -qO- http://localhost:3000/health   # responde ok
```

**Si la imagen pesa menos de ~200MB, ambos servicios están healthy y `/health` responde → imagen de producción lista. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Build muy lento en cada cambio | Copiá `package.json` ANTES del código fuente para cachear la capa de `npm ci` |
| Imagen de más de 1GB | Usá base `alpine` y multi-stage; asegurate de copiar solo artefactos al runner |
| `HEALTHCHECK` falla siempre | Verificá que la app exponga `/health` y que `wget` exista en la imagen (alpine lo trae con `busybox`) |
| Error `COPY` no encuentra archivo | Revisá que `.dockerignore` no esté excluyendo archivos que el `COPY` necesita |
| Contenedor no reinicia si se cae | Agregá `restart: unless-stopped` al servicio |
| El contenedor corre como root | Agregá `USER node` después de copiar archivos (los `COPY` de archivos propios no necesitan root) |
| `npm ci` falla en el runner | Asegurate de copiar `package-lock.json`; sin lockfile `npm ci` no funciona |

---

## Recursos

- [Docker Docs — Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker Docs — Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
- [Docker Docs — Best practices for writing Dockerfiles](https://docs.docker.com/build/building/best-practices/)
- [Docker Compose — Compose file reference](https://docs.docker.com/reference/compose-file/)

## Preguntas de repaso

- **P:** ¿Qué es un multi-stage build y qué ventaja concreta tiene para producción?
  **R:** Separa el build en un stage `builder` (con compiladores) y un `runner` (solo artefactos). La imagen final no lleva compiladores ni devDependencies, por lo que es mucho más chica y con menor superficie de ataque.

- **P:** ¿Por qué copiar `package.json` antes que el código fuente acelera los builds?
  **R:** Porque Docker cachea por capas. Si `package.json` no cambió, la capa del `npm ci` se reutiliza y no se reinstalan dependencias. Si copiás el código primero, cualquier cambio invalida toda la cache.

- **P:** ¿Para qué sirve el archivo `.dockerignore`?
  **R:** Para excluir archivos del build context (`node_modules`, `.git`, `.env`). Sin él, el build es lento y pesado, y archivos sensibles como `.env` quedarían dentro de la imagen.

- **P:** ¿Por qué `npm ci` se usa en el Dockerfile en vez de `npm install`?
  **R:** Porque `npm ci` instala desde el lockfile de forma reproducible y determinística, y borra `node_modules` antes. Es el estándar para builds reproducibles en CI y Docker.

- **P:** ¿Qué diferencia hay entre `CMD` y `HEALTHCHECK` en un Dockerfile?
  **R:** `CMD` define el comando que arranca la app. `HEALTHCHECK` define un comando de diagnóstico que Docker ejecuta periódicamente para saber si el contenedor está sano y decidir si reiniciarlo.

- **P:** ¿Por qué el contenedor de producción no debería correr como root?
  **R:** Para reducir la superficie de ataque: si un atacante escapa del proceso, no tiene privilegios de root. Se usa `USER node` para correr como un usuario sin privilegios.
