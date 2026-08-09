# Setup — MongoDB (Instalación Local + Mongoose)

> **Tópico**: 4 — Backend Core / 6 — Bases de Datos (MongoDB es la base NoSQL principal del stack)
> **Objetivo**: MongoDB corriendo en WSL, accesible desde tu app Node, listo para usar con Mongoose.
> **Prerequisito**: WSL + terminal (`setup-wsl.md`, `setup-terminal.md`).

---

## ¿Por qué MongoDB?

La base documental más usada con Node. Fluye natural con JSON: un documento Mongo ≈ un objeto JS. Ideal para datos semi-estructurados, schemas flexibles, y cuando la relación jerárquica (un user con sus posts embebidos) se lee mejor como documento que como joins.

---

## Checklist

### 1. Instalar MongoDB en WSL (Community Edition)
MongoDB ya no publica paquetes oficiales en los repos de Ubuntu por defecto, usamos el repo oficial:

```bash
# 1. Importar la clave pública de MongoDB
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor

# 2. Agregar el repo (ajustá versión si usás Ubuntu más reciente)
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | \
   sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

# 3. Instalar
sudo apt update
sudo apt install -y mongodb-org
```

> Si preferís la opción más simple: **Docker** (`docker run -d -p 27017:27017 --name mongo mongo:7`). Pero para el tópico 4-6, la instalación nativa te obliga a entender el servicio, y Docker se cubre en el tópico 9.

### 2. Arrancar el servicio
```bash
sudo systemctl enable mongod
sudo systemctl start mongod
# o en WSL sin systemd:
sudo service mongod start

# Verificar
mongosh --eval "db.runCommand({ ping: 1 })"
```
- [ ] El ping devuelve `{ ok: 1 }`

### 3. Verificar acceso desde la app
```bash
# El connect string por defecto:
mongodb://localhost:27017/fullstack_dev
```

### 4. Mongoose (cuando llegues al tópico 5-6)
```bash
npm install mongoose
```
```js
// En tu app:
import mongoose from 'mongoose';
await mongoose.connect('mongodb://localhost:27017/fullstack_dev');
console.log('Connected to MongoDB');
```
- [ ] El server arranca y loguea la conexión sin error

### 5. Comandos útiles del día a día
```bash
mongosh                              # abrir shell
mongosh fullstack_dev                # abrir shell en una base
show dbs                             # listar bases
use fullstack_dev                    # cambiar de base
db.users.find().pretty()             # ver documentos
db.users.find({ email: "x@y.com" })  # con filtro
db.users.insertOne({ name: "Lean" }) # insertar
exit                                 # salir
```

### 6. GUI (opcional cuando necesites inspeccionar visualmente)
- [ ] **MongoDB Compass** (official GUI, gratis). Descargar la versión Linux/AppImage desde [mongodb.com/products/tools/compass](https://www.mongodb.com/products/tools/compass). En WSL con GUI, o instalarlo en Windows apuntando a `localhost:27017`.

---

## Verificación

```bash
mongosh --eval "db.runCommand({ ping: 1 })"   # { ok: 1 }
```
**Si el ping responde `{ ok: 1 }` → MongoDB listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| `mongod: unrecognized service` | Installé mal el paquete o repo. Revisá pasos 1-2. |
| MongoDB arranca y se cae (segfault en WSL) | Aumentar memoria: crear `%UserProfile%\.wslconfig` → `[wsl2] memory=4GB` y `wsl --shutdown` |
| Puerto 27017 ocupado | `sudo lsof -i :27017` para ver quién, o cambiar `net.port` en `mongod.conf` |
| No conecta desde Node (ECONNREFUSED) | El servicio `mongod` no está corriendo: `sudo service mongod start` |
| Mongoose necesita auth en producción | Nunca exponer Mongo sin auth. Configurar `mongod.conf` con `security.authorization: enabled` + crear users en `admin` |

---

## Recursos

- [MongoDB Manual](https://www.mongodb.com/docs/manual/)
- [Mongoose Docs](https://mongoosejs.com/docs/)
- [MongoDB EA — Install on Ubuntu](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/)