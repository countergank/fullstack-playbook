# Setup — Node.js (nvm/fnm + pnpm)

> **Tópico**: 2 — Fundamentos de Programación (sección 2.2, gestión de paquetes)
> **Objetivo**: Node.js instalado con gestor de versiones (para cambiar de versión por proyecto sin romper nada) y pnpm.
> **Prerequisito**: WSL + terminal (`setup-wsl.md`, `setup-terminal.md`).

---

## ¿Por qué un gestor de versiones de Node?

Instalar Node directamente con `apt install nodejs` te da una versión fija que no podés cambiar por proyecto. En el mundo real, un proyecto necesita Node 18, otro Node 20, otro Node 22. Sin un version manager, tendrías que desinstalar y reinstalar constantemente. nvm (o fnm) te permite tener TODAS las versiones instaladas y switchear entre ellas con un comando. Además, no requiere `sudo` porque instala en tu home directory.

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

## Preguntas de repaso

- **P:** ¿Por qué usar nvm en vez de instalar Node directamente con `apt`?
  **R:** Porque `apt` te da una versión fija del repositorio de Ubuntu (que suele ser vieja). nvm te permite instalar cualquier versión, switchear entre ellas por proyecto, y no requiere `sudo`.

- **P:** ¿Qué hace `nvm alias default lts/*`?
  **R:** Define la versión LTS más reciente como la default, para que cada terminal nueva arranque con esa versión sin tener que hacer `nvm use` manualmente.

- **P:** ¿Por qué pnpm es mejor que npm en términos de espacio en disco?
  **R:** Porque pnpm usa hardlinks para compartir paquetes entre proyectos. Si 10 proyectos usan express@4.18, npm lo descarga 10 veces en cada node_modules. pnpm lo guarda una vez en un store global y crea links.

- **P:** ¿Cómo funciona `.nvmrc` y qué ventaja tiene?
  **R:** Es un archivo en la raíz del proyecto que contiene el número de versión de Node (ej: "22"). Al ejecutar `nvm use` sin argumentos, lee automáticamente ese archivo. Podés automatizarlo con herramientas como `avn` o `zsh-nvm` para que haga el switch al entrar al directorio.

- **P:** ¿Qué pasa si un proyecto usa yarn pero tenés pnpm instalado?
  **R:** pnpm puede instalar dependencias de proyectos que usen yarn.lock, pero es mejor usar el gestor que el proyecto especifica. Cada gestor tiene su propio lockfile y algoritmo de resolución. Mezclarlos puede causar inconsistencias.

- **P:** ¿Por qué configurás `npm config set fund false`?
  **R:** Para desactivar los mensajes de "fund" que npm muestra al final de cada instalación, pidiendo que sponsorices paquetes. Son distractores en el output y no aportan al desarrollo.