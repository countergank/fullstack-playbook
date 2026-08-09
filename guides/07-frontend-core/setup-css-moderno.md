# Setup — CSS Moderno

> **Tópico**: 7 — Frontend Core
> **Objetivo**: estilar el `index.html` de la guía anterior con CSS moderno: reset, variables, Flexbox, Grid, media queries mobile-first y tipografía fluida.
> **Prerequisito**: `index.html` del `setup-html.md`.

---

## ¿Por qué CSS moderno?

Porque CSS ya no es "poner colores": Flexbox y Grid arman layouts que antes pedían hacks con floats y position. La regla mental: **Flexbox alinea cosas en UNA dimensión (una fila o una columna), Grid arma la grilla de la página en DOS dimensiones (filas Y columnas)**. Con variables y `clamp()` el CSS queda limpio y mantenible, y mobile-first hace que la página se adapte solo.

---

## Checklist

### 1. Crear `styles.css` y linkearlo

```css
/* styles.css */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
}
```

`box-sizing: border-box` global hace que `width` incluya el padding y el borde (los temidos desbordes se van), y `margin: 0` saca los márgenes por defecto del navegador.

Linkealo en el `<head>` de `index.html`:

```html
<link rel="stylesheet" href="styles.css">
```

- [ ] `styles.css` existe y está linkeado en el `head`

### 2. Definir variables CSS en `:root`

```css
:root {
  --color-primary: #2563eb;
  --color-bg: #f8fafc;
  --color-text: #0f172a;
  --color-surface: #ffffff;
  --spacing: 1rem;
  --radius: 0.5rem;
}

body {
  font-family: system-ui, sans-serif;
  font-size: clamp(1rem, 2vw, 1.25rem);
  line-height: 1.6;
  color: var(--color-text);
  background-color: var(--color-bg);
}
```

Cambiás el color UNA vez en `:root` y todo tu CSS lo hereda vía `var(--...)`.

- [ ] Las variables `--color-*` y `--spacing` están definidas y usadas

### 3. Tipografía fluida con `clamp()`

El `font-size` del `body` ya usa `clamp(1rem, 2vw, 1.25rem)`: mínimo, ideal (relativo al viewport), máximo. Tipografía que escala sin media queries.

```css
h1 { font-size: clamp(2rem, 4vw, 3rem); }
h2 { font-size: clamp(1.5rem, 3vw, 2rem); }
```

- [ ] Los headings usan `clamp()` y unidades relativas (`rem`), no `px`

### 4. Navbar con Flexbox

```css
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--spacing);
  padding: var(--spacing);
  background-color: var(--color-surface);
}

.nav a {
  color: var(--color-primary);
  text-decoration: none;
}
```

- `justify-content: space-between` → separa los links en el eje principal.
- `align-items: center` → centra verticalmente en el eje cruzado.
- `gap` → espaciado SIN márgenes.

En el HTML, la `<nav>` de tu blog:

```html
<nav class="nav">
  <a href="#">Inicio</a>
  <a href="#articulos">Artículos</a>
  <a href="#contacto">Contacto</a>
</nav>
```

- [ ] La navbar alinea logo/links horizontalmente con Flexbox

### 5. Layout de la página con Grid

La regla: cambia estructura según el tamaño del viewport, y un único breakpoint mobile-first:

```css
main {
  display: grid;
  grid-template-columns: 1fr;    /* móvil: una sola columna */
  gap: var(--spacing);
  padding: var(--spacing);
}

@media (min-width: 768px) {
  main {
    grid-template-columns: 200px 1fr 200px;  /* sidebar + contenido flexible + sidebar */
  }
}
```

`200px 1fr 200px` → dos columnas fijas (`aside` y `aside`) y el contenido central flexible (`section`). El `1fr` es "una fracción del espacio restante".

En el HTML, para que las secciones caigan en las columnas correctas:

```html
<main>
  <section id="articulos">...</section>
  <aside>...</aside>
  <aside>...</aside>   <!-- agregar un segundo sidebar opcional -->
</main>
```

> Nota: si tu HTML actual solo tiene una sección y un aside, alcanza con `1fr 200px` — o usá `grid-template-areas` para nombrar zonas. Lo importante ahora es VER que en desktop aparecen columnas y en móvil una.

- [ ] Al achicar la ventana, el layout pasa de 3 columnas a 1

### 6. Verificar el responsive

Con la ventana por debajo de 768px → una columna; por encima → tres columnas. El breakpoint se define en `@media (min-width: 768px)` (mobile-first: base para el celular, el media query SUBE la complejidad).

- [ ] Redimensionás la ventana y el layout cambia en el breakpoint

---

## Verificación

```bash
# Abrí la página y redimensioná la ventana (o modo responsivo F12)
xdg-open index.html
```

**Si el layout pasa de una columna (móvil) a tres columnas (`200px 1fr 200px` en desktop) al cruzar 768px → CSS moderno listo. ✅**

Revisá también que ningún elemento desborde la ventana horizontalmente (resultado del `box-sizing` + `margin: 0`).

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| El link de `styles.css` no aplica nada | Revisá la ruta relativa: `index.html` y `styles.css` están en la misma carpeta |
| La página se desborda horizontalmente | Verificá que `box-sizing: border-box` esté primero y que no uses `width` fija sin flexibilidad |
| El navbar no separa los links | Falta `justify-content: space-between` o los links están en bloques distintos al main axis |
| Grid no cambia con la ventana | El `@media (min-width: 768px)` debe ir DESPUÉS de la regla base, no antes |
| Aplico `--color-x` y no cambia nada | Tipeo del nombre de variable: `var(--color-primario)` ≠ `var(--color-primary)` |

---

## Recursos

- [MDN — Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)
- [MDN — Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-columns)
- [MDN — clamp()](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp)