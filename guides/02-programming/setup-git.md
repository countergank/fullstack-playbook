# Setup — Git

> **Tópico**: 2 — Fundamentos de Programación (sección 2.1)
> **Objetivo**: Git instalado y configurado correctamente (identidad, editor, line endings, aliases, ignore global).
> **Prerequisito**: WSL + terminal (`setup-wsl.md`, `setup-terminal.md`).

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