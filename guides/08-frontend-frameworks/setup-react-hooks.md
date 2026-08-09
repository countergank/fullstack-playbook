# Setup — Los 6 Hooks Esenciales de React

> **Tópico**: 8 — Frameworks & Herramientas Frontend
> **Objetivo**: construir una SPA de gestión de tareas que use TODOS los hooks esenciales (`useState`, `useEffect`, `useRef`, `useMemo`, `useCallback`, `useReducer`) con un uso real y justificado para cada uno.
> **Prerequisito**: el proyecto Vite + React del `setup-vite-react.md` (`~/proyectos/frontend-frameworks/mi-app`).

---

## ¿Por qué esta mini app?

Los hooks son LA mitad de React (concept 8.1). El error clásico es memorizar la firma sin entender cuándo y por qué. Esta guía no usa ejemplos tipo "contador": construís UNA app de tareas donde cada hook resuelve un problema concreto. Si entendés el porqué de cada uno acá, leés cualquier componente real sin miedo.

---

## Checklist

### 1. `useState` — el estado local del formulario

El `useState` es el default para estado local de UN componente. En esta app lo usamos para el texto del input (lo que estás escribiendo ahora mismo) y para el flag de carga. Si la lista fuera simple (solo agregar), un `useState` de array alcanzaría:

```tsx
const [texto, setTexto] = useState('');
const [loading, setLoading] = useState(true);
```

Cuando el estado cambia, el componente se re-renderiza. Esa es toda la magia.

- [ ] El componente declara estado local con `useState` para el input y el flag de carga

### 2. `useReducer` — la lista de tareas con reglas

La lista tiene TRANSICIONES (agregar, completar, borrar, reemplazar desde el server). Ese tipo de estado con varias mutaciones es donde `useReducer` gana: toda la lógica queda en una función pura `(state, action) → newState`, testeable sin UI.

```tsx
// src/types.ts
export interface Task {
  id: number;
  title: string;
  done: boolean;
}
```

```tsx
// src/reducer.ts
import type { Task } from './types';

export type Action =
  | { type: 'set'; tasks: Task[] }
  | { type: 'add'; title: string }
  | { type: 'toggle'; id: number }
  | { type: 'remove'; id: number };

export function reducer(state: Task[], action: Action): Task[] {
  switch (action.type) {
    case 'set':
      return action.tasks;
    case 'add':
      return [{ id: Date.now(), title: action.title, done: false }, ...state];
    case 'toggle':
      return state.map((t) => (t.id === action.id ? { ...t, done: !t.done } : t));
    case 'remove':
      return state.filter((t) => t.id !== action.id);
  }
}
```

- No mutás `state`: cada case devuelve un array NUEVO (inmutabilidad).
- `Date.now()` como id es para la demo; en producción usás el id del server.
- [ ] El reducer cubre `set`, `add`, `toggle` y `remove`

### 3. `useEffect` + cleanup — cargar datos del server al montar

El `useEffect` maneja efectos secundarios (fetch, timers, suscripciones). Acá cargamos tareas de la API falsa de jsonplaceholder UNA vez al montar y limpiamos con el flag `cancelled` (patrón del concept 8.1):

```tsx
// src/App.tsx
import { useCallback, useEffect, useMemo, useReducer, useRef, useState } from 'react';
import { reducer } from './reducer';
import TaskItem from './TaskItem';

interface ApiTodo {
  id: number;
  title: string;
  completed: boolean;
}

function App() {
  const [tasks, dispatch] = useReducer(reducer, []);
  const [newTask, setNewTask] = useState('');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;                          // evita setState tras desmontar

    fetch('https://jsonplaceholder.typicode.com/todos')
      .then((res) => res.json())
      .then((todos: ApiTodo[]) => {
        if (cancelled) return;
        dispatch({
          type: 'set',
          tasks: todos.slice(0, 5).map((t) => ({
            id: t.id,
            title: t.title,
            done: t.completed,
          })),
        });
        setLoading(false);
      })
      .catch(() => {
        if (!cancelled) setLoading(false);
      });

    return () => { cancelled = true; };             // cleanup: corre al desmontar
  }, []);                                           // [] = UNA vez al montar
```

**Por qué `[]` = una vez**: React guarda el efecto en memoria y SOLO lo re-ejecuta si cambia alguna dependencia del array. Con `[]` vacío no hay ninguna que pueda cambiar → corre una vez al montar (y su cleanup al desmontar). Si la lista dependiera de un `userId`, lo pondrías en el array y el efecto re-correría con cada cambio de ese id.

**El loop infinito (advertencia explícita)**: un `setState` dentro de un `useEffect` sin el array de deps = bucle:

```tsx
useEffect(() => {
  setCounter(counter + 1);   // BUG: setState en un efecto SIN deps
});                           // corre en CADA render → setState → re-render → efecto → ...
```

El efecto corre después de cada render; el setState dispara otro render; el efecto vuelve a correr... hasta que la pestaña revienta. La cura casi siempre es definir las deps exactas (o no meter el setState ahí). Regla: si ponés algo en el efecto, el array de deps tiene que reflejar las variables que usás.

- [ ] El fetch corre una sola vez y el flag `cancelled` evita setState tras desmontar
- [ ] Entendés POR QUÉ `[]` = una vez y por qué un efecto sin deps con setState es un loop infinito

### 4. `useRef` — foco del input sin re-render

`useRef` da una referencia mutable que NO provoca re-render al cambiar `.current`. Perfecto para apuntar a un nodo del DOM (autofocus):

```tsx
const inputRef = useRef<HTMLInputElement>(null);

useEffect(() => {
  inputRef.current?.focus();          // foco inicial al abrir la app
}, []);
```

```tsx
<input
  ref={inputRef}
  type="text"
  value={newTask}
  onChange={(e) => setNewTask(e.target.value)}
  placeholder="Escribí una tarea..."
/>
```

> **Clave**: si querés que un cambio RE-RENDERICE la UI usás `useState`, no `useRef`. `inputRef.current = algo` cambia el valor pero no redibuja el componente. Por eso el foco no produce loops: es cambio de DOM, no de estado.

- [ ] El input tiene foco al abrir la app gracias a `useRef`
- [ ] Sabés que `useRef` no re-renderiza (a diferencia de `useState`)

### 5. `useMemo` — el contador de completadas

El contador es un valor DERIVADO del estado: no hace falta guardarlo con setState; se calcula y se memiza. `useMemo` memoriza el RESULTADO y solo recalcula si cambian las deps:

```tsx
const completedCount = useMemo(
  () => tasks.filter((t) => t.done).length,
  [tasks],                                   // solo recalcula si cambia tasks
);
```

Si la lista fuera de 10.000 items ordenándola/filtrándola en cada render, `useMemo` te ahorra el trabajo repetido. Acá es liviano pero muestra el patrón.

> **No envuelvas todo en `useMemo`**: memorizar también tiene costo. Usalo cuando el cálculo es pesado. Primero medí, después optimizá (concept 8.1).

- [ ] El contador de completadas solo se recalcula cuando cambia `tasks`

### 6. `useCallback` + `React.memo` — el toggle estable hacia un hijo memoizado

`useCallback` memoriza una FUNCIÓN: mantiene la misma identidad entre renders. Eso permite que `React.memo` iguale las props y no re-renderice el hijo cuando NADA cambió.

```tsx
const handleToggle = useCallback(
  (id: number) => dispatch({ type: 'toggle', id }),   // dispara una action
  [],                                                  // identidad estable
);
```

El hijo es memoizado:

```tsx
// src/TaskItem.tsx
import { memo } from 'react';
import type { Task } from './types';

const TaskItem = memo(function TaskItem({
  task,
  onToggle,
}: {
  task: Task;
  onToggle: (id: number) => void;
}) {
  return (
    <li>
      <input
        type="checkbox"
        checked={task.done}
        onChange={() => onToggle(task.id)}
      />
      {task.title}
    </li>
  );
});

export default TaskItem;
```

- `React.memo` compara las props con `===`. Si `handleToggle` fuera una función nueva en cada render del padre, la comparación fallaría siempre y el memo no serviría.
- Con `useCallback([])` las props del hijo son estables → solo se re-renderiza el item cuyo `task` cambió.

- [ ] `handleToggle` es estable (misma identidad entre renders) y `TaskItem` es `React.memo`

### 7. Juntar todo en `App.tsx`

```tsx
import { useCallback, useEffect, useMemo, useReducer, useRef, useState } from 'react';
import { reducer } from './reducer';
import TaskItem from './TaskItem';

interface ApiTodo {
  id: number;
  title: string;
  completed: boolean;
}

function App() {
  const [tasks, dispatch] = useReducer(reducer, []);
  const [newTask, setNewTask] = useState('');
  const [loading, setLoading] = useState(true);
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    let cancelled = false;

    fetch('https://jsonplaceholder.typicode.com/todos')
      .then((res) => res.json())
      .then((todos: ApiTodo[]) => {
        if (cancelled) return;
        dispatch({
          type: 'set',
          tasks: todos.slice(0, 5).map((t) => ({
            id: t.id,
            title: t.title,
            done: t.completed,
          })),
        });
        setLoading(false);
      })
      .catch(() => {
        if (!cancelled) setLoading(false);
      });

    return () => { cancelled = true; };
  }, []);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  const completedCount = useMemo(
    () => tasks.filter((t) => t.done).length,
    [tasks],
  );

  const handleToggle = useCallback(
    (id: number) => dispatch({ type: 'toggle', id }),
    [],
  );

  function handleAdd() {
    const title = newTask.trim();
    if (!title) return;
    dispatch({ type: 'add', title });
    setNewTask('');
    inputRef.current?.focus();
  }

  return (
    <main>
      <h1>Gestor de tareas — todos los hooks</h1>

      <form onSubmit={(e) => { e.preventDefault(); handleAdd(); }}>
        <input
          ref={inputRef}
          type="text"
          value={newTask}
          onChange={(e) => setNewTask(e.target.value)}
          placeholder="Escribí una tarea..."
        />
        <button type="submit">Agregar</button>
      </form>

      <p>Completadas: {completedCount} de {tasks.length}</p>

      {loading && <p>Cargando tareas del servidor...</p>}

      <ul>
        {tasks.map((task) => (
          <TaskItem key={task.id} task={task} onToggle={handleToggle} />
        ))}
      </ul>
    </main>
  );
}

export default App;
```

- [ ] La app completa usa los 6 hooks con un rol claro cada uno

### 8. Las Rules of Hooks

- Hooks SOLO al nivel superior del componente: nunca dentro de `if`, loops ni funciones anidadas.
- Hooks SOLO en componentes React o custom hooks. No en funciones comunes.
- El orden de los hooks debe ser el mismo en cada render (por eso no pueden ir condicionados).

```tsx
function MalEjemplo() {
  if (Math.random() > 0.5) {
    const [x] = useState(0);   // PROHIBIDO: hook condicionado
  }
  return <div />;
}
```

- [ ] Tus hooks están al tope del componente, sin condiciones

---

## Verificación

```bash
cd ~/proyectos/frontend-frameworks/mi-app
npm run dev         # http://localhost:5173
```

```text
1. Al abrir la app se cargan 5 tareas remotas (jsonplaceholder) y el input ya tiene el foco.
2. Escribís una tarea nueva y "Agregar" la mete al tope de la lista.
3. Tildás/des-tildás checkboxes y el contador "Completadas: X de N" se actualiza.
4. La consola no muestra el loop infinito de "Maximum update depth" ni warnings de dependencies.
```

**Si la carga remota funciona, agregás/completás/borrás, el input arranca enfocado y el contador se actualiza → los 6 hooks esenciales listos. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Maximum update depth exceeded" | SetState dentro de un `useEffect` sin deps → loop infinito. Definí el array de deps o sacá el setState del efecto |
| El fetch corre mil veces | Falta el array `[]` (o está mal definido). Sin deps el efecto corre en cada render |
| El contador de completadas no cambia | El `useMemo` tiene deps viejas: recordá que `[tasks]` es lo que dispara el recálculo |
| El foco no aparece | `useRef` apunta al DOM pero el input debe tener `ref={inputRef}`. Chequeá que el ref esté conectado |
| Redux/Hooks rules warning | Hooks dentro de un `if` o de un `map` violan las Rules of Hooks: movelos al nivel superior |
| El checkbox funciona pero el hijo se re-renderiza igual | El callback que pasa al hijo debe estar envolto en `useCallback` para que `React.memo` pueda comparar props estables |

---

## Recursos

- [React — Reference de hooks](https://react.dev/reference/react/hooks)
- [React — useState](https://react.dev/reference/react/useState)
- [React — useEffect](https://react.dev/reference/react/useEffect)
- [React — useRef](https://react.dev/reference/react/useRef)
- [React — useMemo](https://react.dev/reference/react/useMemo)
- [React — useCallback](https://react.dev/reference/react/useCallback)
- [React — useReducer](https://react.dev/reference/react/useReducer)
- [JSONPlaceholder — fake REST API](https://jsonplaceholder.typicode.com/)