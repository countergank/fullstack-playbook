# Setup — Git

> **Tópico**: 2 — Fundamentos de Programación (sección 2.1)
> **Objetivo**: Git instalado y configurado correctamente (identidad, editor, line endings, aliases, ignore global).
> **Prerequisito**: WSL + terminal (`setup-wsl.md`, `setup-terminal.md`).

---

## ¿Por qué Git?

Git es el sistema de control de versiones estándar de la industria. No es opcional si querés trabajar en desarrollo profesional. Cada empresa, cada proyecto open source, cada equipo usa Git. No alcanza con saber `add`, `commit`, `push` — necesitás entender el modelo de datos para poder deshacer errores, resolver conflictos, y colaborar sin romper el trabajo de otros. Esta guide te deja con Git configurado correctamente desde el día uno, evitando los problemas clásicos de principiante (emails wrong, line endings, credenciales).

---

## Checklist

### 1. Instalar Git
```bash
sudo apt update && sudo apt install -y git
git --version   # verificar
```

### 2. Identidad (CRÍTICO — se pega a cada commit)
```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-email@ejemplo.com"
```
- [ ] **⚠️ El email debe ser el mismo que usás en GitHub** (Settings → Emails). Si no, tus commits no se atribuyen a tu cuenta

### 3. Editor por defecto
```bash
git config --global core.editor "code --wait"
```

### 4. Line endings (Windows + Linux = conflicto clásico)
```bash
git config --global core.autocrlf input
```
> Lee así: al hacer checkout, Git no convierte los saltos de línea. Así se evitan los diffs gigantes por CRLF/LF.

### 5. Verificar tu configuración
```bash
git config --global --list
```
- [ ] Aparecen `user.name`, `user.email`, `core.editor`, `core.autocrlf`

### 6. `.gitignore` global (para no commitear basura de ningún proyecto)
```bash
# Crear archivo global
code ~/.gitignore_global

# Contenido mínimo:
# Node
node_modules/

# macOS
.DS_Store

# IDE
.vscode/
.idea/

# Env
.env
.env.local
```
git config --global core.excludesfile ~/.gitignore_global
```

### 7. Aliases útiles (velocidad diaria)
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.amend 'commit --amend'
git config --global alias.lg 'log --oneline --graph --all --decorate'
git config --global alias.unstage 'reset HEAD --'
```

### 8. Credential helper (recordar credenciales HTTPS)
```bash
sudo apt install -y libsecret-1-0 libsecret-1-dev gnome-keyring
```
> O más simple para empezar: usar SSH (`setup-ssh-github.md`) y no preocuparte más por credenciales HTTPS.

### 9. (Opcional) Segunda identidad por proyecto
Cuando trabajes en proyectos distintos con identidades distintas (trabajo vs personal):
```bash
# Dentro de un repo específico:
git config user.name "Nombre Profesional"
git config user.email "trabajo@empresa.com"
```

---

## Verificación

```bash
git --version                  # responde
git config --global user.name  # muestra tu nombre
git config --global user.email # muestra tu email de GitHub

# Prueba real: creá un repo temporal y commiteá
cd /tmp && mkdir git-test && cd git-test
git init
echo "prueba" > test.txt
git add test.txt
git commit -m "test: verify git config"
git log --oneline              # muestra tu commit
```

**Si el commit aparece con TU nombre y email → Git configurado. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Please tell me who you are" | Configuraste user.name/email (paso 2) |
| Commits no atribuidos a tu cuenta en GitHub | El email del commit ≠ email de GitHub. Revisá paso 2 |
| Diffs gigantes que solo cambian saltos de línea | `core.autocrlf` mal configurado (paso 4) |
| `git commit` se queda esperando | No definiste editor o está mal. `git config --global core.editor "code"` |
| Push te pide user/password todo el tiempo | Usá SSH (`setup-ssh-github.md`) |

---

## Recursos

- [Pro Git book](https://git-scm.com/book/en/v2)
- [GitHub — Configurar Git](https://docs.github.com/en/get-started/getting-started-with-git/set-up-git)

## Preguntas de repaso

- **P:** ¿Por qué el email de Git debe ser el mismo que usás en GitHub?
  **R:** Porque GitHub atribuye commits a tu cuenta comparando el email del commit con los emails registrados en tu perfil. Si no coinciden, tus commits aparecen como "anonimos" y no se cuentan en tu contribución.

- **P:** ¿Qué hace `core.autocrlf input` y por qué es importante en WSL?
  **R:** Le dice a Git que NO convierta saltos de línea al hacer checkout. En Linux/WSL los saltos son LF, y si Git los convierte a CRLF al checkout, cada archivo aparece como modificado aunque no lo esté.

- **P:** ¿Para qué sirve un `.gitignore` global?
  **R:** Para ignorar archivos que nunca deberían commitearse en NINGÚN proyecto: `node_modules/`, `.DS_Store`, `.env`, archivos de IDE. Así no necesitás crear un `.gitignore` desde cero en cada repo nuevo.

- **P:** ¿Qué ventaja tiene usar aliases como `git lg` en vez del comando completo?
  **R:** Velocidad y ergonomía. `git lg` es más corto que `git log --oneline --graph --all --decorate` y lo usás decenas de veces por día. Los aliases reducen la fricción de las operaciones más comunes.

- **P:** ¿Cómo configurás una identidad diferente para un proyecto específico?
  **R:** Dentro del repo, sin `--global`: `git config user.name "Nombre"` y `git config user.email "email"`. Esto sobreescribe la config global solo para ese repositorio.

- **P:** ¿Qué hace `git config --global core.editor "code --wait"`?
  **R:** Configura VS Code como el editor para mensajes de commit y merge conflicts. El flag `--wait` es crucial: sin él, VS Code abre y Git piensa que cancelaste la operación.