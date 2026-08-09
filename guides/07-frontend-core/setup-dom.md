# Setup — Playground del DOM

> **Tópico**: 7 — Frontend Core
> **Objetivo**: construir una mini app vanilla que crea y borra items de una lista en vivo, manipulando el DOM sin librerías.
> **Prerequisito**: JavaScript (`setup-javascript.md`).

---

## ¿Por qué manipular el DOM a mano?

Porque el navegador arma un árbol en memoria con cada etiqueta HTML como nodo — eso es el DOM — y tu JS lo modifica para cambiar la página sin recargar. Agregar items, borrarlos, escuchar clics y manejar eventos con delegación son las operaciones que después los frameworks (tópico 8) abrazan por vos. Si nunca hiciste el costo manual, no vas a entender por qué existen.

---

## Checklist

### 1. HTML base del playground

```html
<section id="lista-app">
  <h2>Lista de tareas</h2>

  <input type="text" id="nuevo-item" placeholder="Escribí una tarea...">
  <button id="agregar" type="button">Agregar</button>

  <ul id="lista"></ul>
</section>

<script type="module" src="app.js"></script>
```

- [ ] La estructura input + botón + lista existe en el HTML

### 2. Seleccionar elementos con `querySelector`

```js
// app.js
const input = document.querySelector('#nuevo-item');
const boton = document.querySelector('#agregar');
const lista = document.querySelector('#lista');
```

`querySelector('#...')` devuelve el primer elemento que matchea el selector CSS. `querySelectorAll` devuelve todos (en un NodeList iterable).

### 3. Crear y agregar con `document.createElement` + `append`

```js
function agregarItem() {
  const texto = input.value.trim();
  if (!texto) return;

  const li = document.createElement('li');
  li.textContent = texto;

  const botonBorrar = document.createElement('button');
  botonBorrar.textContent = 'Borrar';
  li.append(botonBorrar);

  lista.append(li);
  input.value = '';
  input.focus();
}

boton.addEventListener('click', agregarItem);
```

- `createElement('li')` crea el nodo, `textContent` le pone texto, `append` lo une al árbol.
- `append` acepta nodos Y strings. `.focus()` recupera el cursor en el input.
- El input `type="button"` en el HTML hace que no recargue la página como submit.

- [ ] Clic en "Agregar" crea un item nuevo en la lista sin recargar la página

### 4. Event delegation — un listener para TODOS los items

Cada `<li>` lleva un botón "Borrar", pero no le ponemos un listener a cada uno (y menos a los que se crean después). Un solo listener en el `ul` captura el clic en cualquiera de sus hijos gracias al **event bubbling** (el evento sube: `li` → `ul`):

```js
lista.addEventListener('click', (event) => {
  const botonBorrar = event.target.closest('button');
  if (!botonBorrar) return;                 // el clic no fue en un botón
  botonBorrar.closest('li').remove();       // borro el li que contiene al botón
});
```

- `event.target` → el elemento que recibió el clic real (puede ser el item, el botón o un texto).
- `closest('button')` → sube el árbol hasta encontrar el botón (o devuelve `null`).
- `remove()` → saca el nodo del DOM.

- [ ] Borrar items funciona con un solo listener sobre la lista

### 5. `textContent` vs `innerHTML` — el riesgo XSS

```js
// Peligroso: si el texto tiene HTML, se RENDERIZA (ej: "<img onerror=...>") — XSS
li.innerHTML = texto;

// Seguro: solo texto, no interpreta nada
li.textContent = texto;
```

Regla: si el contenido puede venir del usuario (un input, una API), usá **siempre `textContent`**. `innerHTML` solo cuando construís HTML vos programa a programa con datos confiable.

- [ ] El código usa `textContent` para el texto del usuario

### 6. Probar con items creados DESPUÉS

La delegación brilla acá: agregá dos items, borrá con los botones y verifica que los items nuevos (agregados después de cargar la página) también se borran — aunque su `click` nunca fue "registrado" individualmente.

- [ ] Items agregados DESPUÉS se borran con la delegación

---

## Verificación

```text
1. Agregás items desde el input y aparecen en la lista SIN recargar la página.
2. Borrás items con el botón "Borrar" y desaparecen.
3. Agregás uno nuevo DESPUÉS de cargar la página y también se borra (delegación funcionando).
```

**Si agregar y borrar funcionan sin recarga y los items post-carga caen bajo la delegación → DOM listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| La página se recarga al tocar "Agregar" | El botón debe ser `type="button"` o estar fuera de un `form` (si no, actúa como submit) |
| Borrar items solo funciona en los primeros | Ese es el problema que resuelve la delegación: un listener en el `ul`, no uno por `li` |
| Aparecen textos con HTML literal | Usabas `innerHTML` con texto de input → cambialo por `textContent` |
| `event.target` no es el `li` | El target es el elemento exacto del clic; por eso se usa `closest('button')` antes |

---

## Recursos

- [MDN — Document Object Model](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)
- [MDN — Event bubbling y delegation](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events)
- [MDN — textContent vs innerHTML (XSS)](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)