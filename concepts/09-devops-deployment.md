# 9. DevOps & Deployment

> Objetivo: llevar tu app de "funciona en mi máquina" a "funciona en producción, con confianza, para miles de usuarios". Acá aprendés a empaquetar con Docker (ya lo viste en desarrollo en el tópico 2, ahora en producción), automatizar el pipeline con CI/CD, elegir dónde corre (AWS/Vercel/Railway), separar entornos, y no volar a ciegas: monitoreo, logging centralizado y secrets gestionados como corresponde.

---

## 9.1 Docker

*En criollo:* En el tópico 2 usaste Docker para levantar bases de datos sin ensuciar tu máquina. Acá la pregunta es otra: **¿cómo empaquetás TU aplicación** para que corra igual en producción? La respuesta es un `Dockerfile` — una receta que construye una **imagen** de tu app con todo adentro (código, dependencias, runtime). Esa imagen la subís a un registry y cualquier servidor la corre igual. "En mi máquina funciona" deja de existir porque ahora TODO corre en la misma caja.

*Técnicamente:* Una imagen se construye por **capas** (layers): cada instrucción del `Dockerfile` (`FROM`, `COPY`, `RUN`) genera una capa inmutable cacheable. El truco de producción es el **multi-stage build**: un stage `builder` con todas las herramientas de compilación (que pesa mucho) y un stage `runner` final que solo copia los artefactos ya compilados (que pesa poco). El orden importa: si copiás `package.json` primero y corrés `npm ci`, esa capa se cachea y no se reinstala a menos que cambien las dependencias. → [Docker Docs — Dockerfile reference](https://docs.docker.com/reference/dockerfile/) | → [Docker Docs — Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)

```dockerfile
# ---- Stage 1: builder (pesado, con compilador) ----
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci                        # capa cacheable: solo se re-ejecuta si cambian las deps
COPY . .
RUN npm run build                 # genera dist/ o .next/

# ---- Stage 2: runner (liviano, solo lo necesario) ----
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/package.json /app/package-lock.json ./
RUN npm ci --omit=dev             # solo dependencias de producción
COPY --from=builder /app/dist ./dist
USER node                         # no correr como root
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

Las decisiones que hacen una imagen **de producción** (no de juguete):

| Técnica | Qué resuelve | Cómo |
|---------|-------------|------|
| **Multi-stage** | Imagen final sin compiladores ni devDeps | `builder` compila, `runner` copia solo lo compilado |
| **Base minimal** | Imagen más chica y segura | `node:20-alpine` (~50MB) en vez de `node:20` (~1GB) |
| **`.dockerignore`** | No meter `node_modules`, `.git`, `.env` en el build context | Listá lo que NO se copia |
| **Cache por capas** | Builds rápidos | Copiá `package.json` antes que el código fuente |
| **`--omit=dev`** | No llevar devDependencies a prod | `npm ci --omit=dev` |
| **Non-root user** | Reducir superficie de ataque | `USER node` |
| **`HEALTHCHECK`** | Que el orquestador sepa si el contenedor está sano | `HEALTHCHECK CMD ...` |

`docker-compose.yml` en producción orquesta varios servicios (app + proxy + db) y configura límites:

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

volumes:
  pgdata:
```

> **Nota**: el `compose.yaml` del tópico 2 era para DESARROLLO (bases de datos sueltas). Este `Dockerfile` + `compose.yml` es para PRODUCCIÓN: empaqueta TU app, define health checks, límites de memoria y política de reinicio.

→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.6-docker) para el modelo de imagen vs contenedor y el `compose.yaml` de desarrollo que esto profundiza. → Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.4-vite--el-bundler-y-dev-server-moderno) para qué es el `dist/` que genera el build y que acá se copia al runner.

> **Check de comprensión**
> 1. ¿Qué diferencia hay entre el Docker del tópico 2 y el de este tópico?
>    - R: El tópico 2 usa Docker para servicios de desarrollo (bases de datos ya empaquetadas). Acá empaquetás TU aplicación con un Dockerfile propio para producción, con multi-stage, base minimal y health checks.
> 2. ¿Qué es un multi-stage build y por qué la imagen final es más chica?
>    - R: Separa el build en etapas: un stage `builder` con todas las herramientas de compilación y un stage `runner` que copia solo los artefactos compilados. La imagen final no lleva compiladores ni devDependencies.
> 3. ¿Por qué copiar `package.json` ANTES que el código fuente mejora el cache?
>    - R: Porque Docker cachea por capas. Si `package.json` no cambió, la capa del `npm ci` se reutiliza y no se reinstalan dependencias en cada build. Si copiás el código primero, cualquier cambio invalida toda la cache.
> 4. ¿Para qué sirve `HEALTHCHECK` en un Dockerfile?
>    - R: Define un comando que Docker ejecuta periódicamente para saber si el contenedor está sano. Los orquestadores lo usan para reiniciar contenedores enfermos y para decidir cuándo un servicio está listo.
> 5. ¿Por qué no hay que correr el contenedor como root?
>    - R: Porque reduce la superficie de ataque: si un atacante escapa del proceso, no tiene permisos de root sobre el contenedor. Se usa `USER node` para correr como un usuario sin privilegios.
> 6. ¿Qué hace `restart: unless-stopped` en un compose de producción?
>    - R: Reinicia automáticamente el contenedor si se cae o si el daemon se reinicia, salvo que lo hayas detenido manualmente. Es la política de resiliencia básica para producción.

---

## 9.2 CI/CD

*En criollo:* CI (Integración Continua) es que **cada push dispare tests y build automáticamente** para que un bug se detecte en minutos, no en el deploy. CD (Entrega/Despliegue Continuo) es que ese código que pasó los tests **se despliegue solo** a producción (o quede listo con un botón). La idea: el deploy deja de ser un evento manual, estresante y propenso a error, y pasa a ser una rutina aburrida y repetible. → [GitHub Actions — Understanding GitHub Actions](https://docs.github.com/en/actions/about-github-actions)

*Técnicamente:* Un pipeline es un **workflow** definido en YAML que vive en `.github/workflows/`. Se dispara por **eventos** (`push`, `pull_request`), corre **jobs** (unidades de ejecución en máquinas virtuales paralelas), que se dividen en **steps** (comandos o acciones reutilizables). Los **secrets** (tokens, claves) se inyectan como variables cifradas sin exponerlos en el código. El CD típico es: `build` → `test` → `deploy`, con gates entre etapas y **environments** protegidos para producción.

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build

  test:
    needs: build                     # corre después de build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm test -- --run       # una sola pasada, no watch mode

  deploy:
    needs: test                      # solo si test pasó
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: echo "deploy a producción con ${{ secrets.DEPLOY_TOKEN }}"
```

Conceptos clave que tenés que manejar:

| Concepto | Qué es | Por qué importa |
|----------|--------|-----------------|
| **Trigger** | Qué evento dispara el workflow (`push`, `pull_request`, `schedule`, `workflow_dispatch`) | Controlás cuándo corre el pipeline |
| **Runner** | La máquina (Linux/Windows/macOS) que ejecuta los jobs | El entorno estándar donde corre tu código |
| **Job/Step** | Unidad de trabajo y sus comandos | Paralelismo y aislamiento de fallas |
| **Cache** | Guardar `node_modules` entre runs | Builds más rápidos y baratos |
| **Environment** | Entorno protegido con secrets y reglas de aprobación | No cualquiera deploya a producción |
| **Secret** | Variable cifrada (`${{ secrets.X }}`) | Tokens y claves sin hardcodear |

> **Nota**: el `test` en CI debe correr UNA sola pasada (`--run` / `--ci`), nunca watch mode, porque el runner no es interactivo y el job no terminaría nunca.

→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.1-git--control-de-versiones) para las estrategias de branching que disparan el pipeline. → Ver [Tópico 10: Testing](../concepts/10-testing.md#10.1-unit-tests) para qué tests se ejecutan en el job `test` del CI.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre CI y CD?
>    - R: CI es integrar y validar código continuamente: cada push dispara build y tests para detectar bugs temprano. CD es entregar/desplegar ese código automáticamente a producción una vez que pasó las validaciones.
> 2. ¿Dónde se define un workflow de GitHub Actions y en qué formato?
>    - R: En `.github/workflows/*.yml` (YAML). Cada archivo define un workflow con triggers (`on`), jobs y steps.
> 3. ¿Por qué el job `test` usa `needs: build`?
>    - R: `needs` define dependencia entre jobs: `test` solo corre si `build` terminó con éxito. Esto evita gastar recursos testeando código que ni siquiera compila.
> 4. ¿Qué es un secret en CI/CD y por qué no se hardcodea?
>    - R: Es una variable cifrada (token, clave API) que se referencia como `${{ secrets.NOMBRE }}`. No se hardcodea porque quedaría expuesta en el repositorio y en los logs.
> 5. ¿Por qué el `deploy` tiene `if: github.ref == 'refs/heads/main'`?
>    - R: Para que el deploy solo ocurra en pushes a `main`, no en pull requests o ramas de feature. El deploy a producción debe ser una acción deliberada sobre la rama principal.
> 6. ¿Qué es un `environment` protegido en un workflow?
>    - R: Un entorno con reglas de aprobación y secrets propios. Puede requerir revisión manual o esperar a que ciertos checks pasen antes de que el deploy toque producción.

---

## 9.3 Cloud (AWS, Vercel, Railway)

*En criollo:* El cloud es **alquilar computadoras ajenas** en vez de comprar servidores. La pregunta no es "¿tengo servidor?" sino "¿qué nivel de control necesito y cuánto quiero administrar?". En un extremo alquilás máquinas virtuales (control total, trabajo total); en el otro, subís tu código y una plataforma se encarga de TODO (cero control, cero trabajo). La mayoría de las apps full stack caen en el medio.

*Técnicamente:* Se clasifica en tres modelos de servicio: **IaaS** (máquinas virtuales, redes, discos — vos administrás el SO), **PaaS** (plataforma que corre tu código, vos solo deployás) y **serverless** (funciones que se ejecutan bajo demanda, sin servidor que administrar). → [AWS — What is cloud computing?](https://aws.amazon.com/what-is-cloud-computing/) | → [Vercel — Docs](https://vercel.com/docs) | → [Railway — Docs](https://docs.railway.com/)

| | AWS | Vercel | Railway |
|---|---|---|---|
| Modelo | IaaS + PaaS + serverless (todo) | PaaS orientado a frontend/serverless | PaaS simple y moderno |
| Curva de aprendizaje | Alta (cientos de servicios) | Baja (push y listo) | Baja (push y listo) |
| Control | Total (EC2, ECS, Lambda, S3, RDS) | Bajo (optimizado para Next.js/React) | Medio (servicios + base de datos) |
| Costo | Escala con lo que usás, complejo de predecir | Generoso tier free, luego por uso | Predecible, tier free básico |
| Caso ideal | Empresas, apps grandes, requisitos específicos | Apps frontend/Next.js, portfolios, startups | Proyectos personales, MVPs, apps full stack chicas |

### AWS — los servicios que te vas a cruzar

| Servicio | Qué es | Para qué lo usás |
|----------|--------|------------------|
| **EC2** | Máquina virtual (IaaS) | Servidor propio, control total |
| **ECS / EKS** | Orquestador de contenedores | Correr imágenes Docker (lo de 9.1) |
| **Lambda** | Funciones serverless | Código bajo demanda sin servidor |
| **S3** | Almacenamiento de objetos | Archivos estáticos, uploads, backups |
| **RDS** | Base de datos relacional gestionada | PostgreSQL/MySQL sin administrar el server |
| **Route 53** | DNS | Apuntar tu dominio a los servicios |

### Vercel — el default de frontend

Vercel es la plataforma de Next.js: `git push` y automáticamente detecta el proyecto, hace build y publica. Cada PR genera un **preview deployment** (URL temporal para probar cambios antes de mergear). Ideal para el stack React/Next del tópico 8.

### Railway — el "Heroku moderno"

Railway te deja conectar un repositorio y levantar app + PostgreSQL + Redis con un `Dockerfile` (o autodetección). Simple, con base de datos integrada, ideal para backends y MVPs.

> **Regla de oro**: no hay una respuesta "correcta" universal. Elegí por CONTROL vs CONVENIENCIA. Empezá por lo simple (Vercel/Railway) y subí de nivel a AWS cuando lo necesites de verdad.

→ Ver [Tópico 1: Fundamentos de la Web](../concepts/01-fundamentos-web.md#1.3-protocolo-http) para el modelo cliente-servidor que el cloud materializa en servidores reales. → Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.6-nextjs--el-framework-full-stack-de-react) para por qué Vercel es el deploy natural de Next.js.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre IaaS, PaaS y serverless?
>    - R: IaaS alquila máquinas virtuales (administrás el SO), PaaS ejecuta tu código sin que toques servidores, y serverless ejecuta funciones bajo demanda sin servidor persistente que administrar.
> 2. ¿Cuándo elegirías Vercel y cuándo AWS?
>    - R: Vercel para apps frontend/Next.js donde querés push-and-forget. AWS cuando necesitás control total, servicios específicos (EC2, S3, RDS) o requisitos empresariales.
> 3. ¿Qué es un preview deployment en Vercel?
>    - R: Un deploy temporal con URL propia que se genera por cada PR. Te permite probar los cambios en un entorno real antes de mergear a producción.
> 4. ¿Qué hace el servicio S3 de AWS?
>    - R: Es almacenamiento de objetos (files): sirve archivos estáticos, guarda uploads de usuarios y backups. No es un filesystem tradicional ni una base de datos.
> 5. ¿Qué ventaja tiene RDS sobre montar tu propia base de datos?
>    - R: RDS gestiona la base de datos por vos: backups automáticos, parches, replicación y failover. No administrás el servidor, solo usás la base.
> 6. ¿Por qué Railway se compara con el "Heroku moderno"?
>    - R: Porque conectás un repo y levanta app + base de datos + Redis con mínima configuración (autodetección o Dockerfile), sin la complejidad de administrar infraestructura.

---

## 9.4 Entornos

*En criollo:* No deployás a producción con la base de datos real de los usuarios para "probar algo". Necesitás **copias separadas** de tu app: una para desarrollar (dev), una para probar antes de salir (staging) y la que usan los usuarios (prod). Cada una con su propia base de datos, sus propias claves y su propia configuración. La regla de oro: **nunca apuntes dev a datos reales ni prod a datos de prueba**.

*Técnicamente:* Los entornos se distinguen por **configuración externa** (variables de entorno, archivos `.env`), no por código distinto. El [12-Factor App](https://12factor.net/config) lo define: la config va en el **environment**, no en el código. `NODE_ENV` es la variable canónica (`development`, `test`, `production`) que cambia comportamiento (logs más verbosos en dev, minificación en prod). Cada entorno tiene su `.env` que NUNCA se commitea.

```
~/proyectos/mi-app/
├── .env                 # desarrollo local (NO se commitea)
├── .env.example         # plantilla con las claves (sí se commitea, sin valores reales)
└── src/
    └── config.ts        # lee process.env y valida
```

```ts
// config.ts — una sola fuente de verdad de configuración
const env = {
  NODE_ENV: process.env.NODE_ENV ?? 'development',
  DATABASE_URL: process.env.DATABASE_URL,
  PORT: Number(process.env.PORT ?? 3000),
};

if (!env.DATABASE_URL) {
  throw new Error('DATABASE_URL es obligatoria en todos los entornos');
}
```

| Entorno | Propósito | Base de datos | Datos |
|---------|-----------|---------------|-------|
| **development** | Tu máquina, iterar rápido | Local (Docker) | Ficticios/seed |
| **test** | Correr la suite de tests | Efímera en memoria | Fixtures |
| **staging** | Último paso antes de producción | Réplica de prod | Datos anonimizados |
| **production** | Los usuarios reales | La real | Reales |

> **Regla**: una variable que cambia entre entornos (URL de DB, claves API, puertos) es **config**, y va en el environment. Una lógica que es igual en todos lados es **código**, y va en el repo.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.2-arquitectura-de-una-api-rest) para cómo una API lee configuración al arrancar. → Ver [Tópico 9.7: Secrets management](../concepts/09-devops-deployment.md#9.7-secrets-management) para por qué los `.env` con secretos jamás se commitean.

> **Check de comprensión**
> 1. ¿Por qué necesitás entornos separados y no deployás directo a producción?
>    - R: Para probar cambios en un entorno idéntico a producción (staging) sin arriesgar datos reales ni usuarios. Separás desarrollo, pruebas y producción con sus propias bases y claves.
> 2. ¿Qué dice el factor "config" del 12-Factor App?
>    - R: Que la configuración vive en el environment (variables de entorno), separada del código. El mismo build corre en cualquier entorno cambiando solo la config.
> 3. ¿Qué es `NODE_ENV` y para qué sirve?
>    - R: Es la variable canónica que define el modo de ejecución (`development`, `test`, `production`). Cambia comportamientos como nivel de logs, minificación y mensajes de error.
> 4. ¿Cuál es la diferencia entre `.env` y `.env.example`?
>    - R: `.env` contiene los valores reales del entorno y NUNCA se commitea. `.env.example` es una plantilla con las claves (sin valores reales) que sí se commitea para que otros sepan qué variables necesita la app.
> 5. ¿Por qué staging debe tener datos anonimizados y no una copia exacta de prod?
>    - R: Porque staging es accesible para desarrollo y pruebas. Copiar datos reales de usuarios (emails, contraseñas, PII) a un entorno de prueba es una filtración de datos.
> 6. ¿Qué criterio usás para decidir si algo es "config" o "código"?
>    - R: Si cambia entre entornos (URL de DB, claves, puertos) es config y va en el environment. Si la lógica es idéntica en todos lados, es código y va en el repo.

---

## 9.5 Monitoreo

*En criollo:* Deployaste. ¿Y ahora? Si tu app se cae a las 3 AM, ¿cómo te enterás: por un alerta o por un usuario enojado? Monitorear es **medir la salud de tu app en producción** de forma continua: que responda, que sea rápida, que no tire errores. El objetivo no es mirar dashboards todo el día — es que el SISTEMA te avise cuando algo anda mal, antes de que lo descubran los usuarios.

*Técnicamente:* El monitoreo se divide en tres capas: **métricas** (números agregados como requests/segundo, latencia, uso de CPU), **logs** (eventos puntuales, ver 9.6) y **traces** (el camino de un request a través de los servicios). Prometheus scrapea métricas expuestas en un endpoint `/metrics`; Grafana las visualiza. Los **health checks** (el `HEALTHCHECK` de 9.1, o un endpoint `/health`) son la señal mínima de "estoy vivo". Con eso se definen **SLI/SLO**: objetivos medibles de disponibilidad (ej: 99.9% de uptime mensual). → [Prometheus — Docs](https://prometheus.io/docs/introduction/overview/) | → [Grafana — Docs](https://grafana.com/docs/)

```js
// health check básico en Express — la señal mínima de vida
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok', uptime: process.uptime() });
});
```

| Capa | Pregunta que responde | Herramienta típica |
|------|----------------------|--------------------|
| **Health check** | ¿Está vivo el proceso? | `/health` + orquestador |
| **Métricas** | ¿Cuántos requests? ¿Qué latencia? | Prometheus + Grafana |
| **Logs** | ¿Qué pasó exactamente? | Loki / ELK (ver 9.6) |
| **Traces** | ¿Por qué tarda este request? | OpenTelemetry, Jaeger |
| **Errores/APM** | ¿Dónde crashea el código? | Sentry, New Relic |

### SLI, SLO y alertas — los términos que importan

- **SLI** (Service Level Indicator): la métrica que medís (ej: porcentaje de requests exitosos).
- **SLO** (Service Level Objective): el objetivo que te comprometés (ej: 99.9% de requests exitosos en un mes).
- **Alerta**: cuando un SLO está en riesgo de incumplirse, se dispara una notificación (PagerDuty, Slack, email). Alertás sobre lo que PODÉS arreglar, no sobre ruido.

> **Regla de oro**: un alerta sin acción que tomar es ruido. Alertás solo cuando alguien debe hacer algo al respecto. Si te llegan 500 alertas por día, dejaste de prestar atención a todas.

→ Ver [Tópico 9.6: Logging centralizado](../concepts/09-devops-deployment.md#9.6-logging-centralizado) para la capa de logs que complementa las métricas. → Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.4-manejo-de-errores) para cómo estructurar el manejo de errores que después medís y alertás.

> **Check de comprensión**
> 1. ¿Cuál es el objetivo real de monitorear una app en producción?
>    - R: Detectar problemas de salud y performance ANTES de que los usuarios los sufran. El sistema te alerta proactivamente; no esperás quejas para enterarte de que algo anda mal.
> 2. ¿Qué diferencia hay entre métricas, logs y traces?
>    - R: Métricas son números agregados (requests/seg, latencia). Logs son eventos puntuales ("qué pasó"). Traces siguen el camino de un request a través de los servicios para ver dónde se demora.
> 3. ¿Qué es un health check y por qué es la señal mínima?
>    - R: Un endpoint o comando que indica si el proceso está vivo (ej: `/health` responde 200). Los orquestadores lo consultan para decidir si reiniciar contenedores enfermos.
> 4. ¿Qué son SLI y SLO y cómo se relacionan?
>    - R: SLI es el indicador que medís (porcentaje de requests exitosos). SLO es el objetivo comprometido sobre ese indicador (99.9% de éxito mensual). El SLO define cuánto puede degradarse el SLI sin violar el acuerdo.
> 5. ¿Por qué un alerta sin acción que tomar es ruido?
>    - R: Porque satura el canal de notificación y hace que el equipo deje de prestar atención a alertas reales. Solo se alerta cuando alguien debe hacer algo al respecto.
> 6. ¿Qué hace Prometheus y qué hace Grafana?
>    - R: Prometheus recolecta (scrapea) y almacena métricas desde un endpoint `/metrics`. Grafana las consulta y las visualiza en dashboards con gráficos y alertas.

---

## 9.6 Logging centralizado

*En criollo:* Si tenés una sola instancia, `console.log` alcanza. Pero en producción tenés MUCHAS instancias (o contenedores que se crean y destruyen). ¿Buscás un error y tenés que entrar a 10 máquinas distintas? El logging centralizado es **juntar los logs de TODAS las instancias en un solo lugar**, con una herramienta para buscarlos. Un error en producción se encuentra en segundos, no en horas.

*Técnicamente:* Los logs deben ser **estructurados** (JSON con campos, no texto libre) para poder consultarlos: `{ "level": "error", "message": "...", "requestId": "abc", "timestamp": "..." }`. Cada instancia escribe a **stdout/stderr** y un agente (o la plataforma) los envía a un **agregador** (Loki, ELK, Datadog). Un **correlation ID** (o request ID) generado al inicio de cada request viaja por todos los logs de esa operación para poder rastrear un request de punta a punta. → [12-Factor App — Logs](https://12factor.net/logs) | → [OpenTelemetry — Logs](https://opentelemetry.io/docs/concepts/signals/logs/)

```js
// logging estructurado con correlation ID
function log(level, message, extra = {}) {
  console.log(JSON.stringify({
    level,
    message,
    requestId: extra.requestId ?? null,
    timestamp: new Date().toISOString(),
    ...extra,
  }));
}

app.use((req, res, next) => {
  req.id = crypto.randomUUID();        // correlation ID por request
  log('info', 'request', { requestId: req.id, method: req.method, path: req.path });
  next();
});

// ... en el error handler
app.use((err, req, res, next) => {
  log('error', err.message, { requestId: req.id, stack: err.stack });
  res.status(500).json({ error: 'Internal Server Error' });
});
```

| Práctica | Por qué |
|----------|---------|
| **Logs a stdout/stderr** | El proceso no decide dónde van los logs; la plataforma los captura y enruta |
| **JSON estructurado** | Consultable, filtrable y parseable por el agregador |
| **Correlation ID** | Trazar un request a través de todos los servicios y logs |
| **Niveles (`info`, `warn`, `error`)** | Filtrar ruido y alertar solo sobre errores reales |
| **Nunca loguear secretos** | Passwords y tokens en logs = filtración (ver 9.7) |
| **Redactar PII** | Emails y datos personales no van crudos en logs |

> **Regla de oro**: en producción los logs se **escriben** (a stdout) y la plataforma los **transporta** y **agrega**. Tu app jamás decide "dónde" viven los logs — eso rompe la portabilidad y complica la centralización.

→ Ver [Tópico 5: Frameworks Backend](../concepts/05-frameworks-backend.md#5.6-logging--structured-logging) para las librerías de logging estructurado (pino, winston) que implementan esto. → Ver [Tópico 9.5: Monitoreo](../concepts/09-devops-deployment.md#9.5-monitoreo) para cómo los logs se complementan con métricas y health checks.

> **Check de comprensión**
> 1. ¿Qué problema resuelve el logging centralizado?
>    - R: Con muchas instancias, buscar un error en cada máquina es inviable. Centralizar junta los logs de todas las instancias en un solo lugar consultable, encontrando errores en segundos.
> 2. ¿Por qué los logs deben ser estructurados (JSON) y no texto libre?
>    - R: Porque el agregador puede indexar y consultar campos (level, requestId, status). El texto libre no se puede filtrar ni consultar de forma eficiente.
> 3. ¿Qué es un correlation ID y para qué sirve?
>    - R: Un identificador único generado al inicio de cada request que viaja en todos los logs de esa operación. Permite rastrear un request de punta a punta a través de servicios y logs.
> 4. ¿Por qué la app debe escribir logs a stdout en vez de a un archivo?
>    - R: Porque así la app no decide el destino de los logs. La plataforma captura stdout y los enruta al agregador. Escribir a archivo acopla la app al filesystem y rompe la portabilidad.
> 5. ¿Qué no debe aparecer NUNCA en un log?
>    - R: Secretos (passwords, tokens, claves API) y datos personales (PII como emails o documentos). Loguear secretos es una filtración; la PII debe redactarse.
> 6. ¿Para qué sirven los niveles de log (`info`, `warn`, `error`)?
>    - R: Para categorizar severidad y poder filtrar: en producción ves solo `warn`/`error`, mientras que en desarrollo ves también `info`/`debug`. Permiten alertar solo sobre errores reales.

---

## 9.7 Secrets management

*En criollo:* Un secret es cualquier cosa sensible: contraseña de la base de datos, token de API, clave de firma de JWT, API key de un servicio. El error clásico es meterlos en el código o commitearlos en el repo — y ese repo, tarde o temprano, se filtra o se comparte. Gestionar secrets es **mantenerlos fuera del código y del control de versiones**, y dárselos a la app solo en runtime, desde un lugar seguro y cifrado.

*Técnicamente:* La capa mínima es la **variable de entorno** (inyectada por la plataforma, nunca hardcodeada), con `.env` en `.gitignore` y un `.env.example` como plantilla. La capa robusta es un **secret manager** (AWS Secrets Manager, HashiCorp Vault, Google Secret Manager) que cifra en reposo, rota automáticamente y audita accesos. En CI/CD (9.2), los secrets se guardan cifrados en la plataforma (`secrets.` de GitHub Actions) y se inyectan en runtime sin aparecer en logs. → [12-Factor App — Config](https://12factor.net/config) | → [GitHub Docs — Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)

```bash
# .gitignore — los secrets jamás se commitean
.env
.env.*
!.env.example
```

```bash
# .env.example — plantilla SIN valores reales
DATABASE_URL=postgresql://USER:PASSWORD@HOST:5432/DB
JWT_SECRET=poner-un-valor-aleatorio-largo
STRIPE_API_KEY=sk_test_xxx
```

```bash
# cómo la plataforma inyecta el secret en runtime
DATABASE_URL=$(aws secretsmanager get-secret-value \
  --secret-id prod/database --query SecretString --output text)
```

| Práctica | Qué evita |
|----------|-----------|
| **Nunca commitear `.env`** | Secrets en el historial de git (irrecuperable sin reescribir historia) |
| **`.env.example` commiteado** | Que nadie sepa qué variables necesita la app |
| **Variables de entorno en runtime** | Hardcodear valores sensibles en el código |
| **Secret manager** | Secrets en texto plano, sin rotación ni auditoría |
| **Rotación** | Que una clave robada sirva para siempre |
| **Least privilege** | Que un servicio lea secrets que no necesita |

> **Regla de oro**: si un secret entró al historial de git, **dalo por comprometido** — rotalo (generá uno nuevo) aunque borres el commit, porque sigue en el historial y en los clones. Borrar del código NO basta; hay que rotar.

→ Ver [Tópico 12: Seguridad](../concepts/12-seguridad.md#12.8-secret-management) para secret management en el contexto completo de seguridad. → Ver [Tópico 9.4: Entornos](../concepts/09-devops-deployment.md#9.4-entornos) para cómo cada entorno tiene sus propios secrets sin compartir los de producción.

> **Check de comprensión**
> 1. ¿Qué es un secret y cuál es el error clásico al manejarlo?
>    - R: Un secret es información sensible (password de DB, token, clave JWT, API key). El error clásico es meterlo en el código o commitearlo, dejándolo expuesto en el repo.
> 2. ¿Cuál es la diferencia entre `.env` y `.env.example`?
>    - R: `.env` tiene los valores reales y se excluye de git con `.gitignore`. `.env.example` es una plantilla sin valores reales que sí se commitea para documentar qué variables necesita la app.
> 3. ¿Por qué borrar un secret del código no alcanza si ya se commiteó?
>    - R: Porque el secret queda en el historial de git y en todos los clones. Cualquiera con acceso al repo lo puede recuperar. Hay que ROTAR el secret (generar uno nuevo), no solo borrarlo.
> 4. ¿Qué agrega un secret manager sobre las variables de entorno simples?
>    - R: Cifrado en reposo, rotación automática, auditoría de accesos y control fino de quién lee qué. Las variables de entorno son la capa mínima; el secret manager es la capa robusta.
> 5. ¿Cómo se inyectan secrets en un workflow de GitHub Actions sin exponerlos?
>    - R: Se guardan cifrados en la config del repo (`Settings → Secrets`) y se referencian como `${{ secrets.NOMBRE }}`. Se inyectan en runtime y GitHub los enmascara en los logs.
> 6. ¿Qué es el principio de least privilege aplicado a secrets?
>    - R: Dar a cada servicio/rol acceso SOLO a los secrets que necesita, nada más. Un servicio comprometido no arrastra acceso a todos los secrets del sistema.
