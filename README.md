# Proyecto 3 — API con base de datos y pgAdmin

API REST de usuarios (Node.js + Express) con PostgreSQL y administración vía pgAdmin, orquestada con Docker Compose.

## Descripción

Se requiere una API REST de usuarios que guarde la información en PostgreSQL, administrable desde pgAdmin. La solución se construye con Docker Compose (3 servicios: base de datos, API y pgAdmin), ejecuta las migraciones de base de datos automáticamente al primer arranque y valida los datos de entrada (nombre y email obligatorios, formato de email válido).

## Requisitos

- Docker
- Docker Compose (v2)

## Instrucciones de ejecución

1. Clonar el repositorio.
2. Crear el archivo `.env` a partir de la plantilla:
   ```
   copy .env.example .env
   ```
3. Levantar los 3 servicios:
   ```
   docker compose up -d
   ```
   (o `make up` si tienes `make` instalado)
4. Verificar que los 3 servicios estén corriendo:
   ```
   docker compose ps
   ```
5. Probar el endpoint de salud:
   ```
   curl http://localhost:3000/health
   ```

## Endpoints

| Método | Ruta         | Descripción                          |
|--------|--------------|----------------------------------------|
| GET    | /health      | Estado del servicio                    |
| GET    | /users       | Listar todos los usuarios              |
| GET    | /users/:id   | Obtener un usuario (404 si no existe)  |
| POST   | /users       | Crear usuario (400 si datos inválidos) |
| PUT    | /users/:id   | Actualizar usuario                     |
| DELETE | /users/:id   | Eliminar usuario                       |

Ejemplo de creación válida:
```
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" -d "{\"name\":\"Laura Gomez\",\"email\":\"laura@example.com\"}"
```

Ejemplo de datos inválidos (debe responder 400):
```
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" -d "{\"name\":\"\",\"email\":\"no-es-un-email\"}"
```

## pgAdmin

1. Entra a `http://localhost:5050` (o el puerto definido en `.env`).
2. Inicia sesión con `PGADMIN_EMAIL` / `PGADMIN_PASSWORD` de tu `.env`.
3. Agrega un nuevo servidor:
   - **Host**: `db` (nombre del servicio, no `localhost`)
   - **Puerto**: `5432`
   - **Usuario/contraseña**: los definidos en `DB_USER` / `DB_PASSWORD`
4. Navega a la tabla `users` y verifica los datos creados por la API.

## Makefile

```
make up     # levantar los servicios
make down   # detener los servicios
make logs   # ver logs en vivo
make test   # probar endpoints básicos
```

## Preguntas de reflexión

**¿Por qué el script de migración solo se ejecuta la primera vez que se crea el volumen? ¿Qué harías para volver a ejecutarlo?**
PostgreSQL solo ejecuta los scripts de `/docker-entrypoint-initdb.d` cuando el volumen de datos está vacío (primera inicialización). Para volver a ejecutarlo hay que eliminar el volumen (`docker compose down -v`) y levantar de nuevo.

**¿Por qué en pgAdmin el host de conexión es el nombre del servicio y no localhost?**
Porque pgAdmin corre dentro de su propio contenedor, en la misma red de Docker Compose; `localhost` haría referencia al propio contenedor de pgAdmin, no al de la base de datos. El nombre del servicio (`db`) se resuelve por la red interna de Docker.

**¿Qué ocurre si la API arranca antes de que la base de datos esté lista, y cómo lo previene la configuración del compose?**
La API fallaría al intentar conectarse porque PostgreSQL aún no acepta conexiones. Se previene con el `healthcheck` de `db` (usando `pg_isready`) combinado con `depends_on: condition: service_healthy` en el servicio `api`, que espera a que la base de datos esté realmente lista antes de arrancar.

## Evidencias

- Captura de `docker compose ps` con los 3 servicios corriendo.
- Evidencia del CRUD completo probado con curl (GET, POST, PUT, DELETE).
- Captura de un POST con datos inválidos devolviendo 400.
- Captura de pgAdmin mostrando la tabla `users` con los datos.
