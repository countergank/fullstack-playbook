# Setup — Sistemas de Estilos: CSS Modules, Tailwind y styled-components

> **Tópico**: 8 — Frameworks & Herramientas Frontend
> **Objetivo**: usar las tres estrategias de estilado del concept 8.7 — CSS Modules, Tailwind v4 y styled-components — conviviendo en la misma página React, para que elijás con fundamento.
> **Prerequisito**: proyecto Vite + React (`setup-vite-react.md`). Esta guía supone que borraste el CSS default del template.

---

## ¿Por qué tres estrategias?

CSS puro en una app React choca con un problema: los nombres de clase son globales y se pisan. Cada proyecto resuelve el escopeo distinto. CSS Modules lo hashea por archivo, Tailwind lo resuelve por convención (utility-first), styled-components lo lleva a JS (CSS-in-JS). Acá las probás las tres en un mismo `App.tsx` para que veas qué se siente cada una.

---

## Checklist

### 1. CSS Modules — un botón con clases escopadas

Creá la hoja de estilos con sufijo `.module.css` y un componente que la importe:

```css
/* src/components/Button.module.css */
.button {
  border: none;
  border-radius: 6px;
  padding: 0.5rem 1rem;
  cursor: pointer;
  background: #3b82f6;
  color: #fff;
}
```

```tsx
// src/components/Button.tsx
import styles from './Button.module.css';

export default function Button() {
  return <button className={styles.button}>Button con CSS Module</button>;
}
```

- El nombre de clase se escopa con un hash: `.button` en el CSS se convierte en algo como `Button_button__k3y1x` en el DOM. Un `.button` de otro componente jamás choca.
- No hay anidamiento: importás la hoja como objeto y usás `styles.<clase>`.

- [ ] `Button.tsx` se estila con `styles.button` y la clase generada lleva hash

### 2. Tailwind (v4) — instalar y configurar con Vite

```bash
npm install tailwindcss @tailwindcss/vite
```

> **Ojo con la versión**: la config difiere entre v3 y v4. Con v4 + Vite solo se importa `tailwindcss` y el plugin hace el resto — no hace falta `tailwind.config.js` ni las directivas `@tailwind base/components/utilities` de la v3 (ni PostCSS).

Registrá el plugin de Vite:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

Y en el CSS principal, la directiva de import:

```css
/* src/index.css — reemplazá TODO el contenido del template por esta línea */
@import "tailwindcss";
```

- [ ] `tailwindcss` + `@tailwindcss/vite` instalados, plugin en `vite.config.ts` y el `@import` en `index.css`

### 3. Tailwind — un badge con clases utility

Nada de CSS separado: clases utilitarias directo en el JSX (utility-first, concept 8.7):

```tsx
// src/components/Badge.tsx
export default function Badge({ label }: { label: string }) {
  return (
    <span className="inline-flex items-center rounded-full bg-blue-100 px-3 py-1 text-sm font-medium text-blue-700">
      {label}
    </span>
  );
}
```

- `inline-flex`, `rounded-full`, `bg-blue-100`... son tokens de Tailwind: escala de colores y espaciados consistente, sin reinventar valores.
- El build purga las clases que no usaste → el CSS final queda mínimo.
- [ ] El badge se ve con Tailwind y funciona sin archivo CSS extra

### 4. styled-components — un botón CSS-in-JS tipado

```bash
npm install styled-components
```

```tsx
// src/components/StyledButton.tsx
import styled from 'styled-components';

// el estilo ES un componente; $variant es una prop "transient" (no va al DOM)
const StyledButton = styled.button<{ $variant: 'primary' | 'ghost' }>`
  padding: 0.5rem 1rem;
  border-radius: 6px;
  border: 1px solid #3b82f6;
  cursor: pointer;
  background: ${({ $variant }) => ($variant === 'primary' ? '#3b82f6' : 'transparent')};
  color: ${({ $variant }) => ($variant === 'primary' ? '#fff' : '#3b82f6')};
`;

export default StyledButton;
```

- Template literal con CSS crudo, pero además recibe props y genera clases únicas por componente.
- La `$` en `$variant` es una prop "transient": styled-components la usa para el estilo pero no la reenvía al `<button>` del DOM (con TS la tipás en el genérico `styled.button<...>`).
- [ ] `StyledButton` se estila y el `$variant` cambia la apariencia (probalo con `"primary"` y `"ghost"`)

### 5. Las tres conviven en una misma página

```tsx
// src/App.tsx
import Button from './components/Button';
import Badge from './components/Badge';
import StyledButton from './components/StyledButton';

function App() {
  return (
    <main>
      <h1>Las tres estrategias de estilos</h1>

      <section>
        <h2>CSS Modules</h2>
        <Button />
      </section>

      <section>
        <h2>Tailwind</h2>
        <Badge label="Tailwind v4" />
      </section>

      <section>
        <h2>styled-components</h2>
        <StyledButton $variant="primary">Guardar</StyledButton>
        <StyledButton $variant="ghost">Cancelar</StyledButton>
      </section>
    </main>
  );
}

export default App;
```

- Cada estrategia se ve correcta a la vez: CSS Modules escopea por archivo, Tailwind inyecta utilidades, styled-components genera estilos por componente.

- [ ] En una misma página conviven un CSS Module, utilidades de Tailwind y un styled-component

### 6. Cómo elegir (tl;dr)

| | CSS Modules | Tailwind | styled-components |
|---|---|---|---|
| Escopeo | Automático | Convención de clases | Por componente |
| Velocidad de dev | Media (escribís CSS) | Alta (utility inline) | Alta (CSS en JSX) |
| Curva de aprendizaje | Baja (CSS puro) | Media (aprender clases) | Media (CSS-in-JS) |
| Bundle final | Por uso | Purge automático | Runtime JS extra |
| Team estandarizado | Depende de cada uno | Tokens únicos | Convención del equipo |

- [ ] Sabés argumentar qué estrategia usarías para un proyecto nuevo

---

## Verificación

```bash
cd ~/proyectos/frontend-frameworks/mi-app
npm run dev         # http://localhost:5173
```

```text
1. En la misma página ves los 3 bloques estilados correctamente: botón de CSS Modules, badge de Tailwind, botones de styled-components.
2. Inspeccionando el DOM, cada estrategia generó su mecanismo propio (clases con hash, utilidades de Tailwind, estilos inyectados).
3. El build no se rompe: npm run build termina OK.
```

**Si las TRES estrategias conviven en una misma página y cada una se ve correcta → sistemas de estilos listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Tailwind no genera estilos | Verificá que el `@import "tailwindcss";` esté en `index.css` y el plugin `tailwindcss()` en `vite.config.ts`. En v4 no hay `tailwind.config.js` |
| Clases de Tailwind y CSS Modules se pisan | Son independientes: Tailwind es global, CSS Modules escopea. Si algo se pisa, es porque reutilizás el MISMO nombre en los DOS estilos |
| styled-components recibe `$variant` como atributo HTML | La `$` evita pasar la prop al DOM: usa `$variant`, no `variant` |
| `styled.button<{ $variant: ... }>` da error de tipos | El genérico tipa las props del estilo; asegurate de pasar la prop al render (`<StyledButton $variant="primary">`) |
| Los estilos de un bloque pisan a los de otro | El CSS global (Tailwind, `index.css`) aplica a todo; los estilos escopados (CSS Modules, styled-components) solo a su componente. Revisá orden de import y nombres repetidos |
| Uso las tres en un team real | Elegí UNA y mantenela; conviven en esta guía para demostrar, pero la consistencia es la regla (concept 8.7) |

---

## Recursos

- [CSS Modules — reference](https://github.com/css-modules/css-modules)
- [Tailwind CSS — instalación con Vite](https://tailwindcss.com/docs/installation/using-vite)
- [Tailwind CSS — utility classes](https://tailwindcss.com/docs/utility-first)
- [styled-components — basics](https://styled-components.com/docs/basics)