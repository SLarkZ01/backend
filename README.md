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
