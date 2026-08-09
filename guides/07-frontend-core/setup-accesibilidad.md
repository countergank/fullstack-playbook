# Setup — Herramientas de Accesibilidad (a11y)

> **Tópico**: 7 — Frontend Core
> **Objetivo**: instalar y usar las herramientas que miden accesibilidad — axe DevTools y Lighthouse — y dejar tu página del tópico 7 con accesibilidad en verde.
> **Prerequisito**: `index.html` del tópico 7 (`setup-html.md` + `setup-css-moderno.md`).

---

## ¿Por qué herramientas de accesibilidad?

La accesibilidad la usa TODO el mundo, incluida ~1 de cada 6 personas con discapacidad. No es un checkbox al final: es semántica, es contraste, es teclado. Y como es difícil juzgarla "a ojo", existen herramientas que la miden: **axe DevTools** (extensión de navegador que detecta violaciones concretas) y **Lighthouse** (auditoría automatizada de Chrome). Las dos juntas detectan el 80% de los problemas.

---

## Checklist

### 1. Instalar axe DevTools

- Andá a [deque.com/axe/browser-extensions](https://www.deque.com/axe/browser-extensions/).
- Instalá la extensión para Chrome (o Firefox) desde la Chrome Web Store.
- Fijala en la barra de herramientas.

- [ ] La extensión axe aparece en la barra del navegador

### 2. Correr axe sobre tu página del tópico 7

1. Abrí tu `index.html` (la que creaste en `setup-html.md`).
2. Clic en el ícono de axe → **Scan ALL of my page**.
3. El resultado agrupa las violaciones por **serious / moderate / minor**.

- [ ] axe reporta **0 violaciones serias (serious)** en tu página

### 3. Lighthouse — auditoría de Chrome integrada

1. Abrí DevTools con **F12** → pestaña **Lighthouse**.
2. Elegí las categorías: marca **Accessibility** (y Performance, para ir tanteando).
3. Seleccioná "Mobile" → **Analyze page load**.
4. Esperá el reporte: vas a ver el score de Accessibility y las auditorías falladas con explicación y cómo arreglarlas.

> Lighthouse es una auditoría automatizada: mide contraste, roles, nombres accesibles, jerarquía, etc. Ponele atención a "Contrast" y "Document has a single h1" — suelen aparecer en el tópico 7.

- [ ] Lighthouse genera el reporte y ves el score de Accessibility

### 4. Tab Accessibility en DevTools Elements + emulación de focus

1. Abrí DevTools → pestaña **Elements** → seleccioná un elemento.
2. En el panel derecho, pestaña **Accessibility**: te muestra el nombre accesible, el rol y el árbol de accesibilidad (lo que un lector de pantalla "ve").
3. Para emular el foco visible: DevTools → ⋮ (tres puntos) → **More tools** → **Rendering** → tildá **"Emulate a focused page"**. Ahora podés navegar con Tab y ver EXACTAMENTE qué se enfoca.

- [ ] Viste el árbol de accesibilidad de un elemento y la emulación de focus activada

### 5. Checklist manual de a11y

Recorré tu página y verificá punto por punto:

- [ ] Toda la página se navega con **Tab** y el foco es visible (no está oculto)
- [ ] Hay **un solo `<h1>`** y jerarquía coherente
- [ ] Todo `<input>` tiene su `<label>` (con `for` → `id`)
- [ ] Toda imagen significativa tiene `alt` descriptivo (y `alt=""` las decorativas)
- [ ] El contraste de texto sobre fondo es ≥ **4.5:1** (Lighthouse lo mide)

> El contraste de 4.5:1 es el mínimo WCAG AA para texto normal. Si el Lighthouse marca "Background and foreground colors do not have a sufficient contrast ratio", cambiá el color de `--color-text` / `--color-bg` en tu CSS.

### 6. Corregir lo que reporten y re-escanear

Aplicá los fixes que señalen axe y Lighthouse (los colores de contrast se corrigen en `:root` de `styles.css`; los labels y alt, en el HTML) y volvé a escanear hasta que ambos queden limpios.

- [ ] Después de corregir, re-escaneás y las violaciones bajaron

---

## Verificación

```text
1. Lighthouse (F12 → Lighthouse → Accessibility) da un score ≥ 90.
2. axe DevTools reporta 0 violaciones serias (serious) en tu página del tópico 7.
3. Navegás toda la página con Tab y ves el foco en cada elemento interactivo.
```

**Si Lighthouse da ≥ 90 en Accessibility y axe reporta 0 violaciones serias → accesibilidad lista. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Calculador de contraste | Usá [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) para elegir un par que pase 4.5:1 |
| "Element has no alt text" (Lighthouse/axe) | Agregá `alt` a las imágenes informativas y `alt=""` a las decorativas |
| "Form elements do not have associated labels" | Cada input necesita su `<label for="...">` apuntando al `id` |
| "Document has more than one h1" | Dejá un solo `<h1>` por página |
| El foco "no se ve" al navegar con Tab | No uses `outline: none` sin reemplazarlo por un `:focus-visible` visible |

---

## Recursos

- [axe DevTools — Browser extensions](https://www.deque.com/axe/browser-extensions/)
- [Chrome DevTools — Accessibility reference](https://developer.chrome.com/docs/devtools/accessibility/reference/)
- [MDN — Accessibilidad](https://developer.mozilla.org/es/docs/Web/Accessibility)
- [WebAIM — Contrast Checker](https://webaim.org/resources/contrastchecker/)