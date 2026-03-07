# Solución: Docker Compose para Desarrollo Local con Microservicios

## 📋 Tabla de Contenidos

1. [Contexto del Problema](#contexto-del-problema)
2. [Descripción de la Solución](#descripción-de-la-solución)
3. [Arquitectura Propuesta](#arquitectura-propuesta)
4. [Estructura de Archivos](#estructura-de-archivos)
5. [Configuración Paso a Paso](#configuración-paso-a-paso)
6. [Docker Compose Configuration](#docker-compose-configuration)
7. [Scripts de Utilidad](#scripts-de-utilidad)
8. [Configuración del Backoffice](#configuración-del-backoffice)
9. [Troubleshooting](#troubleshooting)
10. [Mejores Prácticas](#mejores-prácticas)
11. [Consideraciones y Limitaciones](#consideraciones-y-limitaciones)

---

## 🎯 Contexto del Problema

### Situación Actual

En el desarrollo de microservicios con arquitectura distribuida, nos enfrentamos al siguiente escenario:

1. **API de Autenticación (`ave-authentication`)**: Expone un endpoint `/api/auth/login` que genera tokens JWT firmados con una llave privada local (`jwtRS256.key`).

2. **Otras APIs**: Cada microservicio valida los tokens JWT usando su propia llave pública (`jwtRS256.key.pub`).

3. **Problema Principal**: 
   - Las llaves de firma del token son **locales** (distintas a las del servidor de desarrollo)
   - Cuando se hace login localmente, el token se firma con la llave privada local
   - Las APIs en el servidor de desarrollo esperan tokens firmados con la llave privada del servidor
   - Resultado: **Error 401 Unauthorized** al intentar acceder a módulos del backoffice

4. **Escenario de Trabajo**:
   - Cuando trabajamos en una API específica, agregamos la ruta local y desactivamos la validación de tokens (flag de desarrollo)
   - El problema surge cuando trabajamos en el proyecto `ave-authentication` y necesitamos probar con el backoffice
   - El backoffice necesita hacer login y luego usar ese token para acceder a otras APIs

5. **Desafío**:
   - Levantar localmente las 40+ APIs no es viable
   - Necesitamos una solución que permita trabajar de manera fluida como si fuese en el ambiente de desarrollo

---

## 💡 Descripción de la Solución

### Opción 1: Docker Compose con APIs Seleccionadas (Recomendada)

**Concepto**: Levantar solo las APIs necesarias en contenedores Docker, todas compartiendo las mismas llaves JWT para que los tokens generados localmente sean válidos en todas las APIs locales.

### Ventajas

✅ **Aislamiento completo**: No afecta otros entornos (desarrollo, staging, producción)  
✅ **Consistencia de llaves**: Todas las APIs locales usan las mismas llaves JWT  
✅ **Productividad**: Permite probar flujos completos sin depender del servidor de desarrollo  
✅ **Escalabilidad**: Fácil agregar o quitar APIs según necesidad  
✅ **Reproducibilidad**: Todo el equipo puede usar la misma configuración  
✅ **Control total**: Controlas qué APIs están corriendo y en qué puertos  
✅ **Sin cambios en código**: No requiere modificar el código de las APIs existentes  

### Desventajas

⚠️ **Configuración inicial**: Requiere tiempo para configurar el `docker-compose.yml`  
⚠️ **Recursos**: Consume recursos del sistema (RAM, CPU)  
⚠️ **Mantenimiento**: Necesita actualizarse cuando cambian las dependencias de las APIs  

---

## 🏗️ Arquitectura Propuesta

```
┌─────────────────────────────────────────────────────────────┐
│                    DESARROLLADOR LOCAL                       │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Docker Compose Network                      │   │
│  │                                                       │   │
│  │  ┌──────────────────┐  ┌──────────────────┐         │   │
│  │  │  ave-auth        │  │  ave-api-2       │         │   │
│  │  │  :3001           │  │  :3002           │         │   │
│  │  │                  │  │                  │         │   │
│  │  │  JWT Keys:       │  │  JWT Keys:       │         │   │
│  │  │  /jwt-keys/      │  │  /jwt-keys/      │         │   │
│  │  │  (shared volume) │  │  (shared volume) │         │   │
│  │  └──────────────────┘  └──────────────────┘         │   │
│  │                                                       │   │
│  │  ┌──────────────────┐  ┌──────────────────┐         │   │
│  │  │  ave-api-3       │  │  ave-api-4       │         │   │
│  │  │  :3003           │  │  :3004           │         │   │
│  │  │                  │  │                  │         │   │
│  │  │  JWT Keys:       │  │  JWT Keys:       │         │   │
│  │  │  /jwt-keys/      │  │  /jwt-keys/      │         │   │
│  │  └──────────────────┘  └──────────────────┘         │   │
│  │                                                       │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │        Shared Volume: jwt-keys/               │   │   │
│  │  │        - jwtRS256.key (private)              │   │   │
│  │  │        - jwtRS256.key.pub (public)           │   │   │
│  │  └──────────────────────────────────────────────┘   │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Backoffice Frontend                        │   │
│  │           (ave-backoffice)                           │   │
│  │                                                       │   │
│  │  Environment Config:                                 │   │
│  │  - AUTH_API: http://localhost:3001                  │   │
│  │  - API_2: http://localhost:3002                      │   │
│  │  - API_3: http://localhost:3003                      │   │
│  │  - API_4: http://localhost:3004                      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Flujo de Autenticación

1. **Usuario hace login en backoffice** → Request a `http://localhost:3001/api/auth/login`
2. **API de autenticación genera token** → Firma con `jwtRS256.key` (privada)
3. **Backoffice recibe token** → Almacena en localStorage/sessionStorage
4. **Backoffice hace request a otra API** → Envía token en header `Authorization: Bearer <token>`
5. **API destino valida token** → Usa `jwtRS256.key.pub` (pública) del volumen compartido
6. **Token válido** → Request procesado exitosamente ✅

---

## 📁 Estructura de Archivos

```
apis/
├── docker-compose.yml              # Configuración principal de Docker Compose
├── docker-compose.override.yml     # Overrides para desarrollo (opcional)
├── .env.docker                     # Variables de entorno para Docker
├── jwt-keys/                       # Directorio compartido para llaves JWT
│   ├── jwtRS256.key               # Llave privada (NO COMMITEAR)
│   └── jwtRS256.key.pub           # Llave pública
├── scripts/                        # Scripts de utilidad
│   ├── start-local-apis.sh        # Iniciar APIs seleccionadas
│   ├── stop-local-apis.sh         # Detener APIs
│   ├── restart-api.sh             # Reiniciar una API específica
│   ├── logs-api.sh                # Ver logs de una API
│   └── generate-jwt-keys.sh       # Generar nuevas llaves JWT
├── ave-authentication/
│   └── Dockerfile
├── ave-api-2/
│   └── Dockerfile
├── ave-api-3/
│   └── Dockerfile
└── ave-api-4/
    └── Dockerfile
```

### Archivos a Crear

1. **`docker-compose.yml`**: Configuración principal
2. **`.env.docker`**: Variables de entorno compartidas
3. **`scripts/start-local-apis.sh`**: Script para iniciar APIs
4. **`scripts/stop-local-apis.sh`**: Script para detener APIs
5. **`jwt-keys/.gitignore`**: Para no commitear las llaves

---

## 🔧 Configuración Paso a Paso

### Paso 1: Crear Directorio para Llaves JWT

```bash
# Desde la raíz de apis/
mkdir jwt-keys
cd jwt-keys
```

### Paso 2: Generar o Copiar Llaves JWT

#### Opción A: Generar Nuevas Llaves (Recomendado para desarrollo local)

```bash
# Generar par de llaves RSA
openssl genrsa -out jwtRS256.key 2048
openssl rsa -in jwtRS256.key -pubout -out jwtRS256.key.pub

# Verificar permisos (importante para seguridad)
chmod 600 jwtRS256.key
chmod 644 jwtRS256.key.pub
```

#### Opción B: Copiar Llaves del Servidor de Desarrollo

Si necesitas usar las mismas llaves que el servidor de desarrollo (para compatibilidad):

```bash
# Copiar desde el servidor (ajustar ruta según tu caso)
scp user@dev-server:/path/to/jwtRS256.key ./jwtRS256.key
scp user@dev-server:/path/to/jwtRS256.key.pub ./jwtRS256.key.pub
```

⚠️ **Nota de Seguridad**: Las llaves privadas nunca deben committearse al repositorio.

### Paso 3: Crear .gitignore para jwt-keys

```bash
# jwt-keys/.gitignore
jwtRS256.key
jwtRS256.key.pub
*.key
*.key.pub
```

### Paso 4: Preparar Dockerfiles (si no existen)

Cada API debe tener un `Dockerfile`. El de `ave-authentication` ya existe:

```dockerfile
# Ejemplo: ave-authentication/Dockerfile
FROM node:20-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
CMD ["node", "dist/src/main"]
```

### Paso 5: Crear Archivo de Variables de Entorno

Crear `.env.docker` en la raíz de `apis/`:

```env
# .env.docker
# Configuración compartida para todas las APIs

# JWT Keys Paths (dentro del contenedor)
JWT_PRIVATE_KEY_PATH=/app/jwt-keys/jwtRS256.key
JWT_PUBLIC_KEY_PATH=/app/jwt-keys/jwtRS256.key.pub

# MongoDB (ajustar según tu configuración)
MONGODB_URI=mongodb://mongo:27017/ave-local

# Network
NETWORK_NAME=ave-local-network

# Ports base (cada API incrementa en 1)
AUTH_PORT=3001
API_2_PORT=3002
API_3_PORT=3003
API_4_PORT=3004
```

---

## 🐳 Docker Compose Configuration

### docker-compose.yml Completo

```yaml
version: '3.8'

services:
  # MongoDB (opcional, si no usas MongoDB externo)
  mongo:
    image: mongo:7.0
    container_name: ave-mongo-local
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    networks:
      - ave-local-network
    environment:
      - MONGO_INITDB_DATABASE=ave-local
    restart: unless-stopped

  # API de Autenticación
  ave-authentication:
    build:
      context: ./ave-authentication
      dockerfile: Dockerfile
    container_name: ave-auth-local
    ports:
      - "${AUTH_PORT:-3001}:3000"
    volumes:
      - ./jwt-keys:/app/jwt-keys:ro  # Read-only para seguridad
      - ./ave-authentication:/usr/src/app  # Para hot-reload (opcional)
      - /usr/src/app/node_modules  # Excluir node_modules del volumen
    environment:
      # JWT Configuration
      - JWT_PRIVATE_KEY_PATH=/app/jwt-keys/jwtRS256.key
      - JWT_PUBLIC_KEY_PATH=/app/jwt-keys/jwtRS256.key.pub
      - JWT_ISSUER=ave-local
      
      # Database
      - DATABASE_URL=mongodb://mongo:27017/ave-authentication?authSource=admin
      
      # Server Configuration
      - AVE_AUTHENTICATION_PORT=3000
      - NODE_ENV=development
      
      # Swagger
      - AVE_AUTHENTICATION_SWAGGER_SERVER=http://localhost:3001
      - AVE_AUTHENTICATION_SWAGGER_TITLE=AVE Authentication API (Local)
      - AVE_AUTHENTICATION_SWAGGER_DESCRIPTION=API de autenticación para desarrollo local
      
      # PowerSync (ajustar según necesidad)
      - POWERSYNC_PRIVATE_KEY=${POWERSYNC_PRIVATE_KEY:-}
      - POWERSYNC_PUBLIC_KEY=${POWERSYNC_PUBLIC_KEY:-}
      - POWERSYNC_URL=${POWERSYNC_URL:-}
      
      # Other
      - ISOLATED_TOKEN_SECRET=${ISOLATED_TOKEN_SECRET:-local-secret-key}
      - FALLBACK_LANGUAGE=es
      - TEMP_PASSWORD_LENGTH=12
      - TEMP_PASSWORD_MAX_HOURS=8
    networks:
      - ave-local-network
    depends_on:
      - mongo
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000/api/ping"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # API 2 (Ejemplo - ajustar según tus APIs)
  ave-api-2:
    build:
      context: ./ave-api-2
      dockerfile: Dockerfile
    container_name: ave-api-2-local
    ports:
      - "${API_2_PORT:-3002}:3000"
    volumes:
      - ./jwt-keys:/app/jwt-keys:ro
    environment:
      - JWT_PUBLIC_KEY_PATH=/app/jwt-keys/jwtRS256.key.pub
      - DATABASE_URL=mongodb://mongo:27017/ave-api-2?authSource=admin
      - PORT=3000
      - NODE_ENV=development
    networks:
      - ave-local-network
    depends_on:
      - mongo
      - ave-authentication
    restart: unless-stopped

  # API 3 (Ejemplo - ajustar según tus APIs)
  ave-api-3:
    build:
      context: ./ave-api-3
      dockerfile: Dockerfile
    container_name: ave-api-3-local
    ports:
      - "${API_3_PORT:-3003}:3000"
    volumes:
      - ./jwt-keys:/app/jwt-keys:ro
    environment:
      - JWT_PUBLIC_KEY_PATH=/app/jwt-keys/jwtRS256.key.pub
      - DATABASE_URL=mongodb://mongo:27017/ave-api-3?authSource=admin
      - PORT=3000
      - NODE_ENV=development
    networks:
      - ave-local-network
    depends_on:
      - mongo
      - ave-authentication
    restart: unless-stopped

  # API 4 (Ejemplo - ajustar según tus APIs)
  ave-api-4:
    build:
      context: ./ave-api-4
      dockerfile: Dockerfile
    container_name: ave-api-4-local
    ports:
      - "${API_4_PORT:-3004}:3000"
    volumes:
      - ./jwt-keys:/app/jwt-keys:ro
    environment:
      - JWT_PUBLIC_KEY_PATH=/app/jwt-keys/jwtRS256.key.pub
      - DATABASE_URL=mongodb://mongo:27017/ave-api-4?authSource=admin
      - PORT=3000
      - NODE_ENV=development
    networks:
      - ave-local-network
    depends_on:
      - mongo
      - ave-authentication
    restart: unless-stopped

networks:
  ave-local-network:
    driver: bridge
    name: ave-local-network

volumes:
  mongo-data:
    name: ave-mongo-local-data
```

### docker-compose.override.yml (Opcional)

Para desarrollo con hot-reload y otras configuraciones específicas:

```yaml
version: '3.8'

services:
  ave-authentication:
    volumes:
      - ./ave-authentication:/usr/src/app
      - /usr/src/app/node_modules
    command: npm run start:dev  # Hot reload
    environment:
      - NODE_ENV=development
```

---

## 📜 Scripts de Utilidad

### scripts/start-local-apis.sh

```bash
#!/bin/bash

# Script para iniciar las APIs locales en Docker Compose
# Uso: ./scripts/start-local-apis.sh [api1] [api2] ...

set -e

# Colores para output
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m' # No Color

echo -e "${GREEN}🚀 Iniciando APIs locales...${NC}"

# Verificar que existe docker-compose.yml
if [ ! -f "docker-compose.yml" ]; then
    echo -e "${RED}❌ Error: docker-compose.yml no encontrado${NC}"
    exit 1
fi

# Verificar que existen las llaves JWT
if [ ! -f "jwt-keys/jwtRS256.key" ] || [ ! -f "jwt-keys/jwtRS256.key.pub" ]; then
    echo -e "${YELLOW}⚠️  Advertencia: Llaves JWT no encontradas${NC}"
    echo -e "${YELLOW}   Ejecuta: ./scripts/generate-jwt-keys.sh${NC}"
    read -p "¿Continuar de todas formas? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi

# Si se pasan argumentos, iniciar solo esas APIs
if [ $# -gt 0 ]; then
    SERVICES="$@"
    echo -e "${GREEN}Iniciando servicios: ${SERVICES}${NC}"
    docker-compose up -d $SERVICES
else
    # Iniciar todas las APIs por defecto
    echo -e "${GREEN}Iniciando todas las APIs...${NC}"
    docker-compose up -d
fi

# Esperar a que los servicios estén listos
echo -e "${YELLOW}⏳ Esperando a que los servicios estén listos...${NC}"
sleep 5

# Verificar estado
echo -e "${GREEN}📊 Estado de los servicios:${NC}"
docker-compose ps

echo -e "${GREEN}✅ APIs iniciadas correctamente${NC}"
echo -e "${GREEN}📝 Para ver logs: docker-compose logs -f [service-name]${NC}"
```

### scripts/stop-local-apis.sh

```bash
#!/bin/bash

# Script para detener las APIs locales
# Uso: ./scripts/stop-local-apis.sh [api1] [api2] ...

set -e

GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

echo -e "${YELLOW}🛑 Deteniendo APIs locales...${NC}"

if [ $# -gt 0 ]; then
    SERVICES="$@"
    echo -e "Deteniendo servicios: ${SERVICES}"
    docker-compose stop $SERVICES
else
    docker-compose down
fi

echo -e "${GREEN}✅ APIs detenidas${NC}"
```

### scripts/restart-api.sh

```bash
#!/bin/bash

# Script para reiniciar una API específica
# Uso: ./scripts/restart-api.sh ave-authentication

set -e

if [ -z "$1" ]; then
    echo "❌ Error: Debes especificar el nombre del servicio"
    echo "Uso: ./scripts/restart-api.sh <service-name>"
    exit 1
fi

SERVICE=$1

echo "🔄 Reiniciando $SERVICE..."
docker-compose restart $SERVICE
echo "✅ $SERVICE reiniciado"
```

### scripts/logs-api.sh

```bash
#!/bin/bash

# Script para ver logs de una API
# Uso: ./scripts/logs-api.sh ave-authentication [--follow]

set -e

if [ -z "$1" ]; then
    echo "❌ Error: Debes especificar el nombre del servicio"
    echo "Uso: ./scripts/logs-api.sh <service-name> [--follow]"
    exit 1
fi

SERVICE=$1
FOLLOW=${2:-""}

if [ "$FOLLOW" == "--follow" ] || [ "$FOLLOW" == "-f" ]; then
    docker-compose logs -f $SERVICE
else
    docker-compose logs --tail=100 $SERVICE
fi
```

### scripts/generate-jwt-keys.sh

```bash
#!/bin/bash

# Script para generar nuevas llaves JWT
# Uso: ./scripts/generate-jwt-keys.sh

set -e

GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

KEYS_DIR="jwt-keys"

echo -e "${GREEN}🔑 Generando nuevas llaves JWT...${NC}"

# Crear directorio si no existe
mkdir -p $KEYS_DIR

# Verificar si ya existen llaves
if [ -f "$KEYS_DIR/jwtRS256.key" ]; then
    echo -e "${YELLOW}⚠️  Advertencia: Ya existen llaves JWT${NC}"
    read -p "¿Sobrescribir? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        echo "Operación cancelada"
        exit 0
    fi
fi

# Generar llave privada
echo "Generando llave privada..."
openssl genrsa -out "$KEYS_DIR/jwtRS256.key" 2048

# Generar llave pública
echo "Generando llave pública..."
openssl rsa -in "$KEYS_DIR/jwtRS256.key" -pubout -out "$KEYS_DIR/jwtRS256.key.pub"

# Ajustar permisos
chmod 600 "$KEYS_DIR/jwtRS256.key"
chmod 644 "$KEYS_DIR/jwtRS256.key.pub"

echo -e "${GREEN}✅ Llaves generadas exitosamente${NC}"
echo -e "${GREEN}   Privada: $KEYS_DIR/jwtRS256.key${NC}"
echo -e "${GREEN}   Pública: $KEYS_DIR/jwtRS256.key.pub${NC}"
```

### Hacer Scripts Ejecutables

```bash
chmod +x scripts/*.sh
```

---

## 🖥️ Configuración del Backoffice

### Opción A: Variables de Entorno en el Backoffice

En el backoffice (`ave-backoffice`), crear o modificar el archivo de configuración de entorno:

```typescript
// src/config/environment.local.ts o .env.local

export const environment = {
  production: false,
  apiUrls: {
    authentication: 'http://localhost:3001',
    api2: 'http://localhost:3002',
    api3: 'http://localhost:3003',
    api4: 'http://localhost:3004',
  },
  // ... otras configuraciones
};
```

### Opción B: Archivo de Configuración Dinámico

Crear un archivo JSON que el backoffice lea al iniciar:

```json
// public/config/local-apis.json
{
  "apis": {
    "authentication": "http://localhost:3001",
    "api2": "http://localhost:3002",
    "api3": "http://localhost:3003",
    "api4": "http://localhost:3004"
  }
}
```

### Opción C: Script de Inicio del Backoffice

Crear un script que configure las URLs antes de iniciar:

```bash
#!/bin/bash
# scripts/start-backoffice-local.sh

export REACT_APP_AUTH_API=http://localhost:3001
export REACT_APP_API_2=http://localhost:3002
export REACT_APP_API_3=http://localhost:3003
export REACT_APP_API_4=http://localhost:3004

cd ../frontend/new/ave-backoffice
npm start
```

---

## 🔍 Troubleshooting

### Problema 1: Error al leer llaves JWT

**Síntoma**: 
```
Error loading JWT keys: Private key file not found: /app/jwt-keys/jwtRS256.key
```

**Solución**:
1. Verificar que las llaves existen en `jwt-keys/`
2. Verificar que el volumen está montado correctamente en `docker-compose.yml`
3. Verificar permisos: `chmod 600 jwt-keys/jwtRS256.key`

### Problema 2: Puerto ya en uso

**Síntoma**:
```
Error: bind: address already in use
```

**Solución**:
1. Verificar qué proceso usa el puerto: `netstat -ano | findstr :3001` (Windows) o `lsof -i :3001` (Linux/Mac)
2. Cambiar el puerto en `docker-compose.yml` o `.env.docker`
3. Detener el proceso que está usando el puerto

### Problema 3: Token inválido (401)

**Síntoma**: 
- Login exitoso pero requests a otras APIs devuelven 401

**Solución**:
1. Verificar que todas las APIs usan la misma llave pública
2. Verificar que el token se envía correctamente en el header: `Authorization: Bearer <token>`
3. Verificar logs de la API que rechaza: `docker-compose logs ave-api-2`
4. Verificar que el token no ha expirado

### Problema 4: MongoDB connection error

**Síntoma**:
```
MongooseError: connect ECONNREFUSED
```

**Solución**:
1. Verificar que el servicio `mongo` está corriendo: `docker-compose ps mongo`
2. Verificar la URL de conexión en las variables de entorno
3. Si usas MongoDB externo, ajustar `DATABASE_URL` en `.env.docker`

### Problema 5: Build falla

**Síntoma**:
```
ERROR: failed to solve: process "/bin/sh -c npm install" did not complete successfully
```

**Solución**:
1. Verificar que el `Dockerfile` es correcto
2. Limpiar cache de Docker: `docker builder prune`
3. Rebuild sin cache: `docker-compose build --no-cache`

### Comandos Útiles de Debugging

```bash
# Ver logs de todos los servicios
docker-compose logs -f

# Ver logs de un servicio específico
docker-compose logs -f ave-authentication

# Ver estado de los servicios
docker-compose ps

# Entrar a un contenedor
docker-compose exec ave-authentication sh

# Ver variables de entorno de un contenedor
docker-compose exec ave-authentication env

# Reiniciar un servicio
docker-compose restart ave-authentication

# Rebuild y restart
docker-compose up -d --build ave-authentication
```

---

## ✨ Mejores Prácticas

### 1. Gestión de Llaves JWT

- ✅ **Nunca commitees llaves privadas** al repositorio
- ✅ Usa `.gitignore` para excluir `jwt-keys/*.key`
- ✅ Genera llaves diferentes para cada entorno (local, dev, staging, prod)
- ✅ Usa permisos restrictivos: `chmod 600` para privadas, `chmod 644` para públicas
- ✅ Considera usar un secret manager (Vault, AWS Secrets Manager) en producción

### 2. Organización de Docker Compose

- ✅ Separa configuraciones por archivo: `docker-compose.yml` (base) y `docker-compose.override.yml` (desarrollo)
- ✅ Usa variables de entorno desde `.env.docker` en lugar de hardcodear valores
- ✅ Define healthchecks para todos los servicios
- ✅ Usa `depends_on` para definir orden de inicio
- ✅ Usa `restart: unless-stopped` para servicios críticos

### 3. Performance y Recursos

- ✅ Limita recursos si es necesario:
  ```yaml
  deploy:
    resources:
      limits:
        cpus: '0.5'
        memory: 512M
  ```
- ✅ Usa `.dockerignore` para excluir archivos innecesarios del build
- ✅ Considera usar multi-stage builds para imágenes más pequeñas

### 4. Desarrollo

- ✅ Usa volúmenes para hot-reload durante desarrollo
- ✅ Mantén scripts de utilidad actualizados y documentados
- ✅ Documenta qué APIs están incluidas y por qué
- ✅ Crea un README específico para el setup local

### 5. Seguridad

- ✅ Monta volúmenes de llaves como `read-only` (`:ro`)
- ✅ No expongas servicios innecesarios a la red host
- ✅ Usa networks de Docker para aislar servicios
- ✅ No uses credenciales de producción en desarrollo local

---

## ⚠️ Consideraciones y Limitaciones

### Limitaciones

1. **Recursos del Sistema**: 
   - Cada contenedor consume RAM y CPU
   - Con 4 APIs + MongoDB, espera ~2-4GB de RAM
   - Considera cerrar otras aplicaciones pesadas

2. **Sincronización de Datos**:
   - Las bases de datos locales están vacías
   - Necesitarás seedear datos iniciales o migrar desde desarrollo

3. **Dependencias Externas**:
   - Si las APIs dependen de servicios externos (Redis, RabbitMQ, etc.), necesitarás configurarlos también

4. **Actualizaciones**:
   - Cuando cambias código, necesitas rebuild: `docker-compose up -d --build`
   - Con hot-reload configurado, los cambios se reflejan automáticamente

### Cuándo NO Usar Esta Solución

- ❌ Si solo trabajas en una API aislada (usa el método actual)
- ❌ Si no tienes Docker instalado o no puedes usarlo
- ❌ Si necesitas probar con datos reales del servidor de desarrollo constantemente
- ❌ Si el proyecto es muy pequeño (overhead no justificado)

### Cuándo SÍ Usar Esta Solución

- ✅ Cuando trabajas en el proyecto de autenticación y necesitas probar con el backoffice
- ✅ Cuando necesitas probar flujos completos que involucran múltiples APIs
- ✅ Cuando quieres independencia del servidor de desarrollo
- ✅ Cuando trabajas en features que requieren integración entre APIs
- ✅ Cuando quieres un ambiente de desarrollo reproducible para todo el equipo

---

## 📚 Recursos Adicionales

### Documentación

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Networking](https://docs.docker.com/network/)
- [JWT Best Practices](https://tools.ietf.org/html/rfc8725)

### Comandos de Referencia Rápida

```bash
# Iniciar todas las APIs
docker-compose up -d

# Iniciar APIs específicas
docker-compose up -d ave-authentication ave-api-2

# Ver logs
docker-compose logs -f

# Detener todo
docker-compose down

# Rebuild y restart
docker-compose up -d --build

# Ver estado
docker-compose ps

# Limpiar todo (incluyendo volúmenes)
docker-compose down -v
```

---

## 📝 Checklist de Implementación

- [ ] Crear directorio `jwt-keys/` y generar/copiar llaves
- [ ] Crear `.gitignore` para `jwt-keys/`
- [ ] Crear `docker-compose.yml` con las APIs necesarias
- [ ] Crear `.env.docker` con variables de entorno
- [ ] Crear scripts de utilidad (`start-local-apis.sh`, etc.)
- [ ] Verificar que todos los `Dockerfile` existen y son correctos
- [ ] Probar build de cada API: `docker-compose build`
- [ ] Iniciar servicios: `docker-compose up -d`
- [ ] Verificar que todas las APIs están corriendo: `docker-compose ps`
- [ ] Probar login desde el backoffice
- [ ] Probar requests a otras APIs con el token
- [ ] Configurar backoffice para usar URLs locales
- [ ] Documentar qué APIs están incluidas y por qué
- [ ] Compartir configuración con el equipo

---

## 🎯 Próximos Pasos

1. **Revisar este documento** con el equipo
2. **Identificar las 4 APIs específicas** que necesitas
3. **Adaptar el `docker-compose.yml`** con las configuraciones reales de tus APIs
4. **Probar la solución** con un caso de uso real
5. **Iterar y mejorar** según feedback del equipo

---

**¿Preguntas o necesitas ayuda con la implementación?** 

Este documento es un punto de partida. Ajústalo según las necesidades específicas de tu proyecto.
