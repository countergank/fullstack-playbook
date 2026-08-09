# 3. IA & Desarrollo Asistido

> Objetivo: entender cómo funciona la IA que usás para desarrollar, cómo sacarle el máximo provecho con buenos prompts y contexto, y conocer el ecosistema de herramientas que ya son parte del stack profesional.

---

## 3.1 LLMs — Cómo funcionan los Modelos de Lenguaje

*En criollo:* Un LLM (Large Language Model) es un programa que predice la palabra más probable que sigue, dado todo el texto anterior. Entrenado sobre cantidades masivas de texto de internet, libros, y código, "aprende" patrones de lenguaje, razonamiento y programación. No "piensa" como un humano, no "sabe" cosas en el sentido tradicional — es una máquina de completar texto extremadamente sofisticada. El arte de usarlo bien está en darle el contexto justo para que la predicción sea la correcta.

### Conceptos clave

**Tokens**
- Un token es la unidad mínima de texto que el modelo procesa. No es una palabra ni un carácter — es un pedazo de palabra. Ejemplo: "desarrollando" se puede dividir en `["des", "arroll", "ando"]`.
- El modelo cobra por token (entrada + salida) o limita la cantidad de tokens por request.
- Los tokens definen la **ventana de contexto** — cuánto texto "recuerda" el modelo en una conversación.

**Temperatura**
- Controla qué tan "creativo" o "determinista" es el modelo (rango 0 a 2, típicamente 0 a 1).
- **Temperatura baja (0-0.3)**: respuestas precisas, consistentes, predecibles. Ideal para código, hechos, traducción.
- **Temperatura media (0.5-0.7)**: balance entre creatividad y precisión.
- **Temperatura alta (0.8-1.0)**: respuestas variadas, creativas, a veces sorprendentes. Ideal para brainstorming, ideas, contenido creativo.

**Entrenamiento vs Inferencia**
- **Entrenamiento**: el proceso de "enseñarle" al modelo con terabytes de texto. Tarda semanas/meses, cuesta millones. Lo hace la empresa que crea el modelo.
- **Inferencia**: cuando vos usás el modelo (como ahora). El modelo ya está entrenado, solo "predice" tokens basado en lo aprendido. Es rápido y barato en comparación.

**Modelos relevantes en desarrollo**
| Modelo | Creador | Ideal para |
|--------|---------|------------|
| Claude (3.5/4) | Anthropic | Razonamiento profundo, código complejo, análisis |
| GPT-4o | OpenAI | Uso general, código, multimodal (imágenes) |
| Gemini | Google | Contexto largo (1M+ tokens), análisis de código |
| DeepSeek | DeepSeek | Código, razonamiento, open-source |
| Qwen | Alibaba | Código, open-source, fine-tuneable |
| Grok | xAI | Respuestas técnicas directas |

---

## 3.2 Prompt Engineering

Prompt engineering NO es "escribir lindo". Es darle al modelo el CONTEXTO preciso para que produzca la respuesta correcta. El 80% de los malos resultados vienen de prompts ambiguos, incompletos o sin restricciones claras.

### Principios fundamentales

1. **Sé específico, no educado**. "Por favor, ¿podrías..." no suma. "Escribí una función que..." va directo al grano.

2. **Dale contexto, no misterio**. Decile QUÉ estás haciendo, EN QUÉ proyecto, CON QUÉ stack.
   - ❌ "Arreglá este error"
   - ✅ "Estoy en un proyecto Node.js con Express y TypeScript. Esta ruta POST /api/users tira un 500 cuando el email ya existe. Debería devolver 409 Conflict. Acá está el código del controller: [...]"

3. **Mostrale ejemplos (few-shot)**. Si querés un formato específico, mostrale UN ejemplo de lo que esperás. El modelo imita patrones mucho mejor de lo que sigue instrucciones abstractas.

4. **Dividí problemas grandes**. En vez de "haceme un ecommerce", empezá por "diseñemos el schema de la base de datos", después "armemos las rutas de autenticación", después "el carrito de compras". Una conversación por feature.

5. **Restricciones explícitas**. Si algo NO debe hacerse, decilo.
   - "No uses librerías externas"
   - "El código debe funcionar en Node.js 18 sin ESM"
   - "No modifiques los archivos de configuración existentes"

6. **Rol y tono**. Asignale un rol cuando el contexto lo requiera.
   - "Actuá como un senior architect revisando este diseño"
   - "Explicame esto como si fuera mi primer día programando"

### Técnicas avanzadas

- **Chain of Thought (CoT)**: pedile que razone paso a paso antes de responder.
  - "Pensá en voz alta: ¿qué enfoques posibles hay para resolver este problema de caching? Evaluá pros y contras de cada uno y después recomendame uno."

- **System prompts vs user prompts**: los system prompts definen comportamiento BASE (personalidad, reglas, restricciones globales). Los user prompts son las instrucciones puntuales. En herramientas como OpenCode, el system prompt ya está escrito — vos solo escribís user prompts.

- **Iteración**: el primer intento casi nunca es perfecto. "Más específico", "más corto", "en español", "mostrame solo la función, no el archivo entero" — iterar es normal y esperado.

- **Archivos como contexto**: la forma más efectiva de trabajar con código es darle al modelo acceso a los archivos relevantes (como hace OpenCode con sus herramientas Read/Glob/Grep) en vez de copiar y pegar fragmentos.

---

## 3.3 Contexto y Ventanas de Contexto

*En criollo:* La ventana de contexto es la "memoria de corto plazo" del modelo. Es cuánto texto de la conversación puede "ver" al mismo tiempo para generar su respuesta. Si la conversación es más larga que la ventana, el modelo "olvida" lo que estaba al principio. Es como hablar con alguien que solo recuerda las últimas 200 páginas de la conversación — lo de antes, se perdió.

### Tamaños típicos (2025-2026)
| Ventana | Ejemplo de modelos | Equivalente aproximado |
|---------|-------------------|----------------------|
| 128K tokens | GPT-4o, Claude 3.5 | ~300 páginas de un libro |
| 200K tokens | Claude 4 | ~500 páginas |
| 1M+ tokens | Gemini 2.5 Pro | ~3,000 páginas (Guerra y Paz entera) |

### Por qué importa en desarrollo
- Una conversación larga con muchas idas y vueltas empieza a perder contexto del inicio.
- Las instrucciones importantes (reglas del sistema, tu stack, restricciones) DEBEN estar al principio o en el system prompt. Si están al final, el modelo puede "olvidarlas" antes.
- **Estrategia**: poné lo más importante PRIMERO en tu prompt. Restricciones, contexto del proyecto, stack. Después los detalles y ejemplos.

### Cómo se maneja en herramientas reales
- **Sliding window**: el modelo solo "ve" los últimos N tokens. Lo anterior se descarta.
- **Compacción (compaction)**: algunas herramientas resumen la conversación vieja para mantener el contexto esencial sin ocupar toda la ventana. Ejemplo: OpenCode + Engram generan resúmenes de sesión.
- **Memoria externa**: herramientas como Engram guardan decisiones y descubrimientos en una base de datos aparte para que sesiones FUTURAS puedan recuperarlos sin ocupar la ventana actual.

---

## 3.4 SDD — Spec-Driven Development

*En criollo:* SDD es una metodología para desarrollar software CON asistencia de IA de forma estructurada. En vez de tirarle "haceme un login" al chat y rezar, SDD te hace pasar por fases: **explorar → proponer → especificar → diseñar → planificar tareas → implementar → verificar → archivar**. Es como tener un arquitecto, un tech lead y un QA revisando todo ANTES de escribir una línea de código. El resultado: menos idas y vueltas, menos bugs, más coherencia.

### Fases del ciclo SDD

```
explore → propose → spec → design → tasks → apply → verify → archive
                      ↑                         │
                      └──────── design ─────────┘
```

| Fase | ¿Qué produce? | ¿Quién la ejecuta? |
|------|--------------|-------------------|
| **Explore** | Investigación, comparación de enfoques | Sub-agente `sdd-explore` |
| **Propose** | Documento de propuesta (PRD) con alcance, objetivos, no-goals | Sub-agente `sdd-propose` |
| **Spec** | Especificaciones detalladas: requisitos funcionales, escenarios | Sub-agente `sdd-spec` |
| **Design** | Arquitectura técnica, decisiones de stack, diagramas | Sub-agente `sdd-design` |
| **Tasks** | Lista de tareas atómicas, ordenadas, estimadas | Sub-agente `sdd-tasks` |
| **Apply** | Código implementado, tests, documentación | Sub-agente `sdd-apply` |
| **Verify** | Reporte de verificación: tests, coverage, conformidad con spec | Sub-agente `sdd-verify` |
| **Archive** | Cierre del cambio, lecciones aprendidas, specs actualizadas | Sub-agente `sdd-archive` |

### El orquestador
El orquestador (vos o un agente coordinador) no escribe código. Su trabajo es:
- Asegurarse de que cada fase se complete antes de pasar a la siguiente.
- Validar que los artefactos producidos sean coherentes entre sí.
- Decidir rutas (¿hace falta diseño formal o alcanza con spec + tasks?).
- Manejar el presupuesto de revisión y la estrategia de PRs.

### Conceptos del mundo SDD
- **Artefactos**: los documentos que produce cada fase (propuesta, spec, design, tasks, etc.). Se almacenan en un *artifact store*.
- **Artifact store**: dónde viven los artefactos. Puede ser en archivos (`openspec/`) o en memoria persistente (Engram) o ambos.
- **Dependency graph**: `propose → spec → tasks → apply → verify → archive`. No podés implementar sin spec, no podés verificar sin tasks.
- **Review budget**: límite de líneas cambiadas antes de que el orquestador te pida partir el cambio en PRs más chicos (protege al revisor humano del burnout).
- **Gatekeeper**: en modo automático, el orquestador valida cada fase antes de lanzar la siguiente. Si algo falla dos veces, se frena y pide intervención humana.

---

## 3.5 Skills — Conocimiento Modular y Reusable

*En criollo:* Una skill es como un manual de instrucciones empaquetado para la IA. No es código — es un archivo de texto con reglas, procedimientos, ejemplos, y buenas prácticas sobre un tema específico. La IA "carga" la skill cuando la necesita y sigue sus instrucciones al pie de la letra. Es la diferencia entre decirle "hacé un PR" (resultado impredecible) y "cargá la skill `branch-pr` y seguí sus reglas para crear el PR" (resultado consistente y profesional).

### ¿Por qué existen las skills?

- **Consistencia**: cada PR, cada review, cada commit sigue las mismas reglas sin importar qué modelo o sesión lo ejecute.
- **Reusabilidad**: una skill bien escrita se usa en cientos de proyectos sin modificarla.
- **Especialización**: encapsulan conocimiento experto que el modelo base no tiene (ej: cómo hacer commits atómicos con `work-unit-commits`).
- **Evolución**: las skills se mejoran, versionan, y auditan independientemente de los agentes que las usan.

### Anatomía de una skill

```
skills/
├── _shared/              ← referencias compartidas entre skills
│   └── SKILL.md
├── branch-pr/
│   └── SKILL.md          ← el archivo que contiene la skill
├── chained-pr/
│   └── SKILL.md
├── work-unit-commits/
│   └── SKILL.md
└── ...más skills...
```

Cada `SKILL.md` define:
1. **Trigger**: cuándo se activa (ej: "Trigger: creating, opening, or preparing PRs for review").
2. **Instrucciones**: el procedimiento paso a paso que el agente debe seguir.
3. **Reglas**: restricciones, verificaciones, formato de salida esperado.
4. **Ejemplos**: casos concretos de uso.

### Cómo se cargan

*En criollo:* Cuando lanzás un sub-agente, el orquestador mira el registro de skills y dice "para esta tarea, este agente va a necesitar las skills X, Y, Z". Se las pasa en el prompt de lanzamiento. El sub-agente abre cada `SKILL.md`, lo lee, y lo sigue. No las memoriza — las lee en cada ejecución, lo que garantiza que siempre use la versión más reciente.

*Técnicamente:* El orquestador cachea un índice de skills al inicio de la sesión (`skill-registry`). Para cada delegación, matchea skills por contexto de archivos (extensiones, paths) y contexto de tarea (PR, review, testing, etc.). Las skills matcheadas se inyectan como paths en el prompt del sub-agente, que las carga con `Read` antes de ejecutar su tarea. Si la skill no se encuentra, el orquestador usa un fallback.

### Catálogo de skills principales

| Skill | Trigger | ¿Qué hace? |
|-------|---------|------------|
| `branch-pr` | Crear PRs | Verifica issues existentes, prepara el PR con título, body, labels |
| `chained-pr` | PRs > 400 líneas | Divide cambios grandes en PRs encadenados para proteger el foco de revisión |
| `work-unit-commits` | Implementación | Planifica commits como unidades de trabajo revisables, tests + docs con el código |
| `cognitive-doc-design` | Documentación | Diseña docs que reducen carga cognitiva (guías, READMEs, RFCs) |
| `comment-writer` | Feedback en PRs/issues | Escribe comentarios cálidos, directos y colaborativos |
| `judgment-day` | Revisión adversarial | Dos jueces ciegos evalúan el código; discrepancias las resuelve un tercero |
| `go-testing` | Tests en Go | Patrones de testing Go: unit, coverage, golden files, Bubbletea |
| `issue-creation` | Crear issues | Crea y triagea issues de GitHub con evidencia del repositorio |
| `skill-creator` | Crear nuevas skills | Crea skills con frontmatter válido, triggers, y procedimientos |
| `skill-improver` | Auditar skills | Audita y mejora skills existentes: claridad, completitud, efectividad |
| `skill-registry` | Indexar skills | Mantiene el índice de skills disponibles por trigger y path |

### La skill que indexa a las demás

`skill-registry` es una meta-skill: mantiene un índice central de todas las skills instaladas. El orquestador la consulta al inicio de cada sesión para saber qué skills existen y cuándo activarlas. Cuando instalás, creás o modificás una skill, hay que regenerar el registry para que el orquestador la vea.

---

## 3.6 Sub-agentes — Delegación y Trabajo en Paralelo

*En criollo:* Un sub-agente es una instancia FRESCA de IA que lanzás para que haga UNA tarea específica y te devuelva el resultado. No hereda la conversación anterior — arranca de cero con solo lo que vos le pasás en el prompt. Esto es clave: cada sub-agente tiene una ventana de contexto LIMPIA, sin el ruido acumulado de toda la sesión. El orquestador no hace el trabajo — delega. Es como un director de orquesta: no toca instrumentos, pero sabe exactamente a quién llamar para cada parte de la sinfonía.

### ¿Por qué existen los sub-agentes?

- **Contexto limpio**: cada tarea compleja arranca con ventana de contexto fresca. Sin historial irrelevante.
- **Paralelismo**: podés lanzar varios sub-agentes al mismo tiempo si sus tareas no dependen entre sí.
- **Especialización**: hay sub-agentes para explorar código, para escribir specs, para implementar, para revisar. Cada uno con su especialidad y su modelo optimizado.
- **Aislamiento**: si un sub-agente falla o alucina, no contamina al resto. Se descarta y se reintenta.

### Tipos de sub-agentes

| Tipo | Rol | Ejemplos de uso |
|------|-----|-----------------|
| **Exploración** | Leer código, mapear, investigar | `explore`, `sdd-explore` |
| **Planificación** | Crear artefactos SDD | `sdd-propose`, `sdd-spec`, `sdd-design`, `sdd-tasks` |
| **Implementación** | Escribir código, tests | `sdd-apply`, `general` |
| **Verificación** | Validar, testear, revisar | `sdd-verify`, `review-risk`, `review-readability`, `review-reliability`, `review-resilience` |
| **Revisión adversarial** | Evaluar código ciegamente | `jd-judge-a`, `jd-judge-b`, `jd-fix-agent` |
| **Cierre** | Archivar, documentar | `sdd-archive` |
| **Onboarding** | Guiar al usuario | `sdd-onboard`, `sdd-init` |

### Cómo funciona la delegación

1. **El orquestador evalúa**: ¿esta tarea es chica (1-3 archivos, ya entendida)? → la hace inline. ¿Es grande o necesita contexto fresco? → delega.
2. **Resuelve skills**: mira el registry, matchea skills relevantes para la tarea.
3. **Arma el prompt**: incluye paths de skills, referencias a artefactos (topic keys en Engram o paths en OpenSpec), constraintes del proyecto.
4. **Lanza el sub-agente**: una llamada que crea un contexto fresco, lee skills, ejecuta, y devuelve un resultado estructurado.
5. **Valida el resultado**: el orquestador verifica que el resultado cumpla el contrato (status, artifacts, riesgos, next step) antes de aceptarlo.

### Reglas de delegación

- **Bounded read rule**: leer 1-3 archivos para decidir → inline. Leer 4+ archivos para entender → delegar un explorer.
- **Write rule**: un archivo mecánico ya entendido → inline. 2+ archivos no triviales → delegar un writer.
- **Context rule**: lectura que prepara un write o investigación amplia → delegar.
- **Per-action rule**: tests, builds, y verificaciones pueden usar agentes frescos sin cambiar la ruta de implementación.
- **Un solo writer a la vez**: no se lanzan writers paralelos sobre el mismo código sin worktrees aislados.

### Sub-agentes vs el orquestador

| Quién | ¿Hace trabajo? | ¿Guarda contexto? | Responsabilidad |
|-------|---------------|-------------------|-----------------|
| **Orquestador** | NO — coordina, sintetiza | Cachea skills, session preflight, decisiones | Fluir entre fases, validar resultados |
| **Sub-agente** | SÍ — ejecuta UNA tarea | NO — contexto fresco cada vez | Producir un resultado con contrato cumplido |

El orquestador es el único que habla con el usuario. Los sub-agentes son invisibles — trabajan, devuelven, desaparecen.

### Modelos por sub-agente

No todos los sub-agentes usan el mismo modelo. Tareas simples van a modelos rápidos y baratos. Tareas de arquitectura van a modelos potentes:

| Tarea | Modelo típico | Razón |
|-------|--------------|-------|
| Explorar código | Flash / rápido | Mucha lectura, poca generación |
| Arquitectura / diseño | Pro / pesado | Razonamiento profundo, tradeoffs |
| Escribir specs | Balanceado | Estructurado pero no creativo |
| Implementar | Balanceado | Código preciso, consistente |
| Revisar | Balanceado | Evaluación, no generación |
| Archivar | Flash / rápido | Mecánico, copiar y cerrar |

---

## 3.7 Gentle AI — Ecosistema

Gentle AI es un framework que integra agentes de IA con flujos de trabajo de desarrollo profesional. Toma todos los conceptos anteriores — SDD, skills, sub-agentes, memoria, revisión — y los orquesta en un sistema cohesivo que corre sobre OpenCode.

### Componentes clave

**Agentes**
- **Orquestador**: coordina, no ejecuta. Decide qué agente lanzar para cada fase.
- **Agentes de fase SDD**: `sdd-explore`, `sdd-propose`, `sdd-spec`, `sdd-design`, `sdd-tasks`, `sdd-apply`, `sdd-verify`, `sdd-archive`.
- **Agentes de revisión**: `review-risk` (R1), `review-readability` (R2), `review-reliability` (R3), `review-resilience` (R4).
- **Agentes adversariales**: `jd-judge-a`, `jd-judge-b` (revisión ciega), `jd-fix-agent` (corrección).
- **Agentes de soporte**: `general` (tareas generales), `explore` (exploración de código).

**Skills** — ver sección 3.5 para el detalle completo.

**Sub-agentes** — ver sección 3.6 para el detalle completo.

**Protocolos**
- **Engram Protocol**: memoria persistente entre sesiones. Define cuándo y cómo guardar, buscar, y juzgar memorias.
- **Review Integration Protocol**: ciclo de revisión de código (`start → capture → validate → terminal`) con presupuesto de líneas y corrección acotada.
- **SDD Dispatcher Protocol**: enrutamiento de fases SDD con guardas de dependencia. No se implementa sin spec, no se verifica sin tasks.
- **Lossless Blocking Prompts**: menús y decisiones que no se pueden resumir sin perder opciones. Garantiza que el usuario siempre vea todas las alternativas.

**Memoria — Engram** — ver sección 3.8 para el detalle completo.

**Runtime — OpenCode** — ver sección 3.9 para el detalle completo.

### Flujo de trabajo típico con Gentle AI

```
Vos decís: "Quiero agregar autenticación con JWT al proyecto"

Orquestador:
  1. Session preflight → ¿modo interactive/auto? ¿artifact store? ¿PR strategy?
  2. ¿Ya existe init para este proyecto? → si no, lanza sdd-init
  3. ¿Exploración necesaria? → lanza sdd-explore
  4. Propuesta lista → te la muestra, pedís ajustes
  5. Spec escrita → validada contra proposal
  6. Design técnico → validado contra spec
  7. Tasks planificadas → forecast de líneas, ¿hace falta chained PR?
  8. sdd-apply → implementa tasks en batches
  9. sdd-verify → testea, compara contra spec
  10. Review → 4R o Judgment Day según riesgo
  11. sdd-archive → cierra el cambio, guarda aprendizajes
```

---

## 3.8 Engram — Memoria Persistente

*En criollo:* Cada vez que cerrás una conversación con una IA, "olvida" todo. La próxima sesión empieza de cero. Engram es una base de datos externa donde la IA puede guardar y recuperar decisiones, descubrimientos, bugs, y contexto entre sesiones. Es como darle a la IA un cuaderno donde anota todo lo importante, y que puede consultar cuando vuelve a hablar con vos.

### Cómo funciona
- **`mem_save`**: guarda un dato importante (decisión de arquitectura, bug fix, patrón descubierto). Se llama PROACTIVAMENTE, no espera a que el usuario lo pida.
- **`mem_search`**: busca en todas las sesiones anteriores por palabras clave.
- **`mem_context`**: recupera el contexto de sesiones recientes.
- **`mem_session_summary`**: al final de cada sesión, genera un resumen estructurado para la próxima.

### Conceptos
- **Topic key**: un identificador estable para temas que evolucionan (ej: `architecture/auth-model`). Misma key = se actualiza en vez de duplicar.
- **Scope**: `project` (vinculado al proyecto) o `personal` (tuyo, cross-proyecto).
- **Type**: `decision`, `architecture`, `bugfix`, `pattern`, `config`, `discovery`.
- **Conflict resolution**: cuando una memoria nueva potencialmente contradice una existente, Engram pide un juicio humano o automático.

### Por qué importa
Sin Engram, cada sesión con IA empieza de cero — tenés que re-explicar tu stack, tus decisiones, tu contexto. Con Engram, la IA "recuerda" entre sesiones y puede seguir trabajando donde dejaron. Es la diferencia entre un asistente con amnesia y un compañero de equipo.

---

## 3.9 OpenCode — Entorno de Desarrollo

OpenCode es el runtime donde corren los agentes de Gentle AI. Provee:

- **Herramientas**: Read, Write, Edit, Bash, Glob, Grep, Task (sub-agentes), Question (UI de decisiones), WebFetch, CodeGraph.
- **MCP (Model Context Protocol)**: integración con servidores externos como Engram, Context7 (documentación actualizada), CodeGraph (grafo de conocimiento del código).
- **Permisos**: control granular sobre qué puede hacer cada agente (leer archivos, ejecutar comandos, escribir, acceder a red).
- **Modelos**: soporte para múltiples proveedores (OpenAI, Anthropic, Google, DeepSeek, Qwen, etc.) con asignación por agente (un modelo rápido para tareas simples, uno potente para arquitectura).

### Herramientas que usan los agentes
| Herramienta | Uso |
|-------------|-----|
| `Read` | Leer archivos del proyecto |
| `Write` / `Edit` | Escribir o modificar archivos |
| `Bash` | Ejecutar comandos (git, npm, tests, builds) |
| `Glob` | Buscar archivos por patrón |
| `Grep` | Buscar contenido en archivos |
| `Task` | Lanzar sub-agentes para trabajo paralelo |
| `Question` | Preguntar al usuario (menús de opciones) |
| `WebFetch` | Obtener documentación actualizada de la web |
| `CodeGraph` | Explorar el grafo de símbolos del código |

---

## 3.10 Code Review Asistido por IA

La IA no reemplaza al revisor humano, pero acelera MASSIVAMENTE el proceso:

- **Revisión de riesgos (R1)**: seguridad, exposición de datos, dependencias vulnerables.
- **Revisión de legibilidad (R2)**: naming, complejidad, intención del código, mantenibilidad.
- **Revisión de confiabilidad (R3)**: tests, cobertura, edge cases, contratos.
- **Revisión de resiliencia (R4)**: fallbacks, retry, observabilidad, carga, rollback.

**Judgment Day**: protocolo de revisión adversarial donde dos jueces ciegos (modelos distintos, sin ver la review del otro) evalúan el mismo código. Si ambos encuentran un bloqueante, se corrige. Si discrepan, un tercer juez decide. Esto elimina sesgos de modelo.

### Lo que la IA hace bien en review
- Detectar patrones de bugs conocidos.
- Señalar código no testeado.
- Encontrar problemas de seguridad obvios (inyección, secretos expuestos).
- Verificar adherence al estilo del proyecto.

### Lo que NO hace bien
- Entender intención de negocio ("esto está mal para el producto").
- Evaluar tradeoffs de arquitectura a largo plazo.
- Capturar matices culturales o de equipo en el código.

---

## 3.11 Ética, Sesgos y Limitaciones

### Lo que tenés que saber
- **Alucinaciones**: los LLMs inventan hechos, APIs, funciones y librerías que NO existen. Suenan convincentes, pero son ficción. SIEMPRE verificá código y claims técnicos.
- **Sesgos**: los modelos reflejan los sesgos de sus datos de entrenamiento (mayoritariamente en inglés, de internet, con sobrerrepresentación de ciertas perspectivas).
- **Privacidad**: NUNCA subas secretos, tokens, contraseñas, o datos de usuarios reales a un LLM público. Usá entornos locales o enterprise cuando trabajes con datos sensibles.
- **Dependencia**: usar IA no te exime de ENTENDER lo que hace. Si no podés explicar el código que la IA generó, no deberías mergearlo.
- **Responsabilidad**: el código que commiteás es TU responsabilidad, no del modelo. Si la IA generó un bug en producción, el responsable es quien hizo el commit.

### Buenas prácticas
- ✅ Usá IA para acelerar, no para reemplazar pensamiento crítico.
- ✅ Revisá SIEMPRE el código generado antes de commitear.
- ✅ Preguntale a la IA "¿por qué hiciste esto así?" — obligala a justificar sus decisiones.
- ✅ Tratá a la IA como un junior muy rápido pero que a veces miente.
- ❌ No confíes en nombres de APIs, librerías o versiones sin verificarlos.

---

## 3.12 Ecosistema de Herramientas

El panorama va mucho más allá de ChatGPT:

### Asistentes de código
| Herramienta | Enfoque |
|-------------|---------|
| **GitHub Copilot** | Autocompletado inline en el editor. El más maduro, integración nativa con GitHub. |
| **Cursor** | Editor basado en VS Code con IA nativa. Agentes, multi-file edits, @-mentions para contexto. |
| **Cody (Sourcegraph)** | Búsqueda y comprensión de código + generación contextual. Conoce TODO el repositorio. |
| **Codeium / Windsurf** | Autocompletado rápido, gratis para individuos, multi-IDE. |
| **Amazon Q Developer** | Enfocado en AWS, pero tiene capacidades generales de código. |
| **Tabnine** | Autocompletado con modelo local o cloud, privacidad-first. |

### Agentes y flujos de trabajo
| Herramienta | Enfoque |
|-------------|---------|
| **OpenCode** | Runtime de agentes con SDD, memoria, skills, revisión adversarial. |
| **Gentle AI** | Framework de agentes sobre OpenCode — lo que usamos en este roadmap. |
| **Aider** | Agente de código open-source basado en terminal. Edita archivos directamente. |
| **SWE-agent / Devin / OpenHands** | Agentes autónomos que resuelven issues completos. Más experimentales. |
| **Claude Code** | Agente de Anthropic integrado en la terminal. |

### Documentación e investigación
| Herramienta | Enfoque |
|-------------|---------|
| **Context7** | Documentación actualizada de librerías, consultable desde el agente. |
| **Perplexity** | Búsqueda web con IA, cita fuentes, ideal para investigar tecnologías nuevas. |
| **MDN + AI** | MDN integrado con herramientas como Context7 para referencia en tiempo real. |

---

> **Check de comprensión**:
> 1. ¿Podés explicar qué es un token y por qué importa el tamaño de la ventana de contexto?
> 2. ¿Cuáles son las fases del ciclo SDD y qué produce cada una?
> 3. ¿Qué es una skill y cómo se diferencia de un sub-agente?
> 4. ¿Por qué los sub-agentes arrancan con contexto fresco en vez de heredar la conversación?
> 5. ¿Cómo funciona Engram y para qué sirve en un flujo de desarrollo con IA?
> 6. ¿Qué NO deberías hacer con código generado por IA antes de mergearlo?
