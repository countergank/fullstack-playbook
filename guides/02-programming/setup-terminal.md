# Setup — Terminal Moderna (Windows Terminal + Zsh)

> **Tópico**: 2 — Fundamentos de Programación (sección 2.2)
> **Objetivo**: tener una terminal rápida, cómoda y con herramientas modernas para trabajar todos los días.
> **Prerequisito**: WSL instalado (`setup-wsl.md`).

---

## ¿Por qué una terminal moderna?

La terminal es tu herramienta principal como desarrollador. Cada día la usás cientos de veces: git, npm, docker, servidores, scripts. Una terminal moderna con Zsh, Starship, y herramientas como ripgrep y fd no es un lujo — es una inversión en productividad. Los atajos, el autocompletado, y el feedback visual (colores, íconos) reducen la fricción de cada operación. No necesitás ser experto en terminal, pero sí cómodo.

---

## Checklist

### 1. Windows Terminal
- [ ] Instalar Windows Terminal desde Microsoft Store
- [ ] Configurar Ubuntu como perfil por defecto (Settings → Startup → Default profile → Ubuntu)
- [ ] Atajos clave: `Ctrl+Tab` (cambiar tab), `Ctrl+Shift+T` (nueva pestaña), `Ctrl+Shift+W` (cerrar)

### 2. Shell: Zsh + Oh My Zsh
```bash
# Instalar zsh
sudo apt update && sudo apt install -y zsh

# Instalar Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Cambiar shell por defecto a zsh
chsh -s $(which zsh)
```
- [ ] Cerrar sesión y volver a entrar → deberías ver el prompt de Oh My Zsh

### 3. Prompt informativo: Starship
```bash
# Instalar Starship (rust toolchain o bundle precompilado)
curl -sS https://starship.rs/install.sh | sh

# Agregar al final del .zshrc
echo 'eval "$(starship init zsh)"' >> ~/.zshrc
```
- [ ] El prompt ahora muestra branch de git, versiones de node/python, tiempo de ejecución

### 4. Herramientas modernas (reemplazan clásicos)
```bash
# ripgrep > grep (buscar en archivos, rápido, ignora .gitignore)
sudo apt install -y ripgrep

# fd > find (buscar archivos por nombre)
sudo apt install -y fd-find

# bat > cat (con syntax highlighting)
sudo apt install -y bat

# tldr > man (ejemplos prácticos)
sudo apt install -y tldr

# eza > ls (listado con íconos/colores)
sudo apt install -y eza

# fzf (fuzzy finder en terminal)
sudo apt install -y fzf

# htop (monitor de procesos/recursos)
sudo apt install -y htop
```

### 5. Aliases y atajos en `~/.zshrc`
```bash
# Agregar al final del .zshrc
alias ll='eza -la --icons'
alias ls='eza --icons'
alias cat='bat'
alias find='fd'
alias grep='rg'

# Git helpers
alias gs='git status'
alias gp='git push'
alias gl='git log --oneline --graph --all'
alias gc='git commit'
alias ga='git add'
```

- [ ] Recargar: `source ~/.zshrc`

### 6. Extras (opcional pero recomendado)
- [ ] `zsh-autosuggestions` (sugerencias grises mientras escribís)
- [ ] `zsh-syntax-highlighting` (comandos válidos en verde, inválidos en rojo)
- [ ] Nerd Fonts (para que los íconos de eza/starship se vean): instalar "JetBrainsMono Nerd Font" y configurarla en Windows Terminal como fuente

---

## Verificación

```bash
# Desde tu terminal WSL:
zsh --version        # > 5.8
rg --version         # ripgrep instalado
batcat --version     # bat instalado (en Ubuntu se llama batcat, alias a bat)
eza --version        # eza instalado
starship --version   # starship instalado
ls                   # con íconos y colores
```

**Si ves íconos, colores, y los comandos responden → terminal lista. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `batcat` en vez de `bat` | En Ubuntu el paquete se llama `bat` pero el binario es `batcat`. Creá un alias: `alias bat='batcat'` en tu `.zshrc` |
| Starship no aparece después de instalar | Asegurate de agregar `eval "$(starship init zsh)"` al `.zshrc` y hacer `source ~/.zshrc` |
| Los íconos de eza/Starship se ven como cuadrados | No tenés una Nerd Font instalada. Instalá "JetBrainsMono Nerd Font" y configurala en Windows Terminal |
| `zsh-autosuggestions` no funciona | Necesitás clonar el repo en `~/.oh-my-zsh/custom/plugins/` y agregarlo a `plugins=(...)` en `.zshrc` |
| `fd` no se encuentra | En Ubuntu el paquete es `fd-find` y el binario es `fdfind`. Creá un alias: `alias fd='fdfind'` |
| Oh My Zsh tarda en cargar | Desactivá plugins que no uses. Cada plugin añade tiempo de inicio. Mantené solo git y los esenciales |

---

## Recursos

- [Oh My Zsh](https://ohmyz.sh/)
- [Starship](https://starship.rs/)
- [ripgrep](https://github.com/BurntSushi/ripgrep) / [fd](https://github.com/sharkdp/fd) / [bat](https://github.com/sharkdp/bat)
- [Nerd Fonts](https://www.nerdfonts.com/)

## Preguntas de repaso

- **P:** ¿Por qué Zsh es preferible a Bash como shell por defecto?
  **R:** Zsh tiene mejor autocompletado, corrección de typos, temas visuales, y es compatible con la mayoría de scripts de Bash. Con Oh My Zsh, obtenés plugins y temas que Bash no tiene nativamente.

- **P:** ¿Qué hace Starship y por qué es mejor que el prompt por defecto?
  **R:** Starship muestra información contextual en el prompt: branch de git, versión de Node/Python, tiempo de ejecución del último comando, y más. Es rápido (escrito en Rust) y funciona con cualquier shell.

- **P:** ¿Por qué ripgrep es mejor que grep para buscar en código?
  **R:** ripgrep es más rápido (multithreading en Rust), ignora `.gitignore` automáticamente, tiene mejor syntax de regex, y muestra resultados con colores y contexto de forma más legible.

- **P:** ¿Qué hace `fzf` y cómo lo usarías en el día a día?
  **R:** fzf es un fuzzy finder que te permite buscar interactivamente en historial de comandos, archivos, procesos, etc. Con `Ctrl+R` en la terminal, buscás comandos antiguos escribiendo parte del texto.

- **P:** ¿Por qué es importante instalar una Nerd Font?
  **R:** Porque herramientas como Starship y eza usan íconos Unicode especiales (git branch, lenguajes de programación, tipos de archivo) que las fuentes normales no tienen. Sin Nerd Font, esos íconos se ven como cuadrados vacíos.

- **P:** ¿Qué ventaja tiene `zsh-autosuggestions`?
  **R:** Muestra sugerencias grises mientras escribís, basadas en tu historial de comandos. Si escribiste `docker compose up -d` antes, la próxima vez que escribas `doc` te sugiere el comando completo. Lo aceptás con la flecha derecha.