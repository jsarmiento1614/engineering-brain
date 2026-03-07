# User Endpoints - GLIM

## Base URL
`/api/v1/users`

## Endpoints

### GET /users/:id
Obtener información de usuario.

**Response:**
```json
{
  "id": "string",
  "email": "string",
  "name": "string"
}
```

### POST /users
Crear nuevo usuario.

**Request:**
```json
{
  "email": "string",
  "password": "string",
  "name": "string"
}
```

### PUT /users/:id
Actualizar usuario.

### DELETE /users/:id
Eliminar usuario.

## Authentication

Todos los endpoints requieren JWT token en header:
```
Authorization: Bearer <token>
```

## Related

- [[login-flow]]
- [[adr-001-auth-strategy]]
