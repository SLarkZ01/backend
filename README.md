# Backend - PostgreSQL con Docker

Este proyecto usa Spring Boot + PostgreSQL. Para desarrollo local, la base de datos corre en Docker usando `docker-compose`.

## 1) Preparar variables de entorno

1. Copia el archivo de ejemplo:

```bash
cp .env.example .env
```

En PowerShell (Windows):

```powershell
Copy-Item .env.example .env
```

2. Si quieres, cambia usuario/password/puerto en `.env`.

## 2) Levantar PostgreSQL en Docker

```bash
docker compose up -d
```

Verifica estado:

```bash
docker compose ps
```

Ver logs:

```bash
docker compose logs -f postgres
```

## 3) Ejecutar Spring Boot

Con la BD arriba, ejecuta la app normalmente (IDE o Maven wrapper):

```bash
./mvnw spring-boot:run
```

En PowerShell:

```powershell
.\mvnw.cmd spring-boot:run
```

## OpenAPI / Swagger

- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`

La documentacion de endpoints se encuentra separada del controlador en una interfaz contrato:

- `src/main/java/com/proyecto/redes/backend/products/contract/ProductsApi.java`

El controlador implementa esa interfaz y mantiene visibles los mappings HTTP (`@GetMapping`, `@PostMapping`, etc.):

- `src/main/java/com/proyecto/redes/backend/products/controller/ProductController.java`

### Exportar OpenAPI con un solo comando

Windows (PowerShell):

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\export-openapi.ps1
```

Linux/macOS:

```bash
chmod +x scripts/export-openapi.sh
./scripts/export-openapi.sh
```

Archivos generados:

- `docs/openapi/openapi.json`
- `docs/openapi/openapi.yaml`

## CI/CD (GitHub Actions + AWS)

Workflows incluidos:

- `.github/workflows/ci-backend.yml`
  - Ejecuta tests
  - Exporta OpenAPI
  - Publica `openapi.json` y `openapi.yaml` como artifacts
- `.github/workflows/docker-backend.yml`
  - Construye imagen Docker del backend
  - Publica imagen a Amazon ECR (`latest` + SHA)
- `.github/workflows/deploy-ec2.yml`
  - Se conecta por SSH a EC2
  - Actualiza `IMAGE_URI` en `.env`
  - Hace `docker compose pull` y `docker compose up -d`

### Secrets requeridos en GitHub

- `AWS_REGION`
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `EC2_HOST`
- `EC2_USER`
- `EC2_SSH_KEY`

### Archivos para EC2

- `deploy/docker-compose.ec2.yml`
- `deploy/ec2.env.example` (copiar a `.env` en EC2 y ajustar valores)

## Configuracion aplicada

- `docker-compose.yml` levanta un contenedor `postgres:16-alpine`.
- La data persiste en el volumen `postgres_data`.
- `application.properties` usa variables de entorno con valores por defecto para conectar a PostgreSQL.
- JPA usa `ddl-auto=update` para desarrollo.

## Comandos utiles

Detener servicios:

```bash
docker compose down
```

Detener y borrar volumen (elimina datos de la BD):

```bash
docker compose down -v
```
