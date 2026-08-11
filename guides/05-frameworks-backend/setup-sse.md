# Setup — Server-Sent Events (SSE)

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.9)
> **Objetivo**: agregar un endpoint SSE a tu app Express para enviar eventos del servidor al cliente en tiempo real (one-way).
> **Prerequisito**: proyecto Express (`04-backend/setup-express-project.md`).

---

## ¿Por qué SSE?

WebSockets son bidireccionales pero requieren una librería extra, infraestructura adicional, y manejo manual de reconexión. SSE es más simple: usa HTTP común, el navegador se reconecta solo, y no necesitás ninguna librería en el cliente. Si solo necesitás que el servidor envíe datos al cliente (notificaciones, feeds, progreso), SSE es la solución más liviana.

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

## Problemas comunes

| Problema | Solución |
|----------|----------|
| El navegador no recibe eventos | Verificá los 3 headers obligatorios: `Content-Type: text/event-stream`, `Cache-Control: no-cache`, `Connection: keep-alive`. Sin ellos, el navegador no trata la respuesta como stream. |
| El servidor acumula conexiones abiertas | Cada cliente SSE mantiene una conexión HTTP abierta. Usá un Set para trackear clientes y limpiá en `req.on('close')`. Sin cleanup, el server se satura. |
| Los eventos no se reconectan después de un restart del server | `EventSource` reconecta automáticamente, pero pierde el estado. Usá `Last-Event-ID` para reanudar desde donde quedó. |
| Proxy inverso (nginx) corta la conexión | Configurá `proxy_buffering off` y `proxy_read_timeout 86400s` en nginx para SSE. Los proxies suelen bufferizar y cortar streams largos. |

---

## Preguntas de repaso

- **P:** ¿Qué es SSE y en qué se diferencia de WebSockets?
  **R:** SSE (Server-Sent Events) es unidireccional (solo servidor → cliente). WebSockets es bidireccional. SSE usa HTTP común, WebSockets requiere un protocolo diferente.

- **P:** ¿Qué headers son obligatorios para un endpoint SSE?
  **R:** `Content-Type: text/event-stream`, `Cache-Control: no-cache`, y `Connection: keep-alive`.

- **P:** ¿Por qué SSE se reconecta automáticamente?
  **R:** Porque `EventSource` es una API nativa del navegador que implementa reconexión con backoff. WebSocket nativo no la incluye.

- **P:** ¿Cuándo elegir SSE sobre WebSockets?
  **R:** Cuando solo necesitás notificaciones del servidor al cliente (feeds, progreso, logs) y querés la solución más simple sin librerías ni infraestructura extra.

- **P:** ¿Cómo se envían eventos con nombre en SSE?
  **R:** Con `event: <nombre>\ndata: <payload>\n\n`. El cliente escucha con `source.addEventListener('nombre', callback)`.

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