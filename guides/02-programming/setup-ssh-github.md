# Setup — SSH con GitHub

> **Tópico**: 2 — Fundamentos de Programación (sección 2.1)
> **Objetivo**: conectarte a GitHub SIN escribir usuario/contraseña cada vez, usando llaves SSH.
> **Prerequisito**: Git instalado y configurado (`setup-git.md`).

---

## ¿Por qué SSH?

Sin SSH, cada `git push` te pide usuario y contraseña (o un Personal Access Token). Con SSH, configurás una vez y nunca más. Además, SSH es más seguro: la llave privada nunca sale de tu máquina, y GitHub solo verifica la firma criptográfica. En entornos corporativos con 2FA, HTTPS con password directamente no funciona — necesitás tokens. SSH evita toda esa complejidad.

---

## Checklist

### 1. Verificar si ya tenés llaves
```bash
ls -la ~/.ssh
```
- [ ] Si ya ves `id_ed25519` y `id_ed25519.pub` → saltá al paso 3
- [ ] Si no → generá un par nuevo

### 2. Generar par de llaves Ed25519
```bash
ssh-keygen -t ed25519 -C "tu-email@ejemplo.com"
```
- [ ] Aceptar ubicación por defecto (`~/.ssh/id_ed25519`)
- [ ] Poner una **passphrase** (recomendada — protege tu llave si alguien accede a tu máquina)

### 3. Iniciar ssh-agent y agregar la llave
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
- [ ] Si pusiste passphrase, te la pide acá. El agente la recuerda durante la sesión.

### 4. Copiar la llave pública a GitHub
```bash
cat ~/.ssh/id_ed25519.pub
```
- [ ] Copiá TODO el output (empieza con `ssh-ed25519` y termina con tu email)
- [ ] GitHub → Settings → SSH and GPG keys → **New SSH key**
- [ ] Title: algo descriptivo ("Laptop personal") + pastear la llave → Add SSH key

### 5. (Opcional) Persistir la llave en el agente al arrancar
```bash
# Agregar al final de ~/.zshrc
eval "$(ssh-agent -s)" > /dev/null
ssh-add ~/.ssh/id_ed25519 2>/dev/null
```
> Alternativa: usar `ssh-add --apple-use-keychain` (macOS) o `ssh-add -K` (macOS viejo). En Linux, herramientas como `keychain` hacen esto automáticamente.

### 6. Usar URL SSH en lugar de HTTPS al clonar
```bash
# HTTPS (NO recomendado — te pide credenciales)
git clone https://github.com/usuario/repo.git

# SSH (recomendado — sin credenciales)
git clone git@github.com:usuario/repo.git
```
- [ ] Para repos ya clonados con HTTPS, cambiá el remote:
```bash
git remote set-url origin git@github.com:usuario/repo.git
```

### 7. Múltiples llaves para múltiples cuentas (trabajo + personal)
Crear `~/.ssh/config`:
```
# Cuenta personal (default)
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519

# Cuenta de trabajo
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_work_ed25519
```
Luego, en repos de trabajo, apuntar el remote a `git@github-work:usuario/repo.git`.

---

## Verificación

```bash
ssh -T git@github.com
```
```
Hi tu-usuario! You've successfully authenticated, but GitHub does not provide shell access.
```
**Si ves "Hi tu-usuario" → SSH configurado. ✅**

Luego probá con un repo real:
```bash
cd ~
git clone git@github.com:tu-usuario/algun-repo.git
cd algun-repo && git push   # SIN pedir credenciales
```

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Permission denied (publickey)" | La llave pública no está en GitHub, o el agente no la tiene cargada (`ssh-add` y re-verificar) |
| "Host key verification failed" | La primera vez que conectás te pregunta si confiás en el host: escribí `yes` |
| Passphrase pedida en cada push | El ssh-agent no está corriendo: `eval "$(ssh-agent -s)"` + `ssh-add` |
| No puedo conectarme desde firewalls corporativos | El puerto 22 puede estar bloqueado: probá `ssh -T -p 443 git@ssh.github.com` |
| Quiero usar dos cuentas sin estornudar | Usar `~/.ssh/config` con Hosts distintos (paso 7) |

---

## Recursos

- [GitHub Docs — Connecting with SSH](https://docs.github.com/en/authentication/connecting-for-github-with-ssh)
- [GitHub Docs — Adding a new SSH key](https://docs.github.com/en/authentication/connecting-for-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

## Preguntas de repaso

- **P:** ¿Por qué Ed25519 es recomendado sobre RSA para llaves SSH?
  **R:** Ed25519 es más rápido, más seguro, genera llaves más cortas (68 chars vs 3000+ de RSA), y no tiene las vulnerabilidades de implementación que tuvo RSA. Es el algoritmo moderno preferido por OpenSSH.

- **P:** ¿Qué hace `ssh-agent` y por qué es necesario?
  **R:** ssh-agent mantiene las llaves desbloqueadas en memoria para que no tengas que escribir la passphrase en cada operación SSH. Sin él, cada `git push` te pediría la passphrase de la llave.

- **P:** ¿Cómo cambiás un repo ya clonado con HTTPS a SSH?
  **R:** Con `git remote set-url origin git@github.com:usuario/repo.git`. Esto cambia la URL del remote `origin` de HTTPS a SSH sin necesidad de reclonar.

- **P:** ¿Qué significa el error "Host key verification failed"?
  **R:** Es la primera vez que te conectás a ese servidor SSH y no tenés su host key guardada en `~/.ssh/known_hosts`. Escribí `yes` cuando te pregunte si confiás en el host.

- **P:** ¿Cómo manejás dos cuentas de GitHub (personal + trabajo) sin conflictos?
  **R:** Creás un archivo `~/.ssh/config` con dos entradas `Host` distintas, cada una con su propia `IdentityFile`. Luego usás el host alias en el remote: `git@github-work:usuario/repo.git` para trabajo.

- **P:** ¿Qué pasa si perdés tu llave privada?
  **R:** Perdés acceso a todos los repos donde registraste la llave pública correspondiente. Necesitás generar un nuevo par de llaves y registrar la nueva pública en GitHub. Por eso es importante tener backups seguros de la llave privada.