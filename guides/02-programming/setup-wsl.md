# Setup — WSL (Windows Subsystem for Linux)

> **Tópico**: 2 — Fundamentos de Programación (sección 2.2)
> **Objetivo**: tener un kernel Linux real funcionando en tu máquina Windows, listo para la terminal, Git, Node, Docker y todo lo que viene después.
> **Prerequisito**: Windows 10 (2004+) o Windows 11, con virtualización habilitada en BIOS/UEFI.

---

## ¿Por qué?

Docker, Node, PostgreSQL, Redis y prácticamente todas las herramientas de desarrollo modernas están hechas para Linux. WSL2 te da ese Linux sin máquina virtual, sin particionar disco, sin arrancar dos veces. Docker Desktop corre "con backend WSL", no en Windows puro.

---

## Checklist

### 1. Verificar requisitos
- [ ] Windows 10 versión 2004 o superior, o Windows 11
- [ ] Virtualización habilitada en BIOS (VT-x para Intel, AMD-V para AMD)
- [ ] Verificá con: `systeminfo` → busca "Requisitos de Hyper-V" → "Se ha detectado un hipervisor"

### 2. Instalar WSL2

Abrí **PowerShell como administrador** y ejecutá:

```powershell
wsl --install
```

Esto instala WSL2 y una distro de Ubuntu por defecto. **Reiniciá la máquina** cuando termine.

> ⚠️ Al primer arranque de Ubuntu, te va a pedir que crees un **nombre de usuario** y una **contraseña**. Elegí lo que quieras — este será tu usuario dentro de Linux. La contraseña no se muestra mientras la escribís (es normal, es por seguridad).

### 3. Instalar Ubuntu LTS (la última versión con soporte extendido)

*En criollo:* Ubuntu tiene dos tipos de versiones: las **LTS** (Long Term Support, 5 años de actualizaciones de seguridad) y las intermedias (9 meses). Para desarrollo profesional, SIEMPRE usá LTS — no querés que tu entorno se quede sin soporte a los 9 meses.

El `wsl --install` del paso anterior instala la versión por defecto, que puede no ser LTS. Verificá y corregí si hace falta:

```powershell
# Ver todas las distros disponibles (buscá las que dicen "LTS")
wsl --list --online
```

- [ ] Buscá en la lista la versión más reciente que diga **LTS**. Ejemplo: `Ubuntu-24.04`.
- [ ] Instalar la última LTS disponible (reemplazá `Ubuntu-24.04` por la que corresponda):
  ```powershell
  wsl --install -d Ubuntu-24.04
  ```
- [ ] Si el `wsl --install` inicial ya te instaló una versión no-LTS, desinstalala y reinstalá la LTS:
  ```powershell
  wsl --unregister Ubuntu
  wsl --install -d Ubuntu-24.04
  ```

### 4. Verificar que estés en WSL2 (no WSL1)
```bash
wsl --list --verbose
```
- [ ] La distro debe mostrar `VERSION 2`. Si muestra `1`, convertí: `wsl --set-version Ubuntu-24.04 2`

### 5. Actualizar kernel
- [ ] `wsl --update` (mantener WSL al día)

### 6. Instalar WSL desde Microsoft Store (para que las distros aparezcan en el Explorador de Archivos)

Por defecto, WSL se instala como componente de Windows. Pero si además instalás **"Windows Subsystem for Linux" desde la Microsoft Store**, las distros de Linux aparecen en el Explorador de Archivos de Windows como si fueran una carpeta más, facilitando mover archivos entre Windows y Linux.

- [ ] Abrir **Microsoft Store**
- [ ] Buscar **"Windows Subsystem for Linux"** (la app oficial de Microsoft)
- [ ] Instalar
- [ ] Ahora en el Explorador de Archivos, en la barra lateral, vas a ver un ícono de **Linux** con tus distros adentro

> Si no usás la Store, igual podés acceder manualmente a los archivos desde Windows escribiendo `\\wsl$\` en la barra de direcciones del Explorador.

### 7. Configurar paridad Windows/Linux
- [ ] Instalar VS Code + extensión "WSL" (Remote - WSL). Desde WSL, `code .` abre VS Code conectado a tu entorno Linux
- [ ] Instalar Windows Terminal (Microsoft Store) — la terminal moderna que maneja tabs WSL/PowerShell/CMD
- [ ] Actualizar paquetes de Ubuntu: `sudo apt update && sudo apt upgrade -y`

### 8. Entender el filesystem

**Dónde vive cada cosa:**

- Tus archivos Linux viven en `~/` (ej: `/home/tu-usuario/proyectos`)
- Los discos de Windows se montan en `/mnt/c/`, `/mnt/d/`, etc.
- Desde Windows, tus archivos Linux están en `\\wsl$\Ubuntu-24.04\home\tu-usuario` (o en el acceso directo de Linux en el Explorador si instalaste la app de la Store)

**Dónde se guardan físicamente los archivos de WSL en Windows:**

Los discos virtuales de WSL2 se almacenan como archivos `.vhdx` en:
```
%USERPROFILE%\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu-24.04LTS_...\LocalState\ext4.vhdx
```
No modifiques estos archivos manualmente — podés corromper la distro. Para acceder a tus datos, usá la ruta `\\wsl$\` o el acceso desde el Explorador.

> ⚠️ **Regla de oro**: trabajá en `~/` (Linux), no en `/mnt/c/`. Los proyectos en `~/` son mucho más rápidos, Docker monta los volúmenes sin problemas de permisos, y las operaciones de Git/Node no sufren la lentitud del puente Windows-Linux.

---

## Verificación

```bash
# Desde tu terminal WSL:
uname -a            # muestra un kernel Linux ("Linux host ... GNU/Linux")
ls -la              # ves "." y ".." y tu .bashrc etc.
pwd                 # /home/tu-usuario
apt --version       # gestor de paquetes de Ubuntu disponible
code --version      # VS Code abre remoto-WSL correctamente
```

**Si `uname` muestra "Linux" y `apt` responde → WSL está listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `wsl` no reconoce el comando | Asegurate de que Windows esté actualizado (2004+). Si aún falla, instalá WSL desde la Microsoft Store |
| "Please enable the Virtual Machine Platform" | Habilitar la característica "Virtual Machine Platform" en "Activar o desactivar características de Windows" y reiniciar |
| Distro muestra VERSION 1 | `wsl --set-version Ubuntu-24.04 2` |
| Linux no aparece en el Explorador de Archivos | Instalá "Windows Subsystem for Linux" desde Microsoft Store (paso 6) |
| Lentitud en `/mnt/c/` | No trabajes en `/mnt/c/`. Mové el proyecto a `~/` dentro de WSL |
| Sin internet dentro de WSL | Reiniciar WSL: `wsl --shutdown` y volver a entrar. Revisar proxies corporativos |
| Memoria: WSL consume mucha RAM | Crear `%UserProfile%\.wslconfig` y limitar: `[wsl2] memory=8GB` |

---

## Recursos

- [Microsoft Docs — Instalar WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Microsoft Docs — Configuración avanzada (.wslconfig)](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)