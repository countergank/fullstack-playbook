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
> - [GitHub — Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
> - [GitHub — Adding a new SSH key to your account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
> - [Pro Git book](https://git-scm.com/book/en/v2) — gratuito, completo, la referencia definitiva
> - [Conventional Commits](https://www.conventionalcommits.org/) — especificación oficial

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

---

## 2.3 Algoritmos y Estructuras de Datos

> Para desarrollo web NO necesitás ser científico de la computación. Necesitás lo que se usa todos los días.

### Complejidad — notación Big O (conceptual)
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

---

## 2.4 Resolución de Problemas

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

---

## 2.5 Debugging

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

---

> **Check de comprensión**: 
> 1. ¿Podés explicar la diferencia entre `git merge` y `git rebase`, y cuándo usarías cada uno?
> 2. ¿Qué complejidad tiene buscar un elemento en un array con `find` vs en un `Map` con `.get()`?
> 3. ¿Cuál es tu proceso paso a paso cuando encontrás un bug en producción?
