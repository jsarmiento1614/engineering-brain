# Solución Completa: Docker Compose + API Gateway para Desarrollo Local

## 📋 Tabla de Contenidos

1. [Contexto del Problema](#contexto-del-problema)
2. [Análisis de Requerimientos](#análisis-de-requerimientos)
3. [Arquitectura de la Solución](#arquitectura-de-la-solución)
4. [Componentes de la Solución](#componentes-de-la-solución)
5. [Estructura de Archivos](#estructura-de-archivos)
6. [Configuración Paso a Paso](#configuración-paso-a-paso)
7. [Implementación del API Gateway](#implementación-del-api-gateway)
8. [Docker Compose Configuration](#docker-compose-configuration)
9. [Configuración del Backoffice](#configuración-del-backoffice)
10. [Scripts de Utilidad](#scripts-de-utilidad)
11. [Manejo de Tokens](#manejo-de-tokens)
12. [Troubleshooting](#troubleshooting)
13. [Mejores Prácticas](#mejores-prácticas)
14. [Consideraciones de Seguridad](#consideraciones-de-seguridad)

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

4. **Patrón del Backoffice**:
   - El backoffice usa una **URL base única**: `https://csm-api-lucy.aveapplications.com`
   - Accede a diferentes APIs mediante paths: `/location/`, `/routing/`, `/material/`, etc.
   - Construye URLs como: `${domain}/location/api/sub-route/filter`

5. **Restricciones de Seguridad**:
   - No podemos usar las llaves del servidor de desarrollo/staging (políticas de seguridad)
   - No podemos compartir llaves privadas/públicas entre entornos
   - Necesitamos mantener tokens locales y del servidor funcionando simultáneamente

6. **Desafío**:
   - Levantar localmente las 40+ APIs no es viable
   - Necesitamos una solución que permita trabajar de manera fluida como si fuese en el ambiente de desarrollo
   - Mantener el patrón de URL única del backoffice
   - Manejar tokens de diferentes orígenes automáticamente

---

## 🔍 Análisis de Requerimientos

### Requerimientos Funcionales

1. ✅ **URL única para el backoffice**: Mantener el patrón `http://localhost:8080` con paths
2. ✅ **APIs locales**: Levantar solo las APIs necesarias en Docker
3. ✅ **APIs del servidor**: Enrutar requests a APIs del servidor de desarrollo
4. ✅ **Manejo automático de tokens**: 
   - Token local para APIs locales
   - Token del servidor para APIs del servidor
5. ✅ **Transparencia**: El backoffice solo ve una URL base, no maneja complejidad de tokens

### Requerimientos No Funcionales

1. ✅ **Seguridad**: No compartir llaves del servidor
2. ✅ **Flexibilidad**: Fácil agregar/quitar APIs
3. ✅ **Performance**: Cache de tokens del servidor
4. ✅ **Mantenibilidad**: Configuración clara y documentada

---

## 🏗️ Arquitectura de la Solución

### Diagrama de Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                    DESARROLLADOR LOCAL                       │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Backoffice Frontend                        │   │
│  │           (ave-backoffice)                          │   │
│  │           http://localhost:4200                     │   │
│  │                                                       │   │
│  │  Environment:                                        │   │
│  │  const domain = "http://localhost:8080";             │   │
│  │                                                       │   │
│  │  Requests:                                           │   │
│  │  - /authentication/api/auth/login                   │   │
│  │  - /location/api/sub-route/filter                   │   │
│  │  - /routing/api/route/filter                        │   │
│  └──────────────────────────────────────────────────────┘   │
│                       ↓                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           API Gateway (NUEVO)                         │   │
│  │           http://localhost:8080                     │   │
│  │                                                       │   │
│  │  Funciones:                                          │   │
│  │  1. Intercepta requests                             │   │
│  │  2. Detecta destino (local vs servidor)             │   │
│  │  3. Maneja tokens automáticamente                   │   │
│  │  4. Hace proxy al destino correcto                  │   │
│  │                                                       │   │
│  │  Token Cache:                                        │   │
│  │  - Token local (del backoffice)                     │   │
│  │  - Token del servidor (obtenido automáticamente)    │   │
│  └──────────────────────────────────────────────────────┘   │
│                       ↓                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Docker Compose Network                     │   │
│  │                                                       │   │
│  │  ┌──────────────────┐  ┌──────────────────┐         │   │
│  │  │  ave-auth        │  │  ave-master      │         │   │
│  │  │  :3001           │  │  :3002           │         │   │
│  │  │  (local)         │  │  (local)         │         │   │
│  │  │                  │  │                  │         │   │
│  │  │  JWT Keys:       │  │  JWT Keys:       │         │   │
│  │  │  /jwt-keys/      │  │  /jwt-keys/     │         │   │
│  │  │  (local keys)    │  │  (local keys)    │         │   │
│  │  └──────────────────┘  └──────────────────┘         │   │
│  └──────────────────────────────────────────────────────┘   │
│                       ↓                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Internet → Servidor de Desarrollo          │   │
│  │           https://csm-api-lucy.aveapplications.com    │   │
│  │                                                       │   │
│  │  APIs del Servidor:                                  │   │
│  │  - /location/                                        │   │
│  │  - /routing/                                         │   │
│  │  - /material/                                        │   │
│  │  - /order/                                           │   │
│  │  - ... (40+ APIs)                                    │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Flujo de Autenticación y Requests

#### Flujo 1: Login Local

```
1. Backoffice → POST http://localhost:8080/authentication/api/auth/login
   ↓
2. Gateway intercepta:
   - Detecta: /authentication/ → es API local
   - Pasa request tal cual (sin modificar)
   ↓
3. Gateway → Proxy a ave-authentication:3000
   ↓
4. API local genera token con llave privada local
   ↓
5. Gateway → Devuelve respuesta al backoffice
   ↓
6. Backoffice guarda token local
```

#### Flujo 2: Request a API Local

```
1. Backoffice → GET http://localhost:8080/master/api/user/filter
   Header: Authorization: Bearer <token-local>
   ↓
2. Gateway intercepta:
   - Detecta: /master/ → es API local
   - Pasa token local tal cual
   ↓
3. Gateway → Proxy a ave-master:3002
   Header: Authorization: Bearer <token-local>
   ↓
4. API local valida token con llave pública local
   ↓
5. Gateway → Devuelve respuesta al backoffice
```

#### Flujo 3: Request a API del Servidor

```
1. Backoffice → GET http://localhost:8080/location/api/sub-route/filter
   Header: Authorization: Bearer <token-local>
   ↓
2. Gateway intercepta:
   - Detecta: /location/ → es API del servidor
   - Verifica cache de token del servidor
   ↓
3. Si no hay token del servidor:
   - Gateway hace login automático al servidor
   - POST https://csm-api-lucy.aveapplications.com/authentication/api/auth/login
   - Obtiene token del servidor
   - Guarda en cache (con expiración)
   ↓
4. Gateway → Proxy a servidor
   URL: https://csm-api-lucy.aveapplications.com/location/api/sub-route/filter
   Header: Authorization: Bearer <token-del-servidor>
   ↓
5. Servidor valida token y responde
   ↓
6. Gateway → Devuelve respuesta al backoffice
```

---

## 🧩 Componentes de la Solución

### 1. API Gateway

**Propósito**: Servicio intermedio que maneja routing y tokens automáticamente.

**Tecnología**: Node.js + Express (o NestJS para consistencia)

**Funciones**:
- Interceptar requests del backoffice
- Detectar destino (local vs servidor) según configuración
- Manejar tokens:
  - Pasar token local a APIs locales
  - Obtener y usar token del servidor para APIs del servidor
- Cache de tokens del servidor
- Proxy transparente a destinos

### 2. Docker Compose

**Propósito**: Orquestar APIs locales y el Gateway.

**Componentes**:
- APIs locales (solo las necesarias)
- API Gateway
- MongoDB (opcional, si no usas externo)
- Volumen compartido para llaves JWT

### 3. Configuración de Routing

**Propósito**: Definir qué APIs son locales y cuáles van al servidor.

**Formato**: JSON o TypeScript

```typescript
{
  local: {
    '/authentication/': 'http://ave-authentication:3000',
    '/master/': 'http://ave-master:3000',
  },
  server: {
    '/location/': 'https://csm-api-lucy.aveapplications.com/location/',
    '/routing/': 'https://csm-api-lucy.aveapplications.com/routing/',
    '/material/': 'https://csm-api-lucy.aveapplications.com/material/',
  }
}
```

---

## 📁 Estructura de Archivos

```
apis/
├── docker-compose.yml              # Configuración principal
├── docker-compose.override.yml     # Overrides para desarrollo
├── .env.docker                     # Variables de entorno
├── .env.gateway                    # Credenciales del servidor (NO COMMITEAR)
├── jwt-keys/                       # Llaves JWT locales
│   ├── jwtRS256.key               # Llave privada local
│   └── jwtRS256.key.pub           # Llave pública local
│   └── .gitignore
├── api-gateway/                    # NUEVO - Servicio Gateway
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   │   ├── main.ts
│   │   ├── gateway.service.ts     # Lógica de routing
│   │   ├── token-manager.service.ts # Manejo de tokens
│   │   ├── config/
│   │   │   └── routing.config.ts  # Configuración de routing
│   │   └── middleware/
│   │       └── proxy.middleware.ts
│   └── tsconfig.json
├── scripts/
│   ├── start-local-apis.sh
│   ├── stop-local-apis.sh
│   ├── restart-api.sh
│   ├── logs-api.sh
│   ├── generate-jwt-keys.sh
│   └── setup-gateway.sh           # NUEVO
├── ave-authentication/
│   └── Dockerfile
├── ave-master/
│   └── Dockerfile
└── README.md
```

---

## 🔧 Configuración Paso a Paso

### Paso 1: Crear Directorio para Llaves JWT Locales

```bash
cd C:\develop\proyects\freelance\albatros\apis
mkdir jwt-keys
cd jwt-keys
```

### Paso 2: Generar Llaves JWT Locales

```bash
# Generar par de llaves RSA
openssl genrsa -out jwtRS256.key 2048
openssl rsa -in jwtRS256.key -pubout -out jwtRS256.key.pub

# Ajustar permisos
chmod 600 jwtRS256.key
chmod 644 jwtRS256.key.pub
```

### Paso 3: Crear .gitignore para jwt-keys

```bash
# jwt-keys/.gitignore
jwtRS256.key
jwtRS256.key.pub
*.key
*.key.pub
```

### Paso 4: Crear Estructura del API Gateway

```bash
mkdir api-gateway
cd api-gateway
npm init -y
```

### Paso 5: Configurar Variables de Entorno

Crear `.env.gateway` en la raíz de `apis/`:

```env
# .env.gateway
# Credenciales del servidor de desarrollo (NO COMMITEAR)

# Servidor de Desarrollo
SERVER_BASE_URL=https://csm-api-lucy.aveapplications.com
SERVER_AUTH_URL=https://csm-api-lucy.aveapplications.com/authentication/api/auth/login

# Credenciales para login automático al servidor
SERVER_EMAIL=tu-email@example.com
SERVER_PASSWORD=tu-password

# Cache de tokens
TOKEN_CACHE_TTL=3600000  # 1 hora en ms
```

**⚠️ IMPORTANTE**: Agregar `.env.gateway` al `.gitignore`

---

## 🚀 Implementación del API Gateway

### Estructura del Gateway

#### package.json

```json
{
  "name": "ave-api-gateway",
  "version": "1.0.0",
  "description": "API Gateway para desarrollo local con manejo automático de tokens",
  "main": "dist/main.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/main.js",
    "dev": "ts-node-dev --respawn --transpile-only src/main.ts"
  },
  "dependencies": {
    "express": "^4.18.2",
    "http-proxy-middleware": "^2.0.6",
    "axios": "^1.6.0",
    "node-cache": "^5.1.2",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^20.10.0",
    "typescript": "^5.3.0",
    "ts-node-dev": "^2.0.0"
  }
}
```

#### src/config/routing.config.ts

```typescript
export interface RoutingConfig {
  local: Record<string, string>;
  server: Record<string, string>;
}

export const routingConfig: RoutingConfig = {
  local: {
    '/authentication/': 'http://ave-authentication:3000',
    '/master/': 'http://ave-master:3000',
    // Agregar más APIs locales según necesidad
  },
  server: {
    '/location/': 'https://csm-api-lucy.aveapplications.com/location/',
    '/routing/': 'https://csm-api-lucy.aveapplications.com/routing/',
    '/material/': 'https://csm-api-lucy.aveapplications.com/material/',
    '/order/': 'https://csm-api-lucy.aveapplications.com/order/',
    '/business-partner/': 'https://csm-api-lucy.aveapplications.com/business-partner/',
    '/configuration/': 'https://csm-api-lucy.aveapplications.com/configuration/',
    '/common/': 'https://csm-api-lucy.aveapplications.com/common/',
    '/payment/': 'https://csm-api-lucy.aveapplications.com/payment/',
    '/trip/': 'https://csm-api-lucy.aveapplications.com/trip/',
    '/return/': 'https://csm-api-lucy.aveapplications.com/return/',
    '/load-preparation/': 'https://csm-api-lucy.aveapplications.com/load-preparation/',
    '/csm-liquidation/': 'https://csm-api-lucy.aveapplications.com/csm-liquidation/',
    '/integrator/': 'https://csm-api-lucy.aveapplications.com/integrator/',
    '/document-export/': 'https://csm-api-lucy.aveapplications.com/document-export/',
    '/payment-master/': 'https://csm-api-lucy.aveapplications.com/payment-master/',
    '/csm-business-partner/': 'https://csm-api-lucy.aveapplications.com/csm-business-partner/',
    '/material-price-list/': 'https://csm-api-lucy.aveapplications.com/material-price-list/',
    '/document/': 'https://csm-api-lucy.aveapplications.com/document/',
    '/credit/': 'https://csm-api-lucy.aveapplications.com/credit/',
    '/notification/': 'https://csm-api-lucy.aveapplications.com/notification/',
    '/asset/': 'https://csm-api-lucy.aveapplications.com/asset/',
    '/order-report/': 'https://csm-api-lucy.aveapplications.com/order-report/',
    '/fields-catalog/': 'https://csm-api-lucy.aveapplications.com/fields-catalog/',
    '/quota/': 'https://csm-api-lucy.aveapplications.com/quota/',
    // Agregar más APIs del servidor según necesidad
  }
};

/**
 * Detecta si un path corresponde a una API local o del servidor
 */
export function detectDestination(path: string): 'local' | 'server' | null {
  // Normalizar path
  const normalizedPath = path.startsWith('/') ? path : `/${path}`;
  
  // Buscar en APIs locales
  for (const [prefix, _] of Object.entries(routingConfig.local)) {
    if (normalizedPath.startsWith(prefix)) {
      return 'local';
    }
  }
  
  // Buscar en APIs del servidor
  for (const [prefix, _] of Object.entries(routingConfig.server)) {
    if (normalizedPath.startsWith(prefix)) {
      return 'server';
    }
  }
  
  return null;
}

/**
 * Obtiene la URL de destino para un path
 */
export function getTargetUrl(path: string): string | null {
  const normalizedPath = path.startsWith('/') ? path : `/${path}`;
  
  // Buscar en APIs locales
  for (const [prefix, baseUrl] of Object.entries(routingConfig.local)) {
    if (normalizedPath.startsWith(prefix)) {
      const remainingPath = normalizedPath.substring(prefix.length);
      return `${baseUrl}${remainingPath}`;
    }
  }
  
  // Buscar en APIs del servidor
  for (const [prefix, baseUrl] of Object.entries(routingConfig.server)) {
    if (normalizedPath.startsWith(prefix)) {
      const remainingPath = normalizedPath.substring(prefix.length);
      return `${baseUrl}${remainingPath}`;
    }
  }
  
  return null;
}
```

#### src/services/token-manager.service.ts

```typescript
import axios from 'axios';
import NodeCache from 'node-cache';
import * as dotenv from 'dotenv';

dotenv.config({ path: '.env.gateway' });

interface ServerToken {
  access_token: string;
  refresh_token?: string;
  expires_at: number; // Timestamp
}

export class TokenManagerService {
  private cache: NodeCache;
  private serverAuthUrl: string;
  private serverEmail: string;
  private serverPassword: string;
  private tokenCacheTtl: number;

  constructor() {
    this.cache = new NodeCache({ stdTTL: 3600 }); // 1 hora por defecto
    this.serverAuthUrl = process.env.SERVER_AUTH_URL || '';
    this.serverEmail = process.env.SERVER_EMAIL || '';
    this.serverPassword = process.env.SERVER_PASSWORD || '';
    this.tokenCacheTtl = parseInt(process.env.TOKEN_CACHE_TTL || '3600000', 10);
    
    if (!this.serverAuthUrl || !this.serverEmail || !this.serverPassword) {
      console.warn('⚠️  Advertencia: Credenciales del servidor no configuradas');
      console.warn('   El gateway no podrá hacer login automático al servidor');
    }
  }

  /**
   * Obtiene el token del servidor (desde cache o haciendo login)
   */
  async getServerToken(): Promise<string | null> {
    // Verificar cache
    const cachedToken = this.cache.get<ServerToken>('server_token');
    if (cachedToken && cachedToken.expires_at > Date.now()) {
      return cachedToken.access_token;
    }

    // Si no hay token o expiró, hacer login
    return await this.loginToServer();
  }

  /**
   * Hace login al servidor de desarrollo y obtiene token
   */
  private async loginToServer(): Promise<string | null> {
    if (!this.serverAuthUrl || !this.serverEmail || !this.serverPassword) {
      console.error('❌ Error: Credenciales del servidor no configuradas');
      return null;
    }

    try {
      console.log('🔐 Haciendo login automático al servidor...');
      
      const response = await axios.post(this.serverAuthUrl, {
        email: this.serverEmail,
        password: this.serverPassword,
      });

      const tokenData = response.data;
      const accessToken = tokenData.access_token || tokenData.token || tokenData.data?.access_token;

      if (!accessToken) {
        console.error('❌ Error: No se recibió token del servidor');
        return null;
      }

      // Calcular expiración (asumir 30 minutos si no viene en la respuesta)
      const expiresIn = tokenData.expires_in || 1800; // 30 minutos por defecto
      const expiresAt = Date.now() + (expiresIn * 1000);

      // Guardar en cache
      const serverToken: ServerToken = {
        access_token: accessToken,
        refresh_token: tokenData.refresh_token,
        expires_at: expiresAt,
      };

      this.cache.set('server_token', serverToken, Math.floor(expiresIn / 60)); // TTL en minutos

      console.log('✅ Login exitoso al servidor, token guardado en cache');
      return accessToken;

    } catch (error: any) {
      console.error('❌ Error al hacer login al servidor:', error.message);
      if (error.response) {
        console.error('   Status:', error.response.status);
        console.error('   Data:', error.response.data);
      }
      return null;
    }
  }

  /**
   * Limpia el cache de tokens
   */
  clearCache(): void {
    this.cache.flushAll();
    console.log('🗑️  Cache de tokens limpiado');
  }

  /**
   * Obtiene información del token cacheado
   */
  getCacheInfo(): { hasToken: boolean; expiresAt?: number } {
    const cachedToken = this.cache.get<ServerToken>('server_token');
    return {
      hasToken: !!cachedToken,
      expiresAt: cachedToken?.expires_at,
    };
  }
}
```

#### src/services/gateway.service.ts

```typescript
import { Request, Response } from 'express';
import { createProxyMiddleware, Options } from 'http-proxy-middleware';
import { detectDestination, getTargetUrl } from '../config/routing.config';
import { TokenManagerService } from './token-manager.service';

export class GatewayService {
  private tokenManager: TokenManagerService;

  constructor() {
    this.tokenManager = new TokenManagerService();
  }

  /**
   * Middleware principal del gateway
   */
  async handleRequest(req: Request, res: Response): Promise<void> {
    const path = req.path;
    const destination = detectDestination(path);

    if (!destination) {
      res.status(404).json({
        error: 'Not Found',
        message: `No se encontró configuración para el path: ${path}`,
      });
      return;
    }

    const targetUrl = getTargetUrl(path);
    if (!targetUrl) {
      res.status(500).json({
        error: 'Internal Server Error',
        message: 'No se pudo determinar la URL de destino',
      });
      return;
    }

    // Si es API del servidor, obtener token del servidor
    if (destination === 'server') {
      const serverToken = await this.tokenManager.getServerToken();
      
      if (!serverToken) {
        res.status(503).json({
          error: 'Service Unavailable',
          message: 'No se pudo obtener token del servidor. Verifica las credenciales en .env.gateway',
        });
        return;
      }

      // Reemplazar el token local con el token del servidor
      req.headers.authorization = `Bearer ${serverToken}`;
    }

    // Configurar proxy
    const proxyOptions: Options = {
      target: targetUrl,
      changeOrigin: true,
      pathRewrite: this.getPathRewrite(path, destination),
      onProxyReq: (proxyReq, req) => {
        // Log para debugging
        console.log(`[Gateway] ${req.method} ${path} → ${targetUrl}`);
        
        // Asegurar que los headers se pasen correctamente
        if (req.headers.authorization) {
          proxyReq.setHeader('Authorization', req.headers.authorization);
        }
        
        // Pasar otros headers importantes
        if (req.headers['content-type']) {
          proxyReq.setHeader('Content-Type', req.headers['content-type']);
        }
      },
      onError: (err, req, res) => {
        console.error(`[Gateway Error] ${req.method} ${req.path}:`, err.message);
        if (!res.headersSent) {
          res.status(502).json({
            error: 'Bad Gateway',
            message: `Error al conectar con el servicio: ${err.message}`,
          });
        }
      },
    };

    // Crear y ejecutar proxy
    const proxy = createProxyMiddleware(proxyOptions);
    proxy(req, res, () => {
      // Si el proxy no maneja la respuesta, devolver error
      if (!res.headersSent) {
        res.status(500).json({
          error: 'Internal Server Error',
          message: 'Error en el proxy',
        });
      }
    });
  }

  /**
   * Determina cómo reescribir el path según el destino
   */
  private getPathRewrite(path: string, destination: 'local' | 'server'): Record<string, string> {
    // Para APIs locales, quitar el prefijo
    if (destination === 'local') {
      // Ejemplo: /authentication/api/auth/login → /api/auth/login
      const prefix = path.split('/')[1]; // Obtener primer segmento
      return {
        [`^/${prefix}`]: '',
      };
    }

    // Para APIs del servidor, mantener el path completo
    return {};
  }

  /**
   * Health check del gateway
   */
  async healthCheck(): Promise<{ status: string; cache: any }> {
    const cacheInfo = this.tokenManager.getCacheInfo();
    return {
      status: 'ok',
      cache: cacheInfo,
    };
  }
}
```

#### src/main.ts

```typescript
import express from 'express';
import cors from 'cors';
import { GatewayService } from './services/gateway.service';

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Gateway service
const gateway = new GatewayService();

// Health check
app.get('/health', async (req, res) => {
  const health = await gateway.healthCheck();
  res.json(health);
});

// Endpoint para limpiar cache de tokens
app.post('/gateway/clear-cache', (req, res) => {
  // @ts-ignore - acceso directo al tokenManager
  gateway.tokenManager.clearCache();
  res.json({ message: 'Cache limpiado' });
});

// Todas las demás rutas pasan por el gateway
app.use('*', async (req, res) => {
  await gateway.handleRequest(req, res);
});

// Iniciar servidor
app.listen(PORT, () => {
  console.log(`🚀 API Gateway corriendo en http://localhost:${PORT}`);
  console.log(`📋 Health check: http://localhost:${PORT}/health`);
  console.log(`🗑️  Limpiar cache: POST http://localhost:${PORT}/gateway/clear-cache`);
});
```

#### Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /usr/src/app

# Copiar archivos de dependencias
COPY package*.json ./
RUN npm ci --only=production

# Copiar código fuente
COPY . .
RUN npm run build

# Exponer puerto
EXPOSE 3000

# Comando de inicio
CMD ["node", "dist/main.js"]
```

#### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 🐳 Docker Compose Configuration

### docker-compose.yml

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

  # API Gateway
  api-gateway:
    build:
      context: ./api-gateway
      dockerfile: Dockerfile
    container_name: ave-gateway-local
    ports:
      - "8080:3000"
    volumes:
      - ./api-gateway:/usr/src/app
      - /usr/src/app/node_modules
      - ./.env.gateway:/usr/src/app/.env.gateway:ro
    environment:
      - NODE_ENV=development
      - PORT=3000
    networks:
      - ave-local-network
    depends_on:
      - ave-authentication
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # API de Autenticación
  ave-authentication:
    build:
      context: ./ave-authentication
      dockerfile: Dockerfile
    container_name: ave-auth-local
    ports:
      - "3001:3000"  # Expuesto para debugging, pero el gateway usa el puerto interno
    volumes:
      - ./jwt-keys:/app/jwt-keys:ro
    environment:
      - JWT_PRIVATE_KEY_PATH=/app/jwt-keys/jwtRS256.key
      - JWT_PUBLIC_KEY_PATH=/app/jwt-keys/jwtRS256.key.pub
      - JWT_ISSUER=ave-local
      - DATABASE_URL=mongodb://mongo:27017/ave-authentication?authSource=admin
      - AVE_AUTHENTICATION_PORT=3000
      - NODE_ENV=development
      - AVE_AUTHENTICATION_SWAGGER_SERVER=http://localhost:3001
      - AVE_AUTHENTICATION_SWAGGER_TITLE=AVE Authentication API (Local)
      - AVE_AUTHENTICATION_SWAGGER_DESCRIPTION=API de autenticación para desarrollo local
      - ISOLATED_TOKEN_SECRET=${ISOLATED_TOKEN_SECRET:-local-secret-key}
      - FALLBACK_LANGUAGE=es
    networks:
      - ave-local-network
    depends_on:
      - mongo
    restart: unless-stopped

  # API Master (ejemplo - ajustar según necesidad)
  ave-master:
    build:
      context: ./ave-master
      dockerfile: Dockerfile
    container_name: ave-master-local
    ports:
      - "3002:3000"
    volumes:
      - ./jwt-keys:/app/jwt-keys:ro
    environment:
      - JWT_PUBLIC_KEY_PATH=/app/jwt-keys/jwtRS256.key.pub
      - DATABASE_URL=mongodb://mongo:27017/ave-master?authSource=admin
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

### .env.docker

```env
# .env.docker
# Configuración compartida para todas las APIs

# JWT Keys Paths (dentro del contenedor)
JWT_PRIVATE_KEY_PATH=/app/jwt-keys/jwtRS256.key
JWT_PUBLIC_KEY_PATH=/app/jwt-keys/jwtRS256.key.pub

# MongoDB
MONGODB_URI=mongodb://mongo:27017/ave-local

# Network
NETWORK_NAME=ave-local-network

# Ports
GATEWAY_PORT=8080
AUTH_PORT=3001
MASTER_PORT=3002
```

---

## 🖥️ Configuración del Backoffice

### Modificar environment.ts

```typescript
// src/environments/environment.ts

// Cambiar solo esta línea:
const domain = "http://localhost:8080";  // Gateway local

export const environment = {
  production: false,
  
  // ... resto de la configuración ...
  
  // Todas las URLs apuntan al gateway
  API_AUTHENTICATION_URL: `${domain}/authentication/`,
  API_LOCATION_URL: `${domain}/location/`,
  API_ROUTING_URL: `${domain}/routing/`,
  API_MATERIAL_URL: `${domain}/material/`,
  API_BUSINESS_PARTNER_URL: `${domain}/business-partner/`,
  API_MASTER_URL: `${domain}/master/`,
  // ... etc
};
```

**Eso es todo**. El backoffice solo necesita cambiar `domain` a `http://localhost:8080`.

---

## 📜 Scripts de Utilidad

### scripts/setup-gateway.sh

```bash
#!/bin/bash

# Script para configurar el API Gateway
# Uso: ./scripts/setup-gateway.sh

set -e

GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

echo -e "${GREEN}🔧 Configurando API Gateway...${NC}"

# Verificar que existe .env.gateway
if [ ! -f ".env.gateway" ]; then
    echo -e "${YELLOW}⚠️  .env.gateway no existe, creando template...${NC}"
    cat > .env.gateway << EOF
# Credenciales del servidor de desarrollo (NO COMMITEAR)
SERVER_BASE_URL=https://csm-api-lucy.aveapplications.com
SERVER_AUTH_URL=https://csm-api-lucy.aveapplications.com/authentication/api/auth/login
SERVER_EMAIL=tu-email@example.com
SERVER_PASSWORD=tu-password
TOKEN_CACHE_TTL=3600000
EOF
    echo -e "${YELLOW}   Por favor, edita .env.gateway con tus credenciales${NC}"
fi

# Verificar que el gateway tiene las dependencias
if [ -d "api-gateway" ]; then
    echo -e "${GREEN}📦 Instalando dependencias del gateway...${NC}"
    cd api-gateway
    npm install
    cd ..
fi

echo -e "${GREEN}✅ Gateway configurado${NC}"
```

### scripts/start-local-apis.sh (actualizado)

```bash
#!/bin/bash

# Script para iniciar las APIs locales y el gateway
# Uso: ./scripts/start-local-apis.sh [api1] [api2] ...

set -e

GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

echo -e "${GREEN}🚀 Iniciando APIs locales y Gateway...${NC}"

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

# Verificar configuración del gateway
if [ ! -f ".env.gateway" ]; then
    echo -e "${YELLOW}⚠️  .env.gateway no encontrado${NC}"
    echo -e "${YELLOW}   Ejecuta: ./scripts/setup-gateway.sh${NC}"
    read -p "¿Continuar de todas formas? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi

# Si se pasan argumentos, iniciar solo esos servicios
if [ $# -gt 0 ]; then
    SERVICES="$@ api-gateway"  # Siempre incluir el gateway
    echo -e "${GREEN}Iniciando servicios: ${SERVICES}${NC}"
    docker-compose up -d $SERVICES
else
    # Iniciar todas las APIs y el gateway
    echo -e "${GREEN}Iniciando todas las APIs y el Gateway...${NC}"
    docker-compose up -d
fi

# Esperar a que los servicios estén listos
echo -e "${YELLOW}⏳ Esperando a que los servicios estén listos...${NC}"
sleep 5

# Verificar estado
echo -e "${GREEN}📊 Estado de los servicios:${NC}"
docker-compose ps

# Verificar health del gateway
echo -e "${GREEN}🏥 Verificando health del Gateway...${NC}"
sleep 2
curl -s http://localhost:8080/health | jq '.' || echo "Gateway aún no está listo"

echo -e "${GREEN}✅ APIs y Gateway iniciados correctamente${NC}"
echo -e "${GREEN}📝 Gateway disponible en: http://localhost:8080${NC}"
echo -e "${GREEN}📝 Para ver logs: docker-compose logs -f api-gateway${NC}"
```

---

## 🔐 Manejo de Tokens

### Flujo de Tokens

1. **Token Local**:
   - Generado por `ave-authentication` local
   - Firmado con llave privada local
   - Usado para APIs locales
   - El backoffice lo obtiene haciendo login a `/authentication/api/auth/login`

2. **Token del Servidor**:
   - Obtenido automáticamente por el Gateway
   - Firmado con llave privada del servidor
   - Usado para APIs del servidor
   - Cacheado por el Gateway (1 hora por defecto)

### Cache de Tokens

El Gateway cachea el token del servidor para evitar hacer login en cada request:

- **TTL por defecto**: 1 hora
- **Renovación automática**: El Gateway detecta cuando el token expira y obtiene uno nuevo
- **Limpieza manual**: `POST http://localhost:8080/gateway/clear-cache`

### Debugging de Tokens

```bash
# Ver health del gateway (incluye info del cache)
curl http://localhost:8080/health

# Limpiar cache de tokens
curl -X POST http://localhost:8080/gateway/clear-cache

# Ver logs del gateway
docker-compose logs -f api-gateway
```

---

## 🔍 Troubleshooting

### Problema 1: Gateway no puede hacer login al servidor

**Síntoma**: 
```
❌ Error: No se pudo obtener token del servidor
```

**Solución**:
1. Verificar que `.env.gateway` existe y tiene las credenciales correctas
2. Verificar que las credenciales son válidas
3. Verificar conectividad al servidor: `curl https://csm-api-lucy.aveapplications.com/authentication/api/ping`
4. Ver logs del gateway: `docker-compose logs api-gateway`

### Problema 2: 404 en requests

**Síntoma**:
```
No se encontró configuración para el path: /some-path/
```

**Solución**:
1. Agregar el path a `routing.config.ts` en la sección correspondiente (local o server)
2. Rebuild del gateway: `docker-compose up -d --build api-gateway`

### Problema 3: Token local rechazado por API del servidor

**Síntoma**: 401 en requests a APIs del servidor

**Solución**:
1. Verificar que el Gateway está reemplazando el token correctamente
2. Ver logs del gateway para ver qué token está usando
3. Limpiar cache: `POST http://localhost:8080/gateway/clear-cache`
4. Verificar que el Gateway puede hacer login al servidor

### Problema 4: Path rewrite incorrecto

**Síntoma**: 404 en APIs locales aunque están levantadas

**Solución**:
1. Verificar la función `getPathRewrite` en `gateway.service.ts`
2. Ajustar según la estructura de paths de tus APIs
3. Ver logs del gateway para ver qué URL está generando

### Comandos Útiles

```bash
# Ver logs del gateway
docker-compose logs -f api-gateway

# Rebuild solo el gateway
docker-compose up -d --build api-gateway

# Ver health del gateway
curl http://localhost:8080/health

# Probar un request a través del gateway
curl http://localhost:8080/authentication/api/ping

# Limpiar cache de tokens
curl -X POST http://localhost:8080/gateway/clear-cache
```

---

## ✨ Mejores Prácticas

### 1. Seguridad

- ✅ **Nunca commitees `.env.gateway`** al repositorio
- ✅ Usa variables de entorno para credenciales sensibles
- ✅ Rota las credenciales periódicamente
- ✅ No uses credenciales de producción en desarrollo local

### 2. Configuración

- ✅ Mantén `routing.config.ts` actualizado con todas las APIs
- ✅ Documenta qué APIs son locales y cuáles van al servidor
- ✅ Usa nombres descriptivos en la configuración

### 3. Performance

- ✅ Ajusta `TOKEN_CACHE_TTL` según necesidad
- ✅ Monitorea el uso de cache del Gateway
- ✅ Considera implementar refresh token si el servidor lo soporta

### 4. Desarrollo

- ✅ Agrega logging detallado para debugging
- ✅ Implementa health checks robustos
- ✅ Documenta cambios en la configuración de routing

---

## 🔒 Consideraciones de Seguridad

### Credenciales del Servidor

- ⚠️ **NUNCA** commitees `.env.gateway` al repositorio
- ⚠️ Las credenciales solo deben estar en tu máquina local
- ⚠️ Usa un usuario de desarrollo, no de producción
- ⚠️ Considera usar variables de entorno del sistema en lugar de archivo

### Tokens

- ✅ Los tokens del servidor se cachean en memoria (no en disco)
- ✅ Los tokens expiran automáticamente
- ✅ El Gateway no almacena passwords, solo tokens

### Red

- ✅ El Gateway solo corre localmente
- ✅ No expone credenciales al exterior
- ✅ Las APIs locales están en una red Docker aislada

---

## 📝 Checklist de Implementación

- [ ] Crear directorio `jwt-keys/` y generar llaves locales
- [ ] Crear `.gitignore` para `jwt-keys/` y `.env.gateway`
- [ ] Crear estructura del API Gateway
- [ ] Configurar `.env.gateway` con credenciales del servidor
- [ ] Implementar código del Gateway
- [ ] Configurar `routing.config.ts` con todas las APIs
- [ ] Crear `docker-compose.yml` con Gateway y APIs locales
- [ ] Probar build del Gateway: `docker-compose build api-gateway`
- [ ] Iniciar servicios: `docker-compose up -d`
- [ ] Verificar health del Gateway: `curl http://localhost:8080/health`
- [ ] Probar login local: `POST http://localhost:8080/authentication/api/auth/login`
- [ ] Probar request a API local: `GET http://localhost:8080/master/api/...`
- [ ] Probar request a API del servidor: `GET http://localhost:8080/location/api/...`
- [ ] Modificar `environment.ts` del backoffice para usar `http://localhost:8080`
- [ ] Probar flujo completo desde el backoffice
- [ ] Documentar qué APIs están levantadas localmente
- [ ] Compartir configuración con el equipo (sin credenciales)

---

## 🎯 Próximos Pasos

1. **Implementar el Gateway** siguiendo esta documentación
2. **Configurar routing** con las APIs que necesitas
3. **Probar el flujo completo** con el backoffice
4. **Iterar y mejorar** según feedback
5. **Documentar** qué APIs están incluidas y por qué

---

## 📚 Recursos Adicionales

### Documentación

- [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)
- [Express.js](https://expressjs.com/)
- [Docker Compose](https://docs.docker.com/compose/)

### Comandos de Referencia Rápida

```bash
# Iniciar todo
docker-compose up -d

# Ver logs del gateway
docker-compose logs -f api-gateway

# Rebuild gateway
docker-compose up -d --build api-gateway

# Health check
curl http://localhost:8080/health

# Limpiar cache
curl -X POST http://localhost:8080/gateway/clear-cache

# Detener todo
docker-compose down
```

---

**¿Preguntas o necesitas ayuda con la implementación?**

Este documento es una guía completa. Ajústala según las necesidades específicas de tu proyecto.
