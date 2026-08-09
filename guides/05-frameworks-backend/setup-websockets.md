# Setup — WebSockets con socket.io

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.8)
> **Objetivo**: agregar comunicación en tiempo real a tu app Express con socket.io.
> **Prerequisito**: proyecto Express (`setup-express-project.md`).

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

## Recursos

- [socket.io — Get Started](https://socket.io/docs/v4/)
- [socket.io — Rooms](https://socket.io/docs/v4/rooms/)