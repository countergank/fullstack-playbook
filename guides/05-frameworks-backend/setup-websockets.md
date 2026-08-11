# Setup — WebSockets con socket.io

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.8)
> **Objetivo**: agregar comunicación en tiempo real a tu app Express con socket.io.
> **Prerequisito**: proyecto Express (`setup-express-project.md`).

---

## ¿Por qué WebSockets?

HTTP es request-response: el cliente pregunta, el servidor responde. Pero hay casos donde el servidor necesita enviar datos sin que el cliente los pida — notificaciones en tiempo real, chat, dashboards en vivo. WebSockets mantienen una conexión abierta y bidireccional. socket.io simplifica esto: agrega reconexión automática, rooms, y fallback a HTTP polling si WebSocket no está disponible.

---

## Checklist

### 1. Instalar socket.io

```bash
npm install socket.io
```

### 2. Configurar socket.io con Express

```ts
// src/app.ts
import { createServer } from 'node:http';
import { Server } from 'socket.io';

const app = express();
const httpServer = createServer(app);

const io = new Server(httpServer, {
  cors: { origin: process.env.CORS_ORIGIN || 'http://localhost:5173' }
});

io.on('connection', (socket) => {
  console.log(`Client connected: ${socket.id}`);

  socket.on('disconnect', () => {
    console.log(`Client disconnected: ${socket.id}`);
  });
});

httpServer.listen(env.PORT, () => {
  console.log(`Server on http://localhost:${env.PORT}`);
});

export { io };
```

### 3. Emitir eventos desde un controller

```ts
// src/modules/users/users.controller.ts
import { io } from '../../app.js';
import type { Request, Response } from 'express';

export async function createUser(req: Request, res: Response) {
  const user = await userService.create(req.body);

  // Notificar a todos los clientes conectados
  io.emit('user:created', user);

  res.status(201).json({ data: user });
}
```

### 4. Usar rooms (aislar eventos por contexto)

```ts
io.on('connection', (socket) => {
  // Unir al usuario a su room personal
  socket.on('user:join', (userId: string) => {
    socket.join(`user:${userId}`);
  });

  // Emitir solo a ese usuario
  io.to(`user:${userId}`).emit('notification', {
    message: 'Tarea completada',
  });
});
```

### 5. Probar desde el navegador (HTML simple)

```html
<!-- test.html — abrí este archivo en el navegador -->
<script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
<script>
  const socket = io('http://localhost:3000');
  socket.on('user:created', (user) => console.log('Nuevo usuario:', user));
  socket.on('notification', (data) => console.log('Notificación:', data));
</script>
```

### 6. Emitir desde el worker (BullMQ)

- [ ] socket.io instalado y configurado con Express
- [ ] Al hacer POST a `/api/users`, el navegador recibe evento `user:created`
- [ ] Las rooms funcionan: `socket.join()` y `io.to().emit()` aíslan mensajes
- [ ] El worker de BullMQ emite eventos WebSocket al completar jobs

```ts
// src/workers/emailWorker.ts
import { io } from '../app.js';

worker.on('completed', (job) => {
  if (job.name === 'welcome-email') {
    io.emit('email:sent', { to: job.data.to });
  }
});
```

---

## Verificación

```bash
# Abrí test.html en el navegador
# Hacé un POST a /api/users → la consola del navegador loguea "Nuevo usuario: ..."
```

**Si el navegador recibe eventos en tiempo real → WebSockets listos. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `Client connected` nunca aparece en el log | Verificá que el CORS del servidor coincida con el origen del cliente (`origin: 'http://localhost:5173'`). Si no coincide, socket.io rechaza la conexión. |
| El cliente no recibe eventos emitidos desde el controller | Asegurate de que el `io` exportado desde `app.ts` es la misma instancia que usa el controller. Un `new Server()` separado no comparte conexiones. |
| `socket.join()` no funciona como esperado | Las rooms son case-sensitive. Verificá que el nombre de la room es exactamente el mismo en `join` y en `io.to()`. |
| Reconexión infinita del cliente | Si el servidor se cae, socket.io reintenta indefinidamente. Configurá `reconnectionAttempts` y `reconnectionDelay` en el cliente para limitar reintentos. |

---

## Preguntas de repaso

- **P:** ¿Qué diferencia hay entre HTTP y WebSockets?
  **R:** HTTP es request-response (el cliente siempre inicia). WebSockets es bidireccional y persistente — ambos lados pueden enviar datos en cualquier momento.

- **P:** ¿Para qué sirven las rooms en socket.io?
  **R:** Para agrupar sockets y emitir mensajes a un subconjunto de clientes. Ej: todos los usuarios de un proyecto reciben notificaciones, pero no los de otros proyectos.

- **P:** ¿Qué hace `socket.broadcast.emit()`?
  **R:** Envía un evento a todos los clientes conectados EXCEPTO al socket que originó el evento. Útil para chat: todos ven el mensaje menos quien lo envió.

- **P:** ¿Por qué usar socket.io en lugar del WebSocket nativo del navegador?
  **R:** Porque agrega reconexión automática, fallback a HTTP polling, rooms, namespaces, y manejo de heartbeats — todo lo que tendrías que implementar manualmente.

- **P:** ¿Cómo se integra WebSockets con una API REST?
  **R:** REST maneja CRUD y operaciones síncronas. WebSockets notifican cambios en tiempo real. Ej: POST crea un recurso (REST) + `io.emit()` notifica a clientes conectados (WS).

---

## Recursos

- [socket.io — Get Started](https://socket.io/docs/v4/)
- [socket.io — Rooms](https://socket.io/docs/v4/rooms/)