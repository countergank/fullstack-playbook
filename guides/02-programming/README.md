# 02 — Fundamentos de Programación: Orden de Ejecución

> ⚠️ **El orden importa.** Cada guía asume que la anterior fue completada.

## Paso a paso

1. **[setup-wsl.md](setup-wsl.md)** — WSL2 en Windows. La base de TODO.
2. **[setup-docker.md](setup-docker.md)** — Docker Desktop con backend WSL2. Sin esto, las bases de datos se instalan nativo.
3. **[setup-vscode.md](setup-vscode.md)** — VS Code conectado a WSL. Tu editor integrado al entorno Linux.
4. **[setup-terminal.md](setup-terminal.md)** — Terminal moderna: zsh, starship, ripgrep, fd, bat.
5. **[setup-git.md](setup-git.md)** — Git instalado y configurado (identidad, editor, aliases).
6. **[setup-ssh-github.md](setup-ssh-github.md)** — SSH con GitHub. Chau contraseñas.
7. **[setup-node.md](setup-node.md)** — nvm + Node LTS + pnpm. La base de todo proyecto JS/TS.
8. **[setup-http-clients.md](setup-http-clients.md)** — curl + Postman/Insomnia. Probar APIs desde el día uno.

---

## Verificación final

Al terminar todas las guías, ejecutá esto desde tu terminal WSL:

```bash
git --version && node --version && docker --version && code .
```

**Si los 4 comandos responden sin error → el entorno del tópico 2 está completo. ✅**
