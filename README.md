# Taller 02 v2.0 - Aplicación con Arquitectura de Microservicios

Proyecto académico para **Web, APIs y Microservicios**. Implementa una arquitectura distribuida con Java 21, Spring Boot 4.0.8, Spring Cloud Gateway 5.0.3 (Spring Cloud 2025.1.3), PostgreSQL, JWT RSA, Swagger/OpenAPI, Docker Compose y GitHub Actions.

## Arquitectura

```text
Cliente / Postman
       |
       v
API Gateway :8080
   |             |
   v             v
auth-service    user-service
   :8081           :8082
   |                 |
   v                 v
auth_db          users_db
PostgreSQL       PostgreSQL
```

- **api-gateway:** enruta `/api/auth/**` y `/api/users/**` y agrega `X-Correlation-Id`.
- **auth-service:** registro, login, BCrypt y emisión de JWT firmados con RSA.
- **user-service:** CRUD de perfiles y autorización basada en roles del JWT.
- **Database per service:** cada servicio posee su propia base de datos.

## Requisitos

- Docker Desktop con Docker Compose v2.
- Opcional para ejecución sin Docker: Java 21 y Maven 3.9+.
- Postman para ejecutar la colección incluida.

## Arranque rápido con Docker

```bash
cp .env.example .env
docker compose up --build
```

Servicios:

- Gateway: http://localhost:8080
- Auth Swagger: http://localhost:8081/swagger-ui.html
- User Swagger: http://localhost:8082/swagger-ui.html
- Auth PostgreSQL: localhost:5433
- Users PostgreSQL: localhost:5434

## Usuarios demostrativos

Se crean al iniciar `auth-service`:

| Rol | Usuario | Contraseña |
|---|---|---|
| ADMIN | `admin` | `Admin123!` |
| USER | `student` | `Student123!` |

> Estas credenciales y las llaves RSA incluidas son exclusivamente académicas.

## Flujo sugerido

1. `POST /api/auth/login` con `student`.
2. Copiar `accessToken` o dejar que la colección Postman lo almacene.
3. `POST /api/users` con Bearer Token para crear el perfil.
4. `GET /api/users/me`.
5. `PUT /api/users/{studentId}`.
6. Iniciar sesión como `admin`.
7. `GET /api/users` para listar perfiles.
8. `DELETE /api/users/{studentId}` como ADMIN.

## Ejemplos curl

Login:

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"student","password":"Student123!"}'
```

Crear perfil:

```bash
curl -X POST http://localhost:8080/api/users \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Estudiante","lastName":"Demo","phone":"+57 300 111 2233"}'
```

## Postman

Importar:

- `postman/Taller_02_v2.postman_collection.json`
- `postman/Taller_02_v2.postman_environment.json`

Seleccionar el ambiente **Taller 02 v2.0 - Local** y ejecutar las solicitudes en orden.

## Compilación local

Con PostgreSQL disponible en los puertos 5433 y 5434:

```bash
mvn clean verify
```

Luego se pueden ejecutar los módulos individualmente:

```bash
mvn -pl auth-service spring-boot:run
mvn -pl user-service spring-boot:run
mvn -pl api-gateway spring-boot:run
```

## Persistencia

Docker Compose usa los volúmenes `auth-data` y `users-data`. Eliminar un contenedor no elimina los datos del volumen. Para destruir también los datos del taller:

```bash
docker compose down -v
```

## JWT y llaves RSA

`auth-service` conserva la llave privada y pública; `user-service` solo la pública. Las llaves incluidas son de demostración. Para regenerarlas:

```bash
./scripts/generate-rsa-keys.sh
```

En producción, la llave privada debe mantenerse fuera del repositorio y gestionarse como secreto.

## CI/CD

`.github/workflows/ci-cd.yml`:

1. compila y verifica los tres módulos con Java 21;
2. en `push` a `main`, construye las tres imágenes Docker;
3. publica las imágenes en GitHub Container Registry (`ghcr.io`) usando `GITHUB_TOKEN`.

## Conceptos para discutir en clase

- API Gateway y enrutamiento.
- Database per Service.
- Autenticación frente a autorización.
- JWT firmado con RSA.
- BCrypt para contraseñas.
- DTO frente a Entity.
- Validación de entrada.
- Manejo global de errores.
- Migraciones con Flyway.
- Persistencia con volúmenes Docker.
- CI/CD y registro de imágenes.
- Correlation ID para trazabilidad.
