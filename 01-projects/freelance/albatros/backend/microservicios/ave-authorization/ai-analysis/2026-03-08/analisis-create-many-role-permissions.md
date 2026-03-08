# Análisis del Endpoint: `/api/auth_role_permission/create_many/:id`

## 📋 Resumen Ejecutivo

El endpoint `POST /api/auth_role_permission/create_many/:id` permite agregar múltiples permisos a un rol de manera masiva. Este endpoint realiza operaciones complejas que incluyen validaciones, expansión automática de permisos backend, y propagación opcional a todos los usuarios que tienen el rol asignado.

---

## 🔗 Endpoint

**Ruta:** `POST /api/auth_role_permission/create_many/:id`

**Parámetros:**
- `id` (path): MongoDB ObjectId del rol al cual se asignarán los permisos

**Permiso requerido:** `api:authorization-role-permission:create`

**Código de respuesta exitoso:** `201 CREATED`

---

## 📥 DTO de Entrada: `CreateManyRolePermissionsDto`

```typescript
{
  applyToAllUsers?: boolean;  // Opcional, default: true
  permissions: Array<{
    mongoId: string;  // MongoDB ObjectId del permiso
  }>;
}
```

### Campos:

- **`applyToAllUsers`** (opcional, default: `true`):
  - Si es `true` o `undefined/null`: Los permisos se copiarán automáticamente a todos los usuarios que tienen este rol asignado
  - Si es `false`: Solo se actualiza el rol, sin propagar a usuarios

- **`permissions`** (requerido):
  - Array de objetos con el campo `mongoId` que contiene el MongoDB ObjectId del permiso a asignar

---

## 🔄 Flujo de Ejecución

### 1. Validación del Rol

```typescript
// Verifica que el rol existe y no está eliminado
const role = await authRoleRepository.getDocument(
  buildQuery(
    where('_id', Ops.eq(roleMongoId)),
    andWhere('_deleted', Ops.eq(false))
  )
);
```

**Si el rol no existe:** Retorna error `404 NOT_FOUND` con mensaje "Rol no encontrado"

---

### 2. Validación de Permisos

Para cada permiso en el array `permissions`:

```typescript
// Verifica que cada permiso:
// - Existe en la base de datos
// - No está eliminado (_deleted = false)
// - Está activo (statusId = StatusEnum.ACTIVE)
const permission = await authPermissionRepository.getDocument(
  buildQuery(
    where('_id', Ops.eq(permissionId)),
    andWhere('_deleted', Ops.eq(false)),
    andWhere('statusId', Ops.eq(StatusEnum.ACTIVE))
  )
);
```

**Si algún permiso no existe o no está activo:** Retorna error `404 NOT_FOUND` con mensaje específico del permiso

---

### 3. Validación de Tipo de Permisos

```typescript
// Valida que NO se estén agregando permisos API directamente
const apiPermissions = permissions.filter(p => p.isApiPermission === true);
if (apiPermissions.length > 0) {
  // Error: No se pueden agregar permisos API directamente
}
```

**Regla importante:**
- ❌ **NO se pueden agregar permisos API directamente** (`isApiPermission = true`)
- ✅ Solo se pueden agregar permisos App (frontend) (`isApiPermission = false`)
- Los permisos API se agregan **automáticamente** cuando un permiso App los requiere

**Si se intenta agregar permisos API:** Retorna error `400 BAD_REQUEST` con lista de códigos de permisos API rechazados

---

### 4. Verificación de Duplicados

```typescript
// Obtiene todos los permisos ya asignados al rol
const existingRolePermissions = await authRolePermissionRepository.getDocuments(
  buildQuery(
    where('roleMongoId', Ops.eq(roleMongoId)),
    andWhere('_deleted', Ops.eq(false))
  )
);

// Crea un Set con los IDs de permisos existentes
const existingPermissionIds = new Set(
  existingRolePermissions.map(rp => rp.permissionMongoId.toString())
);
```

**Comportamiento:**
- Si un permiso ya está asignado al rol, se **omite** (no se crea duplicado)
- Solo se crean relaciones para permisos nuevos

**Si todos los permisos ya están asignados:** Retorna error `400 BAD_REQUEST` con mensaje "Todos los permisos ya están asignados al rol"

---

### 5. Expansión Automática de Permisos Backend

```typescript
// Obtiene los códigos de los permisos App que se están agregando
const permissionCodes = permissions.map(p => p.code);

// Expande automáticamente los permisos backend requeridos
const expandedBackendCodes = await permissionExpansionService.expandBackendPermissions(
  permissionCodes
);

// Busca los permisos backend en la base de datos
const backendPermissions = await authPermissionModel.find({
  code: { $in: expandedBackendCodes },
  statusId: StatusEnum.ACTIVE,
  _deleted: false
});
```

**Funcionalidad:**
- Cada permiso App puede tener un campo `apiPermissionCodesRequired` que lista los códigos de permisos API que requiere
- El servicio `PermissionExpansionService` expande recursivamente todas las dependencias
- Los permisos backend expandidos se agregan **automáticamente** al rol junto con los permisos App solicitados

**Ejemplo:**
- Si agregas el permiso App `"BUSINESS_PARTNER:CREATE"` que requiere `["API:BUSINESS_PARTNER:CREATE"]`
- El sistema automáticamente también agregará `"API:BUSINESS_PARTNER:CREATE"` al rol

---

### 6. Creación de Relaciones Rol-Permiso

```typescript
// Crea documentos en auth-role-permission para:
// - Los permisos App solicitados
// - Los permisos backend expandidos automáticamente
const createdRolePermissions = await authRolePermissionRepository.createDocumentsBulk(
  rolePermissionsToCreate,
  auditMetadata
);
```

**Datos almacenados en cada documento:**
- `roleMongoId`: ID del rol
- `permissionMongoId`: ID del permiso
- `code`: Código del permiso
- `name`: Nombre del permiso
- `isApiPermission`: Si es permiso API o App
- `isSubAppPermission`: Si es permiso de sub-aplicación
- `subApplication`: Información de la sub-aplicación
- `apiPermissionCodesRequired`: Array de códigos de permisos API requeridos
- `statusId`: Estado (ACTIVE)
- Metadatos de auditoría (`createdBy`, `createdAt`, etc.)

---

### 7. Propagación a Usuarios (Opcional)

```typescript
// Si applyToAllUsers es true (o undefined/null, default true)
const applyToAllUsers = createDto.applyToAllUsers !== false; // Default true

if (applyToAllUsers) {
  await copyPermissionsToUsersWithRole(
    roleMongoId,
    createdRolePermissions,  // Incluye permisos App + permisos backend expandidos
    auditMetadata
  );
}
```

#### Proceso de Copia a Usuarios:

1. **Búsqueda de Usuarios con el Rol:**
   ```typescript
   // Usa aggregate en users_login para obtener todos los usuarios con este rol
   const usersWithRole = await usersLoginModel.aggregate([
     {
       $match: {
         'roles._id': new Types.ObjectId(roleMongoId),
         _deleted: false
       }
     },
     {
       $project: {
         _id: 0,
         userLoginMongoId: '$_id'
       }
     }
   ]);
   ```

2. **Inserción Masiva en `auth-user-role-permission`:**
   ```typescript
   // Para cada usuario, crea documentos en auth-user-role-permission
   // con TODOS los permisos creados (App + backend expandidos)
   const documentsToInsert = rolePermissions.map(rolePermission => ({
     userLoginMongoId: userLoginMongoId,
     roleMongoId: roleMongoId,
     permissionMongoId: rolePermission.permissionMongoId,
     code: rolePermission.code,
     name: rolePermission.name,
     isApiPermission: rolePermission.isApiPermission,
     isSubAppPermission: rolePermission.isSubAppPermission,
     subApplication: rolePermission.subApplication,
     apiPermissionCodesRequired: rolePermission.apiPermissionCodesRequired,
     isCustomPermission: false,  // Siempre false para permisos de rol
     statusId: rolePermission.statusId,
     _deleted: false,
     createdBy: auditMetadata,
     updatedAuditAt: new Date()
   }));
   ```

3. **Procesamiento en Lotes:**
   - Si hay ≤ 10,000 documentos: Inserción directa
   - Si hay > 10,000 documentos: División en lotes de 10,000 e inserción en paralelo

4. **Invalidación de Caché:**
   ```typescript
   // Para cada usuario afectado, invalida su caché de permisos
   await permissionConsolidationService.invalidateCache(userLoginMongoId);
   ```

**Nota importante:** Los errores al copiar permisos a usuarios individuales se registran pero **no fallan la operación completa**. El rol se actualiza exitosamente incluso si algunos usuarios fallan.

---

## 📤 Respuesta Exitosa

```typescript
{
  status: "success",
  statusCode: "201",
  message: "Permisos agregados exitosamente al rol.",
  data: AuthRolePermissionAttributes[]  // Array de permisos creados (App + backend expandidos)
}
```

---

## ⚠️ Errores Posibles

| Código | Mensaje | Causa |
|--------|---------|-------|
| `404` | "Rol no encontrado" | El rol no existe o está eliminado |
| `404` | "Permiso con ID {id} no encontrado o no está activo" | Un permiso no existe o no está activo |
| `400` | "No se pueden agregar permisos API directamente..." | Se intentó agregar permisos API |
| `400` | "Todos los permisos ya están asignados al rol" | Todos los permisos ya existen en el rol |
| `500` | "Error al crear los permisos del rol" | Error interno del servidor |

---

## 🔑 Puntos Clave

### 1. Expansión Automática de Permisos Backend
- Los permisos backend se agregan **automáticamente** cuando un permiso App los requiere
- No es necesario (ni permitido) agregar permisos API manualmente
- La expansión es recursiva: si un permiso backend requiere otro, también se agrega

### 2. Prevención de Duplicados
- El sistema verifica permisos existentes antes de crear
- Los permisos duplicados se omiten silenciosamente
- Solo se crean relaciones nuevas

### 3. Propagación a Usuarios
- Por defecto (`applyToAllUsers = true`), los permisos se copian a todos los usuarios con el rol
- Si `applyToAllUsers = false`, solo se actualiza el rol sin afectar usuarios existentes
- Los permisos se copian a `auth-user-role-permission` con `isCustomPermission = false`

### 4. Procesamiento en Lotes
- Las operaciones masivas se dividen en lotes de 10,000 documentos
- Los lotes se procesan en paralelo para mejor rendimiento
- Usa `allowDiskUse(true)` y `read('secondaryPreferred')` para optimizar agregaciones grandes

### 5. Invalidación de Caché
- Después de copiar permisos a usuarios, se invalida su caché de permisos
- Esto asegura que los usuarios vean los nuevos permisos inmediatamente
- Los errores de invalidación no fallan la operación principal

### 6. Manejo de Errores Robusto
- Los errores al copiar permisos a usuarios individuales se registran pero no fallan la operación
- El rol se actualiza exitosamente incluso si algunos usuarios fallan
- Esto permite operaciones parcialmente exitosas en sistemas grandes

---

## 📊 Ejemplo de Uso

### Request:
```http
POST /api/auth_role_permission/create_many/69a7a9d146a26200e6768e07
Content-Type: application/json

{
  "applyToAllUsers": true,
  "permissions": [
    { "mongoId": "507f1f77bcf86cd799439011" },
    { "mongoId": "507f1f77bcf86cd799439012" }
  ]
}
```

### Escenario:
1. Se solicitan 2 permisos App al rol
2. El sistema valida que ambos existen y están activos
3. Verifica que no son permisos API
4. Verifica que no están duplicados
5. Expande automáticamente los permisos backend requeridos (ej: 3 permisos API)
6. Crea 5 documentos en `auth-role-permission` (2 App + 3 API)
7. Si `applyToAllUsers = true`, copia los 5 permisos a todos los usuarios con el rol
8. Invalida la caché de permisos de todos los usuarios afectados

### Response:
```json
{
  "status": "success",
  "statusCode": "201",
  "message": "Permisos agregados exitosamente al rol.",
  "data": [
    {
      "_id": "...",
      "roleMongoId": "69a7a9d146a26200e6768e07",
      "permissionMongoId": "507f1f77bcf86cd799439011",
      "code": "BUSINESS_PARTNER:CREATE",
      "name": "Crear Socio de Negocio",
      "isApiPermission": false,
      ...
    },
    // ... más permisos (incluyendo los expandidos automáticamente)
  ]
}
```

---

## 🔍 Consideraciones para el Frontend

### 1. Manejo de `applyToAllUsers`
- El frontend debe permitir al usuario decidir si propagar cambios a usuarios
- Por defecto debe ser `true` para mantener consistencia
- Si es `false`, solo se actualiza el rol (útil para cambios graduales)

### 2. No Enviar Permisos API
- El frontend NO debe permitir seleccionar permisos API directamente
- Solo debe mostrar permisos App (`isApiPermission = false`)
- Los permisos API se agregarán automáticamente por el backend

### 3. Manejo de Duplicados
- El frontend puede verificar permisos existentes antes de enviar
- Pero el backend maneja duplicados automáticamente, así que no es crítico

### 4. Feedback al Usuario
- Informar que los permisos backend se agregarán automáticamente
- Mostrar claramente cuántos permisos se agregarán (App + backend expandidos)
- Si `applyToAllUsers = true`, informar cuántos usuarios serán afectados

### 5. Validación de Permisos Activos
- Solo mostrar permisos con `statusId = 1` (ACTIVE)
- Filtrar permisos eliminados (`_deleted = false`)

---

## 📝 Notas Adicionales

- El endpoint usa **soft delete** (`_deleted = true`) en lugar de eliminación física
- Todas las operaciones incluyen metadatos de auditoría (`createdBy`, `updatedBy`, etc.)
- El servicio usa `PermissionExpansionService` y `PermissionConsolidationService` de `@Albatros-Virtual-Ecosystem/ave-auth-decorators-package`
- Los permisos se almacenan en la colección `auth-role-permission`
- Los permisos de usuario se almacenan en `auth-user-role-permission` cuando se propagan

---

## 🔗 Archivos Relacionados

- **Controlador:** `src/authorization/auth-role-permission/auth-role-permission-crud.controller.ts`
- **Servicio:** `src/authorization/auth-role-permission/auth-role-permission.service.ts`
- **DTO:** `src/authorization/auth-role-permission/dto/create-many-role-permissions.dto.ts`
- **Repositorio:** `src/authorization/auth-role-permission/auth-role-permission.repository.ts`

---

**Fecha de análisis:** 2026-03-08
**Versión del código analizado:** Actual
