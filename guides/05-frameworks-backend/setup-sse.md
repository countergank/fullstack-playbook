# Setup — Server-Sent Events (SSE)

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.9)
> **Objetivo**: agregar un endpoint SSE a tu app Express para enviar eventos del servidor al cliente en tiempo real (one-way).
> **Prerequisito**: proyecto Express (`04-backend/setup-express-project.md`).

---

## Checklist

### 1. Crear el endpoint SSE

```ts
// src/modules/events/events.controller.ts
import type { Request, Response } from 'express';

export function streamEvents(req: Request, res: Response) {
  // Headers obligatorios para SSE
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.flushHeaders();

  // Enviar un evento cada 5 segundos
  const interval = setInterval(() => {
    const data = JSON.stringify({
      time: new Date().toISOString(),
      random: Math.floor(Math.random() * 100)
    });
    res.write(`data: ${data}\n\n`);
  }, 5000);

  // Limpiar cuando el cliente se desconecta
  req.on('close', () => {
    clearInterval(interval);
    res.end();
  });
}
```

### 2. Agregar la ruta

```ts
// src/modules/events/events.routes.ts
import { Router } from 'express';
import { streamEvents } from './events.controller.js';

export const eventsRoutes = Router();
eventsRoutes.get('/', streamEvents);
```

```ts
// src/app.ts — registrar la ruta
app.use('/api/events', eventsRoutes);
```

### 3. Probar desde el navegador (cero librerías)

```html
<!-- test-sse.html -->
<script>
  const source = new EventSource('http://localhost:3000/api/events');

  source.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Evento recibido:', data);
  };

  source.onerror = () => console.log('Conexión perdida, reintentando...');
</script>
```

- [ ] Abrir `test-sse.html` en el navegador
- [ ] La consola muestra eventos cada 5 segundos
- [ ] Cerrar la pestaña → el servidor limpia el intervalo

### 4. Enviar eventos con nombre (event types)

```ts
// Evento con nombre — el cliente puede escuchar tipos específicos
res.write(`event: notification\n`);
res.write(`data: ${JSON.stringify({ message: 'Nuevo mensaje' })}\n\n`);

// Cliente:
source.addEventListener('notification', (e) => {
  console.log('Notificación:', JSON.parse(e.data));
});
```

### 5. Emitir SSE desde un controller (ej: al crear un usuario)

```ts
// src/modules/users/users.controller.ts
import { Response } from 'express';

// Mantener una lista de clientes SSE conectados
const clients = new Set<Response>();

export function streamEvents(req: Request, res: Response) {
  // ... (mismo código del paso 1)
  clients.add(res);
  req.on('close', () => clients.delete(res));
}

// Emitir a todos los clientes conectados
export function broadcast(event: string, data: unknown) {
  for (const client of clients) {
    client.write(`event: ${event}\n`);
    client.write(`data: ${JSON.stringify(data)}\n\n`);
  }
}
```

```ts
// En el controller de users:
import { broadcast } from '../events/events.controller.js';

export async function createUser(req: Request, res: Response) {
  const user = await userService.create(req.body);
  broadcast('user:created', user);
  res.status(201).json({ data: user });
}
```

---

## Verificación

```bash
npm run dev
# Abrir test-sse.html en el navegador
# La consola debe mostrar eventos cada 5 segundos
# Cerrar pestaña → no hay errores en el server
```

**Si el navegador recibe eventos cada 5 segundos → SSE listo. ✅**

---

## SSE vs WebSockets: ¿cuándo usar cada uno?

| | SSE | WebSockets |
|---|---|---|
| Dirección | Servidor → Cliente | Bidireccional |
| Protocolo | HTTP | WS (upgrade desde HTTP) |
| Reconexión | Automática (navegador) | Manual (librería) |
| Librería cliente | `EventSource` (nativo) | socket.io o similar |
| Mejor para | Notificaciones, feeds, progreso | Chat, juegos, colaboración en vivo |

---

## Recursos

- [MDN — Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
- [MDN — EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)