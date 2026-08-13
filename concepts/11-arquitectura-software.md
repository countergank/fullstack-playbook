# 11. Arquitectura de Software

> Objetivo: entender cómo organizar un codebase para que sea mantenible, testeable y escalable — no por moda, sino porque define cuánto te cuesta cambiar cada feature en el tiempo.

---

## 11.1 Clean Architecture

> Referencia base: [Uncle Bob — The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

*En criollo:* Pensá en una cebolla. En el centro está lo más valioso de tu sistema: las reglas de negocio, lo que hace que tu app sea TU app. En las capas de afuera están los detalles técnicos que cambian todo el tiempo: la base de datos, el framework web, la UI, las librerías. La regla de oro es una sola: **las capas de adentro no conocen a las de afuera**. Nunca. El negocio no sabe si los datos vienen de PostgreSQL o de un archivo JSON; solo sabe que hay "algo" que le entrega usuarios. Eso hace que cambiar la DB o el framework sea un detalle, no una cirugía.

*Técnicamente:* La dependencia siempre apunta hacia adentro (el centro no importa nada externo):

```
        ┌───────────────────────────────┐
        │  Frameworks & Drivers (HTTP,   │
        │  DB, UI, queues)               │
        │  ┌─────────────────────────┐   │
        │  │  Interface Adapters     │   │
        │  │  (controllers, repos,   │   │
        │  │   presenters)           │   │
        │  │  ┌───────────────────┐  │   │
        │  │  │ Application (use  │  │   │
        │  │  │ cases)            │  │   │
        │  │  │ ┌───────────────┐ │  │   │
        │  │  │ │  Entities /  │ │  │   │
        │  │  │ │  Domain      │ │  │   │
        │  │  │ └───────────────┘ │  │   │
        │  │  └───────────────────┘  │   │
        │  └─────────────────────────┘   │
        └───────────────────────────────┘
```

El mismo caso de uso en TypeScript, sin importar qué framework o DB uses:

```ts
// dominio/entidades/Usuario.ts — NO importa nada externo
export class Usuario {
  constructor(
    readonly id: string,
    readonly email: string,
    private activo: boolean,
  ) {}

  desactivar() {
    this.activo = false;
  }
  estaActivo(): boolean {
    return this.activo;
  }
}

// aplicacion/use-cases/DesactivarUsuario.ts
import { Usuario } from '../../dominio/entidades/Usuario';

export interface UsuarioRepository {
  findById(id: string): Promise<Usuario | null>;
  save(usuario: Usuario): Promise<void>;
}

export class DesactivarUsuario {
  constructor(private readonly repo: UsuarioRepository) {}

  async ejecutar(id: string): Promise<void> {
    const usuario = await this.repo.findById(id);
    if (!usuario) throw new Error('No existe');
    usuario.desactivar();
    await this.repo.save(usuario);
  }
}
```

Fijate que `DesactivarUsuario` depende de la **interfaz** `UsuarioRepository`, no de una implementación concreta. El día que cambies Postgres por Mongo, el caso de uso no se entera. Esta es la esencia de la **regla de dependencia** (dependency rule): el código de negocio apunta hacia adentro, y las flechas de dependencia nunca cruzan el límite del dominio hacia afuera.

> **Check de comprensión**
> 1. ¿Cuál es la regla de dependencia en Clean Architecture y por qué existe?
>    - R: Las capas internas no dependen de las externas; las dependencias siempre apuntan hacia adentro. Existe para que los detalles técnicos (DB, framework) sean intercambiables sin tocar la lógica de negocio.
> 2. ¿Por qué el caso de uso depende de una interfaz `UsuarioRepository` y no de la implementación de PostgreSQL?
>    - R: Porque así el dominio queda desacoplado del motor de datos; se puede reemplazar la implementación sin modificar la lógica del caso de uso.
> 3. ¿Qué capa de Clean Architecture cambiarías si migrás de Express a Fastify?
>    - R: Solo la capa de Frameworks & Drivers (los adapters HTTP); los casos de uso y las entidades quedan intactos.
> 4. ¿Dónde vive la lógica de "desactivar un usuario" y por qué no en el controller?
>    - R: En la entidad `Usuario` (dominio) y en el caso de uso; el controller solo traduce HTTP a una llamada, así la regla es testeable sin levantar el server.
> 5. ¿Qué pasa si una entidad del dominio importa `express`?
>    - R: Se rompe la regla de dependencia: el dominio queda acoplado a un framework web y ya no es independiente ni fácil de testear.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.2-arquitectura-de-una-api-rest) — la estructura en capas del backend que Clean Architecture formaliza.

---

## 11.2 Arquitectura Hexagonal (Ports & Adapters)

> Referencia base: [Alistair Cockburn — Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)

*En criollo:* Es la misma idea que Clean Architecture, contada con otra metáfora: tu aplicación es un **hexágono** (el core), y todo lo que entra y sale lo hace por **puertos** (ports). Los puertos son enchufes estándar; los **adaptadores** (adapters) son los cables que conectan esos enchufes al mundo real: un adapter HTTP para la web, un adapter para Postgres, un adapter para un bus de mensajes. Cambiás el cable sin tocar el enchufe ni el core.

*Técnicamente:* Hay dos tipos de puertos:

| Puerto | Dirección | Ejemplo de adaptador |
|--------|-----------|----------------------|
| **Driving** (primario) | El mundo llama a la app | REST controller, CLI, test runner, evento |
| **Driven** (secundario) | La app llama al mundo | PostgreSQL, Mongo, API externa, SMTP |

```ts
// puertos/driving/UsuarioService.ts — qué ofrece la app al mundo
export interface UsuarioService {
  crear(email: string): Promise<Usuario>;
}

// puertos/driven/UsuarioRepo.ts — qué necesita la app del mundo
export interface UsuarioRepo {
  insertar(usuario: Usuario): Promise<void>;
}

// core/UsuarioServiceImpl.ts — lógica pura, no conoce ni HTTP ni SQL
export class UsuarioServiceImpl implements UsuarioService {
  constructor(private readonly repo: UsuarioRepo) {}

  async crear(email: string): Promise<Usuario> {
    const usuario = new Usuario(crypto.randomUUID(), email);
    await this.repo.insertar(usuario);
    return usuario;
  }
}

// adapters/driven/PostgresUsuarioRepo.ts — el "cable" a Postgres
export class PostgresUsuarioRepo implements UsuarioRepo {
  constructor(private readonly db: Pool) {}
  async insertar(u: Usuario): Promise<void> {
    await this.db.query('INSERT INTO usuarios (id, email) VALUES ($1, $2)', [u.id, u.email]);
  }
}
```

La ventaja de testear es enorme: podés escribir un `FakeUsuarioRepo` en memoria y probar `UsuarioServiceImpl` sin levantar ni Postgres ni el servidor HTTP. Eso es lo que hace la arquitectura hexagonal: **el core es testeable en aislamiento total**.

> **Check de comprensión**
> 1. ¿Qué diferencia hay entre un puerto driving y un puerto driven?
>    - R: El driving es por donde el mundo entra a la app (HTTP, CLI); el driven es por donde la app sale al mundo (DB, API externa).
> 2. ¿Qué es un adaptador y qué problema resuelve?
>    - R: Es la implementación concreta de un puerto (ej: `PostgresUsuarioRepo`); permite conectar el core a infraestructura distinta sin tocar la lógica.
> 3. ¿Por qué podés testear el core sin levantar la base de datos?
>    - R: Porque el core depende de interfaces (puertos), así que le inyectás una implementación en memoria en los tests.
> 4. ¿Cuántos adaptadores puede tener un mismo puerto driven?
>    - R: Tantos como necesites (Postgres, Mongo, in-memory para tests); todos implementan la misma interfaz.
> 5. ¿Qué relación tiene la arquitectura hexagonal con Clean Architecture?
>    - R: Son la misma filosofía con distinta metáfora: ambas aíslan el negocio de los detalles técnicos mediante interfaces y regla de dependencia hacia adentro.

→ Ver [Tópico 5: Frameworks Backend](../concepts/05-frameworks-backend.md#5.2-nestjs--el-framework-opinado) — NestJS aplica inyección de dependencias y puertos/adaptadores por diseño.

---

## 11.3 SOLID

> Referencia base: [Wikipedia — SOLID](https://en.wikipedia.org/wiki/SOLID) · [Uncle Bob — Solid Relevance](https://blog.cleancoder.com/uncle-bob/2020/02/01/SolidRelevance.html)

*En criollo:* SOLID son cinco reglas para que tu código no se convierta en una bola de espagueti. No son un objetivo en sí: son una brújula para decidir CUÁNDO dividir una clase o a quién ponerle una dependencia. La más importante de todas es la **D** (Dependency Inversion): depender de abstracciones, no de concreciones — que es exactamente lo que vimos en las dos secciones anteriores.

*Técnicamente:*

| Letra | Principio | En una frase |
|-------|-----------|--------------|
| **S** | Single Responsibility | Una clase tiene UNA razón para cambiar |
| **O** | Open/Closed | Abierta a extensión, cerrada a modificación |
| **L** | Liskov Substitution | Un subtipo debe poder reemplazar a su base sin romper nada |
| **I** | Interface Segregation | Interfaces chicas y específicas, no una gorda |
| **D** | Dependency Inversion | Dependé de abstracciones, no de implementaciones |

Violación y corrección de las dos más comunes:

```ts
// ❌ SRP violado: esta clase mezcla negocio, persistencia y logueo
class Reporte {
  calcular() { /* lógica */ }
  guardarEnDB() { /* SQL */ }
  loguear() { /* console */ }
}

// ✅ SRP: tres razones de cambio → tres clases
class Reporte {
  calcular() { /* solo lógica */ }
}
class ReporteRepo {
  guardar(reporte: Reporte) { /* solo SQL */ }
}
class Logger {
  info(msg: string) { /* solo logging */ }
}

// ❌ DIP violado: dependencia directa a una clase concreta
class PedidoService {
  private repo = new PostgresRepo(); // acoplado
}

// ✅ DIP: dependemos de una abstracción
class PedidoService {
  constructor(private repo: PedidoRepo) {} // PedidoRepo es una interfaz
}
```

> **Check de comprensión**
> 1. ¿Qué significa "una razón para cambiar" en el principio de Single Responsibility?
>    - R: Que una clase debería tener una sola responsabilidad, así solo cambia cuando cambia esa responsabilidad; si cambia por dos motivos distintos, hay que separarla.
> 2. ¿Por qué la D (Dependency Inversion) es la base de Clean/Hexagonal?
>    - R: Porque manda a depender de abstracciones (interfaces) en vez de concreciones, que es justamente lo que desacopla el dominio de la infraestructura.
> 3. ¿Qué diferencia hay entre Inversión de Dependencia e Inyección de Dependencia?
>    - R: La inversión es el principio (depender de abstracciones); la inyección es la técnica para entregar esas dependencias desde afuera (por constructor, por ejemplo).
> 4. ¿Qué es una interfaz gorda y por qué conviene evitarla (ISP)?
>    - R: Una interfaz con muchos métodos fuerza a los clientes a implementar lo que no usan; conviene dividirla en interfaces chicas y específicas.
> 5. ¿Cómo viola Liskov una clase `Cuadrado` que hereda de `Rectangulo`?
>    - R: Si `Rectangulo.setAncho` no es compatible con las invariantes de `Cuadrado` (ancho debe igualar alto), reemplazar el rectángulo por el cuadrado rompe el comportamiento esperado.

→ Ver [Tópico 11.2: Arquitectura Hexagonal](#11.2-arquitectura-hexagonal-ports--adapters) — SOLID en acción con puertos y adaptadores.

---

## 11.4 Patrones de Diseño

> Referencia base: [Refactoring Guru — Design Patterns](https://refactoring.guru/design-patterns) · [Wikipedia — Software design pattern](https://en.wikipedia.org/wiki/Software_design_pattern)

*En criollo:* Son soluciones probadas a problemas que se repiten una y otra vez. No los inventás vos: ya alguien sufrió ese problema y lo resolvió. Aprender patrones te da un vocabulario compartido: en vez de explicar cinco minutos cómo estructuraste algo, decís "acá uso un Repository" y el otro dev entiende al instante. El riesgo es caer en la fiebre del patrón: usarlos de más complica el código en vez de simplificarlo.

*Técnicamente:* Se agrupan en tres familias:

| Familia | Propósito | Ejemplos |
|---------|-----------|----------|
| **Creacionales** | Cómo crear objetos | Factory, Singleton, Builder |
| **Estructurales** | Cómo componer objetos | Adapter, Facade, Decorator |
| **De comportamiento** | Cómo se comunican los objetos | Observer, Strategy, Repository |

Los que más vas a ver en un backend real:

```ts
// Factory: centraliza la creación de un objeto complejo
class PlanFactory {
  static crear(tipo: 'basico' | 'pro'): Plan {
    if (tipo === 'pro') return new Plan({ limite: 1000, soporte: true });
    return new Plan({ limite: 100, soporte: false });
  }
}

// Strategy: cambiás el algoritmo sin cambiar el cliente
interface PagoStrategy {
  cobrar(monto: number): void;
}
class PagoTarjeta implements PagoStrategy {
  cobrar(m: number) { /* tarjeta */ }
}
class PagoMercadoPago implements PagoStrategy {
  cobrar(m: number) { /* MP */ }
}

// Observer: muchos se enteran de un evento
class EventBus {
  private listeners = new Map<string, Function[]>();
  on(event: string, fn: Function) { /* suscribir */ }
  emit(event: string, data: unknown) { /* notificar a todos */ }
}
```

> ⚠️ **Singleton con cuidado**: en Node.js casi nunca lo necesitás porque los módulos ya son singletons por `require`/`import` cacheado.

> **Check de comprensión**
> 1. ¿Cuál es el valor principal de conocer patrones de diseño?
>    - R: Te da soluciones probadas y un vocabulario compartido para comunicar decisiones de diseño sin explicar cada detalle.
> 2. ¿Qué diferencia hay entre un patrón creacional y uno estructural?
>    - R: El creacional se enfoca en cómo crear objetos (Factory); el estructural en cómo componerlos para formar estructuras mayores (Adapter, Facade).
> 3. ¿Por qué el patrón Strategy es útil para pasarelas de pago?
>    - R: Porque podés cambiar el algoritmo de cobro (tarjeta, MercadoPago) sin modificar el código que lo consume.
> 4. ¿Por qué el patrón Singleton es poco útil en Node.js?
>    - R: Porque el sistema de módulos ya cachea las instancias por archivo, así que un módulo importado se comporta como singleton de forma natural.
> 5. ¿Cuándo un patrón se convierte en un problema en vez de una solución?
>    - R: Cuando se aplica por moda o de más, agregando indirección innecesaria; la regla es usarlo solo cuando resuelve un problema real.

→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md#8.2-manejo-de-estado) — el patrón Observer/Store está detrás del manejo de estado en React.

---

## 11.5 Separación de Concerns

> Referencia base: [Wikipedia — Separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns)

*En criollo:* Es la idea más simple y la más violada. Cada pieza de código se ocupa de UNA sola cosa y nada más. El controller no sabe SQL, el service no sabe HTML, el modelo no sabe HTTP. Cuando mezclás concerns, cualquier cambio chiquito te obliga a leer y tocar código que no tiene nada que ver, y testear se vuelve imposible.

*Técnicamente:* El mismo endpoint con concerns mezclados vs separados:

```ts
// ❌ Todo mezclado: HTTP + validación + SQL + regla de negocio en un handler
app.post('/pedidos', async (req, res) => {
  const { userId, items } = req.body;
  if (!userId || !items?.length) return res.status(400).send('Faltan datos');
  const total = items.reduce((s: number, i: any) => s + i.precio, 0);
  await pool.query('INSERT INTO pedidos (user_id, total) VALUES ($1, $2)', [userId, total]);
  res.json({ ok: true, total });
});

// ✅ Separado por concerns
// controller: traduce HTTP ↔ service
app.post('/pedidos', PedidoController.crear);

class PedidoController {
  static async crear(req: Request, res: Response) {
    const result = await pedidoService.crear(req.body);
    res.status(201).json(result);
  }
}
// service: regla de negocio (calcular total, aplicar descuentos)
// repository: única pieza que conoce SQL
// validación: en su propio middleware/pipe
```

La separación también aplica a la estructura de carpetas. Hay dos estilos y ambos son válidos:

| Estilo | Organización | Mejor cuando |
|--------|--------------|--------------|
| **Por capas** (layer-based) | `controllers/`, `services/`, `repos/` | Proyectos chicos, pocos dominios |
| **Por feature** (feature-based) | `pedidos/`, `usuarios/`, `pagos/` | Proyectos que crecen, equipos por dominio |

> **Check de comprensión**
> 1. ¿Qué significa "separación de concerns" en una frase?
>    - R: Cada módulo o función se ocupa de una única responsabilidad y no se mezcla con las demás (HTTP, negocio, persistencia, validación).
> 2. ¿Por qué mezclar SQL dentro de un controller es un problema?
>    - R: Porque acopla la capa HTTP a un motor de datos; cambiás la consulta y tenés que tocar código de presentación, y no podés testear el handler sin una DB.
> 3. ¿Qué ventaja tiene la estructura por feature frente a la por capas?
>    - R: Agrupa todo lo de un dominio junto (controller, service, repo), lo que escala mejor cuando el proyecto crece y los equipos se dividen por dominio.
> 4. ¿En qué capa debería vivir la validación de un request?
>    - R: En su propio middleware/pipe, antes de llegar al service, para no mezclar validación con regla de negocio.
> 5. ¿Cómo se relaciona separación de concerns con SOLID?
>    - R: Es el paraguas conceptual: la S (Single Responsibility) de SOLID es la separación de concerns aplicada a nivel de clase o función.

→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.3-middlewares) — los middlewares de Express son separación de concerns aplicada al request/response.

---

## 11.6 Monolitos vs Microservicios

> Referencia base: [Martin Fowler — Microservices](https://martinfowler.com/articles/microservices.html) · [Martin Fowler — Monolith First](https://martinfowler.com/bliki/MonolithFirst.html)

*En criollo:* Un **monolito** es toda tu app en un solo proceso: un deploy, una base de datos, un repo. Un **microservicio** divide la app en servicios chicos e independientes que se comunican por la red. La trampa es creer que microservicios es "mejor" siempre. No: un monolito bien estructurado (con Clean/Hexagonal adentro) puede llevarte muy lejos. Microservicios es una solución a problemas de ORGANIZACIÓN y ESCALA, no una moda para empezar un proyecto.

*Técnicamente:*

| Criterio | Monolito | Microservicios |
|----------|----------|----------------|
| Deploy | Uno solo, simple | Muchos, requiere orquestación |
| Escalado | Escalás todo junto | Escalás solo lo que lo necesita |
| Acoplamiento | Fuerte en runtime | Fuerte en la RED (latencia, fallos) |
| Complejidad | En el código | En la infraestructura |
| Equipo ideal | 1–2 equipos | Múltiples equipos autónomos |
| Consistencia | Transacciones ACID fáciles | Eventual, transacciones distribuidas |

La regla práctica de Martin Fowler: **empezá con un monolito modular**. Si después el equipo crece y un dominio necesita escalar distinto o deployar más rápido, **extraé** ese servicio de a poco (strangler fig), no reescribas todo de golpe.

```
Monolito modular (fase 1)          Monolito + 1 servicio (fase 2)
┌──────────────────────────┐      ┌──────────────────────────┐
│ pedidos │ pagos │ users  │      │ pedidos │ pagos │        │
│   (un deploy, una DB)    │  →   │   (monolito)            │
└──────────────────────────┘      └──────────┬───────────────┘
                                             │ red (HTTP/queue)
                                             ▼
                                   ┌──────────────────────────┐
                                   │   users (servicio aparte) │
                                   └──────────────────────────┘
```

> **Check de comprensión**
> 1. ¿Cuál es la principal trampa al elegir microservicios?
>    - R: Asumir que son "mejores" por defecto; agregan enorme complejidad de infraestructura y red, y solo rinden cuando hay escala de equipo o de carga que lo justifique.
> 2. ¿Qué significa que en microservicios el acoplamiento se traslada a la red?
>    - R: Que la comunicación pasa a ser por HTTP/mensajería, con latencia, fallos de red y consistencia eventual, problemas que un monolito no tiene.
> 3. ¿Por qué las transacciones ACID son más fáciles en un monolito?
>    - R: Porque todo vive en un mismo proceso y una misma base; en microservicios una transacción que toca varios servicios requiere patrones como sagas.
> 4. ¿Qué es el patrón "strangler fig" y para qué sirve?
>    - R: Extraer funcionalidad de un monolito a servicios nuevos de forma incremental, sin reescribir todo de golpe.
> 5. ¿Qué problema resuelve un monolito modular que NO resuelve un monolito desordenado?
>    - R: Mantiene los límites internos claros (Clean/Hexagonal), de modo que extraer un servicio después es factible; un monolito acoplado hace imposible esa extracción.

→ Ver [Tópico 11.1: Clean Architecture](#11.1-clean-architecture) — la clave para que un monolito sea "modular" y extraíble.

---

## 11.7 DDD (intro)

> Referencia base: [Martin Fowler — DomainDrivenDesign](https://martinfowler.com/bliki/DomainDrivenDesign.html) · [Domain Language — DDD](https://www.domainlanguage.com/ddd/)

*En criollo:* Domain-Driven Design dice que el código debería hablar el **mismo idioma que el negocio**. Si en la inmobiliaria hablan de "propiedades", "inquilinos" y "contratos", tus clases deberían llamarse `Propiedad`, `Inquilino`, `Contrato` — no `Table1`, `Data2`, `GenericHandler`. Y las reglas del negocio (ej: "no se puede alquilar una propiedad ya alquilada") tienen que estar EN el código, modeladas de forma explícita, no escondidas en un `if` suelto.

*Técnicamente:* El concepto central es el **Ubiquitous Language** (lenguaje ubicuo): un vocabulario compartido entre devs y gente de negocio que se refleja en los nombres del código.

```ts
// ❌ Código que no habla el idioma del negocio
function processData(id: string, flag: boolean) {
  if (flag) db.update('t1', { col2: id });
}

// ✅ Ubiquitous language: el código refleja el dominio
class Contrato {
  constructor(
    readonly propiedad: Propiedad,
    readonly inquilino: Inquilino,
    private vigente: boolean,
  ) {}

  cancelar() {
    if (!this.vigente) throw new Error('El contrato ya no está vigente');
    this.vigente = false;
  }
}
```

Piezas clave del DDD (intro, no hace falta dominarlas todas ya):

| Concepto | Qué es |
|----------|--------|
| **Entidad** | Objeto con identidad propia que cambia con el tiempo (`Usuario`, `Contrato`) |
| **Value Object** | Objeto sin identidad, inmutable, definido por sus valores (`Dinero`, `Email`) |
| **Aggregate** | Cluster de entidades con una raíz que garantiza sus invariantes (`Pedido` con sus `LineaPedido`) |
| **Repository** | Abstracción para persistir y recuperar aggregates |
| **Bounded Context** | Frontera donde un término del dominio significa una cosa concreta |

DDD y Clean/Hexagonal se llevan perfecto: el dominio (entidades, aggregates, value objects) vive en el centro de la cebolla/hexágono, y los repositories son puertos driven.

> **Check de comprensión**
> 1. ¿Qué es el Ubiquitous Language y por qué importa?
>    - R: Un vocabulario compartido entre negocio y desarrollo que se refleja en los nombres del código; evita malentendidos y que el código contradiga al dominio.
> 2. ¿Qué diferencia hay entre una Entidad y un Value Object?
>    - R: La entidad tiene identidad propia y muta con el tiempo; el value object no tiene identidad, es inmutable y se define por sus valores.
> 3. ¿Qué es un Aggregate y qué garantiza su raíz?
>    - R: Un cluster de entidades tratado como una unidad; la raíz del aggregate es la única puerta de entrada y garantiza que las invariantes se cumplan.
> 4. ¿Qué es un Bounded Context?
>    - R: Una frontera donde un término del dominio tiene un significado concreto; la misma palabra puede significar cosas distintas en contextos distintos.
> 5. ¿Cómo encaja DDD con la arquitectura hexagonal?
>    - R: El dominio (entidades, aggregates, value objects) vive en el core, y los repositories son puertos driven, manteniendo el negocio desacoplado de la infraestructura.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.2-postgresql) — los aggregates y repositories del DDD se materializan en el esquema relacional.

→ Ver [Tópico 11.1: Clean Architecture](#11.1-clean-architecture) — el dominio de DDD es el centro de la cebolla.

---
