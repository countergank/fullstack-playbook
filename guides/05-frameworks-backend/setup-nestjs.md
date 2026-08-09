# Setup — NestJS (CLI oficial)

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.2)
> **Objetivo**: crear un proyecto NestJS desde cero usando la CLI oficial, con estructura base y endpoint de prueba.
> **Prerequisito**: Node (`setup-node.md`).

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