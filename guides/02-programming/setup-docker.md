# Setup — Docker Desktop + WSL2

> **Tópico**: 2 — Fundamentos de Programación (sección 2.6)
> **Objetivo**: Docker Desktop instalado en Windows con backend WSL2, verificado con `hello-world`, y con un `compose.yaml` de ejemplo listo para desarrollo.
> **Prerequisito**: WSL2 instalado y funcionando (`setup-wsl.md`).

---

## ¿Por qué Docker?

Docker te permite correr bases de datos, colas, caches y cualquier servicio SIN instalarlo nativo en tu sistema. Un solo comando levanta PostgreSQL + Redis + MongoDB, y otro los apaga. Sin ensuciar tu máquina, sin conflictos de versiones, y con paridad exacta entre desarrollo y producción.

En Windows, Docker corre DENTRO de WSL2 — por eso WSL va primero.

---

## Checklist

### 1. Instalar Docker Desktop (Windows)

- [ ] Descargar [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
- [ ] Instalar. Durante la instalación, asegurate de que esté marcado **"Use WSL 2 instead of Hyper-V"**
- [ ] Reiniciar si lo pide

### 2. Activar la integración con WSL2

- [ ] Abrir Docker Desktop
- [ ] Settings → **Resources** → **WSL Integration**
- [ ] Activar **"Enable integration with my default WSL distro"**
- [ ] Asegurate de que tu distro (Ubuntu) esté seleccionada en la lista
- [ ] Apply & Restart

### 3. Verificar que Docker usa WSL2

Desde tu terminal WSL:
```bash
docker --version
docker info | grep -i "OSType\|Operating System"   # debe decir "Docker Desktop" y "linux"
```
- [ ] `docker --version` responde

### 4. Probar con hello-world
```bash
docker run hello-world
```
- [ ] El mensaje dice "Hello from Docker!" y confirma que la instalación funciona

### 5. Crear tu primer `compose.yaml` para desarrollo

Creá un archivo `compose.yaml` en la raíz de tu proyecto (o en `~/docker-dev/` para practicar):

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: fullstack_dev
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongodata:/data/db

volumes:
  pgdata:
  redisdata:
  mongodata:
```

```bash
# Levantar todos los servicios
docker compose up -d

# Ver que están corriendo
docker compose ps

# Ver logs en vivo
docker compose logs -f

# Apagar todo
docker compose down
```
- [ ] `docker compose ps` muestra los 3 servicios corriendo

### 6. Conectar desde tu app

Con los servicios corriendo, tu app Node se conecta exactamente igual que si estuvieran instalados nativo:

```
# PostgreSQL
DATABASE_URL="postgresql://dev:dev@localhost:5432/fullstack_dev"

# Redis
REDIS_URL="redis://localhost:6379"

# MongoDB
MONGODB_URL="mongodb://localhost:27017/fullstack_dev"
```

> Docker expone los puertos en `localhost` gracias a la integración con WSL2.

### 7. Comandos del día a día

```bash
docker ps                        # contenedores corriendo
docker compose up -d             # levantar servicios
docker compose down              # apagar servicios
docker compose logs -f db        # logs de un servicio específico
docker compose restart redis     # reiniciar un servicio
docker exec -it <id> sh          # entrar al shell de un contenedor
docker system prune -a           # limpiar todo lo que no se usa (libera GB)
docker compose down -v           # apagar Y borrar volúmenes (datos)
```

---

## Verificación

```bash
docker --version                  # 27.x.x
docker ps                         # lista contenedores (vacío o con tus servicios)
docker compose version            # v2.x.x
docker run --rm hello-world       # "Hello from Docker!"
```

**Si `docker run hello-world` responde sin errores → Docker listo. ✅**

---

## Problemas comunes

| Problema | Solución |
|----------|----------|
| "docker: command not found" en WSL | Docker Desktop no está corriendo en Windows, o la integración WSL2 no está activada. Abrí Docker Desktop y verificá Settings → Resources → WSL Integration |
| "Cannot connect to the Docker daemon" | `sudo service docker start` NO aplica — Docker corre en Windows, no en WSL. Abrí Docker Desktop |
| Los puertos no son accesibles desde Windows (`localhost:5432`) | WSL2 expone puertos automáticamente. Si no funciona, reiniciar WSL: `wsl --shutdown` y volver a abrir |
| "permission denied" al montar volúmenes | Asegurate de que el `compose.yaml` esté en un path dentro de `~/` (WSL), no en `/mnt/c/` |
| Docker Desktop consume mucha RAM | Settings → Resources → Advanced → limitar memoria (4-6 GB suele alcanzar) |
| Quiero borrar TODO y empezar de cero | `docker compose down -v` (borra volúmenes) + `docker system prune -a` (borra imágenes no usadas) |

---

## Recursos

- [Docker Desktop — WSL 2 best practices](https://docs.docker.com/desktop/wsl/)
- [Docker Compose — Getting Started](https://docs.docker.com/compose/gettingstarted/)
- [Play with Docker (práctica online)](https://labs.play-with-docker.com/)