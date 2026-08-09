# Setup — Web APIs

> **Tópico**: 7 — Frontend Core
> **Objetivo**: usar las APIs del navegador que todo fullstack conoce — `fetch`, `localStorage`, `IntersectionObserver` y `navigator.geolocation` — con mini ejemplos que corren en tu página.
> **Prerequisito**: JavaScript (`setup-javascript.md`).

---

## ¿Por qué las Web APIs?

El navegador expone decenas de APIs listas para usar, sin instalar nada: red, almacenamiento, observadores, geolocalización. Son la interfaz entre tu JS y las capacidades del navegador. Cuando más adelante consumas tu propio backend con `fetch`, o persistás el tema de una app con `localStorage`, no vas a instalar ninguna librería: ya está ahí.

---

## Checklist

### 1. `fetch` — GET con manejo de `res.ok`

`fetch` hace requests HTTP desde el cliente. El gotcha clásico: **no lanza error en 404/500** — solo en fallo de red. Siempre chequeá `res.ok`.

```js
// app.js (o un bloque aparte en tu página)
async function cargarPosts() {
  try {
    const res = await fetch('https://jsonplaceholder.typicode.com/posts');
    if (!res.ok) {
      throw new Error(`HTTP ${res.status}: ${res.statusText}`);
    }
    const posts = await res.json();
    console.table(posts.slice(0, 5));
  } catch (err) {
    console.error('Fallo el fetch:', err);
  }
}

cargarPosts();
```

- `await` SOLO dentro de funciones `async`, y el error se atrapa con `try/catch` (o `.catch()`).
- `res.json()` devuelve OTRA Promise → necesita su propio `await`.

- [ ] `cargarPosts()` devuelve datos en la consola y maneja el caso `!res.ok`

### 2. `localStorage` — persistir la preferencia de tema

`localStorage` guarda key-values en el navegador del usuario (¡nunca secretos ahí!). Perfecto para preferencias. Ejemplo de tema claro/oscuro:

```html
<button id="toggle-tema" type="button">Cambiar tema</button>
<div id="app">Hola, mundo con tema!</div>
```

```css
body.dark {
  background-color: #0f172a;
  color: #f8fafc;
}
```

```js
const botonTema = document.querySelector('#toggle-tema');

// 1. Al arrancar, restaurá lo guardado
const temaGuardado = localStorage.getItem('tema');
if (temaGuardado === 'dark') {
  document.body.classList.add('dark');
}

// 2. Al hacer clic, guardá el cambio
botonTema.addEventListener('click', () => {
  const esDark = document.body.classList.toggle('dark');
  localStorage.setItem('tema', esDark ? 'dark' : 'light');
});
```

- `localStorage.getItem(key)` lee (devuelve `null` si no existe).
- `localStorage.setItem(key, value)` escribe — valores SIEMPRE strings.
- Cerrar y reabrir la pestaña, o refrescar (F5), sobrevive.

- [ ] El tema elegido sobrevive a recargar la página

### 3. `IntersectionObserver` — lazy loading de imágenes

Observa cuándo un elemento entra al viewport y te avisa. Caso real: cargar imágenes recién cuando el usuario scrollea hasta ellas.

```html
<img data-src="foto-1.jpg" alt="Ilustración de ejemplo" width="600" height="400">
<img data-src="foto-2.jpg" alt="Otra ilustración" width="600" height="400">
```

> Poner `width` y `height` evita saltos de layout. El `src` real se asigna cuando la imagen es visible.

```js
const imagenes = document.querySelectorAll('img[data-src]');

const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (!entry.isIntersecting) return;          // todavía no se ve

    const img = entry.target;
    img.src = img.dataset.src;                  // asigno el src real
    img.removeAttribute('data-src');
    observer.unobserve(img);                    // dejo de observarla
  });
}, { rootMargin: '200px' });                    // cargá 200px antes

imagenes.forEach((img) => observer.observe(img));
```

Sin imagenes reales a mano, el efecto se nota igual con `console.log(img.dataset.src)` en lugar de asignar el `src`: vas a ver cómo se "cargan" solo al hacer scroll. Y una vez que aprendas this, el "infinite scroll" es agregar más items cuando el observador detecta el último.

- [ ] Las imágenes cargan su `src` recién cuando scrolleás hasta ellas

### 4. `navigator.geolocation` — posición del usuario (opcional)

Pide permiso al usuario y devuelve su posición:

```js
function mostrarPosicion() {
  if (!navigator.geolocation) {
    console.warn('Tu navegador no soporta geolocalización');
    return;
  }

  navigator.geolocation.getCurrentPosition(
    (pos) => {
      console.log('Latitud:', pos.coords.latitude);
      console.log('Longitud:', pos.coords.longitude);
    },
    (err) => console.warn('Error de geolocalización:', err.message)
  );
}

// Ejecutala a pedido: document.querySelector('#mi-ubicacion').addEventListener('click', mostrarPosicion)
```

> Requiere que la página se sirva por HTTPS (o `localhost`). Si corrés el archivo por `file://` puede no funcionar — por eso es OPCIONAL acá.

---

## Verificación

```text
1. Elegís el tema oscuro, refrescás con F5 y sigue oscuro (localStorage).
2. Scrolleás la página y las imágenes cargan "en el momento" (IntersectionObserver).
```

**Si la preferencia de tema sobrevive a la recarga y las imágenes cargan al hacer scroll → Web APIs listas. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `localStorage.getItem` devuelve `null` | Es normal la primera vez: no hay nada guardado todavía. Chequeá el `null` antes de usar |
| `localStorage` te da problema con objetos | Guardá `JSON.stringify(obj)` y leé `JSON.parse(...)` — solo guarda strings |
| Las imágenes cargan todas de una | Si `rootMargin` es chico o las imágenes ya están cerca del viewport, se cargan casi juntas. Aumentá `rootMargin` o ponelas lejos |
| Geolocation no devuelve nada | Necesita HTTPS o `localhost` (por `file://` Chrome lo bloquea) — es opcional por esto |
| El scroll no dispara el observer | Verificá que el elemento tenga alto real (las imágenes sin `src` pueden tener altura 0: pone `width`/`height`) |

---

## Recursos

- [MDN — Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN — Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [MDN — Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [MDN — Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API)