# Setup — CI/CD con GitHub Actions

> **Tópico**: 9 — DevOps & Deployment (sección 9.2)
> **Objetivo**: automatizar build → test → deploy con un workflow de GitHub Actions que se dispara en cada push y despliega solo desde `main` usando secrets cifrados y un environment protegido.
> **Prerequisito**: un repo en GitHub con una app que tenga `npm test` y `npm run build` funcionando (la del `setup-docker.md` sirve).

---

## ¿Por qué?

Sin CI, cada cambio se prueba a mano y el deploy es un ritual estresante y propenso a error. Con un pipeline, cada push dispara build y tests automáticamente: un bug se detecta en minutos, no en producción. El CD hace que el deploy sea una rutina repetible que cualquiera del equipo puede disparar sin miedo. Es la diferencia entre "ojalá no se rompa nada" y "sé con certeza que pasó todos los tests antes de salir".

---

## Checklist

### 1. Preparar los scripts del `package.json`

- [ ] Asegurate de que `npm test` corra en modo no-interactivo (una sola pasada). Para Vitest: `"test": "vitest run"`. Para Jest: `"test": "jest --ci"`.
- [ ] Verificá que `npm run build` funcione de forma limpia.

### 2. Crear el workflow de CI

- [ ] Creá `.github/workflows/ci.yml`:

```yaml
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
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm test
```

### 3. Agregar el job de deploy (CD)

- [ ] Agregá un job `deploy` que solo corra en `main` y use un environment protegido:

```yaml
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: |
          echo "Deployando a producción..."
          docker build -t mi-app:prod .
          echo "Push a registry con token cifrado"
        env:
          REGISTRY_TOKEN: ${{ secrets.REGISTRY_TOKEN }}
```

### 4. Configurar secrets en GitHub

- [ ] En GitHub: **Settings → Secrets and variables → Actions → New repository secret**.
- [ ] Creá `REGISTRY_TOKEN` (u otros secrets que use tu deploy). NUNCA los escribas en el YAML.

### 5. Configurar el environment protegido

- [ ] En GitHub: **Settings → Environments → New environment** → nombrarlo `production`.
- [ ] Opcional: activá **Required reviewers** para que el deploy a producción requiera aprobación manual.

### 6. Pushear y ver el pipeline

- [ ] `git add . && git commit -m "ci: add GitHub Actions pipeline" && git push`
- [ ] En GitHub: pestaña **Actions** → verificá que `build` y `test` corren en verde.
- [ ] Abrí un PR: verificá que el workflow corre también en el PR (gracias al trigger `pull_request`).

---

## Verificación

- Un push a `main` dispara el workflow y `build` → `test` → `deploy` corren en orden.
- Un PR dispara `build` y `test`, pero NO `deploy` (por el `if: github.ref == 'refs/heads/main'`).
- El job `deploy` muestra el secret enmascarado (`***`) en los logs, nunca su valor real.

**Si los tres jobs pasan en verde y el deploy respeta el environment → pipeline listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `npm ci` falla en CI | El `package-lock.json` no está commiteado o difiere de `package.json`. Comitealo y regenerá con `npm install` |
| El test cuelga en CI | Estás corriendo watch mode. Usá `vitest run` / `jest --ci` (una sola pasada) |
| `deploy` corre en PRs | Verificá el `if: github.ref == 'refs/heads/main'`; los PRs corren con el contexto del PR, no de main |
| Secret aparece en logs | No uses `echo $SECRET` en un step; GitHub enmascara `${{ secrets.X }}` pero no lo protege si lo logueás a mano |
| `environment` no existe | Creá el environment en Settings → Environments antes de referenciarlo, o el job falla |
| Build distinto en CI vs local | Fijá la versión de Node (`node-version: 20`) para que CI y local usen el mismo runtime |
| No corre el workflow | El archivo debe estar en `.github/workflows/` y el `on:` debe incluir el evento que disparaste |

---

## Recursos

- [GitHub Docs — Understanding GitHub Actions](https://docs.github.com/en/actions/about-github-actions)
- [GitHub Docs — Workflow syntax](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions)
- [GitHub Docs — Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
- [GitHub Docs — Using environments for deployment](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/managing-environments-for-deployment)

## Preguntas de repaso

- **P:** ¿Cuál es la diferencia entre un trigger `push` y `pull_request` en un workflow?
  **R:** `push` dispara el workflow cuando se pushea a una rama (típicamente `main`). `pull_request` lo dispara cuando se abre o actualiza un PR, permitiendo validar cambios antes de mergear.

- **P:** ¿Para qué sirve `needs: build` en el job `test`?
  **R:** Define dependencia entre jobs: `test` solo corre si `build` terminó con éxito. Evita gastar recursos de CI testeando código que no compila.

- **P:** ¿Por qué el deploy usa `if: github.ref == 'refs/heads/main'`?
  **R:** Para que el deploy ocurra solo en pushes a `main`, no en PRs ni ramas de feature. El deploy a producción debe ser deliberado y sobre la rama principal.

- **P:** ¿Dónde se guardan los secrets y cómo se usan en el workflow?
  **R:** En GitHub Settings → Secrets, cifrados. Se usan como `${{ secrets.NOMBRE }}` y se inyectan como variables de entorno en runtime, enmascarados en los logs.

- **P:** ¿Qué es un `environment` protegido y para qué sirve?
  **R:** Un entorno con secrets y reglas propias (como required reviewers). Protege el deploy a producción exigiéndo aprobación manual o condiciones antes de ejecutarse.

- **P:** ¿Por qué el test en CI debe correr en modo no-interactivo?
  **R:** Porque el runner no es interactivo: un watch mode nunca termina y el job queda colgado hasta el timeout. En CI se corre una sola pasada (`vitest run`, `jest --ci`).
