# Setup — MongoDB Compass

> **Tópico**: 6 — Bases de Datos & Persistencia
> **Objetivo**: MongoDB Compass instalado y conectado a MongoDB en Docker, listo para explorar documentos y agregaciones.
> **Prerequisito**: Docker + MongoDB corriendo (`setup-mongodb.md`).

---

## ¿Por qué MongoDB Compass?

Es la GUI oficial de MongoDB, gratis y multiplataforma. La línea de comandos (`mongosh`) sirve, pero para ver documentos, modelar datos y armar el aggregation pipeline con feedback visual, Compass es otra cosa. Y como el `compose.yaml` de `setup-mongodb.md` no usa autenticación, conectar es pegar la URI y listo.

---

## Checklist

### 1. Descargar e instalar MongoDB Compass

Andá a [mongodb.com/products/tools/compass](https://www.mongodb.com/products/tools/compass) y bajá la edición **Community** (gratis). Instalá con los valores por defecto.

- [ ] Compass abre sin errores

### 2. Conectar a MongoDB en Docker

1. Asegurate de que MongoDB esté corriendo: `docker compose up -d mongo`.
2. Abrí Compass. En la pantalla de conexión pegá la URI del compose de `setup-mongodb.md`:

```
mongodb://localhost:27017/fullstack_dev
```

3. **Connect**. El compose no define usuario ni password — no hace falta completar nada más.

- [ ] La conexión es exitosa

### 3. Explorar bases y colecciones

- En el panel izquierdo ves las bases; la nuestra es `fullstack_dev`.
- Hacé clic en la base para ver sus colecciones (ej: `users`). Cada colección muestra sus documentos.
- Doble clic en un documento → **Edit** para verlo formateado.

### 4. Insertar un documento manual

1. Clic en la colección → **Add Data** → **Insert Document**.
2. Escribí el JSON y dale **Insert**:

```js
{ name: "Lean", email: "lean@ejemplo.com", createdAt: new Date() }
```

### 5. Correr el aggregation pipeline

En la pestaña **Aggregations** armás el pipeline paso a paso y lo ejecutás para ver el resultado de cada stage:

```js
[
  { $match: { name: { $ne: null } } },
  { $count: "total" }
]
```

---

## Verificación

```bash
docker compose up -d mongo
```

Abrí Compass y conectá a `mongodb://localhost:27017/fullstack_dev`.

**Si ves la base `fullstack_dev` con sus colecciones → MongoDB Compass listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "Unable to connect" / connection refused | MongoDB no está corriendo: `docker compose up -d mongo` |
| Compass pide usuario y password | El compose no usa auth: dejalo vacío y usá `mongodb://localhost:27017/fullstack_dev` |
| Puerto 27017 ocupado | Otra instancia corriendo — mirá en `setup-mongodb.md` cómo cambiar el puerto |
| No veo la base | Todavía no se creó: la base aparece al insertar el primer documento |

---

## Preguntas de repaso

- **P:** ¿Cuál es la URI de conexión para MongoDB Compass con el compose de esta guía?
  **R:** `mongodb://localhost:27017/fullstack_dev`.
- **P:** ¿Por qué Compass no pide usuario ni password con esta configuración?
  **R:** Porque el `compose.yaml` no define variables de autenticación (`MONGO_INITDB_ROOT_USERNAME`, etc.), así que MongoDB arranca sin auth.
- **P:** ¿Cómo insertás un documento manualmente desde Compass?
  **R:** Clic en la colección → Add Data → Insert Document, escribir el JSON y darle Insert.
- **P:** ¿Qué ventaja tiene la pestaña Aggregations de Compass sobre escribir el pipeline en código?
  **R:** Permite armar el pipeline stage por stage con feedback visual inmediato, viendo el resultado de cada stage antes de agregar el siguiente.
- **P:** ¿Por qué la base `fullstack_dev` puede no aparecer en Compass hasta que insertes el primer documento?
  **R:** Porque MongoDB crea las bases de datos de forma lazy: la base no existe físicamente hasta que se inserta el primer documento en ella.

---

## Recursos

- [MongoDB Compass](https://www.mongodb.com/products/tools/compass)
- [Compass Docs](https://www.mongodb.com/docs/compass/)
