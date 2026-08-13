# Setup — Monitoreo y Logging Centralizado

> **Tópico**: 9 — DevOps & Deployment (secciones 9.5 y 9.6)
> **Objetivo**: darle observabilidad a tu app en producción: health checks, logging estructurado en JSON con correlation ID, y métricas básicas de request. Así detectás problemas antes que los usuarios.
> **Prerequisito**: una app deployada (la del `setup-docker.md`) con Express (o similar) corriendo detrás de un endpoint HTTP.

---

## ¿Por qué?

Deployar no es el final — es el principio. Sin observabilidad operás a ciegas: te enterás de una caída por un usuario enojado, y buscar un error significa bucear en logs de muchas instancias. Un health check te dice si el proceso está vivo, el logging estructurado te deja encontrar errores en segundos, y las métricas te muestran tendencias antes de que se vuelvan incidentes. Es la diferencia entre reaccionar y anticiparte.

---

## Checklist

### 1. Agregar health check

- [ ] Creá un endpoint `GET /health` que responda 200 con datos del proceso:

```js
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok', uptime: process.uptime() });
});
```

- [ ] Verificá que el `HEALTHCHECK` del Dockerfile (guía anterior) lo esté usando.

### 2. Implementar logging estructurado

- [ ] Creá un helper `log()` que emita JSON a stdout:

```js
function log(level, message, extra = {}) {
  console.log(JSON.stringify({
    level,
    message,
    timestamp: new Date().toISOString(),
    ...extra,
  }));
}
```

- [ ] Reemplazá los `console.log` sueltos de tu app por `log('info' | 'warn' | 'error', ...)`.

### 3. Agregar correlation ID

- [ ] Creá un middleware que genere un ID único por request y lo propague a todos los logs:

```js
const crypto = require('node:crypto');

app.use((req, res, next) => {
  req.id = crypto.randomUUID();
  log('info', 'request', { requestId: req.id, method: req.method, path: req.path });
  next();
});
```

- [ ] Asegurate de incluir `requestId` en los logs del error handler y de cualquier lógica posterior.

### 4. Loguear errores en el error handler

- [ ] En el error handler, logueá el error con su correlation ID:

```js
app.use((err, req, res, next) => {
  log('error', err.message, { requestId: req.id, stack: err.stack });
  res.status(500).json({ error: 'Internal Server Error' });
});
```

### 5. Agregar métricas básicas de request

- [ ] Creá un middleware que cuente requests y mida latencia, y exponelo en `/metrics`:

```js
let requestCount = 0;

app.use((req, res, next) => {
  requestCount++;
  const start = Date.now();
  res.on('finish', () => {
    log('info', 'request_finished', {
      requestId: req.id,
      status: res.statusCode,
      durationMs: Date.now() - start,
    });
  });
  next();
});

app.get('/metrics', (req, res) => {
  res.set('Content-Type', 'text/plain');
  res.send(`# HELP http_requests_total Total requests\n# TYPE http_requests_total counter\nhttp_requests_total ${requestCount}\n`);
});
```

### 6. Verificar que nada sensible se loguea

- [ ] Buscá en tu código logs que incluyan passwords, tokens o emails crudos y eliminá/redactalos.
- [ ] Confirmá que los logs no incluyen `DATABASE_URL` ni `JWT_SECRET`.

---

## Verificación

```bash
# corré la app y probá:
curl -s localhost:3000/health          # {"status":"ok",...}
curl -s localhost:3000/metrics         # http_requests_total ...
curl -s localhost:3000/alguna-ruta     # luego revisá el stdout
```

- El stdout muestra líneas JSON con `level`, `message`, `timestamp` y `requestId`.
- Un request que falla genera un log `level: error` con el MISMO `requestId` que su request de entrada.

**Si `/health` responde, los logs son JSON con correlation ID y `/metrics` expone el contador → observabilidad lista. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| Logs ilegibles (texto libre) | Usá el helper `log()` que emite JSON; no `console.log` con concatenación |
| No podés rastrear un request | Falta el correlation ID: generá `req.id` en un middleware ANTES de cualquier log |
| `/metrics` vacío o sin formato | El content-type debe ser `text/plain` y el formato debe respetar el de Prometheus (`# HELP`, `# TYPE`, métrica) |
| Logs con secretos | Revisá que no loguees objetos completos de config; redactá o eliminá campos sensibles |
| `process.uptime()` devuelve 0 o raro | Es normal en instancias muy jóvenes; es tiempo en segundos desde que arrancó el proceso |
| Mucho ruido en los logs | Usá niveles: en producción solo `warn`/`error`; reservá `info`/`debug` para desarrollo |

---

## Recursos

- [12-Factor App — Logs](https://12factor.net/logs)
- [OpenTelemetry — Logs](https://opentelemetry.io/docs/concepts/signals/logs/)
- [Prometheus — Metrics types](https://prometheus.io/docs/concepts/metric_types/)
- [Node.js — console](https://nodejs.org/api/console.html)

## Preguntas de repaso

- **P:** ¿Para qué sirve un health check en producción?
  **R:** Es la señal mínima de que el proceso está vivo. Los orquestadores lo consultan periódicamente para decidir si reiniciar contenedores enfermos o si un servicio está listo para recibir tráfico.

- **P:** ¿Por qué los logs deben ser estructurados (JSON) en vez de texto libre?
  **R:** Porque el agregador puede indexar y consultar campos específicos (`level`, `requestId`, `status`). El texto libre no se puede filtrar ni consultar de forma eficiente.

- **P:** ¿Qué es un correlation ID y cómo se implementa?
  **R:** Un identificador único generado al inicio de cada request (por ej. `crypto.randomUUID()`) que se propaga a todos los logs de esa operación. Se genera en un middleware temprano y se incluye en cada log posterior.

- **P:** ¿Por qué la app escribe logs a stdout en vez de a un archivo?
  **R:** Porque así la app no decide el destino de los logs. La plataforma captura stdout y los enruta al agregador. Escribir a archivo acopla la app al filesystem y complica la centralización.

- **P:** ¿Qué información NO debería aparecer nunca en un log?
  **R:** Secretos (passwords, tokens, claves) y datos personales (PII). Loguear secretos es una filtración; la PII debe redactarse antes de escribir el log.

- **P:** ¿Cómo medís latencia por request con un middleware?
  **R:** Capturás `Date.now()` al entrar, y en el evento `finish` de la respuesta calculás `Date.now() - start`. Eso te da la duración de cada request, que logueás o exportás como métrica.
