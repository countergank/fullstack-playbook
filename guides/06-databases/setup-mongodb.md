# Setup — MongoDB (Docker)

> **Tópico**: 6 — Bases de Datos & Persistencia
> **Objetivo**: MongoDB corriendo en Docker, accesible desde tu app Node, listo para usar con Mongoose.
> **Prerequisito**: Docker (`setup-docker.md`).

---

## ¿Por qué MongoDB en Docker?

La base documental más usada con Node. Un `docker compose up -d mongo` y está corriendo. Sin instalar paquetes, sin configurar servicios, sin pelear con versiones.

---

## Checklist

### 1. Agregar MongoDB a tu `compose.yaml`

```yaml
services:
  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongodata:/data/db

volumes:
  mongodata:
```

### 2. Levantar y verificar

```bash
docker compose up -d mongo
docker compose ps

# Verificar que responde (usando mongosh dentro del contenedor)
docker compose exec mongo mongosh --eval "db.runCommand({ ping: 1 })"
```
- [ ] El ping devuelve `{ ok: 1 }`

### 3. Connect string para tu app

```
mongodb://localhost:27017/fullstack_dev
```

### 4. Mongoose (cuando configures tu proyecto backend)

```bash
npm install mongoose
```

```js
import mongoose from 'mongoose';
await mongoose.connect('mongodb://localhost:27017/fullstack_dev');
console.log('Connected to MongoDB');
```

### 5. Comandos útiles

```bash
docker compose up -d mongo                             # levantar MongoDB
docker compose exec mongo mongosh                      # shell interactivo
docker compose exec mongo mongosh fullstack_dev --eval "db.users.find()"  # query rápida

# Dentro de mongosh:
# show dbs                          listar bases
# use fullstack_dev                 cambiar de base
# db.users.find().pretty()          ver documentos
# db.users.insertOne({ name: "Lean" })
# exit
```

### 6. GUI (opcional)

- [ ] **MongoDB Compass** (oficial, gratis). Descargar desde [mongodb.com/products/tools/compass](https://www.mongodb.com/products/tools/compass). Conecta a `mongodb://localhost:27017`.

---

## Verificación

```bash
docker compose exec mongo mongosh --eval "db.runCommand({ ping: 1 })"   # { ok: 1 }
```

**Si el ping responde `{ ok: 1 }` → MongoDB listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Puerto 27017 ocupado | Otra instancia de MongoDB corriendo. `docker compose down`, cambiar `ports: - "27018:27017"` |
| "Connection refused" desde Node | Contenedor no está corriendo: `docker compose up -d mongo` |
| Datos se pierden | Agregar `volumes:` como en el paso 1 |
| Mongoose no conecta | Verificar URL: `mongodb://localhost:27017/nombre-base` |

---

## Preguntas de repaso

- **P:** ¿Qué comando verificás dentro del contenedor para confirmar que MongoDB responde?
  **R:** `docker compose exec mongo mongosh --eval "db.runCommand({ ping: 1 })"` que debe devolver `{ ok: 1 }`.
- **P:** ¿Cuál es el connection string para conectar Mongoose a MongoDB en Docker?
  **R:** `mongodb://localhost:27017/fullstack_dev`.
- **P:** ¿Qué comando dentro de mongosh lista todas las bases de datos?
  **R:** `show dbs`.
- **P:** ¿Por qué es importante el volumen `mongodata:/data/db` en el compose?
  **R:** Porque persiste los datos de MongoDB fuera del contenedor. Sin él, al hacer `docker compose down` se pierden todos los documentos.
- **P:** ¿Cómo insertás un documento desde mongosh?
  **R:** `db.coleccion.insertOne({ campo: "valor" })`, por ejemplo `db.users.insertOne({ name: "Lean" })`.

---

## Recursos

- [MongoDB Docker Image](https://hub.docker.com/_/mongo)
- [Mongoose Docs](https://mongoosejs.com/docs/)