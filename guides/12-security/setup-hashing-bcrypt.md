# Setup — Hashing de Passwords con bcrypt (y argon2id)

> **Tópico**: 12 — Seguridad (sección 12.7)
> **Objetivo**: hashear passwords con bcrypt (hash, verify, salt rounds, comparación timing-safe) y conocer argon2id como alternativa recomendada por OWASP.
> **Prerequisito**: una app con registro/login (la del tópico 4 sirve) y un lugar donde persistir el hash (Postgres, SQLite, en memoria para práctica).

---

## ¿Por qué?

Guardar passwords en texto plano es un regalo para quien robe tu DB: con un volcado se lleva todas las cuentas. Hashear convierte la password en una cadena de un solo sentido, pero no cualquier hash sirve. SHA256/MD5 son rápidos — un atacante con GPU prueba miles de millones por segundo. bcrypt y argon2id son **deliberadamente lentos y costosos** (en CPU, y argon2 también en memoria), lo que vuelve inviable la fuerza bruta. Esa lentitud es exactamente la defensa. Además, el salt automático garantiza que dos usuarios con la misma password tengan hashes distintos.

---

## Checklist

### 1. Instalar la librería

- [ ] `npm install bcrypt` (o `npm install argon2` si elegís argon2id).
- [ ] Elegí el **work factor**: bcrypt `cost=12` (≈250ms por hash) o argon2 `memoryCost=19456, timeCost=2`.

### 2. Hashear al registrar

- [ ] En el registro, hasheá la password ANTES de guardar. Nunca guardes el texto plano:

```js
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12; // 2^12 iteraciones

app.post('/api/registro', async (req, res) => {
  const { email, password } = req.body; // asumí que ya validaste

  const hash = await bcrypt.hash(password, SALT_ROUNDS);
  await db.usuarios.create({ email, passwordHash: hash }); // ← guardás el hash

  res.status(201).json({ email });
});
```

### 3. Verificar al loguear (timing-safe)

- [ ] En el login, compará con `bcrypt.compare` (nunca `===`):

```js
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body;

  const usuario = await db.usuarios.findOne({ email });
  if (!usuario) return res.status(401).json({ error: 'Credenciales inválidas' });

  const ok = await bcrypt.compare(password, usuario.passwordHash); // timing-safe
  if (!ok) return res.status(401).json({ error: 'Credenciales inválidas' });

  // acá emitís la sesión o el JWT
  res.json({ token: '...' });
});
```

### 4. (Alternativa) argon2id

- [ ] Si preferís argon2id (recomendado por OWASP para proyectos nuevos):

```js
import argon2 from 'argon2';

const hash = await argon2.hash(password, {
  type: argon2.argon2id, // variante resistente a side-channel
  memoryCost: 19456,      // 19 MiB
  timeCost: 2,
  parallelism: 1,
});

const ok = await argon2.verify(hash, password);
```

### 5. Proteger contra timing attacks y leaks

- [ ] Usá el mismo mensaje de error para "usuario no existe" y "password incorrecta" (no reveles qué falló).
- [ ] No loguees nunca la password ni el hash; logueá solo el email o el id.

### 6. Verificar el resultado

- [ ] Confirmá que en la DB quedó un hash (que empieza con `$2b$` en bcrypt o `$argon2id$`), no la password (ver sección Verificación).

---

## Verificación

```bash
# El hash guardado NO es la password (empieza con $2b$ en bcrypt)
node -e "import('bcrypt').then(async ({default:bcrypt}) => {
  const hash = await bcrypt.hash('super-secreto', 12);
  console.log(hash);                    // $2b$12$... (no la password)
  console.log(await bcrypt.compare('super-secreto', hash));  // true
  console.log(await bcrypt.compare('super-secreta', hash));  // false (un caracter de diff)
  console.log(await bcrypt.compare('super-secreto', hash + 'x')); // false
})"
```

**Si el hash empieza con `$2b$` (o `$argon2id$`), la password correcta da `true`, una con un caracter distinto da `false`, y el mismo texto da hashes distintos en dos corridas (salt) → hashing correcto. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| El login tarda mucho (más de 1s) | Bajá el work factor (cost=10 o memoryCost menor) o mové el hash a un worker thread |
| `bcrypt.compare` devuelve `false` con password correcta | Verificá que estás comparando contra el hash guardado (no la password) y que no hay doble hash accidental |
| Dos usuarios con la misma password tienen hashes iguales | Eso NO debe pasar con bcrypt; si pasa, es que guardaste un hash sin salt — usá `bcrypt.hash` (agrega salt automático) |
| Querés migrar hashes viejos (MD5/SHA1) | Hacé re-hash en el próximo login exitoso (migración progresiva), nunca compares con el hash viejo |
| `argon2.verify` es lento en CI | Bajá `memoryCost`/`timeCost` solo en tests; en producción mantené parámetros altos |
| Guardaste la password en texto plano por error | Tratalo como incidente: migrá a hash, forzá reset de passwords y rotá cualquier secreto expuesto |

---

## Recursos

- [OWASP — Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [bcrypt — npm](https://www.npmjs.com/package/bcrypt)
- [argon2 — npm](https://www.npmjs.com/package/argon2)

## Preguntas de repaso

- **P:** ¿Por qué hasheás la password y no la guardás en texto plano?
  **R:** Para que un volcado de la DB no exponga las passwords reales. El hash es de un solo sentido: no se puede revertir a la password original.

- **P:** ¿Qué son los salt rounds (work factor) y por qué importan?
  **R:** Son las iteraciones de hashing (2^cost). Definen cuán caro es cada hash: suficiente para que tarde ~250ms, lo que hace inviable la fuerza bruta, pero sin degradar el login.

- **P:** ¿Por qué `bcrypt.compare` en vez de `===`?
  **R:** Porque `bcrypt.compare` compara en tiempo constante (timing-safe). `===` puede filtrar cuántos caracteres coinciden por el tiempo de respuesta, ayudando a un atacante.

- **P:** ¿Por qué SHA256 o MD5 son malos para passwords?
  **R:** Porque son rápidos: un atacante con GPU prueba miles de millones de combinaciones por segundo. bcrypt/argon2 son deliberadamente lentos y costosos.

- **P:** ¿Qué agrega argon2id sobre bcrypt?
  **R:** Un parámetro de memoria además del de CPU, lo que lo hace resistente a ataques con GPU. Es el recomendado por OWASP para proyectos nuevos.

- **P:** ¿Por qué el error de login debe ser el mismo para "usuario no existe" y "password incorrecta"?
  **R:** Para no filtrar qué emails están registrados. Distintos mensajes permiten a un atacante enumerar usuarios válidos del sistema.
