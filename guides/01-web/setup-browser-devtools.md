# Setup — Browser DevTools

> **Tópico**: 7 — Frontend Core
> **Objetivo**: dominar las cuatro pestañas que más vas a usar en todo el stack — Network, Console, Application y Sources.
> **Prerequisito**: navegador moderno (Chrome recomendado).

---

## ¿Por qué DevTools?

Todo el tópico 7 —JavaScript, `fetch`, DOM, Web APIs— se depura y se entiende adentro de DevTools. Network es donde vas a VER cada request de tu frontend contra el backend; Console es donde aterrizan tus logs y errores; Application, donde vive la data que persiste el navegador. Hacete amigo de estas pestañas ANTES de escribir lógica: la herramienta es parte del oficio, no un extra.

---

## Checklist

### 1. Abrir DevTools

- En Chrome o Firefox: **F12** (o clic derecho → **Inspeccionar**).
- **Ctrl+Shift+J** abre directo en Console; **Ctrl+Shift+I** abre el panel general.
- Con la ventana chica, algunos paneles se ocultan; ampliá la ventana o usá el botón de "devices/responsive".

- [ ] DevTools abre con F12

### 2. Network tab — ver los requests

1. Abrí la pestaña **Network**.
2. Entrá a una página con tráfico (ej: [jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users) o cualquier web).
3. **Recargá con F5**: los requests aparecen en vivo.
   - ¿Vacío? Tildá **Preserve log** y recargá.

**Filtros y datos útiles:**
- **Filtrar por tipo**: clic en **XHR/Fetch** para ver SOLO llamadas de API (nada de imágenes ni CSS).
- **Timing / Waterfall**: clic en un request → pestaña **Timing**: ves dónde se fue el tiempo (DNS, conexión, espera del servidor, descarga).
- **Contenido**: pestaña **Response** (lo que devolvió la API) y **Headers** (status, CORS, etc.).

**Throttling (simular red lenta):**
- Arriba en Network, el dropdown "No throttling" → elegí **Slow 3G** o **Fast 3G**.
- Recargá: ahora VES la latencia real de una API en cámara lenta.
- **Si no ves nada en Network → recargá la página (F5):** DevTools muestra solo lo que pasa DESPUÉS de abrirse.

- [ ] Interceptás un request XHR/Fetch y ves su Timing (waterfall) y su Response

### 3. Console — logs, tablas y `debugger`

1. Abrí la pestaña **Console**.
2. Corré estas líneas directo:

```js
console.log('Hola, devtools');
console.table([
  { name: 'Lean', role: 'dev' },
  { name: 'Ana', role: 'dev' }
]);
```

3. **Filtros de nivel**: arriba, los dropdowns/checkboxes de *Info / Warnings / Errors* — tildá y destildá para quedarte solo con lo que te importa.

4. El statement `debugger` pausa donde lo pongas:

```js
function suma(a, b) {
  debugger;                 // la ejecución se pausa ACÁ
  return a + b;
}

suma(2, 3);
```

- [ ] La consola muestra tu `console.table` y el `debugger` pausa la ejecución

### 4. Application — almacenamiento y cookies

1. Abrí **Application**.
2. En el panel izquierdo vas a ver:
   - **Local Storage** y **Session Storage**: los key-values que persiste cada dominio.
   - **Cookies**: las cookies de la página (ojito: nunca datos sensibles acá).
   - **Service Workers**: si tu app tiene uno, aparece acá (lo vas a tocar con PWA más adelante).

**Probalo en la consola:**

```js
localStorage.setItem('tema', 'dark');
localStorage.getItem('tema');       // "dark"
sessionStorage.setItem('pref', 'x');
```

- Refrescá con **F5** y verificá que `tema` sigue en Local Storage (al contrario que Session Storage, que muere con la pestaña).

- [ ] Guardás y leés un valor en Local Storage que sobrevive al refresh

### 5. Sources — breakpoints y watch

1. Abrí **Sources**.
2. En el panel **Page** (izquierda), navegá hasta un archivo JS (ej: `app.js` de tu proyecto del tópico 7).
3. Clic en el número de línea → se marca en azul (breakpoint).
4. Recargá la página: la ejecución se pausa en esa línea.
5. Controles de depuración (arriba, a la derecha):
   - **Step over** (⏭): ejecuta la línea sin entrar en funciones.
   - **Step into** (⏬): entra en la función que se está llamando.
   - **Step out** (⏫): sale de la función actual.
6. **Watch** (panel derecho): clic en **+** y escribí una expresión, ej: `document.title` — se reevalúa en cada paso.

- [ ] Pausás la ejecución en un breakpoint, caminás con step over/into y una expresión en Watch muestra un valor

---

## Verificación

```text
1. En Network: aplicás throttling "Fast 3G", recargás y ves un request XHR con su waterfall (Timing).
2. En Sources: ponés un breakpoint en un script, recargás, la ejecución se pausa y una expresión en Watch muestra un valor.
```

**Si interceptás un request con throttling Y pausás en un breakpoint viendo variables en Watch → DevTools listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| No aparece nada en Network | Recargá la página (F5) — DevTools solo muestra lo que pasa DESPUÉS de abrirse |
| Faltan pestañas (Network, Application) | Chrome desactualizado: actualizalo. Con la ventana muy chica también se ocultan |
| No veo las llamadas de API | Filtrá por **XHR/Fetch** y tildá **Preserve log** |
| El `debugger` no pausa | Asegurate de que la línea se ejecute de verdad y que DevTools esté abierto |
| `localStorage` se pasa entre pestañas | Es por diseño (comparte el dominio); `sessionStorage` es el aislado por pestaña |

---

## Recursos

- [Chrome DevTools — Docs](https://developer.chrome.com/docs/devtools/)
- [Chrome DevTools — Network reference](https://developer.chrome.com/docs/devtools/network/reference/)
- [Chrome DevTools — Sources overview](https://developer.chrome.com/docs/devtools/javascript/)

---

## Preguntas de repaso

- **P:** ¿Por qué DevTools solo muestra requests que ocurren DESPUÉS de abrirlo?
  **R:** DevTools intercepta el tráfico de red desde el momento en que se abre; los requests anteriores ya fueron procesados por el navegador y no se registran retroactivamente. Usá "Preserve log" para mantener el historial entre recargas.

- **P:** ¿Cuál es la diferencia entre `localStorage` y `sessionStorage`?
  **R:** `localStorage` persiste entre recargas y cierres del navegador (mismo dominio); `sessionStorage` se borra al cerrar la pestaña. Ambos comparten el mismo API (`setItem`, `getItem`, `removeItem`).

- **P:** ¿Qué pestaña de DevTools usarías para ver cuánto tiempo tarda un request en cada fase (DNS, conexión, espera, descarga)?
  **R:** La pestaña **Network**, seleccionando un request y mirando el tab **Timing** (waterfall), que desglosa el tiempo por fase.

- **P:** ¿Cómo simularías una conexión lenta para probar cómo se comporta tu app en 3G?
  **R:** En la pestaña Network, usá el dropdown de throttling (arriba, donde dice "No throttling") y elegí "Slow 3G" o "Fast 3G". Esto afecta TODOS los requests hasta que lo desactivés.

- **P:** ¿Qué hace el statement `debugger` en JavaScript y qué condición debe cumplirse para que funcione?
  **R:** Pausa la ejecución del código en esa línea, como un breakpoint programático. DevTools debe estar ABIERTO cuando se ejecute esa línea; si está cerrado, el `debugger` se ignora.

- **P:** ¿Por qué una cookie con `HttpOnly` no es accesible desde `document.cookie`?
  **R:** El flag `HttpOnly` le dice al navegador que esa cookie solo se envíe en requests HTTP, no que sea legible por JavaScript. Esto previene que scripts maliciosos (XSS) roben la cookie de sesión.