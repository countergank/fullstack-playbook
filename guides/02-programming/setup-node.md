# Setup — Node.js (nvm/fnm + pnpm)

> **Tópico**: 2 — Fundamentos de Programación (sección 2.2, gestión de paquetes)
> **Objetivo**: Node.js instalado con gestor de versiones (para cambiar de versión por proyecto sin romper nada) y pnpm.
> **Prerequisito**: WSL + terminal (`setup-wsl.md`, `setup-terminal.md`).

---

## Checklist

### 1. Instalar nvm (Node Version Manager)
> nvm instala Node por usuario, sin `sudo`, y permite switchear versiones al instante. `fnm` es la alternativa más rápida (Rust) — elegí uno, yo recomiendo nvm para empezar por la comunidad y documentación.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# Recargar shell (el instalador ya agrega las líneas al .zshrc)
source ~/.zshrc

# Verificar
nvm --version
```

### 2. Instalar la versión LTS de Node
```bash
nvm install --lts
nvm use --lts
node --version   # v22.x.x (LTS)
npm --version    # viene con Node
```
- [ ] `node --version` responde con la LTS

### 3. Definir default (para no tener que `nvm use` en cada terminal)
```bash
nvm alias default lts/*
```

### 4. pnpm — gestor de paquetes rápido y eficiente
```bash
npm install -g pnpm
pnpm --version
```
> pnpm ahorra espacio (hardlinks, no duplica node_modules) y es mucho más rápido que npm. Es el estándar en muchos proyectos modernos.

### 5. Configurar npm (registry + level de auditoría)
```bash
npm config set audit true
npm config set fund false   # desactivar avisos de "sponsor"
```

### 6. Soportar múltiples versiones por proyecto con `.nvmrc`
Dentro de cada proyecto:
```bash
echo "22" > .nvmrc
nvm use              # lee el .nvmrc automáticamente
```

---

## Verificación

```bash
node --version   # v22.x.x LTS
npm --version    # responde
pnpm --version   # responde
nvm ls           # muestra la versión instalada con flecha en la activa
nvm current      # v22.x.x
```

**Si `node -v` responde con una versión LTS y `pnpm` existe → Node listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `nvm: command not found` | No recargaste `~/.zshrc`, o instalaste con otro shell (bash vs zsh). `source ~/.zshrc` |
| `node / npm not found` al abrir VS Code | VS Code no heredó las variables de nvm. Reiniciar VS Code, o abrirlo desde WSL (`code .` dentro de WSL) |
| Permisos de npm (EPERM/EACCES) | En Linux/no sudo NO deberías necesitar sudo con nvm. Si te falta, revisá que nvm esté activo |
| Proyecto pide Node 18 pero tenés 22 | `nvm install 18 && nvm use 18` en ese proyecto |
| pnpm no instala bien en algún repo | Puede que el repo use npm/yarn específicamente. Usá el gestor del repo |

---

## Recursos

- [nvm (Node Version Manager)](https://github.com/nvm-sh/nvm)
- [Node.js — Releases (LTS)](https://nodejs.org/en/about/previous-releases)
- [pnpm](https://pnpm.io/)