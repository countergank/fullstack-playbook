# Setup — Terminal Moderna (Windows Terminal + Zsh)

> **Tópico**: 2 — Fundamentos de Programación (sección 2.2)
> **Objetivo**: tener una terminal rápida, cómoda y con herramientas modernas para trabajar todos los días.
> **Prerequisito**: WSL instalado (`setup-wsl.md`).

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

## Recursos

- [Oh My Zsh](https://ohmyz.sh/)
- [Starship](https://starship.rs/)
- [ripgrep](https://github.com/BurntSushi/ripgrep) / [fd](https://github.com/sharkdp/fd) / [bat](https://github.com/sharkdp/bat)
- [Nerd Fonts](https://www.nerdfonts.com/)