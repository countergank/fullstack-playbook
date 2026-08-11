# 2. Fundamentos de Programación

> Objetivo: dominar las herramientas y el pensamiento que usás todos los días como desarrollador, sin importar el lenguaje o el framework.

---

## 2.1 Git — Control de Versiones

Git NO es opcional. Es la herramienta de colaboración más importante del ecosistema. No alcanza con saber `add`, `commit`, `push`. Tenés que entender el modelo de datos.

### Conceptos fundamentales

> Cada concepto se explica primero en criollo y después técnicamente, para que quede claro de verdad.

**Working directory, staging area, repository (.git)**

*En criollo:* Imaginá que tenés un escritorio. El **working directory** es tu escritorio — los archivos que ves y tocás. El **staging area** es una bandeja de "listo para guardar" donde ponés lo que querés que entre en la próxima foto. El **repository** (`.git`) es el álbum de fotos — ahí quedan registrados todos los estados anteriores de tu proyecto, y podés volver a cualquiera cuando quieras.

*Técnicamente:* Git tiene tres estados para cada archivo: *modified* (cambiado en working directory pero no staged), *staged* (marcado para el próximo commit), y *committed* (guardado en la base de datos de Git). El staging area — también llamado *index* — es lo que hace a Git diferente de otros VCS. Te permite construir commits atómicos seleccionando exactamente qué cambios incluir, en vez de commitear todo el working directory.

**Commits como snapshots, no diffs**

*En criollo:* Cuando sacás una foto con el celular, no guardás "lo que cambió desde la foto anterior". Guardás la imagen completa. Git hace lo mismo: cada commit es una foto COMPLETA de todo el proyecto en ese instante. Si un archivo no cambió, no lo duplica — guarda una referencia a la versión anterior, lo cual es súper eficiente.

*Técnicamente:* Git modela la historia como un DAG (Directed Acyclic Graph) de snapshots. Cada commit apunta a un objeto *tree* que representa el directorio raíz completo, que a su vez apunta a otros *trees* (subdirectorios) y *blobs* (archivos). Los blobs se identifican por SHA-1 de su contenido, lo que permite deduplicación automática. Git NO almacena deltas entre versiones como SVN — almacena objetos completos. La compresión (packfiles) es una optimización de almacenamiento, no del modelo conceptual.

**Hashing (SHA-1)**

*En criollo:* Cada cosa que guardás en Git — cada archivo, cada carpeta, cada commit — recibe un "DNI" único de 40 caracteres. Ese DNI se calcula a partir del contenido, así que si dos archivos son idénticos, tienen el mismo DNI. Si cambiás aunque sea una coma, el DNI cambia completamente. Esto significa que Git SABE si alguien adulteró algo — es imposible modificar un commit sin cambiar su hash.

*Técnicamente:* Git usa SHA-1 (migrando a SHA-256) como función de hash criptográfico para identificar todos sus objetos. Un commit es un objeto con: tree (hash del árbol raíz), parent(s) (hashes de commits anteriores), author, committer, message. El hash del commit se calcula hasheando ese contenido. Como cada commit referencia el hash de su padre, cualquier modificación en cualquier punto de la historia cambia TODOS los hashes subsiguientes — esto es lo que hace a Git inmutable y detectable ante corrupción.

**HEAD, branches, tags**

*En criollo:* **HEAD** es un post-it que dice "estás acá". Si apunta a un branch, te movés con el branch. Si apunta directo a un commit (detached HEAD), estás flotando. Un **branch** es otro post-it que se mueve solo — cada vez que commiteás, el branch avanza al nuevo commit. Un **tag** es una chapa grabada en la pared — lo ponés en un commit específico y no se mueve más. Ideal para marcar releases (`v1.0.0`).

*Técnicamente:* HEAD es una referencia simbólica (normalmente `refs/heads/main`). Un branch es un puntero mutable a un commit — al hacer commit, Git crea el nuevo commit y actualiza el puntero del branch actual. Un tag (annotated) es un objeto Git completo con autor, fecha, mensaje y firma GPG posible, que apunta a un commit. `git checkout` mueve HEAD; `git reset` mueve HEAD y opcionalmente el branch.

**Reflog**

*En criollo:* Git tiene un "historial de navegación" secreto donde guarda TODO lo que hiciste: cada commit, cada merge, cada rebase, cada reset, cada checkout. Incluso si "perdiste" commits por un reset o un rebase mal hecho, el reflog los sigue teniendo por ~90 días. Es tu red de seguridad.

*Técnicamente:* `.git/logs/` almacena un historial de cada movimiento de HEAD y de cada branch. `git reflog` muestra esto en orden cronológico inverso. Cada entrada tiene: hash anterior, hash nuevo, autor, timestamp, y acción. Los commits "inaccesibles" desde branches/tags todavía son accesibles desde el reflog hasta que el garbage collector (`git gc`) los limpie (por defecto, 90 días para objetos inalcanzables con reflog).

---

### Operaciones esenciales — de básico a avanzado

#### Primer nivel: el día a día
| Comando | Para qué |
|---------|----------|
| `git init` | Crear un repositorio nuevo en la carpeta actual |
| `git clone <url>` | Clonar un repositorio remoto a tu máquina |
| `git status` | Ver qué archivos cambiaron, cuáles están staged, en qué branch estás |
| `git add <file>` | Agregar archivos al staging area |
| `git add -p` | Agregar cambios por partes (hunks) — control granular |
| `git commit -m "mensaje"` | Crear un commit con los cambios staged |
| `git commit --amend` | Editar el último commit (mensaje o contenido). SOLO si no lo pusheaste |
| `git push` | Subir commits locales al remoto |
| `git pull` | Bajar commits del remoto y mergearlos a tu branch actual (`fetch` + `merge`) |
| `git pull --rebase` | Bajar commits del remoto y rebasear tu branch arriba de ellos (historia lineal) |
| `git fetch` | Bajar cambios del remoto SIN mergearlos — solo actualiza `origin/main` |
| `git merge <branch>` | Fusionar otra rama en la actual |
| `git log --oneline --graph --all` | Ver el historial como un árbol visual |

#### Segundo nivel: lo que te salva
| Comando | Para qué |
|---------|----------|
| `git diff` | Ver cambios en working directory (no staged) |
| `git diff --staged` | Ver cambios que ya están staged (lo que va en el próximo commit) |
| `git diff <branch1>..<branch2>` | Ver diferencias entre dos ramas |
| `git reset --soft HEAD~1` | Deshacer un commit, dejando los cambios en staging |
| `git reset --mixed HEAD~1` | Deshacer un commit, dejando los cambios en working directory (sin staged) |
| `git reset --hard HEAD~1` | Deshacer un commit Y los cambios — peligroso, usá con cuidado |
| `git stash` | Guardar cambios temporalmente (working directory limpio) |
| `git stash pop` | Recuperar los cambios guardados y borrar el stash |
| `git stash list` | Ver todos los stashes guardados |
| `git revert <commit>` | Crear un NUEVO commit que deshace los cambios de otro (seguro, no reescribe historia) |
| `git cherry-pick <commit>` | Traer un commit específico de otra rama a la actual |
| `git checkout -- <file>` | Descartar cambios en un archivo (volver al último commit) |

#### Tercer nivel: operaciones avanzadas
| Comando | Para qué |
|---------|----------|
| `git rebase -i HEAD~3` | Reescribir los últimos 3 commits (squash, reorder, edit, drop) |
| `git rebase main` | Rebasear tu branch sobre main (reescribe historia, más limpio que merge) |
| `git bisect start` / `git bisect good/bad` | Búsqueda binaria para encontrar qué commit introdujo un bug |
| `git blame <file> -L 10,20` | Ver quién modificó cada línea, en qué commit y cuándo |
| `git log -p --follow <file>` | Seguir la historia completa de un archivo incluso si fue renombrado |
| `git reflog` | Recuperar commits "perdidos" — el historial secreto de Git |
| `git clean -fd` | Eliminar archivos no trackeados (generados, temporales). Cuidado |

---

### Estrategias de branching

- **GitHub Flow**: `main` + feature branches cortas. Simple, ideal para CI/CD y deploy continuo. Una feature = un branch = un PR.
- **Git Flow**: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`. Más ceremonial, útil para productos con releases versionados y múltiples ambientes.
- **Trunk-Based Development**: branches de vida muy corta (horas, no días), merge frecuente a `main`. Feature flags para código no listo. Lo usan Google, Facebook, y equipos con CI/CD maduro.

---

### Mensajes de commit — Conventional Commits

```
feat(auth): add JWT refresh token rotation
^--^ ^--^  ^----------------------------^
│     │     │
│     │     └── descripción en presente e imperativo, máximo 72 chars
│     └── alcance opcional (auth, api, ui, db)
└── tipo: feat | fix | chore | docs | refactor | test | perf | ci | style
```

Ejemplos reales:
```
feat(api): add user registration endpoint
fix(cart): prevent negative quantities on checkout
refactor(db): extract query builder to shared module
chore(deps): bump prisma to 5.14.0
```

Un buen commit message explica **qué** cambió y **por qué**, no **cómo** (el cómo está en el diff). Si tu commit necesita un párrafo de explicación, usá el body:

```
fix(session): handle expired refresh tokens gracefully

Previously an expired refresh token would throw an unhandled 500 error
because the error handler wasn't catching TokenExpiredError from the JWT
library. Now it returns 401 with a clear message, and the client can
redirect to login.

Closes #142
```

---

### Conflictos de merge

*En criollo:* Dos personas (o vos en dos ramas) tocaron la misma parte del mismo archivo. Git no sabe cuál versión querés — te pide que decidas. Es como cuando dos personas editan el mismo párrafo en Google Docs y aparece el historial de versiones. Los marcadores `<<<<<<<`, `=======`, `>>>>>>>` te muestran tu versión y la de la otra rama. Borrás lo que no va, dejás lo correcto, y le decís a Git "listo, resolví".

*Técnicamente:* Git detecta conflictos a nivel de líneas (y líneas adyacentes con `merge.conflictStyle = diff3`). Al hacer merge o rebase, si dos cambios afectan la misma región, Git pausa la operación y marca el archivo como `unmerged`. Tenés tres opciones: (1) resolver manualmente editando el archivo, (2) `git checkout --ours/--theirs <file>` para elegir todo un lado, (3) usar una herramienta visual de merge. Después de resolver, `git add <file>` y `git merge --continue` (o `git rebase --continue`).

**Prevención**: branches cortos, pulls frecuentes de main, comunicación con el equipo sobre qué archivos están tocando.

---

### SSH y GitHub

#### ¿Qué es SSH y para qué sirve con Git?

*En criollo:* SSH (Secure Shell) es un protocolo que crea un túnel encriptado entre tu máquina y un servidor. Con Git, lo usás para conectarte a GitHub/GitLab/Bitbucket SIN tener que escribir usuario y contraseña cada vez que hacés push o pull. En vez de contraseña, usás un par de llaves: una **llave privada** que guardás en tu máquina (nunca la compartas, es tu identidad) y una **llave pública** que subís a GitHub. Cuando hacés `git push`, GitHub verifica que la llave privada de tu máquina coincide con la pública que registraste. Todo el tráfico va encriptado.

*Técnicamente:* SSH usa criptografía asimétrica (RSA, Ed25519 o ECDSA). La llave privada se almacena en `~/.ssh/id_ed25519` (o `id_rsa`). La llave pública se almacena en `~/.ssh/id_ed25519.pub` y se registra en GitHub en Settings → SSH and GPG keys. Durante la conexión, el cliente SSH usa la llave privada para firmar un desafío del servidor, demostrando posesión sin revelar la llave. El agente SSH (`ssh-agent`) mantiene las llaves desbloqueadas en memoria para no pedir passphrase en cada operación.

#### Configuración paso a paso

```bash
# 1. Verificar si ya tenés llaves SSH
ls -la ~/.ssh

# 2. Si no tenés (o querés una nueva), generar par de llaves Ed25519 (recomendado)
ssh-keygen -t ed25519 -C "tu-email@ejemplo.com"
# Presioná Enter para aceptar la ubicación por defecto (~/.ssh/id_ed25519)
# Ingresá una passphrase (recomendado) o dejá vacío

# 3. Iniciar el agente SSH y agregar la llave
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 4. Copiar la llave pública al portapapeles
cat ~/.ssh/id_ed25519.pub
# Copiá TODO el output (empieza con ssh-ed25519 y termina con tu email)

# 5. En GitHub: Settings → SSH and GPG keys → New SSH key
# Pegá la llave pública, ponele un título descriptivo (ej: "Laptop personal")

# 6. Verificar la conexión
ssh -T git@github.com
# Deberías ver: "Hi tu-usuario! You've successfully authenticated..."

# 7. Ahora cloná usando la URL SSH en vez de HTTPS
git clone git@github.com:usuario/repo.git
# En vez de: git clone https://github.com/usuario/repo.git
```

#### ¿HTTPS o SSH?

| Criterio | HTTPS | SSH |
|----------|-------|-----|
| Configuración inicial | Más simple, solo usuario y token | Requiere generar llaves y subir la pública |
| Uso diario | Pide token/password cada vez (o usás credential helper) | Una vez configurado, no pide nada |
| Firewalls corporativos | Funciona en casi todos lados | Puede estar bloqueado (puerto 22) |
| Múltiples cuentas | Configurable con credential helpers | Configurable con `~/.ssh/config` y distintos hosts |
| Recomendación | Para empezar rápido | Para uso profesional diario |

> **Referencias oficiales**:
> - [Pro Git book](https://git-scm.com/book/en/v2) — gratuito, completo, la referencia definitiva
> - [GitHub — Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
> - [Conventional Commits](https://www.conventionalcommits.org/) — especificación oficial

> **Check de comprensión**:
> 1. ¿Cuál es la diferencia entre working directory, staging area y repository en Git?
>    - R: El working directory son tus archivos actuales, el staging area es la bandeja de "listo para guardar" donde seleccionás qué cambios van al próximo commit, y el repository (.git) es el álbum de fotos con todos los estados anteriores del proyecto.
> 2. ¿Por qué Git usa snapshots en vez de diffs?
>    - R: Porque cada commit guarda una foto completa del proyecto, no solo lo que cambió. Los archivos que no cambiaron se referencian por su hash SHA-1 sin duplicarlos, lo que hace el modelo eficiente y simple.
> 3. ¿Qué pasa si modificás un commit antiguo en la historia de Git?
>    - R: Cambia el hash de ese commit y TODOS los hashes subsiguientes, porque cada commit referencia el hash de su padre. Esto hace a Git inmutable y detectable ante corrupción.
> 4. ¿Cuál es la diferencia entre `git reset --soft`, `--mixed` y `--hard`?
>    - R: `--soft` deshace el commit pero deja los cambios en staging; `--mixed` los deja en working directory (sin staged); `--hard` los destruye completamente. Solo `--soft` y `--mixed` son recuperables fácilmente.
> 5. ¿Cuándo usarías `git rebase` en vez de `git merge`?
>    - R: Usá rebase cuando querés una historia lineal y limpia (tu branch se "reubica" sobre main). Usá merge cuando querés preservar la historia exacta de cómo se desarrollaron las ramas, incluyendo los puntos de divergencia.
> 6. ¿Qué es el reflog y por qué es tu red de seguridad?
>    - R: Es el historial secreto de cada movimiento de HEAD en Git. Guarda commits "perdidos" por ~90 días incluso después de un reset o rebase mal hecho, permitiendo recuperarlos con `git reflog` y `git checkout`.

→ Ver [Tópico 1: Fundamentos de la Web](../concepts/01-fundamentos-web.md#1.3-dns) para entender DNS (prerequisito de SSH).
→ Ver [Tópico 8: Frameworks Frontend](../concepts/08-frameworks-herramientas-frontend.md) para ver cómo Git se usa en equipos con React/Vite.

---

## 2.2 Línea de Comandos / Terminal

El frontend moderno también se opera desde la terminal. No hace falta ser sysadmin, pero sí moverte con soltura.

### WSL — Windows Subsystem for Linux

*En criollo:* Si tu máquina es Windows, WSL2 te da un kernel Linux REAL corriendo al lado de Windows — sin máquina virtual, sin particionar el disco, sin arrancar dos veces. Abrís una terminal y tenés Bash, Node, Docker, PostgreSQL y todas las herramientas que el mundo profesional usa por defecto, pero escondidas detrás de un solo comando. Docker no corre "bien" en Windows puro: corre EN WSL.

*Técnicamente:* WSL2 ejecuta un kernel Linux real en una VM ligera administrada por el hypervisor de Windows (Hyper-V). El filesystem se monta en rutas estilo Linux (`/home/usuario/...`) con acceso transparente a los drives de Windows vía `/mnt/c/...`. VS Code se integra nativamente (Remote – WSL). Docker Desktop permite elegir "WSL 2 backend" para que los contenedores corran en Linux, no en el daemon de Windows.

```bash
# Instalar WSL2 (Windows PowerShell como administrador)
wsl --install

# Ver distros disponibles e instalar Ubuntu
wsl --list --online
wsl --install -d Ubuntu

# Actualizar kernel WSL (mantenerlo al día)
wsl --update

# Entrar a tu distro de Linux
wsl

# Configurar versión de WSL por distro (2 es la recomendada)
wsl --set-version Ubuntu 2

# Ruta de tu sistema de archivos Linux dentro de Windows
\\wsl$\Ubuntu\home\tu-usuario
```

**Check rápido**: ¿`ls -la` te muestra archivos con `.` y `..`? ¿`pwd` te muestra `/home/tu-usuario`? ¿`apt` o `apt-get` existen? → Tenés Linux funcionando.

> Guía de instalación paso a paso: `guides/02-programming/setup-wsl.md`

### Lo indispensable
- **Navegación**: `ls`, `cd`, `pwd`, `tree`
- **Archivos**: `touch`, `mkdir`, `cp`, `mv`, `rm`, `cat`, `less`, `head`, `tail`
- **Permisos**: `chmod` (`755`, `644`), `chown`
- **Búsqueda**: `grep -r`, `find`, `locate`
- **Pipes y redirección**: `|`, `>`, `>>`, `2>&1`
- **Procesos**: `ps aux`, `kill`, `htop`, `&` (background), `Ctrl+Z` / `fg` / `bg`
- **Variables de entorno**: `export`, `.env` files, `printenv`
- **Aliases y configuración**: `.bashrc` / `.zshrc`, alias útiles

### Herramientas modernas que reemplazan clásicos
| Clásico | Moderno | Para qué |
|---------|---------|----------|
| `grep` | `rg` (ripgrep) | Buscar en archivos, más rápido, ignora `.gitignore` por defecto |
| `find` | `fd` | Buscar archivos por nombre, más rápido, sintaxis más limpia |
| `cat` | `bat` | Mostrar archivos con syntax highlighting y números de línea |
| `ls` | `eza` / `lsd` | Listar directorios con íconos y colores |
| `man` | `tldr` | Documentación con ejemplos prácticos, no la enciclopedia |

### Gestión de paquetes
- **npm / yarn / pnpm** — el que uses, entendé `node_modules`, `package.json`, `package-lock.json`, versionado semántico (`^` vs `~`).
- **nvm / fnm** — gestor de versiones de Node.js. Fundamental para proyectos con distintas versiones.

> **Check de comprensión**:
> 1. ¿Qué es WSL2 y por qué es mejor que una máquina virtual tradicional para desarrollo?
>    - R: WSL2 corre un kernel Linux real en una VM ligera de Hyper-V, sin el overhead de virtualizar hardware completo. Arranca en segundos, pesa menos, y tiene integración directa con Windows (acceso a `/mnt/c/`, VS Code Remote).
> 2. ¿Por qué deberías trabajar en `~/` dentro de WSL en vez de `/mnt/c/`?
>    - R: Porque el puente Windows-Linux es lento para operaciones de archivos. Git, Node, y Docker funcionan mucho mejor en el filesystem nativo de Linux (`~/`), sin problemas de permisos ni lentitud.
> 3. ¿Cuál es la diferencia entre `grep` y `ripgrep (rg)`?
>    - R: ripgrep es más rápido porque usa multithreading y regex optimizadas en Rust, ignora `.gitignore` por defecto, y tiene mejor manejo de encoding. `grep` es el clásico POSIX que viene en todo sistema.
> 4. ¿Qué hacen los pipes (`|`) y la redirección (`>`, `>>`) en la terminal?
>    - R: El pipe (`|`) pasa la salida de un comando como entrada del siguiente. `>` redirige la salida a un archivo (sobreescribe), `>>` agrega al final sin borrar lo existente.
> 5. ¿Por qué usar un gestor de versiones de Node (nvm/fnm) en vez de instalar Node directamente?
>    - R: Porque distintos proyectos requieren distintas versiones de Node. nvm/fnm permiten switchear versiones por proyecto sin sudo, sin conflictos, y con `.nvmrc` para automatizarlo.
> 6. ¿Qué diferencia hay entre `chmod 755` y `chmod 644`?
>    - R: `755` da lectura+ejecución al owner y lectura+ejecución a grupo/otros (para scripts/directorios). `644` da lectura+escritura al owner y solo lectura a grupo/otros (para archivos de configuración).

→ Ver [Tópico 1: Fundamentos de la Web](../concepts/01-fundamentos-web.md#1.2-protocolo-http) para entender HTTP (se prueba desde terminal con curl).
→ Ver [Tópico 9: DevOps](../concepts/09-devops-deployment.md) para ver cómo la terminal se usa en CI/CD y deployment.

> **Referencias oficiales**:
> - [Microsoft Docs — WSL](https://learn.microsoft.com/en-us/windows/wsl/)
> - [nvm-sh/nvm](https://github.com/nvm-sh/nvm)
> - [ripgrep](https://github.com/BurntSushi/ripgrep) / [fd](https://github.com/sharkdp/fd)

---

## 2.3 Algoritmos y Estructuras de Datos

> Para desarrollo web NO necesitás ser científico de la computación. Necesitás lo que se usa todos los días.

### Complejidad — notación Big O (conceptual)

*En criollo:* Big O te dice cómo escala tu código cuando crecen los datos. O(1) es instantáneo sin importar cuántos datos tengas. O(n) crece proporcionalmente: el doble de datos, el doble de tiempo. O(n²) es peligroso: el doble de datos significa CUATRO veces más tiempo. En el día a día, tu trabajo es evitar O(n²) accidental.

*Técnicamente:* La notación Big O describe el límite superior del crecimiento de una función en términos del tamaño de entrada n, ignorando constantes y términos de menor orden. Se analiza el peor caso. Ejemplos: acceso a array por índice es O(1) porque la CPU calcula `base + index * size` directamente. Búsqueda lineal es O(n) porque en el peor caso recorres todo el array. Merge sort es O(n log n) porque divide recursivamente (log n niveles) y mergea en cada nivel (n operaciones).

- **O(1)** — constante. Acceso por índice en array, acceso por key en objeto/hash map.
- **O(log n)** — logarítmica. Búsqueda binaria, operaciones en árbol balanceado.
- **O(n)** — lineal. Recorrer un array una vez, `map`, `filter`, `find`.
- **O(n log n)** — linealítmica. Algoritmos de ordenamiento eficientes (merge sort, quicksort).
- **O(n²)** — cuadrática. Loop anidado. SIEMPRE preguntate si podés evitarlo.

Lo que importa en el día a día: reconocer cuándo estás escribiendo un O(n²) sin querer (un `find` dentro de un `map` o un `includes` dentro de un `filter`).

### Estructuras de datos que sí o sí usás
| Estructura | En JS/TS | Uso típico |
|------------|----------|------------|
| Array / Lista | `[]` | Colecciones ordenadas, stacks, queues |
| Hash Map / Objeto | `{}`, `Map` | Lookups por key O(1), cachés, agrupar datos |
| Set | `new Set()` | Valores únicos, deduplicación |
| Stack (LIFO) | `[]` + `push/pop` | Historial de navegación, undo/redo, call stack |
| Queue (FIFO) | `[]` + `push/shift`* | Colas de tareas, BFS, event loop |
| Tree | objetos anidados | DOM, sistema de archivos, JSON, menús jerárquicos |
| Graph | listas de adyacencia | Redes sociales, dependencias, rutas, recomendaciones |

> *`shift()` es O(n) en arrays. Para queues reales, usar una implementación con dos stacks o una linked list.

### Algoritmos que aparecen en el trabajo real
- **Búsqueda en arrays**: `find`, `filter`, `some`, `every`, `includes`. Entender cuándo usan O(n) y cuándo podés usar un `Map` para O(1).
- **Ordenamiento**: `sort()` con compare function. Entender que `sort()` convierte a string por defecto.
- **Recorrido de objetos anidados**: recursión (cuidado con stack overflow) o iteración con stack/queue.
- **Agrupación** (`groupBy`): de un array de objetos a un objeto donde la key es una propiedad.
- **Memoización**: cachear resultados de funciones caras para evitar recomputar.
- **Debounce y throttle**: patrones para limitar la frecuencia de ejecución (búsqueda en tiempo real, scroll, resize).
- **Depth-first vs Breadth-first**: cuándo usar cada uno. DFS para búsqueda profunda (recursivo), BFS para "más cercano primero" (cola).

> **Check de comprensión**:
> 1. ¿Cuál es la diferencia de complejidad entre buscar en un array con `find` vs en un `Map` con `.get()`?
>    - R: `find` es O(n) porque recorre elemento por elemento hasta encontrar. `.get()` en un Map es O(1) porque usa hashing para acceder directamente a la posición de memoria.
> 2. ¿Por qué `sort()` en JavaScript convierte elementos a string por defecto?
>    - R: Porque la especificación de ECMAScript define que sin compare function, `sort()` convierte cada elemento a string y los compara por su código UTF-16. Por eso `[10, 2, 1].sort()` da `[1, 10, 2]`. Siempre pasá `(a, b) => a - b` para números.
> 3. ¿Cuándo usarías un `Set` en vez de un array?
>    - R: Cuando necesitás valores únicos (deduplicación automática), membership testing O(1) con `.has()`, o operaciones de conjuntos (unión, intersección, diferencia).
> 4. ¿Qué problema tiene usar `shift()` en un array como queue?
>    - R: `shift()` es O(n) porque mueve todos los elementos restantes una posición hacia adelante. Para queues reales, usá dos stacks o una linked list donde enqueue y dequeue sean O(1).
> 5. ¿Qué es memoización y cuándo la usarías?
>    - R: Es cachear el resultado de una función pura para no recomputarlo con los mismos inputs. La usás cuando una función es cara (cálculos pesados, recursión) y se llama repetidamente con los mismos argumentos.
> 6. ¿Cuál es la diferencia práctica entre DFS y BFS?
>    - R: DFS va profundo primero (recursión o stack), ideal para explorar caminos completos. BFS va por niveles (cola), ideal para encontrar el camino más corto o "lo más cercano primero".

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md#6.3-índices) para ver cómo los índices usan árboles balanceados (O(log n)).
→ Ver [Tópico 13: Performance](../concepts/13-performance-optimizacion.md#13.5-n1-problem) para entender el problema N+1 (O(n²) accidental en queries).

> **Referencias oficiales**:
> - [MDN — Indexed collections (Arrays)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
> - [MDN — Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
> - [MDN — Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)

---

## 2.4 Resolución de Problemas

*En criollo:* Programar es resolver problemas, no escribir código. El código es solo la herramienta. Un desarrollador senior no sabe más funciones de memoria — tiene un MEJOR proceso para abordar lo desconocido. La diferencia entre un junior y un senior no es cuánto sabe, es cómo reacciona cuando NO sabe.

*Técnicamente:* La resolución de problemas en ingeniería de software sigue un ciclo sistemático: (1) especificación formal del problema (inputs, outputs, constraints), (2) descomposición en subproblemas verificables independientemente, (3) diseño de algoritmo con análisis de complejidad, (4) implementación con invariantes verificables, (5) testing con casos límite (edge cases), y (6) refactorización manteniendo los tests verdes. Este ciclo se aplica tanto a un bug de producción como a una feature nueva.

Un desarrollador profesional no memoriza soluciones — construye un proceso para encontrarlas.

### El método
1. **Entender el problema** antes de tocarlo. Reformularlo con tus palabras. Dibujarlo. Preguntar.
2. **Dividir en subproblemas**. Un problema grande está compuesto de varios chicos. Atacá de a uno.
3. **Escribir pseudocódigo** o un plan antes de escribir código real. Si no podés explicarlo en español, no podés programarlo.
4. **Resolver el caso más simple primero** (happy path). Después agregás edge cases, errores, validaciones.
5. **Testear a mano** con datos reales antes de automatizar tests. El test manual es más rápido que debuggear un test mal escrito.
6. **Iterar**: tu primera solución probablemente no sea la mejor. Hacela funcionar → hacela correcta → hacela rápida (en ese orden).

### Errores comunes de principiantes
- Saltar directo a escribir código sin entender el problema.
- Intentar resolver TODO de una vez en vez de atacar de a un subproblema.
- No leer el mensaje de error completo (el stack trace te dice EXACTAMENTE qué y dónde falló).
- Cambiar varias cosas a la vez y no saber cuál funcionó (o rompió).
- No buscar en Google/Stack Overflow antes de preguntar. La regla: 15 minutos de búsqueda activa, después pedí ayuda.

> **Check de comprensión**:
> 1. ¿Cuál es el primer paso antes de tocar código cuando te enfrentás a un problema?
>    - R: Entender el problema completamente. Reformularlo con tus propias palabras, dibujarlo, y asegurarte de saber qué se espera como resultado. Si no podés explicarlo en español, no podés programarlo.
> 2. ¿Por qué es peligroso cambiar varias cosas a la vez al debuggear?
>    - R: Porque si el bug desaparece, no sabés cuál cambio lo arregló. Si aparece otro bug, no sabés cuál cambio lo causó. Aprendés cero. Cambiá una cosa a la vez y verificá.
> 3. ¿Qué significa "resolver el happy path primero"?
>    - R: Implementar el flujo ideal sin edge cases, errores ni validaciones. Una vez que funciona el caso normal, agregás manejo de errores, validaciones de input, y casos límite de a uno.
> 4. ¿Cuánto tiempo deberías buscar solo antes de pedir ayuda?
>    - R: La regla es 15 minutos de búsqueda activa (Google, Stack Overflow, docs, logs). Menos es pedir sin intentar; más es perder tiempo que el equipo podría ahorrarte en 30 segundos.
> 5. ¿Por qué escribir pseudocódigo antes de código real ayuda?
>    - R: Porque te obliga a pensar en la lógica sin distraerte con sintaxis. Si no podés escribir el algoritmo en español paso a paso, tu código va a ser confuso y probablemente incorrecto.
> 6. ¿Qué orden deberías seguir al mejorar una solución?
>    - R: Hacela funcionar → hacela correcta → hacela rápida. Primero que compile y produzca resultado, luego que pase todos los tests y edge cases, recién después optimizá performance.

→ Ver [Tópico 5: Debugging](../concepts/02-fundamentos-programacion.md#2.5-debugging) para las técnicas específicas de debugging.
→ Ver [Tópico 10: Testing](../concepts/10-testing.md) para automatizar la verificación de tus soluciones.

> **Referencias oficiales**:
> - [The Pragmatic Programmer — Debugging](https://pragprog.com/titles/tpp20/) — libro clásico sobre el arte de resolver problemas
> - [Rubber Duck Debugging](https://rubberduckdebugging.com/) — la técnica explicada

---

## 2.5 Debugging

*En criollo:* Debuggear no es adivinar — es investigar como detective. Cada error es una pista, cada `console.log` es una pregunta que le hacés al código. El debugger no piensa por vos: vos pensás, el debugger te muestra lo que pasa paso a paso. La diferencia entre un dev que debuggea en 5 minutos y uno que tarda 2 horas no es la herramienta, es el MÉTODO.

*Técnicamente:* El debugging sistemático usa breakpoints para pausar la ejecución en puntos estratégicos, inspección de variables en el scope actual, el call stack para entender la cadena de llamadas que llevó al error, y watch expressions para monitorear valores en tiempo real. En Node.js, `--inspect-brk` pausa en la primera línea; en el browser, los DevTools permiten breakpoints condicionales que solo pausan cuando una expresión es verdadera.

Debuggear es una habilidad, no un botón. El debugger es una herramienta, no un sustituto del razonamiento.

### Herramientas
| Herramienta | Entorno | Para qué |
|-------------|---------|----------|
| `console.log` / `console.table` / `console.group` | Cualquiera | Rápido, pero no escalable |
| `debugger;` statement | Browser + DevTools abiertas | Breakpoint inline sin configurar |
| Chrome/Edge DevTools Sources tab | Frontend | Breakpoints condicionales, watch, call stack, scope |
| VS Code Debugger | Backend (Node.js) | Breakpoints, step over/into/out, variables |
| `node --inspect-brk` | Backend (Node.js) | Debuggear desde el inicio del script |
| `curl -v` / Postman | Backend (APIs) | Inspeccionar requests/responses HTTP |

### Técnicas
- **Rubber duck debugging**: explicá el código línea por línea a un pato de goma (o a un compañero). El 80% de las veces encontrás el bug antes de terminar de explicarlo.
- **Binary search debugging**: comentá o deshabilitá la mitad del código. Si el bug desaparece, está en la mitad deshabilitada. Repetí. Llegás al bug en O(log n).
- **Read the error message**: en serio. El 90% de los bugs de principiantes se resuelven leyendo el mensaje de error completo. El stack trace te da archivo y línea exacta.
- **Reproduce it first**: si no podés reproducir el bug a voluntad, no podés saber si lo arreglaste. Escribí los pasos exactos.
- **Change one thing at a time**: si tocás tres variables y el bug desaparece, no sabés cuál era la causa. Aprendizaje cero.

### Fuentes comunes de bugs en web dev
- **Type coercion** en JavaScript (`==` vs `===`, `0 == ""`, `[] == false`)
- **Async/await mal manejado**: olvidar `await`, promesas no retornadas, race conditions
- **Mutación inesperada**: modificar un objeto/array que otro código sigue referenciando
- **Closures con variables mutables** en loops
- **CORS bloqueando requests** cross-origin
- **Cache** del navegador mostrando datos viejos

> Referencia: [MDN — What are browser developer tools?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Tools_and_setup/What_are_browser_developer_tools)

> **Check de comprensión**:
> 1. ¿Qué es "rubber duck debugging" y por qué funciona?
>    - R: Es explicar tu código línea por línea a un objeto inanimado (o compañero). Funciona porque al verbalizar tu razonamiento, tu cerebro detecta inconsistencias que pasaban desapercibidas al leer en silencio. El 80% de las veces encontrás el bug antes de terminar.
> 2. ¿Cómo funciona el "binary search debugging"?
>    - R: Comentás o deshabilitás la mitad del código. Si el bug desaparece, está en la mitad deshabilitada. Si persiste, está en la mitad activa. Repetís dividiendo a la mitad hasta aislar el bug en O(log n) pasos.
> 3. ¿Cuál es la diferencia entre `console.log` y usar breakpoints del debugger?
>    - R: `console.log` te muestra valores en un momento específico pero no te deja interactuar. Un breakpoint pausa la ejecución y te permite inspeccionar TODAS las variables, el call stack, el scope, y ejecutar código en ese contexto exacto.
> 4. ¿Por qué es crítico reproducir un bug antes de intentar arreglarlo?
>    - R: Porque si no podés reproducirlo a voluntad, no podés saber si tu fix realmente lo resolvió o si fue suerte. Escribir los pasos exactos de reproducción es el primer paso para cualquier fix verificable.
> 5. ¿Qué es type coercion en JavaScript y por qué causa bugs?
>    - R: Es la conversión automática de tipos que hace JS con `==`. Ejemplos: `0 == ""` es true, `[] == false` es true, `"0" == false` es true. Por eso siempre usás `===` (comparación estricta sin conversión).
> 6. ¿Qué pasa cuando olvidás un `await` en una función async?
>    - R: La promesa se ejecuta pero no esperás su resultado. Tu código sigue con un objeto Promise pendiente en vez del valor resuelto, lo que causa errores tipo "[object Promise]" o undefined en operaciones posteriores.

→ Ver [Tópico 1: Fundamentos de la Web](../concepts/01-fundamentos-web.md#1.8-cors) para entender errores de CORS que aparecen al debuggear APIs.
→ Ver [Tópico 4: Backend Core](../concepts/04-backend-core.md#4.5-manejo-de-errores) para ver cómo estructurar el manejo de errores en el servidor.

---

## 2.6 Docker

*En criollo:* Un contenedor es como una caja que contiene TODO lo que una aplicación necesita para correr: código, dependencias, configuración, y hasta el sistema operativo mínimo. A diferencia de una máquina virtual (que virtualiza hardware entero y pesa gigas), un contenedor comparte el kernel de tu máquina y solo empaqueta lo que realmente usa. Arranca en segundos, pesa megas, y corre igual en tu máquina, en la de tu compañero, y en producción.

### Contenedor vs Máquina Virtual

| | Contenedor (Docker) | Máquina Virtual (VirtualBox, VMware) |
|---|---|---|
| Arranque | Segundos | Minutos |
| Tamaño | MB | GB |
| Aislamiento | Proceso (comparte kernel) | Completo (kernel propio) |
| Performance | Casi nativa | Overhead del hypervisor |
| Uso típico | Una app por contenedor | Un sistema operativo completo |

### Por qué Docker es FUNDAMENTAL para desarrollo

- **"En mi máquina funciona" deja de existir**: el contenedor es idéntico en tu máquina, en staging y en producción.
- **Bases de datos sin instalarlas nativo**: PostgreSQL, MongoDB, Redis corren en contenedores. No ensucian tu sistema, no tenés que configurar servicios. Un `docker compose up` y tenés todo corriendo.
- **Un comando para levantar todo el stack**: frontend, backend, base de datos, Redis, todo en un `compose.yaml`.
- **Paridad con producción**: si producción usa Docker (y el 90% de los deploys modernos sí), desarrollar en Docker significa que no hay sorpresas al deployar.

### Imagen vs Contenedor

- **Imagen**: el plano / receta. Define qué sistema operativo base, qué dependencias, qué archivos, qué comando ejecutar al arrancar. Es inmutable.
- **Contenedor**: la instancia corriendo de una imagen. Podés tener 10 contenedores de la misma imagen de PostgreSQL, cada uno con sus propios datos.

### Docker Compose — el orquestador de desarrollo

```yaml
# compose.yaml — define todos los servicios que necesita tu app
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: fullstack_dev
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

```bash
docker compose up -d    # levanta todo en background
docker compose down     # apaga todo
docker compose ps       # ver qué está corriendo
```

### Comandos esenciales

```bash
docker ps                    # contenedores corriendo
docker compose up -d         # levantar servicios definidos en compose.yaml
docker compose down          # apagar servicios
docker compose logs -f       # ver logs en vivo de todos los servicios
docker compose logs db       # logs solo de un servicio
docker exec -it <id> bash    # entrar a la terminal de un contenedor
docker system prune -a       # limpiar imágenes/vólumes no usados (liberar disco)
```

### Docker + WSL2

En Windows, Docker Desktop usa WSL2 como backend. Los contenedores corren DENTRO de WSL2, no en Windows. Esto significa que:
- Tenés rendimiento de Linux nativo.
- Los archivos de tu proyecto en `~/` (dentro de WSL) se pueden montar en contenedores sin problemas de permisos.
- La red de WSL2 expone los puertos de los contenedores automáticamente en `localhost`.

> Guía de instalación: `guides/02-programming/setup-docker.md`

> **Check de comprensión**:
> 1. ¿Cuál es la diferencia fundamental entre un contenedor y una máquina virtual?
>    - R: Un contenedor comparte el kernel del host y solo empaqueta la app + dependencias (MB, segundos de arranque). Una VM virtualiza hardware completo con su propio kernel (GB, minutos de arranque).
> 2. ¿Qué es una imagen Docker y qué es un contenedor?
>    - R: La imagen es el plano/receta inmutable (qué OS, qué dependencias, qué comando). El contenedor es la instancia corriendo de esa imagen. Podés tener 10 contenedores de la misma imagen.
> 3. ¿Por qué Docker elimina el problema de "en mi máquina funciona"?
>    - R: Porque el contenedor es idéntico en desarrollo, staging y producción. Las dependencias, versiones y configuración están empaquetadas en la imagen, no dependen del sistema operativo del desarrollador.
> 4. ¿Qué hace `docker compose up -d` y qué significa el flag `-d`?
>    - R: Levanta todos los servicios definidos en `compose.yaml`. El flag `-d` (detached) los corre en background, liberando la terminal. Sin `-d`, los logs se muestran en la terminal y bloquea.
> 5. ¿Qué diferencia hay entre `docker compose down` y `docker compose down -v`?
>    - R: `down` apaga los contenedores y red pero PRESERVA los volúmenes (datos persistentes). `down -v` además BORRA los volúmenes, eliminando todos los datos. Usá `-v` solo cuando querés empezar de cero.
> 6. ¿Por qué en WSL2 los archivos del proyecto deben estar en `~/` y no en `/mnt/c/`?
>    - R: Porque el puente Windows-Linux añade latencia significativa en operaciones de I/O. Docker monta volúmenes mucho más rápido desde el filesystem nativo de Linux, y evita problemas de permisos entre sistemas.

→ Ver [Tópico 6: Bases de Datos](../concepts/06-bases-datos.md) para ver cómo PostgreSQL, MongoDB y Redis corren en contenedores.
→ Ver [Tópico 9: DevOps](../concepts/09-devops-deployment.md) para ver cómo Docker se usa en producción con CI/CD.

> **Referencias oficiales**:
> - [Docker Docs — Get Started](https://docs.docker.com/get-started/)
> - [Docker Compose Docs](https://docs.docker.com/compose/)

