# Setup — NestJS (CLI oficial)

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.2)
> **Objetivo**: crear un proyecto NestJS desde cero usando la CLI oficial, con estructura base y endpoint de prueba.
> **Prerequisito**: Node (`setup-node.md`).

---

## ¿Por qué NestJS?

Express te da libertad total pero también toda la responsabilidad: estructura, validación, testing, inyección de dependencias — todo lo armás vos. NestJS viene con arquitectura predefinida (módulos, controladores, servicios), TypeScript nativo, DI built-in, y una CLI que genera boilerplate. Es ideal para equipos grandes o proyectos enterprise donde la consistencia importa más que la flexibilidad.

---

## Checklist

### 1. Instalar la CLI de NestJS

```bash
npm install -g @nestjs/cli
nest --version
```

### 2. Crear el proyecto

```bash
nest new mi-backend-nest
```

- [ ] Elegir **npm** como package manager
- [ ] La CLI genera toda la estructura: `src/`, `app.module.ts`, `app.controller.ts`, `app.service.ts`, `main.ts`, `test/`

### 3. Estructura generada

```
src/
├── app.controller.ts       # endpoint de prueba (GET /)
├── app.controller.spec.ts  # test unitario del controller
├── app.module.ts           # módulo raíz
├── app.service.ts          # servicio inyectable
└── main.ts                 # entry point (bootstrap)
```

### 4. Probar que funciona

```bash
cd mi-backend-nest
npm run start:dev
```

```bash
curl http://localhost:3000
# "Hello World!"
```

### 5. Generar un módulo completo (controller + service + module)

```bash
nest generate resource users
```

- [ ] Elegir **REST API**
- [ ] Elegir **Y** para generar CRUD entry points
- [ ] Esto crea `src/users/` con DTOs, controller, service, module, entity y tests

### 6. Estructura después de generar un recurso

```
src/
├── app.module.ts
├── main.ts
└── users/
    ├── dto/
    │   ├── create-user.dto.ts
    │   └── update-user.dto.ts
    ├── entities/
    │   └── user.entity.ts
    ├── users.controller.ts
    ├── users.module.ts
    └── users.service.ts
```

### 7. Probar los endpoints generados

```bash
# NestJS CLI ya definió las rutas REST:
curl http://localhost:3000/users          # GET — listar
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Lean", "email": "lean@ejemplo.com"}'
```

---

## Verificación

```bash
nest --version                    # CLI instalada
curl http://localhost:3000         # "Hello World!"
curl http://localhost:3000/users   # [] (vacío, pero el endpoint existe)
```

**Si `nest --version` responde y los endpoints funcionan → NestJS listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `nest: command not found` después de `npm install -g @nestjs/cli` | Verificá que el PATH de npm global está en tu `.bashrc`/`.zshrc`. Ejecutá `npm config get prefix` y agregá `<prefix>/bin` al PATH. |
| `Nest can't resolve dependencies of the XService` | El provider no está registrado en el `providers` array del módulo, o tiene una dependencia que no está importada en el módulo. |
| `Error: ENOENT: no such file or directory` al generar un recurso | Estás fuera del directorio del proyecto. `nest generate` necesita ejecutarse desde la raíz donde está `nest-cli.json`. |
| El endpoint devuelve 404 | Verificá que el controller está registrado en el `controllers` array del módulo y que el módulo está importado en `AppModule`. |

---

## Preguntas de repaso

- **P:** ¿Qué ventaja tiene NestJS sobre Express para proyectos grandes?
  **R:** Arquitectura predefinida (módulos, controllers, services), TypeScript nativo, inyección de dependencias, y CLI que genera código boilerplate. Todo el equipo sigue la misma estructura.

- **P:** ¿Qué hace `nest generate resource`?
  **R:** Genera un módulo completo con controller, service, DTOs, entity, module y tests — todo conectado y con CRUD endpoints listos.

- **P:** ¿Qué es la inyección de dependencias en NestJS?
  **R:** Un patrón donde las dependencias se proveen externamente por el contenedor IoC. Los servicios se decoran con `@Injectable()` y se inyectan vía constructor.

- **P:** ¿NestJS reemplaza a Express?
  **R:** No exactamente. NestJS usa Express como HTTP adapter por defecto (pero puede usar Fastify). Es una capa de arquitectura sobre el motor HTTP.

- **P:** ¿Cuándo NO conviene NestJS?
  **R:** Para proyectos pequeños, MVPs rápidos, o cuando estás aprendiendo. La curva de aprendizaje es media-alta y el boilerplate puede ser excesivo para algo simple.

---

## Diferencias clave con Express

| Expres | NestJS |
|--------|--------|
| Creás archivos a mano | `nest generate resource` |
| Rutas en `*.routes.ts` | Decorators `@Get()`, `@Post()` en el controller |
| Inyección manual | `@Injectable()` + constructor injection |
| Sin estructura forzada | Módulos, controllers, services, DTOs por convención |
| `npm init` + `npm install express` | `nest new` |

---

## Recursos

- [NestJS — First Steps](https://docs.nestjs.com/first-steps)
- [NestJS — CLI](https://docs.nestjs.com/cli/overview)