# Gapsi Test - Docker Compose Setup

Sistema de gestión de proveedores. Stack dockerizado con frontend React, backend Spring Boot y Redis para caché.

## 📋 Arquitectura

```
┌─────────────────────────────────────────────────────┐
│              HTTP (80)                            │
│            localhost:80                           │
└───────────────────┬─────────────────────────────────┘
                    │
            ┌───────▼────────┐
            │ Nginx (Alpine) │
            │   Frontend     │
            │  - React 19    │
            │  - Material UI │
            │  - Redux       │
            │  - Vite        │
            └────────┬────────┘
                     │ Proxy /api → gapsi-backend:8080
            ┌────────▼──────────┐
            │  Spring Boot      │
            │    Backend        │
            │  - Java 21        │
            │  - H2 Database    │
            │  - REST + GraphQL │
            └────────┬──────────┘
                     │
            ┌────────▼──────────┐
            │  Redis 7-Alpine   │
            │      Cache        │
            │  - Sessions       │
            │  - 256MB limit    │
            │  - LRU eviction   │
            └───────────────────┘
```

## 🚀 Requisitos Previos

- Docker 20.10+
- Docker Compose 2.0+
- 2GB RAM mínimo

## 📦 Instalación Rápida

1. **Asegurar que las imágenes estén construidas:**

   ```bash
   # Backend
   cd ../gapsi-test-backend
   ./mvnw clean package -DskipTests
   docker build -t oscarygutierrezg/gapsi-test-backend:latest .

   # Frontend
   cd ../gapsi-test-frontend
   npm run build
   docker build -t oscarygutierrezg/gapsi-test-frontend:latest .
   ```
2. **Configurar variables de entorno:**

   ```bash
   cd ../gapsi-test-docker-compose
   # El archivo .env ya existe con la configuración por defecto
   # Editar si necesitas cambiar valores
   ```
3. **Iniciar los servicios:**

   ```bash
   docker-compose up -d
   ```
4. **Verificar el estado:**

   ```bash
   docker-compose ps
   ```

## 🎯 Acceso a la Aplicación

- **Frontend:** http://localhost:80
- **Backend API REST:** http://localhost:8080/gapsi-test-ecommerce/api/v1/suppliers
- **Backend GraphQL:** http://localhost:8080/gapsi-test-ecommerce/graphql
- **GraphiQL:** http://localhost:8080/gapsi-test-ecommerce/graphiql
- **H2 Console:** http://localhost:8080/gapsi-test-ecommerce/h2-console
- **Health Check:** http://localhost:8080/gapsi-test-ecommerce/actuator/health
- **Redis:** localhost:6379 (password: gapsi2024)

## 📝 Comandos Disponibles

```bash
# Iniciar servicios
docker-compose up -d

# Ver logs en tiempo real (todos los servicios)
docker-compose logs -f

# Ver logs de un servicio específico
docker-compose logs -f gapsi-backend
docker-compose logs -f gapsi-frontend
docker-compose logs -f redis

# Detener servicios (sin eliminar contenedores)
docker-compose stop

# Iniciar servicios detenidos
docker-compose start

# Reiniciar servicios
docker-compose restart

# Detener y eliminar contenedores
docker-compose down

# Eliminar servicios, volúmenes y redes
docker-compose down -v

# Ver estado de servicios
docker-compose ps

# Ver uso de recursos
docker stats
```

## 🔧 Configuración

### Variables de Entorno (.env)

```env
# Spring Profile
SPRING_PROFILES_ACTIVE=dev

# Variables de entorno para la aplicación
SERVER_PORT=8080
CORS_ALLOWED_ORIGINS=http://localhost:80,http://gapsi-frontend

# Redis Configuration
REDIS_HOST=gapsi-redis
REDIS_PORT=6379
REDIS_PASSWORD=gapsi2024
REDIS_DATABASE=0

# Logging Configuration
LOG_PATH=/opt/gapsi/logs
```

## 🏗 Build de Imágenes

### Backend

```bash
cd ../gapsi-test-backend

# Compilar con Maven
./mvnw clean package -DskipTests

# Construir imagen Docker
docker build -t oscarygutierrezg/gapsi-test-backend:latest .

# (Opcional) Subir a Docker Hub
docker push oscarygutierrezg/gapsi-test-backend:latest
```

### Frontend

```bash
cd ../gapsi-test-frontend

# Instalar dependencias (si es necesario)
npm install

# Compilar aplicación React
npm run build

# Construir imagen Docker
docker build -t oscarygutierrezg/gapsi-test-frontend:latest .

# (Opcional) Subir a Docker Hub
docker push oscarygutierrezg/gapsi-test-frontend:latest
```

## 🔍 Monitoreo y Logs

### Logs en Consola

```bash
# Ver todos los logs
docker-compose logs -f

# Solo backend
docker-compose logs -f gapsi-backend

# Solo frontend
docker-compose logs -f gapsi-frontend

# Solo Redis
docker-compose logs -f redis

# Ver últimas 100 líneas
docker-compose logs --tail=100 gapsi-backend
```

### Logs Persistentes (Backend)

Los logs del backend se guardan en `./logs/`:

```bash
# Ver logs generales
tail -f logs/gapsi-backend.log

# Ver solo errores
tail -f logs/gapsi-backend-error.log

# Buscar en logs
grep "ERROR" logs/gapsi-backend.log

# Ver logs archivados
ls -lh logs/archive/
```

### Inspección de Contenedores

```bash
# Ver detalles de un contenedor
docker inspect gapsi-backend
docker inspect gapsi-frontend
docker inspect gapsi-redis

# Entrar a un contenedor
docker exec -it gapsi-backend bash
docker exec -it gapsi-frontend sh
docker exec -it gapsi-redis redis-cli -a gapsi2024

# Ver uso de recursos en tiempo real
docker stats
```

## 🛠 Troubleshooting

### Redis no inicia

```bash
# Verificar logs de Redis
docker-compose logs redis

# Verificar health check
docker exec gapsi-redis redis-cli -a gapsi2024 ping

# Debería responder: PONG
```

### Backend no inicia o reinicia constantemente

```bash
# Ver logs completos
docker-compose logs gapsi-backend

# Ver si Redis está healthy
docker-compose ps

# Verificar health check manualmente
docker exec gapsi-backend curl -f http://localhost:8080/gapsi-test-ecommerce/actuator/health

# Verificar logs persistentes
tail -100 logs/gapsi-backend-error.log
```

### Frontend no carga o muestra errores

```bash
# Verificar logs
docker-compose logs gapsi-frontend

# Verificar que el backend esté healthy
docker-compose ps

# Probar conexión al backend desde el frontend
docker exec gapsi-frontend wget -O- http://gapsi-backend:8080/gapsi-test-ecommerce/actuator/health
```

### Error de conexión entre contenedores

```bash
# Verificar que estén en la misma red
docker network inspect gapsi-test-docker-compose_gapsi-network

# Probar conectividad
docker exec gapsi-frontend ping gapsi-backend
docker exec gapsi-backend ping redis

# Verificar variables de entorno
docker exec gapsi-backend env | grep REDIS
```

### Reiniciar servicios individuales

```bash
# Reiniciar solo backend
docker-compose restart gapsi-backend

# Reiniciar solo frontend
docker-compose restart gapsi-frontend

# Reiniciar solo Redis
docker-compose restart redis

# Reiniciar todo
docker-compose restart
```

### Limpiar y empezar de nuevo

```bash
# Detener y eliminar todo
docker-compose down -v

# Eliminar logs
rm -rf logs/*

# Volver a iniciar
docker-compose up -d
```

## 📊 Health Checks

Los servicios incluyen health checks automáticos:

- **Redis:**

  - Comando: `redis-cli --pass gapsi2024 ping`
  - Intervalo: cada 30s
  - El backend espera a que Redis esté healthy
- **Backend:**

  - Ruta: `/gapsi-test-ecommerce/actuator/health`
  - Intervalo: cada 30s
  - Start period: 90s (tiempo de espera inicial)
  - El frontend espera a que el backend esté healthy
- **Frontend:**

  - Se inicia automáticamente después de que el backend esté healthy
  - No tiene health check propio (nginx es muy liviano)

## 🔄 Actualización

### Actualizar desde Docker Hub

```bash
# Descargar nuevas imágenes
docker-compose pull

# Recrear contenedores con nuevas imágenes
docker-compose up -d
```

### Actualizar desde código fuente

```bash
# 1. Backend
cd ../gapsi-test-backend
git pull
./mvnw clean package -DskipTests
docker build -t oscarygutierrezg/gapsi-test-backend:latest .

# 2. Frontend
cd ../gapsi-test-frontend
git pull
npm install
npm run build
docker build -t oscarygutierrezg/gapsi-test-frontend:latest .

# 3. Reiniciar servicios
cd ../gapsi-test-docker-compose
docker-compose down
docker-compose up -d
```

## 📄 Estructura de Archivos

```
gapsi-test-docker-compose/
├── docker-compose.yml       # Definición de servicios Docker
├── .env                     # Variables de entorno (no versionado)
├── .gitignore              # Archivos ignorados por git
├── README.md               # Este archivo
├── logs/                   # Logs persistentes del backend
│   ├── gapsi-backend.log
│   ├── gapsi-backend-error.log
│   └── archive/            # Logs rotados (30 días)
└── scripts/                # Scripts de utilidad (opcional)
```

## 🔐 Configuración de Seguridad

### Producción

Para usar en producción, cambia los siguientes valores en `.env`:

```env
# Cambiar profile a prod
SPRING_PROFILES_ACTIVE=prod

# Cambiar password de Redis
REDIS_PASSWORD=tu-password-seguro-aqui

# Configurar CORS solo para tu dominio
CORS_ALLOWED_ORIGINS=https://tu-dominio.com

# Usar base de datos real (PostgreSQL/MySQL)
DB_URL=jdbc:postgresql://host:5432/database
DB_USERNAME=usuario
DB_PASSWORD=password-seguro
```

## 🤝 Contribuir

1. Fork el proyecto
2. Crear rama feature (`git checkout -b feature/AmazingFeature`)
3. Commit cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abrir Pull Request

## 📝 Licencia

Este proyecto es privado y confidencial.

---

**Desarrollado para prueba técnica GAPSI** 🚀

```

### Contenedores

| Contenedor | Imagen | Puerto | CPU | RAM | Uso |
|------------|--------|--------|-----|-----|-----|
| **tecnowave-frontend** | oscarygutierrezg/tecnowave-frontend | 443 | 0.5 | 256MB | Interfaz web Angular + Nginx |
| **tecnowave-backend** | oscarygutierrezg/tecnowave-backend | 8080 | 2.5 | 2.5GB | API REST Spring Boot + JasperReports |
| **tecnowave-redis** | redis:7-alpine | 6379 | 0.5 | 512MB | Caché distribuido + sesiones |

### Especificaciones del Servidor

- **CPU:** 4 vCPUs
- **RAM:** 8 GB
- **Disco:** 160 GB SSD
- **OS:** Ubuntu 22.04/24.04 LTS
- **Usuarios concurrentes:** 10
- **Base de datos:** PostgreSQL (servidor externo - Digital Ocean Managed Database)

---

## 📦 Requisitos Previos

### En el servidor Ubuntu:

```bash
# Docker Engine
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Docker Compose Plugin
sudo apt-get update
sudo apt-get install docker-compose-plugin -y

# Verificar instalación
docker --version
docker compose version
```

### Base de Datos PostgreSQL

- PostgreSQL 15+ (servidor separado recomendado)
- Usuario con permisos completos en schema `public`
- SSL/TLS habilitado
- Configuración recomendada:
  - `max_connections`: 50
  - `shared_buffers`: 2GB
  - `effective_cache_size`: 6GB

---

## 🚀 Instalación

### 1. Clonar/Copiar Archivos

```bash
# Opción A: Desde Git
git clone <repository-url> /tmp/tecnowave-setup
cd /tmp/tecnowave-setup

# Opción B: Subir manualmente
scp -r tecnowave-docker-compose/ root@servidor:/tmp/tecnowave-setup
ssh root@servidor
cd /tmp/tecnowave-setup
```

### 2. Ejecutar Script de Instalación

```bash
# Ejecutar con sudo
sudo bash scripts/setup-service.sh
```

**El script automáticamente:**

- ✅ Crea `/opt/tecnowave/` con subdirectorios
- ✅ Copia `docker-compose.yml` y scripts
- ✅ Configura servicio systemd `tecnowave.service`
- ✅ Instala limpieza automática de logs (diaria a las 2:00 AM)
- ✅ Configura logrotate
- ✅ Habilita inicio automático en boot

### 3. Estructura Resultante

```
/opt/tecnowave/
├── docker-compose.yml          # Orquestación de contenedores
├── .env                         # Variables de entorno (crear manualmente)
├── logs/                        # Logs centralizados
│   ├── tecnowave-backend.log
│   ├── tecnowave-backend-error.log
│   └── archive/                 # Logs rotativos (30 días)
├── certificates/                # Certificados SSL PostgreSQL
│   └── ca-certificate.crt
├── ssl/                         # Certificados HTTPS
│   ├── fullchain.pem
│   └── privkey.pem
└── scripts/
    └── cleanup-logs.sh         # Script de limpieza de logs
```

---

## ⚙️ Configuración

### 1. Crear Archivo .env

```bash
sudo nano /opt/tecnowave/.env
```

**Contenido mínimo requerido:**

```bash
# PostgreSQL Database
DB_URL=jdbc:postgresql://host:25060/tecnowave_db
DB_USERNAME=tecnowave-user
DB_PASSWORD=your_db_password
SSL_CERT_PATH=/opt/certificates/ca-certificate.crt

# Logging
LOG_PATH=/opt/tecnowave/logs

# Brevo Email API
BREVO_API_KEY=your_brevo_api_key
BREVO_API_URL=https://api.brevo.com/v3
EMAIL_FROM=noreply@tecnowavespa.com
EMAIL_FROM_NAME=Tecnowave
EMAIL_SANDBOX_MODE=false

# Security
ENCRYPTOR_SECRET=K9mP2xL8nQ4vR6wZ3fG5hJ1dS0aY8bN4
ENCRYPTOR_TRANSFORMATION=AES
JWT_SECRET=9K7mP2xL8nQ4vR6wZ3fG5hJ1dS0aY8bN4cT6eU2iO9pM7qW5rX3tV1yA8zF4gH6jK

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=tecnowave2024
REDIS_DATABASE=0
REDIS_TIMEOUT=5000
REDIS_POOL_MAX_ACTIVE=20
REDIS_POOL_MAX_IDLE=10
REDIS_POOL_MIN_IDLE=5

# CORS
CORS_ALLOWED_ORIGINS=https://tecnowavespa.online,https://www.tecnowavespa.online
```

### 2. Copiar Certificados SSL PostgreSQL

```bash
# Subir certificado desde tu máquina local
scp ca-certificate.crt root@servidor:/opt/tecnowave/certificates/

# O descargar desde Digital Ocean
# (Panel → Databases → tecnowave_db → Connection Details → Download CA Certificate)
```

### 3. Permisos

```bash
sudo chmod 600 /opt/tecnowave/.env
sudo chmod 644 /opt/tecnowave/certificates/ca-certificate.crt
```

---

## 🌐 Despliegue

### Iniciar Aplicación

```bash
# Método 1: Con systemd (recomendado)
sudo systemctl start tecnowave
sudo systemctl status tecnowave

# Método 2: Manual
cd /opt/tecnowave
docker compose up -d
```

### Verificar Estado

```bash
# Ver contenedores
docker ps

# Logs en tiempo real
sudo journalctl -u tecnowave -f

# Health checks
curl http://localhost:8080/api/actuator/health
curl -I https://tecnowavespa.online
```

---

## 🎛 Comandos Disponibles

### Gestión del Servicio

```bash
# Iniciar servicio
sudo systemctl start tecnowave

# Detener servicio
sudo systemctl stop tecnowave

# Reiniciar servicio
sudo systemctl restart tecnowave

# Ver estado
sudo systemctl status tecnowave

# Habilitar inicio automático
sudo systemctl enable tecnowave

# Deshabilitar inicio automático
sudo systemctl disable tecnowave
```

### Gestión de Contenedores

```bash
# Ver contenedores activos
docker ps

# Ver logs de un contenedor
docker logs tecnowave-backend -f
docker logs tecnowave-frontend -f
docker logs tecnowave-redis -f

# Entrar a un contenedor
docker exec -it tecnowave-backend bash
docker exec -it tecnowave-redis redis-cli

# Ver uso de recursos
docker stats

# Ver redes
docker network ls
docker network inspect tecnowave-network
```

### Logs del Sistema

```bash
# Logs de systemd (servicio)
sudo journalctl -u tecnowave -f
sudo journalctl -u tecnowave --since "1 hour ago"
sudo journalctl -u tecnowave --since today

# Logs de aplicación (backend)
sudo tail -f /opt/tecnowave/logs/tecnowave-backend.log
sudo tail -f /opt/tecnowave/logs/tecnowave-backend-error.log

# Ver logs archivados
ls -lh /opt/tecnowave/logs/archive/

# Buscar errores
sudo grep -i "error" /opt/tecnowave/logs/tecnowave-backend.log
```

---

## 📊 Monitoreo y Logs

### Health Checks

```bash
# Backend health
curl http://localhost:8080/api/actuator/health

# Frontend (debe responder 200)
curl -I https://tecnowavespa.online

# Redis
docker exec tecnowave-redis redis-cli --pass tecnowave2024 ping
```

### Métricas del Backend

```bash
# Endpoints de Actuator
curl http://localhost:8080/api/actuator/metrics
curl http://localhost:8080/api/actuator/info
curl http://localhost:8080/api/actuator/env
```

### Monitoreo de Recursos

```bash
# CPU y RAM en tiempo real
docker stats

# Disco
df -h /opt/tecnowave
du -sh /opt/tecnowave/logs/*

# Conexiones Redis
docker exec tecnowave-redis redis-cli --pass tecnowave2024 INFO stats

# Conexiones PostgreSQL (desde servidor DB)
SELECT count(*) FROM pg_stat_activity WHERE datname = 'tecnowave_db';
```

### Limpieza Automática de Logs

```bash
# Ver estado del timer
sudo systemctl status tecnowave-cleanup.timer
sudo systemctl list-timers tecnowave-cleanup.timer

# Ejecutar limpieza manual
sudo bash /opt/tecnowave/scripts/cleanup-logs.sh

# Ver última ejecución
sudo journalctl -u tecnowave-cleanup.service -n 50
```

**Política de limpieza:**

- 📅 Ejecución diaria a las **2:00 AM**
- 🗄️ Retención de logs: **30 días**
- 📦 Compresión automática con GZIP
- 🚮 Limpieza de logs JSON de Docker
- 💾 Cap total: 10 GB (aplicación) + 5 GB (errores)

---

## 🔄 Actualización

### Actualizar Contenedores

```bash
# Método 1: Con systemd (recomendado)
sudo systemctl reload tecnowave

# Método 2: Manual
cd /opt/tecnowave
docker compose pull
docker compose up -d --remove-orphans
```

### Actualizar Configuración

```bash
# 1. Editar .env
sudo nano /opt/tecnowave/.env

# 2. Reiniciar servicio
sudo systemctl restart tecnowave
```

### Rollback

```bash
# Ver historial de imágenes
docker images | grep tecnowave

# Usar tag específico en docker-compose.yml
# image: oscarygutierrezg/tecnowave-backend:v1.2.3

# Aplicar
sudo systemctl restart tecnowave
```

---

## 🔒 SSL/HTTPS

### Configurar Certificado Let's Encrypt

```bash
# 1. Instalar Certbot
sudo apt update
sudo apt install certbot python3-certbot-nginx -y

# 2. Detener servicios
sudo systemctl stop tecnowave

# 3. Obtener certificado
sudo certbot certonly --standalone \
  -d tecnowavespa.online \
  -d www.tecnowavespa.online \
  --email adm@tecnowavespa.com \
  --agree-tos \
  --no-eff-email

# 4. Copiar certificados
sudo mkdir -p /opt/tecnowave/ssl
sudo cp /etc/letsencrypt/live/tecnowavespa.online/fullchain.pem /opt/tecnowave/ssl/
sudo cp /etc/letsencrypt/live/tecnowavespa.online/privkey.pem /opt/tecnowave/ssl/
sudo chmod 644 /opt/tecnowave/ssl/*.pem

# 5. Iniciar servicios
sudo systemctl start tecnowave
```

### Renovación Automática

```bash
# Crear script de renovación
sudo nano /etc/cron.d/certbot-renew
```

**Contenido:**

```bash
0 3 * * * root certbot renew --quiet --deploy-hook "cp /etc/letsencrypt/live/tecnowavespa.online/*.pem /opt/tecnowave/ssl/ && systemctl restart tecnowave"
```

### Verificar SSL

```bash
# Test local
curl -I https://tecnowavespa.online

# SSL Labs Test
open https://www.ssllabs.com/ssltest/analyze.html?d=tecnowavespa.online
```

---

## 🐛 Troubleshooting

### Backend no inicia

```bash
# Ver logs detallados
sudo journalctl -u tecnowave -n 100
docker logs tecnowave-backend --tail 200

# Verificar .env
sudo cat /opt/tecnowave/.env | grep -v PASSWORD

# Probar conexión a base de datos
docker exec -it tecnowave-backend bash
curl http://localhost:8080/api/actuator/health
```

### Error: "permission denied for schema public"

```sql
-- Conectar como admin a PostgreSQL
GRANT ALL ON SCHEMA public TO "tecnowave-user";
GRANT USAGE, CREATE ON SCHEMA public TO "tecnowave-user";
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO "tecnowave-user";
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO "tecnowave-user";
```

### Redis Connection Failed

```bash
# Verificar contenedor
docker ps | grep redis

# Test conexión
docker exec tecnowave-redis redis-cli --pass tecnowave2024 ping

# Ver logs
docker logs tecnowave-redis

# Verificar password en .env
sudo grep REDIS_PASSWORD /opt/tecnowave/.env
```

### Frontend no carga

```bash
# Ver logs de nginx
docker logs tecnowave-frontend

# Verificar puerto 443
sudo netstat -tlnp | grep 443

# Test directo
curl -k https://localhost:443

# Verificar certificados
sudo ls -la /opt/tecnowave/ssl/
```

### DNS no resuelve

```bash
# Verificar propagación
dig tecnowavespa.online +short
nslookup tecnowavespa.online

# Test desde diferentes DNS
dig @8.8.8.8 tecnowavespa.online
dig @1.1.1.1 tecnowavespa.online

# Verificar registros A en Namecheap
# @ → 146.190.241.51
# www → 146.190.241.51
```

### Contenedores reiniciando

```bash
# Ver razón del reinicio
docker inspect tecnowave-backend | grep -A 10 "State"

# Ver recursos disponibles
free -h
df -h

# Ajustar límites en docker-compose.yml si es necesario
```

---

## 🔧 Mantenimiento

### Backup de Configuración

```bash
# Backup completo de /opt/tecnowave
sudo tar -czf /backup/tecnowave-$(date +%Y%m%d).tar.gz \
  /opt/tecnowave/.env \
  /opt/tecnowave/docker-compose.yml \
  /opt/tecnowave/certificates/ \
  /opt/tecnowave/ssl/

# Descargar a local
scp root@servidor:/backup/tecnowave-*.tar.gz ~/backups/
```

### Limpieza Manual

```bash
# Limpiar imágenes no usadas
docker image prune -a

# Limpiar volúmenes no usados
docker volume prune

# Limpiar todo el sistema Docker
docker system prune -a --volumes

# Limpiar logs manualmente
sudo bash /opt/tecnowave/scripts/cleanup-logs.sh
```

### Base de Datos

```bash
# Backup PostgreSQL (desde servidor DB)
pg_dump -h host -U tecnowave-user tecnowave_db > backup.sql

# Restore
psql -h host -U tecnowave-user tecnowave_db < backup.sql

# Verificar migraciones Flyway
SELECT * FROM flyway_schema_history ORDER BY installed_rank DESC;
```

### Monitoreo a Largo Plazo

Considerar instalar:

- **Prometheus + Grafana:** Métricas y dashboards
- **Netdata:** Monitoreo en tiempo real del sistema
- **Uptime Kuma:** Monitoreo uptime y alertas
- **Logstash/ELK:** Análisis avanzado de logs

---

## 📞 Soporte

### Información del Sistema

```bash
# Versiones
docker --version
docker compose version
cat /etc/os-release

# Estado general del servidor
sudo systemctl status tecnowave
docker ps
df -h
free -h
uptime
```

### Logs para Debugging

```bash
# Generar reporte completo
{
  echo "=== System Info ==="
  uname -a
  docker --version
  
  echo -e "\n=== Service Status ==="
  sudo systemctl status tecnowave --no-pager
  
  echo -e "\n=== Containers ==="
  docker ps -a
  
  echo -e "\n=== Backend Logs (últimas 50 líneas) ==="
  docker logs tecnowave-backend --tail 50
  
  echo -e "\n=== Resources ==="
  docker stats --no-stream
} > /tmp/tecnowave-debug-$(date +%Y%m%d-%H%M%S).log
```

---

## 📝 Notas

- **Puerto 8080:** Expuesto para el backend (solo accesible localmente o vía proxy Nginx)
- **Puerto 443:** Expuesto para el frontend (HTTPS público)
- **Puerto 6379:** Redis (solo interno en red Docker)
- **JVM Heap:** Configurado para 2GB max (-Xmx2g)
- **Logs:** Máximo 30 días de retención, comprimidos automáticamente
- **Health checks:** Backend tarda 90s en estar listo (start_period)

---

## 🔗 Enlaces Útiles

- **Aplicación:** https://tecnowavespa.online
- **API Health:** http://servidor:8080/api/actuator/health
- **Swagger:** http://servidor:8080/api/swagger-ui.html
- **Docker Hub:** https://hub.docker.com/u/oscarygutierrezg

---

## 📄 Licencia

Uso interno - Importadora Tecnowave SPA © 2026
