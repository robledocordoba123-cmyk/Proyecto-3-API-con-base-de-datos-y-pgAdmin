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

### Creación del repositorio
![Repositorio creado en GitHub](evidencias/e01.png)

### Commit inicial y construcción de la imagen
Se sube todo el código (API, Dockerfile, docker-compose, script de migración, Makefile), se instalan dependencias y se levanta el proyecto con Docker Compose (3 servicios: `db`, `api`, `pgadmin`).

![Commit inicial](evidencias/e02.png)
![Push del commit inicial](evidencias/e03.png)
![Push completado](evidencias/e04.png)
![Instalación de dependencias](evidencias/e05.png)
![Construcción de la imagen y servicios](evidencias/e06.png)

### Resolución de conflicto de puerto entre proyectos
El puerto 3000 estaba ocupado por el contenedor del Proyecto 1; se detiene y se reconstruye el servicio `api` para que quede correctamente publicado.

![Conflicto de puerto detectado](evidencias/e07.png)
![Contenedor recreado](evidencias/e08.png)
![Servicios corriendo correctamente](evidencias/e09.png)

### Pruebas de la API: salud, CRUD y validación
Se prueban el endpoint de salud, el listado (con los usuarios de ejemplo creados por la migración automática), la creación, actualización y eliminación de usuarios.

**Demostración de validación — POST con datos inválidos devuelve 400:**
```
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" -d '{"name":"","email":"no-es-un-email"}'

{"error":"name y email son obligatorios"}
HTTP_STATUS:400
```

![Endpoint de salud](evidencias/e10.png)
![Listado de usuarios](evidencias/e11.png)
![Creación de usuario (201)](evidencias/e12.png)
![Actualización de usuario (200)](evidencias/e13.png)
![Eliminación de usuario (204)](evidencias/e14.png)

### Verificación en pgAdmin
Se configura pgAdmin, se conecta a la base de datos usando `db` como host (el nombre del servicio, no `localhost`), y se confirma que los datos creados por la API se reflejan en la tabla `users`.

![Acceso a pgAdmin](evidencias/e15.png)
![Panel principal de pgAdmin](evidencias/e16.png)
![Registro del servidor](evidencias/e17.png)
![Configuración de la conexión](evidencias/e18.png)
![Servidor conectado](evidencias/e19.png)
![Bases de datos disponibles](evidencias/e20.png)
![Tabla users en el árbol de pgAdmin](evidencias/e21.png)
![Datos de la tabla users](evidencias/e22.png)

### Flujo de Pull Request
Se crea una rama corta, se abre un Pull Request hacia `main`, se fusiona y se limpia la rama, dejando el repositorio con una sola rama (`main`).

![Rama y push](evidencias/e23.png)
![Pull Request creado](evidencias/e24.png)
![Limpieza final de la rama](evidencias/e25.png)

## Autor

Manuela Cordoba
