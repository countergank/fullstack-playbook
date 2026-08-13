# Setup — Estructura del Proyecto

> **Tópico**: 11 — Arquitectura de Software (secciones 11.1, 11.2 y 11.5)
> **Objetivo**: armar la estructura de carpetas y capas de un backend siguiendo Clean Architecture y Hexagonal (Ports & Adapters), con la regla de dependencia respetada.
> **Prerequisito**: `setup-express-project.md` (04-backend), Node y TypeScript configurados.

---

## ¿Por qué?

La estructura de carpetas NO es cosmética: es lo que hace que la regla de dependencia (las capas internas no conocen a las externas) se cumpla o se rompa. Un proyecto que mezcla HTTP con SQL en el mismo archivo funciona el primer mes, pero a los seis meses cada cambio te obliga a tocar tres lugares, los tests son imposibles y migrar de librería es una pesadilla. Separar en capas te da un mapa: sabés DÓNDE vive cada cosa sin abrir un solo archivo, y podés cambiar la base de datos o el framework sin tocar la lógica de negocio.

---

## Checklist

### 1. Crear la estructura de carpetas por capas

- [ ] Crear los directorios base:

```bash
mkdir -p src/domain/entities \
         src/domain/repositories \
         src/application/use-cases \
         src/infrastructure/db \
         src/infrastructure/http \
         src/infrastructure/http/controllers
```

### 2. Entender qué va en cada capa

- [ ] Revisar la tabla de responsabilidades (NO saltear este paso):

| Capa | Qué contiene | Qué NO contiene |
|------|--------------|-----------------|
| `domain/` | Entidades, value objects, interfaces de repositorios | Imports de Express, SQL, `console.log` |
| `application/` | Casos de uso (orquestan el dominio) | Acceso directo a HTTP o DB |
| `infrastructure/db/` | Implementaciones de repositorios (SQL, ORM) | Reglas de negocio |
| `infrastructure/http/` | Controllers, routers, middlewares | Cálculo de totales, validaciones de negocio |

### 3. Definir la entidad del dominio

- [ ] Crear `src/domain/entities/usuario.ts` sin ningún import externo:

```ts
export class Usuario {
  constructor(
    readonly id: string,
    readonly email: string,
    private activo: boolean,
  ) {}

  desactivar(): void {
    this.activo = false;
  }

  estaActivo(): boolean {
    return this.activo;
  }
}
```

### 4. Definir el puerto (interfaz) del repositorio

- [ ] Crear `src/domain/repositories/usuario-repository.ts`:

```ts
import { Usuario } from '../entities/usuario';

export interface UsuarioRepository {
  findById(id: string): Promise<Usuario | null>;
  save(usuario: Usuario): Promise<void>;
}
```

### 5. Escribir el caso de uso

- [ ] Crear `src/application/use-cases/desactivar-usuario.ts` que dependa SOLO de la interfaz:

```ts
import { UsuarioRepository } from '../../domain/repositories/usuario-repository';

export class DesactivarUsuario {
  constructor(private readonly repo: UsuarioRepository) {}

  async ejecutar(id: string): Promise<void> {
    const usuario = await this.repo.findById(id);
    if (!usuario) throw new Error('Usuario no encontrado');
    usuario.desactivar();
    await this.repo.save(usuario);
  }
}
```

### 6. Implementar el adaptador de base de datos

- [ ] Crear `src/infrastructure/db/postgres-usuario-repository.ts`:

```ts
import { Pool } from 'pg';
import { Usuario } from '../../domain/entities/usuario';
import { UsuarioRepository } from '../../domain/repositories/usuario-repository';

export class PostgresUsuarioRepository implements UsuarioRepository {
  constructor(private readonly pool: Pool) {}

  async findById(id: string): Promise<Usuario | null> {
    const { rows } = await this.pool.query('SELECT * FROM usuarios WHERE id = $1', [id]);
    if (!rows[0]) return null;
    return new Usuario(rows[0].id, rows[0].email, rows[0].activo);
  }

  async save(usuario: Usuario): Promise<void> {
    await this.pool.query('UPDATE usuarios SET activo = $1 WHERE id = $2', [usuario.estaActivo(), usuario.id]);
  }
}
```

### 7. Conectar todo en el controller

- [ ] Crear `src/infrastructure/http/controllers/usuario-controller.ts` que arma el grafo de dependencias:

```ts
import { Request, Response } from 'express';
import { Pool } from 'pg';
import { DesactivarUsuario } from '../../../application/use-cases/desactivar-usuario';
import { PostgresUsuarioRepository } from '../../db/postgres-usuario-repository';

export class UsuarioController {
  static async desactivar(req: Request, res: Response): Promise<void> {
    const pool = req.app.locals.pool as Pool;
    const useCase = new DesactivarUsuario(new PostgresUsuarioRepository(pool));
    await useCase.ejecutar(req.params.id);
    res.status(204).send();
  }
}
```

### 8. Verificar la regla de dependencia

- [ ] Confirmar que `domain/` NO importa nada de `infrastructure/` ni de `application/`:

```bash
grep -rn "from '.*infrastructure" src/domain/   # debe devolver NADA
grep -rn "from 'express" src/domain/ src/application/  # debe devolver NADA
```

---

## Verificación

```bash
grep -rn "from '.*infrastructure" src/domain/   # sin resultados = regla cumplida
grep -rn "from 'express" src/domain/ src/application/   # sin resultados = dominio desacoplado
npx tsc --noEmit   # compila sin errores de tipos
```

**Si los dos `grep` no devuelven nada y `tsc --noEmit` pasa → estructura por capas lista. ✅**

---

## Problemas comunes

| Problema | Causa | Solución |
|---|---|---|
| El dominio importa `express` o `pg` | Alguien puso lógica de infraestructura en una entidad o caso de uso | Mover ese código a `infrastructure/`; el dominio solo conoce sus propias interfaces |
| El caso de uso depende de `PostgresUsuarioRepository` (concreto) | Inyectaste la implementación en vez de la interfaz | Tipar el constructor con `UsuarioRepository` y armar la instancia concreta en el controller o un contenedor de DI |
| Todo el código quedó en `controllers/` | Se usó estructura por capas pero sin respetar las capas | Repartir: validación en middleware, negocio en `application/`, SQL en `infrastructure/db/` |
| `tsc` no encuentra módulos al mover archivos | Los imports relativos (`../../`) quedaron rotos | Revisar la profundidad de cada import o configurar `paths` en `tsconfig.json` |
| Cientos de `new X(new Y(new Z()))` en los controllers | No hay un punto central de composición de dependencias | Crear un `container.ts` (composition root) que construya el grafo una sola vez |

---

## Recursos

- [Uncle Bob — The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Alistair Cockburn — Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [TypeScript — Module Resolution](https://www.typescriptlang.org/docs/handbook/module-resolution.html)
- [Wikipedia — Separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns)

---

## Preguntas de repaso

- **P:** ¿Por qué la carpeta `domain/` no puede importar nada de `infrastructure/`?
**R:** Porque eso rompería la regla de dependencia: el dominio (la capa más valiosa) quedaría acoplado a detalles técnicos como la base de datos o el framework. Si el dominio importa infraestructura, ya no podés cambiar la DB ni testear la lógica en aislamiento.

- **P:** ¿Qué diferencia hay entre `UsuarioRepository` (interfaz) y `PostgresUsuarioRepository` (clase)?
**R:** La interfaz es un puerto (driven) que define QUÉ operaciones necesita el dominio; la clase es un adaptador que define CÓMO se implementan contra Postgres. El dominio depende de la interfaz, nunca de la clase concreta.

- **P:** ¿Dónde se decide qué implementación concreta se usa (Postgres vs Mongo)?
**R:** En el composition root — en este ejemplo el controller, pero idealmente un `container.ts` o el entry point de la app. Ese es el ÚNICO lugar que conoce las implementaciones concretas y arma el grafo de dependencias.

- **P:** ¿Qué pasa si necesitás migrar de Postgres a MongoDB?
**R:** Creás un nuevo adaptador `MongoUsuarioRepository` que implemente la misma interfaz `UsuarioRepository`, lo registrás en el composition root, y el dominio y los casos de uso quedan intactos. Cero cambios en `domain/` y `application/`.

- **P:** ¿Por qué la estructura por capas es importante para testear?
**R:** Porque te permite inyectar un `FakeUsuarioRepository` (en memoria) y probar el caso de uso sin levantar ni la base de datos ni el servidor HTTP. Sin separación de capas, el test se vuelve un test de integración lento y frágil.

- **P:** ¿Cuándo conviene pasar de estructura por capas a estructura por feature?
**R:** Cuando el proyecto crece y un dominio (pedidos, pagos, usuarios) tiene tantos archivos que la carpeta `controllers/` mezcla dominios distintos. Ahí conviene agrupar por feature (`src/pedidos/`, `src/pagos/`) manteniendo las mismas capas dentro de cada feature.
