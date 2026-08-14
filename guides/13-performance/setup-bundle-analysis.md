# Setup — Bundle Analysis

> **Tópico**: 13 — Performance & Optimización (sección 13.4)
> **Objetivo**: visualizar qué pesa en tu bundle JS con `rollup-plugin-visualizer` (Vite) o `webpack-bundle-analyzer`, detectar dependencias infladas y reducirlas con tree-shaking y code splitting.
> **Prerequisito**: un proyecto frontend con bundler (Vite o webpack) y un `package.json` propio. Concepto 13 completo (al menos 13.3 y 13.4).

---

## ¿Por qué?

No podés optimizar lo que no medís. El bundle analysis te muestra, en un mapa visual (treemap), cuánto pesa cada dependencia de tu JS. Casi siempre aparece la misma sorpresa: trajiste una librería de cientos de KB para usar una sola función, o `lodash`/`moment` enteros cuando con una importación puntual alcanza. Este reporte es el paso previo obligatorio a cualquier optimización: de ahí salen las decisiones de code splitting y de eliminar dependencias. Optimizar sin el reporte es disparar a ciegas.

---

## Checklist

### 1. Instalar el visualizador

- [ ] En Vite (usa Rollup debajo), instalá el plugin:

```bash
npm install -D rollup-plugin-visualizer
```

- [ ] En webpack, instalá el analyzer:

```bash
npm install -D webpack-bundle-analyzer
```

### 2. Configurar el plugin

- [ ] En Vite, agregá el plugin a `vite.config.js`:

```js
// vite.config.js
import { defineConfig } from 'vite';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      open: true,        // abre el reporte en el navegador al terminar el build
      gzipSize: true,    // mostrá el tamaño real que viaja por la red (gzip)
      brotliSize: true,
      filename: 'stats.html', // reporte HTML interactivo
    }),
  ],
});
```

- [ ] En webpack, agregá el plugin a `webpack.config.js`:

```js
// webpack.config.js
const BundleAnalyzerPlugin =
  require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static', // genera report.html en vez de levantar un server
      openAnalyzer: false,
    }),
  ],
};
```

### 3. Generar el reporte

- [ ] Corré un build de producción (nunca el dev server, que no minifica ni tree-shakea igual):

```bash
npm run build
```

- [ ] Abrí el reporte generado (`stats.html` en Vite, `report.html` en webpack) y mirá el treemap.

### 4. Leer el treemap

- [ ] Identificá los rectángulos más grandes: son las dependencias que más pesan.
- [ ] Buscá librerías importadas enteras (`lodash`, `date-fns`, `moment`) y reducilas:

```js
// MAL: trae todo lodash (~70kb gzip) por una función
import { debounce } from 'lodash';

// BIEN: solo esa función (el tree-shaking elimina el resto)
import debounce from 'lodash/debounce';
```

- [ ] Si ves `moment`, reemplazalo por `date-fns` (tree-shakeable) o por el `Intl` nativo.

### 5. Reducir el chunk inicial

- [ ] Mové lo que no sea del primer viewport a un chunk aparte con `import()` dinámico:

```js
// El parser pesado solo se descarga cuando el usuario sube un archivo
async function procesarArchivo(file) {
  const { parse } = await import('papaparse'); // ~100kb fuera del bundle inicial
  return parse(await file.text());
}
```

### 6. Verificar el resultado

- [ ] Regenerá el reporte y confirmá que el bundle total (y el chunk inicial) bajó (ver sección Verificación).

---

## Verificación

```bash
# Tamaño del bundle antes y después (gzip es lo que viaja por red)
npm run build
ls -lh dist/assets/        # mirá los .js y .css generados
gzip -c dist/assets/index-*.js | wc -c   # bytes gzip del bundle principal

# También podés ver el reporte en modo consola (Vite)
npx vite build --report
```

**Si el treemap se abre, ves cada dependencia con su peso (gzip), y tras eliminar una librería inflada el bundle gzip bajó → bundle analysis correcto. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| El reporte muestra la librería entera aunque importás una función | Usá la importación puntual (`lodash/debounce`) o verificá que la librería sea tree-shakeable (ESM); si es CommonJS, no se puede tree-shakear |
| No ves el reporte después del build | `open: true` requiere entorno gráfico; usá `filename` y abrilo manual, o `analyzerMode: 'static'` en webpack |
| El tamaño en el reporte es mucho mayor al que "viaja" | Estás mirando el tamaño sin comprimir; activá `gzipSize: true` para ver el real |
| `moment` aparece y no lo querés tocar | Migrá a `date-fns` por partes, importando solo las funciones que usás |
| Dos versiones de `react` en el treemap | Alguna dependencia trae su copia de React; alineá versiones con `npm dedupe` o `overrides` en `package.json` |
| El chunk inicial sigue gigante después de mover cosas | Verificá que el `import()` dinámico esté en un punto que no se ejecute al arrancar (por ejemplo, dentro de un handler, no en el top-level) |

---

## Recursos

- [rollup-plugin-visualizer — npm](https://www.npmjs.com/package/rollup-plugin-visualizer)
- [webpack-bundle-analyzer — npm](https://www.npmjs.com/package/webpack-bundle-analyzer)
- [Vite — Build options](https://vite.dev/config/build-options)
- [MDN — Dynamic import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import)

## Preguntas de repaso

- **P:** ¿Por qué el bundle analysis va ANTES de optimizar?
  **R:** Porque te da datos reales (qué pesa y cuánto) en vez de intuición. Sin medir, optimizás a ciegas y probablemente en el lugar equivocado.

- **P:** ¿Qué herramienta usás con Vite y por qué no `webpack-bundle-analyzer`?
  **R:** Con Vite usás `rollup-plugin-visualizer` porque Vite usa Rollup debajo, y el plugin se integra con su pipeline. `webpack-bundle-analyzer` es para proyectos con webpack.

- **P:** ¿Por qué activás `gzipSize` en el reporte?
  **R:** Porque el navegador recibe los archivos comprimidos con gzip/brotli; el tamaño gzip es el que realmente viaja por la red, no el tamaño en disco.

- **P:** ¿Qué significa que una librería sea "tree-shakeable" y por qué importa?
  **R:** Que el bundler puede eliminar el código que no usás (exporta en ESM). Si no es tree-shakeable (CommonJS), importar una función trae la librería entera.

- **P:** ¿Por qué generás el reporte con un build de producción y no con el dev server?
  **R:** Porque el build de producción minifica, tree-shakea y genera los chunks reales que el usuario descarga; el dev server no refleja el tamaño final.

- **P:** ¿Qué problema indica ver dos versiones de `react` en el treemap?
  **R:** Que una dependencia trae su propia copia de React (versión incompatible), duplicando tamaño y pudiendo causar bugs. Se resuelve alineando las versiones a una sola.
