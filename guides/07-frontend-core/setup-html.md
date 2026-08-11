# Setup — HTML Semántico

> **Tópico**: 7 — Frontend Core
> **Objetivo**: página HTML5 con estructura semántica válida: `header`, `nav`, `main`, `article`, `section`, `aside`, `footer`, jerarquía de headings con un único `<h1>` y un formulario accesible.
> **Prerequisito**: navegador moderno (Chrome recomendado).

---

## ¿Por qué HTML semántico?

Porque el HTML no es solo lo que se VE, es lo que se ENTIENDE: un buscador, un lector de pantalla y otro developer leen tu markup sin mirar una línea de CSS. `<header>`, `<nav>` y `<article>` le dicen a todos qué ES cada cosa; un `<div>` genérico no dice nada. Acá nace tu página del tópico 7: la que después estilás con CSS y le colgás JavaScript. Sin un esqueleto correcto, todo lo demás se apoya en arena. Nada de frameworks ni bundlers todavía — HTML plano que abrís con doble clic.

---

## Checklist

### 1. Crear la carpeta del proyecto

Usá una carpeta genérica de práctica:

```bash
mkdir -p ~/proyectos/frontend-core
cd ~/proyectos/frontend-core
```

Todas las guías del tópico 7 van a vivir acá.

### 2. Crear `index.html` con la estructura base HTML5

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Frontend Core — Mi blog</title>
</head>
<body>
</body>
</html>
```

- `<!DOCTYPE html>` → le dice al navegador que es HTML5 (modo estándar, sin quirks).
- `<html lang="es">` → idioma de la página (clave para lectores de pantalla y SEO).
- `<meta charset="UTF-8">` → para que las tildes y caracteres se muestren bien.
- `<meta name="viewport">` → responsive en móviles.

- [ ] `index.html` guardado con la estructura base

### 3. Agregar los elementos semánticos

Reemplazá el `<body>` vacío por la estructura con significado:

```html
<body>
  <header>
    <nav>
      <a href="#">Inicio</a>
      <a href="#articulos">Artículos</a>
      <a href="#contacto">Contacto</a>
    </nav>
  </header>

  <main>
    <h1>Mi blog semántico</h1>

    <section id="articulos">
      <h2>Artículos recientes</h2>
      <article>
        <h3>Por qué importa el HTML semántico</h3>
        <p>Un lector de pantalla anuncia la estructura con los headings y cada sección con su etiqueta.</p>
      </article>
      <article>
        <h3>Un solo h1 por página</h3>
        <p>La jerarquía h1 → h2 → h3 es la estructura del documento, no un tema de tamaño de fuente.</p>
      </article>
    </section>

    <aside>
      <h2>Sobre mi</h2>
      <p>Estudiante fullstack, aprendiendo los fundamentos antes de tocar frameworks.</p>
    </aside>
  </main>

  <footer>
    <p>&copy; 2026 Mi blog semántico</p>
  </footer>
</body>
```

- [ ] La página tiene `header`, `nav`, `main`, `section`, `article`, `aside` y `footer`

### 4. Verificar la jerarquía de headings

- **Un solo `<h1>`** por página (acá es "Mi blog semántico").
- Los headings siguen la jerarquía sin saltos: `h1` → `h2` (`Artículos recientes`, `Sobre mi`) → `h3` (los artículos).
- El tamaño de fuente lo decide CSS, los headings son estructura del documento.

- [ ] Hay exactamente un `<h1>` y la jerarquía no saltea niveles

### 5. Agregar un formulario accesible

Dentro de `<main>`, después del `<aside>`:

```html
<section id="contacto">
  <h2>Contacto</h2>

  <form>
    <fieldset>
      <legend>Datos de contacto</legend>

      <div>
        <label for="nombre">Nombre</label>
        <input type="text" id="nombre" name="nombre">
      </div>

      <div>
        <label for="email">Email</label>
        <input type="email" id="email" name="email">
      </div>

      <button type="submit">Enviar</button>
    </fieldset>
  </form>
</section>
```

- Cada `<input>` tiene su `<label>` con `for` apuntando al `id` del input: clic en el label enfoca el input y el lector de pantalla anuncia el campo.
- `<fieldset>` + `<legend>` agrupan un conjunto de campos ("Datos de contacto" es la etiqueta del grupo).

- [ ] Todos los inputs tienen `<label for>` y el grupo está en `<fieldset>`/`<legend>`

### 6. Abrir en el navegador

```bash
xdg-open index.html
```

O directamente doble clic en el archivo. NO necesitás servidor ni npm: es HTML plano.

- [ ] `index.html` renderiza en el navegador

---

## Verificación

```bash
# 1. La página abre y muestra el contenido con la jerarquía correcta
xdg-open index.html

# 2. Validá con el validador Nu de W3C:
#    https://validator.w3.org/nu/ → pegá el código o subí el archivo
```

**Si el validador de W3C informa "Document checking completed. No errors or warnings to show." → HTML semántico listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| El validador marca "Stray start tag" o errores de estructura | Revisá que las etiquetas abran y cierren en pares; HTML es anidado, no entrelazado |
| "Multiple H1" en el validador | Dejá UN solo `<h1>`; los títulos de sección van en `h2`, los sub-títulos en `h3` |
| El `label` no funciona al hacer clic | El `for` del label debe coincidir EXACTO con el `id` del input |
| Todo es `<div>` por costumbre | Preguntate "¿qué ES este contenido?" — si hay un elemento semántico, úsalo |

---

## Recursos

- [MDN — HTML Semantics](https://developer.mozilla.org/en-US/docs/Glossary/Semantics)
- [MDN — Elementos de sección](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements)
- [W3C — Nu HTML Validator](https://validator.w3.org/nu/)

## Preguntas de repaso

- **P:** ¿Por qué `<header>` es mejor que `<div class="cabecera">`?
  **R:** `<header>` comunica semánticamente que es la cabecera de la página o sección. Los buscadores, lectores de pantalla y otros developers entienden qué ES sin mirar CSS. Un `<div>` es genérico y no dice nada.

- **P:** ¿Cuántos `<h1>` debe tener una página y qué pasa si ponés más?
  **R:** Exactamente uno. Múltiples `<h1>` confunden la jerarquía del documento, penalizan SEO y hacen que los lectores de pantalla anuncien mal la estructura.

- **P:** ¿Cómo asociás un `<label>` a un `<input>` correctamente?
  **R:** Usando `for` en el label que apunta al `id` del input: `<label for="email">Email</label>` + `<input id="email">`. Esto permite clic en el label para enfocar el input y anuncia el campo en lectores de pantalla.

- **P:** ¿Qué diferencia hay entre `<section>` y `<article>`?
  **R:** `<section>` agrupa contenido temático con su propio heading (h2-h6). `<article>` envuelve contenido autocontenido que tiene sentido por sí solo (un post, una noticia). Un article puede tener sections dentro.

- **P:** ¿Para qué sirve `<fieldset>` y `<legend>` en un formulario?
  **R:** `<fieldset>` agrupa campos relacionados y `<legend>` pone un título al grupo. Los lectores de pantalla anuncian el legend cuando el usuario navega por los campos del grupo.