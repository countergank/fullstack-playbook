# Setup — Mongoose en tu Proyecto Express

> **Tópico**: 5 — Frameworks & Herramientas Backend (sección 5.5)
> **Objetivo**: integrar Mongoose en tu proyecto Express, definir schemas y modelos, y conectar a MongoDB en Docker.
> **Prerequisito**: proyecto Express (`04-backend/setup-express-project.md`), MongoDB corriendo en Docker (`06-databases/setup-mongodb.md`).

---

## Checklist

### 1. Instalar Mongoose

```bash
npm install mongoose
```

### 2. Conectar a MongoDB

```ts
// src/shared/db/mongoose.ts
import mongoose from 'mongoose';

const MONGO_URL = process.env.MONGO_URL || 'mongodb://localhost:27017/fullstack_dev';

export async function connectMongo() {
  await mongoose.connect(MONGO_URL);
  console.log('Connected to MongoDB');
}
```

Llamar `connectMongo()` al inicio de `src/app.ts`:
```ts
import { connectMongo } from './shared/db/mongoose.js';

await connectMongo();
app.listen(env.PORT, () => console.log(`Server on http://localhost:${env.PORT}`));
```

### 3. Definir un modelo

```ts
// src/modules/users/user.model.ts
import mongoose from 'mongoose';

const userSchema = new mongoose.Schema({
  email:    { type: String, required: true, unique: true, lowercase: true },
  name:     { type: String, trim: true },
  password: { type: String, required: true, select: false },
  role:     { type: String, enum: ['user', 'admin'], default: 'user' }
}, { timestamps: true });

// Hook: hashear password antes de guardar
userSchema.pre('save', async function(next) {
  if (this.isModified('password')) {
    const bcrypt = await import('bcryptjs');
    this.password = await bcrypt.hash(this.password, 12);
  }
  next();
});

export const User = mongoose.model('User', userSchema);
```

### 4. Usar el modelo en un repository

```ts
// src/modules/users/users.repository.ts
import { User } from './user.model.js';

export const userRepository = {
  findByEmail: (email: string) => User.findOne({ email }),
  findById: (id: string) => User.findById(id),
  create: (data: Record<string, unknown>) => User.create(data),
  list: (skip: number, limit: number) => User.find().skip(skip).limit(limit),
};
```

### 5. Agregar `MONGO_URL` al `.env`

```
MONGO_URL=mongodb://localhost:27017/fullstack_dev
```

---

## Verificación

```bash
npm run dev
# El log debe mostrar "Connected to MongoDB"

# Crear un usuario de prueba:
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Lean", "email": "lean@ejemplo.com", "password": "secreto123"}'
```

**Si el server conecta a MongoDB y el POST crea un documento → Mongoose listo. ✅**

---

## Mongoose vs Prisma

| | Prisma (SQL) | Mongoose (MongoDB) |
|---|---|---|
| Schema | `schema.prisma` (declarativo) | `new Schema({...})` (JS) |
| Tipos | Generados automáticamente | Manual o con `@types/mongoose` |
| Migraciones | `prisma migrate dev` | No hay — schemaless por naturaleza |
| Relaciones | `@relation` + joins | `populate()` (similar a join) |
| Hooks | No tiene (usar service layer) | `pre('save')`, `post('find')` |
| Mejor para | Datos relacionales, type-safety | Documentos flexibles, schemas que cambian |

---

## Recursos

- [Mongoose — Getting Started](https://mongoosejs.com/docs/index.html)
- [Mongoose — Schemas](https://mongoosejs.com/docs/guide.html)