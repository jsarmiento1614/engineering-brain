# Configurar Variable de Entorno GITHUB_PERSONAL_ACCESS_TOKEN

## Opción 1: Variable de Entorno del Sistema (Recomendada)

### Método A: Interfaz Gráfica de Windows

1. Presiona `Win + R`, escribe `sysdm.cpl` y presiona Enter
2. Ve a la pestaña **"Opciones avanzadas"**
3. Haz clic en **"Variables de entorno"**
4. En "Variables de usuario", haz clic en **"Nueva"**
5. Nombre de variable: `GITHUB_PERSONAL_ACCESS_TOKEN`
6. Valor de variable: `tu_token_de_github_aqui`
7. Haz clic en **"Aceptar"** en todas las ventanas
8. **Reinicia Cursor** para que cargue la nueva variable

### Método B: PowerShell (Permanente)

Abre PowerShell como Administrador y ejecuta:

```powershell
[System.Environment]::SetEnvironmentVariable("GITHUB_PERSONAL_ACCESS_TOKEN", "tu_token_aqui", "User")
```

Luego **reinicia Cursor**.

### Método C: PowerShell (Temporal - Solo para esta sesión)

```powershell
$env:GITHUB_PERSONAL_ACCESS_TOKEN = "tu_token_aqui"
```

**Nota:** Esta configuración se pierde al cerrar PowerShell.

## Opción 2: Directamente en el archivo mcp.json (Menos Segura)

Si prefieres tener el token directamente en el archivo (como tienes con GitLab), puedes cambiar:

```json
"env": {
  "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_tu_token_aqui"
}
```

**⚠️ Advertencia:** Esto expone el token en texto plano en el archivo de configuración.

## Opción 3: Archivo .env (Si Cursor lo soporta)

Crear un archivo `.env` en `c:\Users\USER\.cursor\`:

```
GITHUB_PERSONAL_ACCESS_TOKEN=tu_token_aqui
```

## Crear un Personal Access Token en GitHub

1. Ve a: https://github.com/settings/tokens
2. Haz clic en **"Generate new token"** → **"Generate new token (classic)"**
3. Dale un nombre descriptivo (ej: "Cursor MCP")
4. Selecciona los permisos necesarios:
   - ✅ `repo` (acceso completo a repositorios)
   - ✅ `read:org` (si necesitas acceso a organizaciones)
   - ✅ `read:user` (información del usuario)
5. Haz clic en **"Generate token"**
6. **Copia el token inmediatamente** (solo se muestra una vez)

## Verificar que Funciona

1. Reinicia Cursor completamente
2. Verifica en la configuración de MCP que el servidor GitHub esté activo
3. Prueba hacer una consulta relacionada con GitHub desde el chat

## Nota Importante

La sintaxis `${GITHUB_PERSONAL_ACCESS_TOKEN}` en el archivo `mcp.json` indica que Cursor buscará una variable de entorno con ese nombre. Si no la encuentra, el servidor MCP no funcionará correctamente.
