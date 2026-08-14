# 14. Prácticas Profesionales

> Objetivo: entender que ser un buen desarrollador no es solo escribir código — es trabajar con otros. Este tópico cubre los procesos y habilidades blandas que separan a un programador de un profesional: cómo se organiza el trabajo (Agile/Scrum), cómo se revisa y documenta el código, cómo se comunica, cómo se levantan y estiman requerimientos, y cómo se trabaja en equipo sin fricción. Sin esto, tu código por bueno que sea no llega a producción, o llega y nadie puede mantenerlo.

> Nota: este es el único tópico sin guides (no hay herramientas que instalar), por eso cada sección se desarrolla con más profundidad conceptual. Las prácticas acá son transferibles a cualquier stack y a cualquier equipo.

---

## 14.1 Agile / Scrum

> Referencia base: [Scrum Guide](https://scrumguides.org/) · [Manifesto Ágil](https://agilemanifesto.org/iso/es/manifesto.html) · [Atlassian — Scrum](https://www.atlassian.com/agile/scrum)

*En criollo:* Agile es una forma de encarar el desarrollo que prioriza **entregar valor chico y frecuente** en vez de planear todo y construir en secreto durante meses. Scrum es el framework ágil más usado: un equipo chico (3–9 personas), ciclos cortos llamados *sprints* (1–4 semanas), y una serie de reuniones cortas para mantener a todos alineados. La idea de fondo es simple: **es más barato corregir el rumbo cada dos semanas que después de un año**.

*Técnicamente:* Scrum define tres roles, cinco eventos y tres artefactos. No es un proceso rígido: es un marco que tu equipo adapta (de ahí "Scrum Guide", no "Scrum Law").

**Roles**

| Rol | Qué hace | Qué NO hace |
|-----|----------|-------------|
| Product Owner (PO) | Define y prioriza el backlog por valor de negocio | No asigna tareas técnicas al equipo |
| Scrum Master (SM) | Facilita el proceso, remueve impedimentos, protege al equipo | No es el jefe; es un coach |
| Developers (Dev Team) | Estiman, diseñan, construyen y entregan el incremento | No reciben tareas en cascada: se auto-organizan |

**Eventos** (todos con *timebox* fijo)

| Evento | Duración típica | Propósito |
|--------|-----------------|-----------|
| Sprint Planning | ≤8h (sprint de 1 mes) | Elegir QUÉ se hace y CÓMO |
| Daily Scrum | 15 min | Sincronizar: qué hice, qué voy a hacer, qué me bloquea |
| Sprint Review | ≤4h | Mostrar el incremento al stakeholder y recibir feedback |
| Sprint Retrospective | ≤3h | Mejorar el proceso (no el producto) |

**Artefactos**

- **Product Backlog**: lista ordenada de TODO lo que podría hacerse (features, bugs, tech debt). Solo el PO la prioriza.
- **Sprint Backlog**: subconjunto del backlog comprometido para el sprint + el plan de cómo hacerlo.
- **Increment**: la suma de todo lo terminado en el sprint, que debe cumplir la **Definition of Done**.

La **Definition of Done (DoD)** es el contrato de calidad del equipo: qué significa "terminado" (¿testeado? ¿documentado? ¿deployeado?). Sin una DoD explícita, "terminado" significa "compila en mi máquina", y eso es una bomba de tiempo.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre Agile y Scrum?
>    - R: Agile es una filosofía/valores (entregar valor frecuente, adaptarse al cambio); Scrum es un framework concreto (roles, eventos, artefactos) que implementa esos valores.
> 2. ¿Qué es un sprint y cuánto suele durar?
>    - R: Es un ciclo de trabajo acotado en el tiempo (1–4 semanas) al final del cual el equipo entrega un incremento potencialmente releasable.
> 3. ¿Cuál es la diferencia entre Product Backlog y Sprint Backlog?
>    - R: El Product Backlog es la lista completa y priorizada de todo lo pendiente (lo gestiona el PO); el Sprint Backlog es el subconjunto que el equipo se comprometió a hacer en el sprint actual.
> 4. ¿Qué es la Definition of Done y por qué es crítica?
>    - R: Es el contrato de calidad del equipo que define qué significa "terminado" (testeado, documentado, deployeado). Sin ella, "terminado" es ambiguo y se acumula deuda técnica invisible.
> 5. ¿Qué rol NO debería asumir el Scrum Master?
>    - R: El de jefe. El Scrum Master es un facilitador/coach que remueve impedimentos y protege el proceso; no asigna tareas ni manda sobre el equipo, que es auto-organizado.
> 6. ¿Para qué sirve la Retrospective y en qué se diferencia de la Review?
>    - R: La Review muestra el producto al stakeholder para obtener feedback sobre QUÉ se construyó; la Retro es interna y mejora CÓMO se trabaja (el proceso), no el producto.

→ Ver [Tópico 3: IA & Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.4-sdd-spec-driven-development) — SDD organiza el trabajo en fases con artefactos, una filosofía compatible con Agile (iterar chico y validar temprano).

---

## 14.2 Code Review

> Referencia base: [Google Engineering Practices — Code Review](https://google.github.io/eng-practices/review/) · [Conventional Commits](https://www.conventionalcommits.org/) · [GitHub — About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests)

*En criollo:* El code review es que OTRA persona lea tu código antes de que se mezcle a `main`. No es un examen ni una caza de culpables: es una red de seguridad y una herramienta de aprendizaje mutuo. Un buen review caza bugs que vos no ves (estás "ciego" de tu propio código), mantiene la coherencia del estilo, y enseña a todo el equipo los patrones que convienen. La regla de oro: **revisá el código, nunca a la persona**.

*Técnicamente:* El flujo estándar es el **pull request** (PR): un branch con tu cambio, una descripción de contexto, y revisores asignados que dejan comentarios y aprueban o piden cambios.

**Qué mirar en un review** (de lo más a lo menos crítico):

1. **Corrección**: ¿hace lo que dice? ¿los casos borde están cubiertos?
2. **Seguridad y performance**: ¿inyección, secretos expuestos, N+1, bucles innecesarios?
3. **Diseño**: ¿el código está en el lugar correcto? ¿respeta la separación de concerns?
4. **Tests**: ¿el cambio trae tests que cubren el comportamiento nuevo?
5. **Legibilidad**: ¿los nombres dicen lo que hacen? ¿hay código muerto?
6. **Estilo**: formato, lint, convenciones del repo.

**Buenas prácticas (de los dos lados)**

| Como autor | Como revisor |
|------------|--------------|
| PRs chicos (ideal <400 líneas cambiadas) | Respondé en <24h, un PR viejo se pudre |
| Describí el PORQUÉ, no el qué (el diff ya muestra el qué) | Distinguí "bloqueante" de "sugerencia" (nit) |
| Un PR = un propósito (una feature, un fix, una refactor) | Preguntá en vez de afirmar ("¿por qué acá y no en X?") |
| No te lo tomes personal | Señalá lo bueno también |

**Conventional Commits** es la convención de mensajes de commit que hace los reviews y el changelog predecibles:

```
feat(api): agrega endpoint de login con JWT
fix(orders): corrige cálculo de total con descuentos
refactor(db): extrae helper de paginación
```

El mensaje tiene la forma `tipo(alcance): descripción`, donde `tipo` ∈ `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, etc. Eso permite versionado semántico automático y que cualquier persona sepa el impacto de un commit leyendo una línea.

> **Check de comprensión**
> 1. ¿Cuál es el propósito principal de un code review?
>    - R: Que otra persona revise el código antes de mezclarlo, para cazar bugs, mantener coherencia de estilo, compartir conocimiento y reducir el riesgo de regresiones.
> 2. ¿Por qué es mejor revisar en un orden de crítico a cosmético (corrección → seguridad → diseño → estilo)?
>    - R: Porque el costo de un bug o una falla de seguridad es enorme, mientras que el estilo es automatizable con lint; concentrar la energía del revisor en lo que más impacta.
> 3. ¿Qué significa "PRs chicos" y por qué importa?
>    - R: PRs con pocas líneas cambiadas (ideal <400), que son más fáciles de revisar a fondo. Un PR gigante abruma al revisor, que termina aprobando por cansancio sin revisar de verdad.
> 4. ¿Cuál es la diferencia entre un comentario bloqueante y un "nit"?
>    - R: El bloqueante impide el merge (un bug, un riesgo); el nit es una sugerencia cosmética o de preferencia que el autor puede aceptar o ignorar sin trabar el PR.
> 5. ¿Para qué sirve Conventional Commits y cómo se estructura un mensaje?
>    - R: Estandariza los mensajes con la forma `tipo(alcance): descripción` (ej. `fix(orders): ...`), lo que habilita versionado semántico automático y hace el historial y el changelog predecibles.
> 6. ¿Por qué el autor debería describir el PORQUÉ de un cambio y no el QUÉ?
>    - R: Porque el diff ya muestra QUÉ cambió; lo que el revisor no puede inferir es la motivación y las decisiones detrás, que son el contexto que hace el review útil.

→ Ver [Tópico 3: IA & Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.10-code-review-asistido-por-ia) — el review asistido por IA que acelera (pero no reemplaza) la revisión humana.
→ Ver [Tópico 11: Arquitectura de Software](../concepts/11-arquitectura-software.md#11.3-solid) — SOLID como el criterio de "diseño correcto" que buscás en un review.

---

## 14.3 Documentación Técnica

> Referencia base: [Write the Docs](https://www.writethedocs.org/) · [Documentation System (diátaxis)](https://diataxis.fr/) · [ADR — Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)

*En criollo:* La documentación es la interfaz entre tu código y el futuro — incluyendo el futuro vos, que en seis meses no se va a acordar de nada. No documentes para el "por las dudas": documentá lo que NO se puede deducir leyendo el código. El código dice el *cómo*; la documentación debe decir el *por qué*. Un README decente, las decisiones de arquitectura registradas, y una API documentada valen más que mil comentarios que explican lo obvio.

*Técnicamente:* Hay tipos de documentación con propósitos distintos. El framework **diátaxis** los clasifica por su objetivo, no por su formato:

| Tipo | Responde | Ejemplo |
|------|----------|---------|
| Tutorial | "¿Cómo empiezo desde cero?" | Guía paso a paso de onboarding |
| How-to | "¿Cómo resuelvo X puntual?" | "Cómo agregar un índice a una tabla" |
| Reference | "¿Qué hace esta función exactamente?" | API docs generadas (JSDoc, OpenAPI) |
| Explanation | "¿Por qué se hizo así?" | ADRs, design docs |

**Reglas de oro**

- **El README primero**: qué es el proyecto, cómo correrlo, cómo testearlo, cómo contribuir. Es lo primero (y a veces lo único) que lee alguien nuevo.
- **Docs as code**: la documentación vive en el repo, en Markdown, versionada con el código y revisada en PR como cualquier otro cambio.
- **ADRs (Architecture Decision Records)**: registran decisiones de arquitectura con contexto, consecuencias y alternativas rechazadas. Sin ADRs, en un año nadie sabe POR QUÉ se eligió PostgreSQL y no MongoDB, y alguien lo "corrige" rompiendo todo.
- **No documentes lo obvio**: `// suma dos números` es ruido; documentá el porqué no trivial.
- **API con ejemplos**: una referencia de API sin un ejemplo de request/response es inútil.

```markdown
<!-- ADR mínimo: contexto, decisión, consecuencias, alternativas -->
# ADR-003: PostgreSQL como base principal

## Contexto
Necesitamos transacciones ACID y joins complejos para el módulo de órdenes.

## Decisión
Usamos PostgreSQL como base relacional principal; MongoDB solo para datos documentales sueltos.

## Consecuencias
- Positivo: integridad transaccional, SQL maduro.
- Negativo: un motor más que operar y backupear.

## Alternativas rechazadas
- MongoDB para todo: pierde integridad transaccional en órdenes.
```

> **Check de comprensión**
> 1. ¿Qué debería documentar la documentación: el cómo o el porqué?
>    - R: El porqué. El código ya muestra el cómo; la documentación debe capturar las decisiones, contexto y motivaciones que no se deducen leyendo el código.
> 2. ¿Cuál es la diferencia entre un tutorial y un how-to según diátaxis?
>    - R: El tutorial guía a alguien que no sabe nada desde cero (progresión de aprendizaje); el how-to resuelve un problema puntual para alguien que ya entiende el contexto.
> 3. ¿Qué es un ADR y qué problema resuelve?
>    - R: Un registro de decisión de arquitectura con contexto, decisión, consecuencias y alternativas rechazadas. Resuelve el "por qué se eligió esto" que se pierde con el tiempo y previene reversiones por ignorancia.
> 4. ¿Qué significa "docs as code" y qué ventaja da?
>    - R: Tratar la documentación como código: vive en el repo en Markdown, versionada con el código y revisada en PR. Ventaja: la doc nunca se desincroniza del código y es revisable como cualquier cambio.
> 5. ¿Por qué documentar lo obvio es perjudicial?
>    - R: Porque agrega ruido que se desactualiza y entierra la información valiosa; un comentario que explica `suma dos números` no aporta y se convierte en mentira cuando el código cambia.
> 6. ¿Por qué una API de referencia necesita ejemplos de request/response?
>    - R: Porque la referencia describe la firma pero no muestra el uso; un ejemplo real es lo que un consumidor copia y pega para entender cómo llamar al endpoint.

→ Ver [Tópico 3: IA & Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.4-sdd-spec-driven-development) — SDD produce especificaciones y diseños que son documentación viva del sistema.

---

## 14.4 Comunicación Efectiva

> Referencia base: [Atlassian — Remote work & async](https://www.atlassian.com/software/confluence/templates) · [Basecamp — Shape Up (comunicación escrita)](https://basecamp.com/shapeup) · [Rands — Writing](https://randsinrepose.com/)

*En criollo:* En un equipo, la mayor fuente de bugs no es el código: es el **malentendido**. Dos personas con visiones distintas de la misma tarea, un requerimiento interpretado de dos formas, un "dale, ya lo veo" que nadie vio. Comunicar bien es, sobre todo, comunicar **por escrito y de forma asíncrona**: un mensaje claro que otra persona entiende sin que tengas que explicarlo en persona. En equipos remotos esto deja de ser una cortesía y pasa a ser infraestructura.

*Técnicamente:* Los principios de comunicación efectiva en ingeniería:

1. **Async-first**: escribí para que te lean cuando quieran, no cuando estés. Documentá decisiones en el PR, en el issue, en el doc — no en un mensaje efímero de chat.
2. **Contexto antes que pregunta**: al pedir ayuda, incluí QUÉ intentaste, QUÉ esperabas y QUÉ pasó (el error completo). "No funciona" obliga al otro a adivinar.
3. **Una idea por mensaje**: no mezcles tres pedidos en un párrafo; se pierden dos.
4. **Voz activa y específica**: "el endpoint `/login` devuelve 500 con un token expirado" en vez de "hay un problema con el login".

**Update de status efectivo** (async daily):

```
- Ayer: cerré el PR de login (mergeado) y arranqué el módulo de pagos.
- Hoy: sigo con pagos; quiero terminar el flujo de refund.
- Bloqueos: necesito acceso a la API sandbox del gateway de pagos.
```

**Dar y recibir feedback**

| Mal feedback | Buen feedback |
|--------------|---------------|
| "Tu código es un desastre" | "En `pagos.ts` el handler hace 3 cosas; ¿podemos separarlo en funciones?" |
| Vago y tardío | Específico, sobre el hecho, y cerca del momento |
| Sobre la persona | Sobre el comportamiento/artefacto |

- **En público lo positivo, en privado lo correctivo.** Nada desmotiva más que una crítica pública.
- **Pedí feedback**: "¿qué habrías hecho distinto en este diseño?" te hace crecer más rápido que esperar a que te lo den.

> **Check de comprensión**
> 1. ¿Por qué la comunicación asíncrona es "infraestructura" en equipos remotos?
>    - R: Porque sin horarios compartidos, la única forma de que la información fluya es que esté escrita y accesible; el chat efímero y las decisiones habladas se pierden o bloquean a quien no estuvo.
> 2. ¿Qué debería incluir un pedido de ayuda para que sea efectivo?
>    - R: Qué intentaste, qué esperabas que pasara y qué pasó realmente (con el error completo). "No funciona" sin contexto obliga al otro a adivinar y pierde tiempo.
> 3. ¿Qué diferencia un buen feedback de uno malo?
>    - R: El bueno es específico, sobre el artefacto o comportamiento (no sobre la persona), y cercano al momento del hecho; el malo es vago, tardío y personal.
> 4. ¿Por qué "en público lo positivo, en privado lo correctivo"?
>    - R: Porque la crítica pública humilla y desmotiva, y hace que la persona se ponga a la defensiva en vez de aprender; el elogio público refuerza comportamientos que querés que se repitan.
> 5. ¿Qué problema evita "una idea por mensaje"?
>    - R: Evita que varios pedidos mezclados en un párrafo hagan que el lector responda solo uno y se pierdan los demás; cada mensaje debe tener un único foco accionable.
> 6. ¿Por qué pedir feedback activamente te hace crecer más rápido?
>    - R: Porque no esperás a que te lo den (que suele ser tarde o nunca); preguntar dirigido ("¿qué harías distinto?") obtiene perspectivas accionables sobre tus decisiones.

→ Ver [Tópico 3: IA & Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.6-sub-agentes-delegacion-y-trabajo-en-paralelo) — la delegación efectiva a sub-agentes depende de escribir prompts con contexto claro, la misma habilidad que la comunicación async.

---

## 14.5 Toma de Requerimientos

> Referencia base: [Mountain Goat Software — User Stories](https://www.mountaingoatsoftware.com/agile/user-stories) · [INVEST criteria](https://en.wikipedia.org/wiki/INVEST_(mnemonic)) · [Given-When-Then](https://martinfowler.com/bliki/GivenWhenThen.html)

*En criollo:* Tomar requerimientos es traducir lo que un cliente O CREE que quiere en algo que un equipo puede construir sin ambigüedad. El cliente dice "quiero que la app sea rápida"; el requerimiento de verdad es "la lista de órdenes debe cargar en menos de 2 segundos con 10.000 registros". El artefacto estándar es la **user story** con sus **criterios de aceptación**: la historia describe el QUÉ desde la perspectiva del usuario, y los criterios definen de forma verificable cuándo está "terminado".

*Técnicamente:* Una user story sigue el formato clásico:

> Como **[rol]**, quiero **[capacidad]**, para **[beneficio]**.

```
Como usuario registrado, quiero filtrar mis órdenes por estado, para encontrar rápido las que están pendientes de pago.
```

**Criterios de aceptación** (formato Given-When-Then, el mismo que usan los tests BDD):

```
Given que estoy logueado y tengo 50 órdenes
When  filtro por "pendiente"
Then  solo veo las órdenes con estado "pendiente"
And   el total se recalcula para reflejar el filtro
```

Un criterio de aceptación **debe ser verificable**: si no podés escribir un test que lo valide, es una opinión, no un requerimiento. "La app debe ser rápida" no es un criterio; "cargar en <2s" sí.

**Características de una buena historia (INVEST)**

| Letra | Significa | Pregunta de control |
|-------|-----------|---------------------|
| I | Independent | ¿Puede desarrollarse sin depender de otra historia? |
| N | Negotiable | ¿Es el "cómo" flexible? |
| V | Valuable | ¿Aporta valor al usuario o negocio? |
| E | Estimable | ¿Puede estimarse con razonable certeza? |
| S | Small | ¿Es lo bastante chica para un sprint? |
| T | Testable | ¿Tiene criterios de aceptación verificables? |

**Errores clásicos**

- **Scope creep**: el cliente agrega "de paso" un requisito a mitad de sprint. Defensa: nuevo pedido → nuevo backlog → se prioriza, no se inyecta en el sprint en curso.
- **Asumir en vez de preguntar**: "me imaginé que lo querías así". El costo de una suposición errada crece exponencialmente cuanto más tarde se descubre.
- **Requerimientos de solución en vez de problema**: "quiero un botón rojo acá" es una solución; el requerimiento real es "quiero que el usuario note el estado de error". Si capturás la solución y no el problema, atás el diseño a una ocurrencia.

> **Check de comprensión**
> 1. ¿Cuál es la diferencia entre un deseo del cliente y un requerimiento bien tomado?
>    - R: El deseo es vago y subjetivo ("que sea rápida"); el requerimiento es específico y verificable ("cargar en <2s con 10k registros"), con criterios de aceptación que un test puede validar.
> 2. ¿Cuáles son las tres partes de una user story y qué expresa cada una?
>    - R: Rol (quién), capacidad (qué quiere), beneficio (por qué lo quiere). El formato "Como [rol], quiero [capacidad], para [beneficio]" ancla la historia en valor de usuario.
> 3. ¿Qué es el formato Given-When-Then y por qué importa?
>    - R: Es la estructura de los criterios de aceptación (contexto, acción, resultado esperado). Importa porque cada cláusula es directamente traducible a un test, haciendo el requerimiento verificable.
> 4. ¿Qué significa la "T" de INVEST y por qué es crítica?
>    - R: Testable: la historia debe tener criterios de aceptación verificables. Sin T, no hay forma objetiva de saber cuándo está terminada, y "terminado" se vuelve negociable.
> 5. ¿Qué es el scope creep y cómo se maneja sin matar la relación con el cliente?
>    - R: Es la acumulación de requisitos nuevos a mitad de sprint. Se maneja redirigiendo todo pedido nuevo al backlog para priorizarlo en el próximo sprint, en vez de inyectarlo en el sprint en curso.
> 6. ¿Por qué es un error capturar la solución ("un botón rojo") en vez del problema ("que se note el error")?
>    - R: Porque atás el diseño a una ocurrencia puntual y perdés el objetivo real; capturar el problema deja al equipo libre para elegir la mejor solución y evita construir lo que no se necesitaba.

→ Ver [Tópico 3: IA & Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.4-sdd-spec-driven-development) — la fase de spec de SDD formaliza exactamente esta disciplina: requerimientos con escenarios Given-When-Then.

---

## 14.6 Estimación

> Referencia base: [Mountain Goat Software — Story Points](https://www.mountaingoatsoftware.com/agile/story-points) · [Planning Poker](https://www.planningpoker.com/) · [#NoEstimates](https://oikosofyseries.com/no-estimates-book)

*En criollo:* Estimar es responder "¿cuánto va a tardar esto?" — y es de las cosas que peor hacemos por instinto. El error clásico es estimar en horas y pensando solo en escribir código, olvidando el diseño, las pruebas, la revisión, los imprevistos y el despliegue. La estimación no es una promesa: es una herramienta de planificación que se vuelve más precisa con la práctica y con datos históricos, no con más decimales.

*Técnicamente:* La unidad estándar en equipos ágiles es el **story point**: una medida RELATIVA de tamaño/esfuerzo/complejidad/riesgo, no de tiempo. Una historia de 2 puntos es (más o menos) el doble de grande que una de 1; no "2 horas".

**Escalas comunes**

| Escala | Valores | Cuándo |
|--------|---------|--------|
| Fibonacci modificada | 1, 2, 3, 5, 8, 13, 20, 40, 100 | La más usada; los saltos reflejan incertidumbre creciente |
| T-shirt sizes | XS, S, M, L, XL | Estimación rápida de features grandes |
| Planning Poker | — | Técnica de consenso, no una escala |

**Planning Poker**: todos estiman en privado, muestran su carta a la vez, y los extremos (el que puso 3 y el que puso 13) explican su razonamiento. Repetís hasta converger. Evita el *anchoring* (que el primero que habla arrastre al resto) y el *groupthink*.

**Velocity**: la suma de puntos COMPLETADOS en sprints pasados. Sirve para predecir cuánto podés comprometer en el próximo sprint. Se calcula con historia real, no con deseos:

```
Velocity promedio = (20 + 24 + 19 + 23) / 4 = 21.5 puntos por sprint
```

**Reglas de oro**

- Estimá el tamaño RELATIVO entre historias ("esto es el doble de aquello"), no horas absolutas.
- Estimá en EQUIPO: distintas personas ven riesgos distintos.
- Estimá incluyendo TODO: diseño, code, tests, review, docs, deploy, imprevistos.
- Aceptá la incertidumbre: una historia de 13+ probablemente está mal definida y debería partirse.
- No confundas estimación con compromiso: la estimación es un pronóstico, no una promesa de entrega.

> **Check de comprensión**
> 1. ¿Por qué se estima en story points y no en horas?
>    - R: Porque los puntos miden tamaño/esfuerzo relativo entre historias (independiente de quién la haga y de su velocidad), mientras que las horas son falsas de precisión y varían por persona y por contexto.
> 2. ¿Qué es el "anchoring" y cómo lo evita Planning Poker?
>    - R: Es el sesgo donde la primera estimación dicha en voz alta arrastra al resto del equipo. Planning Poker lo evita haciendo que todos estimen en privado y muestren a la vez, antes de discutir.
> 3. ¿Qué es la velocity y para qué se usa?
>    - R: Es el promedio de puntos COMPLETADOS en sprints pasados. Se usa para predecir cuánto puede comprometer el equipo en el próximo sprint, basado en historia real.
> 4. ¿Por qué una historia de 13 o más puntos es una señal de alarma?
>    - R: Porque probablemente está mal definida o es demasiado grande; la incertidumbre es alta, y casi siempre conviene partirla en historias más chicas y estimables.
> 5. ¿Qué elementos se olvidan típicamente al estimar y hay que incluir?
>    - R: El diseño, las pruebas, la revisión, la documentación, el despliegue y los imprevistos. Estimar solo "escribir código" subestima sistemáticamente.
> 6. ¿Por qué la estimación no es un compromiso de entrega?
>    - R: Porque es un pronóstico con incertidumbre inherente; tratarla como promesa lleva a calidad sacrificada o a crunch cuando el pronóstico falla. El compromiso real se define al inicio de cada sprint con la información disponible.

→ Ver [Tópico 3: IA & Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.4-sdd-spec-driven-development) — la fase de tasks de SDD descompone el trabajo en unidades estimables y planifica por review budget, análogo a estimar con puntos.

---

## 14.7 Trabajo en Equipo

> Referencia base: [Google — Project Aristotle (re:Work)](https://rework.withgoogle.com/print/guides/5721312655835136/) · [Pair Programming](https://martinfowler.com/articles/on-pair-programming.html) · [Mob Programming](https://www.agilealliance.org/glossary/mob-programming/)

*En criollo:* El trabajo en equipo no es "dividir el trabajo y que cada uno haga lo suyo en su silo": es construir una red donde el conocimiento fluye y el todo rinde más que la suma de las partes. El estudio **Project Aristotle** de Google encontró que lo que más predice un equipo de alto rendimiento NO es la inteligencia individual de sus miembros, sino la **seguridad psicológica**: que nadie tenga miedo de preguntar, admitir un error o proponer algo tonto. Un equipo donde preguntar es castigado es un equipo donde los errores se esconden hasta que explotan.

*Técnicamente:* Los mecanismos concretos de colaboración:

**Seguridad psicológica** (el predictor #1 de equipos efectivos)

- El error se discute como sistema a mejorar, no como culpa a castigar ("¿qué falló en el proceso?" en vez de "¿quién la pifió?").
- Preguntar siempre está bien; "no sé" es una respuesta válida y productiva.
- Las ideas se critican por su mérito, no por quién las propone.

**Prácticas de colaboración**

| Práctica | Qué es | Cuándo rinde |
|----------|--------|--------------|
| Pair programming | Dos personas, una máquina: una escribe (driver), la otra revisa en vivo (navigator) | Código complejo, onboarding, decisiones de diseño |
| Mob programming | Todo el equipo, una máquina, rotando | Features críticas, refinamiento de historias |
| Code review | Revisión asíncrona por PR | El default de todo cambio |
| Knowledge sharing | Charlas técnicas, demos, pairing cruzado | Evitar el "silo de conocimiento" |

**El silo de conocimiento** es el riesgo #1 de un equipo: cuando solo UNA persona sabe cómo funciona un módulo. Si se va (vacaciones, otro trabajo, accidente), ese conocimiento se va con ella. Las defensas: rotar tareas, documentar, pair programming, y no dejar que nadie sea "dueño exclusivo" de nada crítico.

**Roles y responsabilidades claras**

- Cada tarea tiene UN responsable final (accountability), aunque varios contribuyan.
- El responsable no es quien "hace todo", sino quien se asegura de que se haga y se comunique.
- Los acuerdos se escriben: un "dale, arreglamos así" hablado se desvanece; un comentario en el issue queda.

**Resolución de conflictos**

1. Atacá el problema, no a la persona.
2. Buscá el dato objetivo: medí, probá, benchmarkeá — no discutas opiniones como si fueran hechos.
3. Escalá temprano: un conflicto arrastrado semanas pudre al equipo; llevarlo a una persona de confianza no es fracasar, es gestionar.

> **Check de comprensión**
> 1. ¿Qué encontró el Project Aristotle de Google sobre los equipos de alto rendimiento?
>    - R: Que el predictor #1 no es la inteligencia individual de los miembros, sino la seguridad psicológica: la confianza de que nadie será castigado por preguntar, errar o proponer.
> 2. ¿Qué es la seguridad psicológica y por qué importa más que el talento?
>    - R: Es la certeza de poder hablar sin miedo a represalias. Importa más porque sin ella los errores se esconden, la gente no pide ayuda y el conocimiento no fluye — anulando cualquier talento individual.
> 3. ¿Cuál es la diferencia entre pair programming y mob programming?
>    - R: En pair son DOS personas con una máquina (driver escribe, navigator revisa en vivo); en mob es TODO el equipo con una máquina rotando el teclado.
> 4. ¿Qué es un silo de conocimiento y cómo se combate?
>    - R: Es cuando solo una persona sabe cómo funciona un módulo. Se combate rotando tareas, documentando, haciendo pair programming y evitando dueños exclusivos de componentes críticos.
> 5. ¿Qué significa que una tarea tenga "un responsable final" si varios contribuyen?
>    - R: Que hay una única persona accountable que se asegura de que la tarea se complete y se comunique, aunque no sea la que hace todo el trabajo; evita el "pensé que lo hacías vos".
> 6. ¿Por qué "atacá el problema, no a la persona" es la base de la resolución de conflictos?
>    - R: Porque personalizar la crítica activa la defensividad y rompe la seguridad psicológica; enfocar el problema permite discutir con datos objetivos y encontrar una solución sin dañar la relación.

→ Ver [Tópico 3: IA & Desarrollo Asistido](../concepts/03-ia-desarrollo-asistido.md#3.6-sub-agentes-delegacion-y-trabajo-en-paralelo) — trabajar con sub-agentes es un equipo de colaboradores: delegación clara, contexto y revisión, igual que con personas.

---

> **Check de comprensión — cierre del tópico**
> 1. ¿Por qué las prácticas profesionales de este tópico importan aunque no haya código que instalar?
>    - R: Porque determinan si el código llega a producción y se mantiene en el tiempo; un desarrollador brillante sin procesos de revisión, comunicación y documentación genera deuda técnica y fricción de equipo.
