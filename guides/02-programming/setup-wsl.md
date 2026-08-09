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
- [ ] Abrir **PowerShell como administrador**
- [ ] Ejecutar `wsl --install` (instala WSL2 + Ubuntu por defecto)
- [ ] **Reiniciar la máquina** cuando termine

> Si tu Windows es viejo o `wsl --install` falla, usar el método manual:
> ```powershell
> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
> ```
> Reiniciar, luego `wsl --set-default-version 2`.

### 3. Configurar tu distro
- [ ] Ver distros disponibles: `wsl --list --online`
- [ ] Instalar Ubuntu (o la que prefieras): `wsl --install -d Ubuntu`
- [ ] Al primer ingreso, crear **tu usuario** y contraseña (será tu usuario en Linux)

### 4. Verificar que estés en WSL2 (no WSL1)
```bash
wsl --list --verbose
```
- [ ] La distro debe mostrar `VERSION 2`. Si muestra `1`, convertí: `wsl --set-version Ubuntu 2`

### 5. Actualizar kernel
- [ ] `wsl --update` (mantener WSL al día)

### 6. Configurar paridad Windows/Linux
- [ ] Instalar VS Code + extensión "WSL" (Remote - WSL). Desde WSL, `code .` abre VS Code conectado a tu entorno Linux
- [ ] Instalar Windows Terminal (Microsoft Store) — la terminal moderna que maneja tabs WSL/PowerShell/CMD
- [ ] Agregar alias de tiempo: `sudo apt update && sudo apt upgrade -y`

### 7. Entender el filesystem
- [ ] Tus archivos Linux viven en `~/` (dentro de WSL)
- [ ] Los discos de Windows se montan en `/mnt/c/`, `/mnt/d/`, etc.
- [ ] Desde Windows, tu home Linux está en: `\\wsl$\Ubuntu\home\tu-usuario`
- [ ] ⚠️ **Regla de oro**: trabajá en `~/` (Linux), no en `/mnt/c/` — es más rápido y no corrompe permisos

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
| `wsl` no reconoce el comando | Windows viejo → habilitar features manualmente (paso 2) |
| "Please enable the Virtual Machine Platform" | Habilitar VirtualMachinePlatform y reiniciar |
| Distro muestra VERSION 1 | `wsl --set-version Ubuntu 2` |
| Lentitud en `/mnt/c/` | No trabajes en `/mnt/c/`. Mové el proyecto a `~/` |
| Sin internet dentro de WSL | Reiniciar WSL: `wsl --shutdown` y volver a entrar. Revisar proxies corporativos |
| Memoria: WSL consume mucha RAM | Crear `%UserProfile%\.wslconfig` y limitar: `[wsl2] memory=8GB` |

---

## Recursos

- [Microsoft Docs — Instalar WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Microsoft Docs — Configuración avanzada (.wslconfig)](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)