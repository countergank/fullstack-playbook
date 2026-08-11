# Setup — VS Code + WSL

> **Tópico**: 2 — Fundamentos de Programación (sección 2.2)
> **Objetivo**: un editor configurado para desarrollo profesional, conectado a WSL con las extensiones esenciales.
> **Prerequisito**: WSL instalado (`setup-wsl.md`).

---

## ¿Por qué VS Code?

VS Code es el editor más usado en el mundo del desarrollo web (74% según la encuesta de Stack Overflow 2024). No es el más ligero ni el más potente, pero es el que mejor equilibrio ofrece entre rendimiento, extensibilidad, y comunidad. Su integración nativa con WSL, Git, terminals, y language servers lo convierte en el centro de operaciones del desarrollador full stack. No necesitás un IDE pesado como IntelliJ para JavaScript/TypeScript — VS Code con las extensiones correctas cubre el 95% de los casos de uso.

---

## Checklist

### 1. Instalar VS Code
- [ ] Descargar e instalar VS Code desde [code.visualstudio.com](https://code.visualstudio.com/)
- [ ] Durante la instalación marcar "Add to PATH" (para poder usar `code` desde la terminal)

### 2. Conectar VS Code a WSL (lo más importante)
- [ ] Instalar la extensión **"WSL"** (Microsoft, Remote - WSL)
- [ ] En VS Code en Windows: `Ctrl+Shift+P` → "WSL: Connect to WSL"
- [ ] O abrir directamente desde la terminal WSL: `code .`
- [ ] El indicador verde en la esquina inferior izquierda dice "WSL: Ubuntu" → estás conectado
- [ ] Abrir la carpeta del proyecto Y abrir la terminal integrada (`Ctrl+ñ`) → confirmar que el shell es zsh de WSL

### 3. Extensiones esenciales (base)
| Extensión | Para qué |
|-----------|----------|
| **WSL** (Microsoft) | Conectar VS Code al entorno Linux |
| **GitLens** | Blame, historial, comparación de ramas en el editor |
| **ESLint** | Detectar problemas de código en tiempo real |
| **Prettier** | Formateo consistente de código |
| **Error Lens** | Mostrar errores inline (sobre la línea) |
| **Path Intellisense** | Autocompletar rutas de archivos/imports |
| **Material Icon Theme** | Íconos de archivos para reconocer tipos de un vistazo |

### 4. Extensiones por lenguaje (instalar cuando llegues al tópico)
| Tópico | Extensiones |
|--------|-------------|
| Frontend (7-8) | Auto Rename Tag, CSS Peek, Live Server (opcional), Tailwind CSS IntelliSense (si usás Tailwind) |
| Backend (4-5) | Thunder Client (o REST Client) para probar APIs, Prisma (extensión oficial para el ORM) |
| Testing (10) | Jest Runner / Vitest Runner, Playwright Test Runner |
| Docker (9) | Docker (Microsoft) — ver contenedores, logs, compose |

### 5. Configuración personal (`settings.json`)
`Ctrl+Shift+P` → "Preferences: Open User Settings (JSON)":
```jsonc
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "editor.renderWhitespace": "boundary",
  "editor.rulers": [100],
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "workbench.colorTheme": "Default Dark Modern",
  "terminal.integrated.defaultProfile.linux": "zsh"
}
```

### 6. Atajos que deberías dominar
| Atajo | Acción |
|-------|--------|
| `Ctrl+P` | Ir a cualquier archivo |
| `Ctrl+Shift+P` | Panel de comandos |
| `Ctrl+Shift+F` | Buscar en todo el proyecto |
| `Ctrl+B` | Toggle sidebar |
| `Ctrl+ñ` / `` Ctrl+` `` | Terminal integrada |
| `F2` | Renombrar símbolo en todo el proyecto |
| `Shift+Alt+F` | Formatear documento |
| `Ctrl+Shift+E` | Explorador de archivos |
| `Ctrl+G` | Ir a línea específica |

### 7. WSL + Extensions: no todo se instala igual
- Las extensiones del editor (tema, iconos) se instalan en **Windows**.
- Los **language servers y Linters** (ESLint, Prettier, Tallwind) deben instalarse en **WSL** (Remote) para que usen las herramientas del proyecto. VS Code lo maneja solo: al conectarte a WSL, las extensiones "Remote" te las sugiere él.

---

## Verificación

```bash
# Desde VS Code con WSL conectado:
code --version          # VS Code responde
git status              # usa la identidad/config de WSL
node --version          # si instalaste Node, responde
```
- [ ] La terminal integrada de VS Code abre con zsh en `/home/tu-usuario`
- [ ] Abrir archivo, hacer algo mal (ej: faltar un `;`) → Error Lens muestra el error inline
- [ ] Escribir código y guardar → Prettier formatea al guardar

**Si la terminal integrada es zsh dentro de WSL → VS Code conectado. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `code` no se reconoce en terminal | Reiniciar VS Code; o manualmente agregar VS Code al PATH |
| VS Code no ve Node instalado en WSL | Abriste VS Code desde Windows pero no conectado a WSL. Usá `code .` desde la terminal WSL |
| Extensiones instaladas pero no funcionan en WSL | Instalarlas "in WSL" por separado (VS Code lo sugiere) |
| Fuerte consumes RAM | Desactivar extensiones que no uses; VS Code separa extensiones por remote. |

---

## Recursos

- [VS Code — WSL docs](https://code.visualstudio.com/docs/remote/wsl)
- [VS Code — Keybindings](https://code.visualstudio.com/docs/getstarted/keybindings)

## Preguntas de repaso

- **P:** ¿Por qué las extensiones de language server deben instalarse "en WSL" y no en Windows?
  **R:** Porque el language server necesita acceder a las herramientas del proyecto (Node, TypeScript compiler, etc.) que viven dentro del entorno Linux de WSL. Si se instala solo en Windows, no encuentra los binaries ni el node_modules.

- **P:** ¿Qué hace `"editor.formatOnSave": true` y por qué es útil?
  **R:** Formatea automáticamente el archivo cada vez que guardás. Combinado con Prettier, elimina discusiones de estilo en code reviews y mantiene consistencia en todo el equipo.

- **P:** ¿Cómo abrís VS Code conectado a WSL desde la terminal?
  **R:** Con el comando `code .` ejecutado desde dentro de la terminal WSL. VS Code detecta que está en WSL y se conecta automáticamente al remote.

- **P:** ¿Qué extensión te muestra el blame de cada línea directamente en el editor?
  **R:** GitLens. Muestra quién modificó cada línea, en qué commit y cuándo, sin necesidad de correr `git blame` en la terminal.

- **P:** ¿Por qué es importante `"editor.rulers": [100]`?
  **R:** Muestra una línea vertical a los 100 caracteres, ayudando a mantener las líneas dentro del límite recomendado. Líneas muy largas son difíciles de leer en diffs y code reviews.

- **P:** ¿Qué atajo usás para buscar un archivo por nombre en todo el proyecto?
  **R:** `Ctrl+P`. Escribís parte del nombre y VS Code lo encuentra instantáneamente usando fuzzy matching.