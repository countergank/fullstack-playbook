# 3. IA & Desarrollo Asistido

> Objetivo: entender cómo funciona la IA que usás para desarrollar, cómo sacarle el máximo provecho con buenos prompts y contexto, y conocer el ecosistema de herramientas que ya son parte del stack profesional.

---

## 3.1 LLMs — Cómo funcionan los Modelos de Lenguaje

*En criollo:* Un LLM (Large Language Model) es un programa que predice la palabra más probable que sigue, dado todo el texto anterior. Entrenado sobre cantidades masivas de texto de internet, libros, y código, "aprende" patrones de lenguaje, razonamiento y programación. No "piensa" como un humano, no "sabe" cosas en el sentido tradicional — es una máquina de completar texto extremadamente sofisticada. El arte de usarlo bien está en darle el contexto justo para que la predicción sea la correcta.

*Técnicamente:* Los LLMs son redes neuronales transformer con arquitecturas decoder-only (GPT, Claude) o encoder-decoder (T5). Cada capa aplica atención multi-cabeza sobre embeddings de tokens, seguida de feed-forward networks con activaciones como SwiGLU. La inferencia usa sampling strategies: greedy (temperatura 0), nucleus/top-p sampling, o temperature scaling sobre la distribución softmax. El entrenamiento combina pre-entrenamiento auto-regresivo (next-token prediction con cross-entropy loss) y fine-tuning con RLHF (Reinforcement Learning from Human Feedback) o DPO (Direct Preference Optimization).
→ Ver [Transformer Architecture — Jay Alammar](https://jalammar.github.io/illustrated-transformer/)
→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.8-algoritmos) para entender complejidad computacional del entrenamiento

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

> **Check de comprensión**
> 1. ¿Qué significa que un LLM es una "máquina de completar texto" y no un sistema que "piensa"?
>    - R: El modelo no tiene comprensión ni intención — solo calcula probabilidades del siguiente token basándose en patrones aprendidos durante el entrenamiento.
> 2. ¿Cuál es la diferencia entre entrenamiento e inferencia?
>    - R: El entrenamiento es el proceso de enseñarle al modelo con terabytes de texto (costoso y lento); la inferencia es usar el modelo ya entrenado para predecir tokens (rápido y barato).
> 3. ¿Cómo afecta la temperatura a las respuestas del modelo?
>    - R: Temperatura baja (0-0.3) produce respuestas deterministas y precisas; temperatura alta (0.8-1.0) genera respuestas más creativas y variadas pero menos predecibles.
> 4. ¿Qué es un token y por qué no equivale a una palabra?
>    - R: Un token es un fragmento de texto (puede ser parte de una palabra, una palabra completa, o un carácter especial). El modelo procesa tokens, no palabras, y el conteo varía según el idioma y el tokenizer.
> 5. ¿Por qué la ventana de contexto limita lo que el modelo puede hacer en una sola request?
>    - R: La ventana de contexto define cuántos tokens (entrada + salida) puede procesar simultáneamente. Si el input excede ese límite, el modelo no puede "ver" todo el texto y pierde información.

---

## 3.2 Prompt Engineering

*En criollo:* Prompt engineering NO es "escribir lindo". Es darle al modelo el CONTEXTO preciso para que produzca la respuesta correcta. El 80% de los malos resultados vienen de prompts ambiguos, incompletos o sin restricciones claras.

*Técnicamente:* Un prompt se compone de tokens que el modelo procesa como una secuencia. La calidad del output depende de cómo la distribución de probabilidad del siguiente token se condicione sobre el input. Técnicas como few-shot prompting inyectan ejemplos que modifican la distribución hacia patrones deseados. Chain of Thought activa el razonamiento paso a paso porque el modelo genera tokens intermedios que condicionan cada paso siguiente. System prompts se prependean al prompt del usuario con un rol special (a menudo tokens de control como `<|system|>`) que el modelo fue entrenado para obedecer con mayor prioridad.
→ Ver [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
→ Ver [Anthropic Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags)
→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.1-pensamiento-computacional) para la base de descomposición de problemas

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

> **Check de comprensión**
> 1. ¿Por qué ser específico es más importante que ser educado en un prompt?
>    - R: El modelo no responde a cortesía — responde a claridad. Un prompt vago genera respuestas vagas; uno específico con contexto, restricciones y ejemplos produce resultados precisos.
> 2. ¿Qué es few-shot prompting y por qué funciona mejor que instrucciones abstractas?
>    - R: Few-shot prompting es mostrarle al modelo 1 o más ejemplos del formato esperado. Funciona porque los LLMs son excelentes imitando patrones concretos, no siguiendo descripciones abstractas.
> 3. ¿Cuál es la diferencia entre system prompt y user prompt?
>    - R: El system prompt define comportamiento base (personalidad, reglas globales, restricciones permanentes); el user prompt es la instrucción puntual de la conversación. El system prompt tiene mayor prioridad.
> 4. ¿Por qué conviene dividir problemas grandes en prompts separados?
>    - R: Cada prompt consume ventana de contexto y mezcla objetivos reduce la precisión. Una conversación por feature mantiene el contexto limpio y permite iterar sobre cada parte independientemente.
> 5. ¿Qué técnica usarías para que el modelo evalúe múltiples enfoques antes de recomendar uno?
>    - R: Chain of Thought (CoT): pedirle que razone paso a paso, enumerando opciones, evaluando pros y contras, y luego recomendando. Esto fuerza al modelo a generar tokens de razonamiento intermedios.

---

## 3.3 Contexto y Ventanas de Contexto

*En criollo:* La ventana de contexto es la "memoria de corto plazo" del modelo. Es cuánto texto de la conversación puede "ver" al mismo tiempo para generar su respuesta. Si la conversación es más larga que la ventana, el modelo "olvida" lo que estaba al principio. Es como hablar con alguien que solo recuerda las últimas 200 páginas de la conversación — lo de antes, se perdió.

*Técnicamente:* La ventana de contexto (context window) es el máximo de tokens que el transformer puede atender en una sola forward pass. Se mide en tokens de entrada + salida. Architecturas como RoPE (Rotary Position Embedding) y ALiBi permiten extender más allá del límite de entrenamiento, pero la calidad degrada en posiciones lejanas (lost-in-the-middle phenomenon). La compacción usa summarization models para condensar historia antigua; la memoria externa (como Engram) persiste datos fuera de la ventana en una base de datos semántica consultable.
→ Ver [Anthropic — Long Context Windows](https://docs.anthropic.com/en/docs/build-with-claude/context-windows)
→ Ver [Google Gemini — Context Caching](https://ai.google.dev/gemini-api/docs/caching)
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.3-dns) para la analogía de caché vs memoria persistente

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

> **Check de comprensión**
> 1. ¿Qué pasa cuando una conversación excede la ventana de contexto del modelo?
>    - R: El modelo pierde acceso a los tokens más antiguos. Dependiendo de la estrategia, puede descartarlos (sliding window), resumirlos (compaction), o necesitar que se los re-envíen manualmente.
> 2. ¿Cuál es la diferencia entre sliding window y compaction?
>    - R: Sliding window descarta los tokens más viejos sin guardar nada; compaction genera un resumen del contenido antiguo para preservar la información esencial en menos tokens.
> 3. ¿Por qué las instrucciones importantes deben ir al principio del prompt?
>    - R: En ventanas largas, los modelos tienden a prestar más atención al inicio y al final del contexto (efecto "primacy/recency"). Además, si hay compaction, lo del principio tiene más chance de sobrevivir en el resumen.
> 4. ¿Cómo resuelve Engram el problema de la ventana de contexto limitada?
>    - R: Engram persiste información fuera de la ventana del modelo en una base de datos. El agente consulta Engram cuando necesita contexto de sesiones anteriores sin ocupar tokens de la conversación actual.
> 5. ¿Cuántas páginas de texto equivalen aproximadamente a 128K tokens?
>    - R: Aproximadamente 300 páginas de un libro. Esto da una idea de cuánto puede "recordar" un modelo como GPT-4o o Claude 3.5 en una sola conversación.

---

## 3.4 SDD — Spec-Driven Development

*En criollo:* SDD es una metodología para desarrollar software CON asistencia de IA de forma estructurada. En vez de tirarle "haceme un login" al chat y rezar, SDD te hace pasar por fases: **explorar → proponer → especificar → diseñar → planificar tareas → implementar → verificar → archivar**. Es como tener un arquitecto, un tech lead y un QA revisando todo ANTES de escribir una línea de código. El resultado: menos idas y vueltas, menos bugs, más coherencia.

*Técnicamente:* SDD implementa un pipeline de transformación de artefactos donde cada fase produce un documento estructurado que sirve como input de la siguiente. El dependency graph es DAG (Directed Acyclic Graph): `propose → spec → design → tasks → apply → verify → archive`. Cada transición tiene guardas de validación (no se avanza si el artefacto no cumple el schema). El artifact store puede ser filesystem-based (OpenSpec con archivos YAML/Markdown) o database-based (Engram con observaciones semánticas), o híbrido. El review budget protege al revisor humano limitando líneas cambiadas por PR.
→ Ver [Spec-Driven Development — Concepto original](https://www.anthropic.com/engineering/spec-driven-development)
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.1-arquitectura) para ver cómo SDD se aplica a un proyecto real

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

> **Check de comprensión**
> 1. ¿Cuáles son las 8 fases del ciclo SDD en orden?
>    - R: Explore → Propose → Spec → Design → Tasks → Apply → Verify → Archive. Cada fase produce un artefacto que alimenta la siguiente.
> 2. ¿Qué produce la fase de Spec y en qué se diferencia de Design?
>    - R: Spec produce requisitos funcionales y escenarios (el QUÉ debe hacer el sistema); Design produce arquitectura técnica y decisiones de stack (el CÓMO se implementa).
> 3. ¿Qué es el artifact store y qué opciones existen?
>    - R: Es dónde se guardan los artefactos de cada fase. Puede ser en archivos del filesystem (OpenSpec), en memoria persistente (Engram), o ambos (híbrido).
> 4. ¿Por qué el orquestador NO escribe código?
>    - R: El orquestador coordina y valida — su responsabilidad es asegurar coherencia entre artefactos, manejar el presupuesto de revisión, y decidir cuándo delegar a sub-agentes especializados.
> 5. ¿Qué es el review budget y por qué existe?
>    - R: Es un límite de líneas cambiadas por PR. Existe porque revisar cambios grandes causa burnout al revisor humano; si se excede, el orquestador pide dividir el cambio en PRs más chicos.

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

→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.6-modularidad) para entender por qué la modularidad aplica tanto a código como a conocimiento
→ Ver [Tópico 7: Frontend Core](../concepts/07-frontend-core.md#7.1-html) para ver un ejemplo de cómo las skills de frontend se estructuran

> **Check de comprensión**
> 1. ¿Qué es una skill y qué NO es?
>    - R: Una skill es un archivo de texto con reglas, procedimientos y buenas prácticas sobre un tema. NO es código ejecutable ni un plugin — es conocimiento estructurado que la IA lee y sigue.
> 2. ¿Por qué las skills garantizan consistencia entre sesiones y modelos?
>    - R: Porque se leen en cada ejecución desde el filesystem, no se memorizan. Así siempre se usa la versión más reciente, sin importar qué modelo o sesión las ejecuta.
> 3. ¿Qué define cada SKILL.md?
>    - R: Cuatro componentes: Trigger (cuándo se activa), Instrucciones (procedimiento paso a paso), Reglas (restricciones y formato), y Ejemplos (casos concretos de uso).
> 4. ¿Cómo sabe el orquestador qué skills cargar para una tarea?
>    - R: Consulta el skill-registry (índice central) y matchea por contexto de archivos (extensiones, paths) y contexto de tarea (PR, review, testing, etc.).
> 5. ¿Qué pasa si una skill no se encuentra al momento de ejecutar?
>    - R: El orquestador usa un fallback — continúa sin esa skill pero puede producir resultados menos consistentes o completos.

---

## 3.6 Sub-agentes — Delegación y Trabajo en Paralelo

*En criollo:* Un sub-agente es una instancia FRESCA de IA que lanzás para que haga UNA tarea específica y te devuelva el resultado. No hereda la conversación anterior — arranca de cero con solo lo que vos le pasás en el prompt. Esto es clave: cada sub-agente tiene una ventana de contexto LIMPIA, sin el ruido acumulado de toda la sesión. El orquestador no hace el trabajo — delega. Es como un director de orquesta: no toca instrumentos, pero sabe exactamente a quién llamar para cada parte de la sinfonía.

*Técnicamente:* Un sub-agente es una invocación de la herramienta `Task` (o equivalente) que crea un contexto de ejecución aislado. Recibe como input: paths de skills, referencias a artefactos (topic keys de Engram o paths de OpenSpec), y constraints del proyecto. Su output es un envelope estructurado con status, artifacts modificados, y next step. El aislamiento garantiza que un fallo o alucinación no contamine la sesión principal. La delegación sigue reglas heurísticas: bounded read (1-3 archivos = inline, 4+ = delegar), write rule (2+ archivos no triviales = delegar), y per-action rule (tests/builds = agente fresco).
→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.6-modularidad) para la analogía con funciones y responsabilidades únicas

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

> **Check de comprensión**
> 1. ¿Por qué los sub-agentes arrancan con contexto fresco en vez de heredar la conversación?
>    - R: Porque la ventana de contexto es limitada y la conversación acumulada tiene ruido irrelevante. Un contexto fresco garantiza que el sub-agente se concentre solo en su tarea específica.
> 2. ¿Qué regla determina cuándo delegar vs hacer inline?
>    - R: Bounded read rule: si necesitás leer 1-3 archivos para decidir, hacé inline. Si necesitás 4+ archivos para entender, delegá un explorer. Para escritura: 2+ archivos no triviales = delegar.
> 3. ¿Qué pasa si un sub-agente falla o alucina?
>    - R: Como está aislado, no contamina al resto. Se descarta su resultado y se reintenta (posiblemente con otro modelo o más contexto).
> 4. ¿Por qué no se lanzan writers paralelos sobre el mismo código?
>    - R: Porque crearían conflictos de escritura. Solo se permite paralelismo con worktrees aislados o tareas que no tocan los mismos archivos.
> 5. ¿Qué diferencia hay entre el orquestador y un sub-agente?
>    - R: El orquestador coordina, valida y habla con el usuario — NO ejecuta. El sub-agente ejecuta UNA tarea específica con contexto fresco y devuelve un resultado estructurado.

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

*En criollo:* Gentle AI es un framework que integra agentes de IA con flujos de trabajo de desarrollo profesional. Toma todos los conceptos anteriores — SDD, skills, sub-agentes, memoria, revisión — y los orquesta en un sistema cohesivo que corre sobre OpenCode.

*Técnicamente:* Gentle AI es una CLI (`gentle-ai`) que provee comandos para interactuar con el ecosistema SDD desde terminal (`sdd-status`, `sdd-continue`, `review status`, `engram search`). Se integra con OpenCode como capa de orquestación sobre el runtime. Los protocolos (Engram, Review Integration, SDD Dispatcher, Lossless Blocking Prompts) definen contratos formales entre componentes. La TUI de Gentle AI gestiona perfiles SDD, instalación de componentes (Engram, plugins), y monitoreo de estado.
→ **Nota de excepción**: Gentle AI no tiene documentación pública estable. La fuente oficial es el repositorio del proyecto: [gentleman-programming/gentle-ai](https://github.com/gentleman-programming/gentle-ai).
→ Ver [Tópico 3: SDD](#3.4-sdd--spec-driven-development) para las fases que Gentle AI orquesta

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

> **Check de comprensión**
> 1. ¿Qué rol cumple Gentle AI en el ecosistema de herramientas?
>    - R: Gentle AI es el framework que orquesta todos los componentes (SDD, skills, sub-agentes, memoria, revisión) sobre el runtime de OpenCode. Es la capa de coordinación.
> 2. ¿Cuáles son los 4 protocolos principales de Gentle AI?
>    - R: Engram Protocol (memoria persistente), Review Integration Protocol (ciclo de review 4R), SDD Dispatcher Protocol (enrutamiento de fases con guardas), y Lossless Blocking Prompts (decisiones sin pérdida de opciones).
> 3. ¿Qué diferencia hay entre los agentes de fase SDD y los agentes de revisión?
>    - R: Los de fase SDD ejecutan el pipeline de desarrollo (explorar, proponer, spec, design, etc.); los de revisión evalúan código ya escrito desde distintas perspectivas (riesgo, legibilidad, confiabilidad, resiliencia).
> 4. ¿Qué es Judgment Day y cuándo se usa?
>    - R: Es un protocolo de revisión adversarial donde dos jueces ciegos (modelos distintos) evalúan el mismo código independientemente. Se usa para cambios de alto riesgo o cuando hay discrepancias en la review normal.
> 5. ¿Por qué Gentle AI no tiene documentación pública estable?
>    - R: Es un proyecto en desarrollo activo sin docs públicas formales. La fuente oficial es su repositorio de GitHub y los posts de blog de los autores. Esta es una excepción documentada en este playbook.

---

## 3.8 Engram — Memoria Persistente

*En criollo:* Cada vez que cerrás una conversación con una IA, "olvida" todo. La próxima sesión empieza de cero. Engram es una base de datos externa donde la IA puede guardar y recuperar decisiones, descubrimientos, bugs, y contexto entre sesiones. Es como darle a la IA un cuaderno donde anota todo lo importante, y que puede consultar cuando vuelve a hablar con vos.

*Técnicamente:* Engram es un servidor MCP (Model Context Protocol) que expone operaciones CRUD semánticas sobre observaciones. Usa FTS5 (Full-Text Search en SQLite) para búsquedas por texto completo y embeddings semánticos para matching contextual. Cada observación tiene: id, title, type (decision/architecture/bugfix/pattern/config/discovery), scope (project/personal), content, topic_key (para upserts), y timestamps con decay policies. El conflict resolution usa JudgeBySemantic: cuando una nueva observación potencialmente contradice una existente, se genera un pending judgment que requiere resolución (automática o humana).
→ **Nota de excepción**: Engram no tiene documentación pública estable fuera de su repositorio. Fuente oficial: [gentleman-programming/engram](https://github.com/gentleman-programming/engram).
→ Ver [Tópico 2: Fundamentos de Programación](../concepts/02-fundamentos-programacion.md#2.7-bases-de-datos) para entender cómo SQLite + FTS5 habilita la búsqueda semántica

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

> **Check de comprensión**
> 1. ¿Qué hace `mem_save` y cuándo debe llamarse?
>    - R: `mem_save` guarda una observación (decisión, bug, descubrimiento) en la base de datos persistente. Debe llamarse PROACTIVAMENTE después de cualquier decisión o hallazgo importante, no cuando el usuario lo pida.
> 2. ¿Qué es un topic_key y por qué es útil?
>    - R: Es un identificador estable para temas que evolucionan (ej: `architecture/auth-model`). Usar la misma key permite actualizar (upsert) en vez de duplicar observaciones sobre el mismo tema.
> 3. ¿Qué pasa cuando una memoria nueva contradice una existente?
>    - R: Engram genera un pending conflict judgment. Puede resolverse automáticamente (si la confianza es alta y la relación es compatible/scoped) o pedir intervención humana (si es supersedes/conflicts_with).
> 4. ¿Cuál es la diferencia entre scope `project` y `personal`?
>    - R: `project` vincula la observación a un proyecto específico; `personal` es cross-proyecto, para preferencias o patrones que aplican en cualquier contexto.
> 5. ¿Por qué `mem_session_summary` es obligatorio antes de cerrar una sesión?
>    - R: Porque genera un resumen estructurado (Goal, Instructions, Discoveries, Accomplished, Next Steps, Relevant Files) que la próxima sesión usa para recuperar contexto sin empezar de cero.

---

## 3.9 OpenCode — Entorno de Desarrollo

*En criollo:* OpenCode es el runtime donde corren los agentes de Gentle AI. Provee herramientas (leer, escribir, ejecutar comandos), integración con modelos de IA (OpenAI, Anthropic, Google, etc.), y control de permisos para que cada agente solo haga lo que debe. Es como el sistema operativo donde viven los agentes.

*Técnicamente:* OpenCode es una TUI (Terminal UI) construida en Go que actúa como host de agentes LLM. Expone herramientas como MCP tools (Read, Write, Edit, Bash, Glob, Grep, Task, Question, WebFetch, CodeGraph) y soporta múltiples proveedores vía configuración JSON. El sistema de permisos granular controla qué herramientas puede invocar cada agente. Los perfiles (profiles) permiten switchear entre configuraciones de modelos por fase SDD. El archivo de configuración es `~/.config/opencode/opencode.json`.
→ **Nota de excepción**: OpenCode no tiene documentación pública estable fuera de su sitio y repositorio. Fuente oficial: [opencode.ai](https://opencode.ai) y [npm @opencode-ai/cli](https://www.npmjs.com/package/@opencode-ai/cli).
→ Ver [Tópico 3: Skills](#3.5-skills--conocimiento-modular-y-reusable) para entender cómo las skills se cargan dentro de OpenCode

### Qué provee OpenCode

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

> **Check de comprensión**
> 1. ¿Qué es OpenCode y qué rol cumple en el ecosistema Gentle AI?
>    - R: OpenCode es el runtime (TUI en Go) donde corren los agentes. Provee herramientas, integración con modelos, MCP para servidores externos, y control de permisos. Es la base sobre la que Gentle AI orquesta el flujo SDD.
> 2. ¿Qué es MCP (Model Context Protocol) y para qué sirve?
>    - R: MCP es un protocolo estándar que permite a los agentes integrarse con servidores externos como Engram (memoria), Context7 (documentación), y CodeGraph (grafo de código), sin que el modelo base necesite conocer esos servicios.
> 3. ¿Por qué OpenCode soporta múltiples proveedores de modelos?
>    - R: Porque diferentes tareas requieren diferentes capacidades: modelos rápidos y baratos para exploración, modelos potentes para arquitectura. Los perfiles permiten asignar modelos por fase SDD.
> 4. ¿Cómo funciona el sistema de permisos de OpenCode?
>    - R: Controla granularmente qué herramientas puede invocar cada agente (leer archivos, ejecutar comandos, escribir, acceder a red). El usuario puede permitir o denegar desde la UI o configurar en `opencode.json`.
> 5. ¿Cuál es la herramienta que permite lanzar sub-agentes en paralelo?
>    - R: La herramienta `Task` crea un contexto de ejecución aislado y lanza un sub-agente con skills y artefactos específicos para una tarea concreta.

---

## 3.10 Code Review Asistido por IA

*En criollo:* La IA no reemplaza al revisor humano, pero acelera MASSIVAMENTE el proceso. En vez de que una persona lea cada línea, la IA pre-clasifica los problemas por tipo (seguridad, legibilidad, confiabilidad, resiliencia) y el revisor se concentra en lo que realmente importa: intención de negocio y tradeoffs de arquitectura.

*Técnicamente:* El ciclo de review sigue un protocolo `start → capture → validate → terminal` con presupuesto de líneas (400 changed lines como threshold). 4R ejecuta 4 reviews secuenciales (R1-Risk, R2-Readability, R3-Reliability, R4-Resilience). Judgment Day usa dos modelos independientes como jueges ciegos; si hay discrepancia, un tercer modelo desempata. Cada review produce un envelope estructurado con hallazgos, severidad, y fix suggestions. El review budget protege al revisor humano del burnout por diffs grandes.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.3-manejo-de-errores) para ejemplos de lo que R3 (confiabilidad) busca en código de backend

### Las 4 revisiones (4R)

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

> **Check de comprensión**
> 1. ¿Qué evalúa cada una de las 4 revisiones (4R)?
>    - R: R1 evalúa riesgos de seguridad y dependencias vulnerables; R2 evalúa legibilidad, naming y mantenibilidad; R3 evalúa confiabilidad, tests y edge cases; R4 evalúa resiliencia, fallbacks y observabilidad.
> 2. ¿Cómo funciona Judgment Day y por qué elimina sesgos?
>    - R: Dos modelos distintos evalúan el mismo código independientemente (ciegos entre sí). Si ambos encuentran un bloqueante, se corrige. Si discrepan, un tercer juez decide. Al usar modelos distintos, se eliminan sesgos específicos de un modelo.
> 3. ¿Qué cosas la IA hace BIEN en code review?
>    - R: Detectar patrones de bugs conocidos, señalar código no testeado, encontrar problemas de seguridad obvios (inyección, secretos), y verificar adherence al estilo del proyecto.
> 4. ¿Qué cosas la IA NO hace bien en code review?
>    - R: Entender intención de negocio, evaluar tradeoffs de arquitectura a largo plazo, y capturar matices culturales o de equipo.
> 5. ¿Qué es el review budget y por qué importa?
>    - R: Es un límite de ~400 líneas cambiadas por PR. Importa porque revisar diffs grandes causa burnout al revisor humano; si se excede, el orquestador pide dividir el cambio en PRs encadenados.

---

## 3.11 Ética, Sesgos y Limitaciones

*En criollo:* Los LLMs son herramientas poderosas pero imperfectas. Inventan cosas que suenan reales (alucinaciones), reflejan los sesgos de internet, y no entienden privacidad. Usarlos bien significa verificar siempre, nunca confiar ciegamente, y recordar que la responsabilidad final es tuya.

*Técnicamente:* Las alucinaciones son un fenómeno inherente a los modelos auto-regresivos: generan tokens con alta probabilidad según su distribución de entrenamiento, sin mecanismo de verificación factual. Los sesgos emergen de la distribución skewed de los datos de entrenamiento (mayoritariamente inglés, cultura tech occidental). La privacidad se compromete cuando prompts con datos sensibles se envían a APIs de terceros. No existe "grounding" automático — el modelo no consulta fuentes externas salvo que se le provean vía RAG o herramientas como Context7.
→ Ver [OWASP Top 10 for LLMs](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
→ Ver [Tópico 3: Ética](#3.11-ética-sesgos-y-limitaciones) — auto-referencia: esta sección es el punto de referencia para todo el playbook

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

> **Check de comprensión**
> 1. ¿Qué son las alucinaciones de los LLMs y por qué ocurren?
>    - R: Son respuestas que suenan convincentes pero son ficticias (APIs inexistentes, funciones inventadas). Ocurren porque el modelo genera tokens por probabilidad, no por verificación factual.
> 2. ¿Por qué nunca debés subir secretos o datos sensibles a un LLM público?
>    - R: Porque los prompts se envían a servidores de terceros y pueden ser almacenados, usados para entrenamiento, o filtrados. Usá entornos locales o enterprise para datos sensibles.
> 3. ¿Qué responsabilidad tenés sobre el código que genera la IA?
>    - R: El código que commiteás es TU responsabilidad, no del modelo. Si la IA generó un bug en producción, el responsable es quien hizo el commit y lo mergeó sin verificar.
> 4. ¿Qué deberías hacer SIEMPRE antes de mergear código generado por IA?
>    - R: Revisar cada línea, entender qué hace, verificar que las APIs y librerías existen, y asegurarte de que pasa los tests. Si no podés explicar el código, no deberías mergearlo.
> 5. ¿Cómo deberías tratar a la IA como compañero de desarrollo?
>    - R: Como un junior muy rápido pero que a veces miente: usala para acelerar, pero verificá todo, pedile que justifique sus decisiones, y nunca delegues tu pensamiento crítico.

---

## 3.12 Ecosistema de Herramientas

*En criollo:* El panorama va mucho más allá de ChatGPT. Hay herramientas para autocompletado en el editor (Copilot, Cursor), agentes que editan archivos directamente (Aider, Claude Code), y servicios que proveen documentación actualizada (Context7). Cada una tiene su nicho y su fortaleza.

*Técnicamente:* El ecosistema se clasifica en 3 capas: (1) Asistentes de código — operan a nivel de editor/IDE con autocompletado inline o chat contextual; (2) Agentes autónomos — operan a nivel de terminal/filesystem, pueden ejecutar comandos y editar múltiples archivos; (3) Infraestructura de soporte — documentación en tiempo real (Context7), grafos de conocimiento (CodeGraph), y memoria persistente (Engram). La integración entre capas (como Gentle AI + OpenCode + Engram) crea un flujo de trabajo cohesivo.
→ Ver [Tópico 1: Fundamentos Web](../concepts/01-fundamentos-web.md#1.9-httpstls) para entender por qué la seguridad importa incluso en herramientas de desarrollo

### Capa 1: Asistentes de código
| Herramienta | Enfoque |
|-------------|---------|
| **GitHub Copilot** | Autocompletado inline en el editor. El más maduro, integración nativa con GitHub. |
| **Cursor** | Editor basado en VS Code con IA nativa. Agentes, multi-file edits, @-mentions para contexto. |
| **Cody (Sourcegraph)** | Búsqueda y comprensión de código + generación contextual. Conoce TODO el repositorio. |
| **Codeium / Windsurf** | Autocompletado rápido, gratis para individuos, multi-IDE. |
| **Amazon Q Developer** | Enfocado en AWS, pero tiene capacidades generales de código. |
| **Tabnine** | Autocompletado con modelo local o cloud, privacidad-first. |

### Capa 2: Agentes y flujos de trabajo
| Herramienta | Enfoque |
|-------------|---------|
| **OpenCode** | Runtime de agentes con SDD, memoria, skills, revisión adversarial. |
| **Gentle AI** | Framework de agentes sobre OpenCode — lo que usamos en este roadmap. |
| **Aider** | Agente de código open-source basado en terminal. Edita archivos directamente. |
| **SWE-agent / Devin / OpenHands** | Agentes autónomos que resuelven issues completos. Más experimentales. |
| **Claude Code** | Agente de Anthropic integrado en la terminal. |

### Capa 3: Documentación e investigación
| Herramienta | Enfoque |
|-------------|---------|
| **Context7** | Documentación actualizada de librerías, consultable desde el agente. |
| **Perplexity** | Búsqueda web con IA, cita fuentes, ideal para investigar tecnologías nuevas. |
| **MDN + AI** | MDN integrado con herramientas como Context7 para referencia en tiempo real. |

> **Check de comprensión**
> 1. ¿En qué se diferencia GitHub Copilot de Cursor?
>    - R: Copilot es un asistente de autocompletado inline que se integra en tu editor existente; Cursor es un editor completo (basado en VS Code) con IA nativa, agentes multi-file, y @-mentions para contexto.
> 2. ¿Qué hace Aider que Copilot no hace?
>    - R: Aider es un agente autónomo basado en terminal que edita archivos directamente y puede ejecutar comandos. Copilot solo sugiere código inline — no ejecuta ni edita archivos por su cuenta.
> 3. ¿Por qué Context7 es importante para evitar alucinaciones?
>    - R: Porque consulta documentación oficial actualizada en vez de depender del conocimiento de entrenamiento del modelo (que puede estar desactualizado). El agente busca la doc viva antes de responder.
> 4. ¿Qué capa del ecosistema cubre Gentle AI + OpenCode + Engram?
>    - R: Las 3 capas: OpenCode como runtime de agentes (capa 2), Gentle AI como orquestador de flujos SDD (capa 2), y Engram como infraestructura de memoria persistente (capa 3).
> 5. ¿Cuál es la diferencia principal entre un asistente de código y un agente autónomo?
>    - R: El asistente sugiere código que vos aceptás o rechazás (Copilot, Tabnine); el agente autónomo puede ejecutar comandos, editar múltiples archivos, y tomar decisiones sin intervención constante (Aider, Claude Code, SWE-agent).

---

> **Nota sobre fuentes de este tópico (excepción documentada)**: Gentle AI, Engram, OpenCode y CodeGraph no tienen documentación pública estable comparable a MDN o las docs de React/Node. Las fuentes oficiales son los repositorios de GitHub de cada proyecto y los posts de blog de sus autores. Esta es una excepción reconocida en el spec de auditoría — no se inventan links; se usan los que existen y se documenta la ausencia.
