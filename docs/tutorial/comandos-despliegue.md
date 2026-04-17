# Comandos usados en el despliegue (Spring Boot + PostgreSQL + Docker + ECR + EC2 + GitHub Actions)

Este documento resume los comandos usados para el despliegue, que hace cada uno y en que momento usarlo.

## 1) Desarrollo local (backend + Docker DB)

### 1.1 Levantar PostgreSQL en local con Docker
```bash
docker compose up -d
```
Sirve para iniciar contenedor de PostgreSQL en segundo plano.
Usalo antes de arrancar Spring Boot en local.

```bash
docker compose ps
```
Sirve para ver estado de contenedores (`running`, `healthy`, etc.).
Usalo para confirmar que la base de datos esta arriba.

```bash
docker compose logs -f postgres
```
Sirve para ver logs en tiempo real de PostgreSQL.
Usalo cuando la DB no arranca o hay errores de conexion.

```bash
docker compose down
```
Sirve para detener contenedores sin borrar datos del volumen.

```bash
docker compose down -v
```
Sirve para detener y borrar el volumen (elimina datos).
Usalo solo cuando quieras reset completo.

## 2) Spring Boot local

```powershell
.\mvnw.cmd spring-boot:run
```
Sirve para correr la app localmente en Windows.

```powershell
.\mvnw.cmd -q test
```
Sirve para ejecutar tests del proyecto.
Usalo antes de push/PR.

## 3) OpenAPI (generar contrato API)

### 3.1 Exportar OpenAPI automaticamente (Windows)
```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\export-openapi.ps1
```
Genera `docs/openapi/openapi.json` y `docs/openapi/openapi.yaml`.

### 3.2 Exportar OpenAPI automaticamente (Linux/macOS/CI)
```bash
chmod +x scripts/export-openapi.sh
./scripts/export-openapi.sh
```
Hace lo mismo en Linux/macOS/CI.

### 3.3 Endpoints OpenAPI
```text
http://localhost:8080/v3/api-docs
http://localhost:8080/swagger-ui.html
```
Sirven para ver el contrato y probar endpoints.

## 4) GitHub CLI (validar workflows)

```bash
gh --version
```
Verifica que GitHub CLI esta instalado.

```bash
gh auth status
```
Confirma sesion activa en GitHub.

```bash
gh workflow list
```
Lista workflows del repo.

```bash
gh run list --limit 10
```
Muestra ejecuciones recientes.

```bash
gh run view <RUN_ID>
```
Muestra resumen de ejecucion.

```bash
gh run view <RUN_ID> --log-failed
```
Muestra logs de pasos fallidos.

## 5) Docker de backend (build local)

```bash
docker build -t techstock-backend:local .
```
Construye imagen Docker local del backend.

## 6) AWS ECR + CI (imagen en registro)

### 6.1 Secrets en GitHub
- `AWS_REGION` (ejemplo: `us-east-2`)
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

### 6.2 Ejecutar build/push manual
```bash
gh workflow run "Docker Build and Push (ECR)" --ref main
```
Lanza build y push a ECR manualmente.

## 7) Crear y preparar EC2 (Ubuntu)

### 7.1 Conexion SSH desde local
```bash
ssh -i "D:\permiso\backend-redes.pem" ubuntu@3.19.222.150
```
Conecta a la instancia EC2.

### 7.2 Si PowerShell bloquea la clave `.pem` por permisos
Si aparece `UNPROTECTED PRIVATE KEY FILE`, ejecutar en PowerShell:

```powershell
$pem = "D:\permiso\backend-redes.pem"
icacls $pem /inheritance:r
icacls $pem /remove:g *S-1-5-11 *S-1-5-32-545 *S-1-1-0
icacls $pem /grant:r "${env:USERDOMAIN}\${env:USERNAME}:(R)"
```

Verificar permisos:

```powershell
icacls $pem
```

Probar SSH de nuevo:

```powershell
ssh -i "D:\permiso\backend-redes.pem" ubuntu@3.19.222.150
```

## 8) Instalar herramientas en EC2

```bash
sudo apt update -o Acquire::ForceIPv4=true
sudo apt install -y docker.io docker-compose-v2 curl unzip
```
Instala Docker y Compose en Ubuntu.

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip -q awscliv2.zip
sudo ./aws/install --update
aws --version
```
Instala AWS CLI v2.

```bash
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
docker --version
docker compose version
```
Habilita Docker y verifica instalacion.

## 9) Archivos de despliegue en EC2

```bash
mkdir -p ~/techstock-backend
cd ~/techstock-backend
ls -la
```
Prepara carpeta esperada por el workflow.

Archivos requeridos:
- `docker-compose.ec2.yml`
- `.env`

## 10) Deploy automatico a EC2 (GitHub Actions)

### 10.1 Secrets extra en GitHub
- `EC2_HOST`
- `EC2_USER`
- `EC2_SSH_KEY`

### 10.2 Ejecutar deploy manual
```bash
gh workflow run "Deploy to EC2" --ref main
```

## 11) Verificacion post-deploy

### 11.1 En EC2
```bash
docker compose --env-file .env -f docker-compose.ec2.yml ps
```
Verifica estado de contenedores.

```bash
docker compose --env-file .env -f docker-compose.ec2.yml logs --no-color backend
```
Verifica logs de arranque del backend.

### 11.2 Desde internet
```bash
curl -sS "http://3.19.222.150:8080/v3/api-docs"
```
Confirma que la API esta publica.

## 12) Troubleshooting rapido

### 12.1 Error SSH `UNPROTECTED PRIVATE KEY FILE`
Causa: permisos abiertos del `.pem` en Windows.
Solucion: ajustar ACL con `icacls` (seccion 7.2).

### 12.2 Deploy falla por `i/o timeout` en puerto 22
Causa: Security Group bloquea SSH desde GitHub runner.
Solucion: abrir temporalmente puerto 22 o migrar a SSM.

### 12.3 Deploy falla con `NoCredentials` en EC2
Causa: variables AWS no se pasaban al shell remoto.
Solucion: pasar `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` en paso SSH del workflow.

### 12.4 API no responde publicamente
Causa: puerto `8080` no permitido en Security Group.
Solucion: agregar inbound `Custom TCP 8080`.
