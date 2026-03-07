# Configuración de GitHub MCP en Cursor

## ✅ Sí, puedes agregar GitHub vía MCP

GitHub ofrece un servidor MCP (Model Context Protocol) oficial que puedes integrar en Cursor.

## Pasos de Configuración

### 1. Crear un Personal Access Token (PAT) en GitHub

1. Ve a GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Genera un nuevo token con los siguientes permisos:
   - `repo` (acceso completo a repositorios)
   - `read:org` (si necesitas acceso a organizaciones)
   - `read:user` (información del usuario)
   - `read:gpg_key` (si necesitas verificar commits)

### 2. Configurar la Variable de Entorno

**En Windows (PowerShell):**
```powershell
# Temporal (solo para esta sesión)
$env:GITHUB_PERSONAL_ACCESS_TOKEN = "tu_token_aqui"

# Permanente (para todas las sesiones)
[System.Environment]::SetEnvironmentVariable("GITHUB_PERSONAL_ACCESS_TOKEN", "tu_token_aqui", "User")
```

**O crear un archivo `.env` en la raíz del proyecto:**
```
GITHUB_PERSONAL_ACCESS_TOKEN=tu_token_aqui
```

### 3. Archivo de Configuración MCP

Ya se ha creado el archivo `.vscode/mcp.json` en tu proyecto con la configuración básica.

### 4. Reiniciar Cursor

Después de configurar, reinicia Cursor para que cargue la nueva configuración MCP.

## Configuración Alternativa (OAuth)

Si prefieres usar OAuth en lugar de PAT, puedes usar el servidor remoto de GitHub:

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

**Nota:** OAuth requiere VS Code/Cursor versión 1.101 o superior y acceso a GitHub Copilot.

## Funcionalidades Disponibles

Una vez configurado, podrás:
- ✅ Gestionar repositorios, issues y pull requests
- ✅ Analizar código y seguridad
- ✅ Automatizar flujos de trabajo de GitHub Actions
- ✅ Crear y actualizar issues mediante lenguaje natural
- ✅ Consultar información de repositorios y commits

## Verificación

Para verificar que está funcionando:
1. Abre Cursor
2. Verifica que el servidor MCP de GitHub aparezca en la lista de servidores MCP disponibles
3. Prueba hacer una consulta relacionada con GitHub desde el chat

## Referencias

- [Documentación oficial de GitHub MCP](https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp/use-the-github-mcp-server)
- [Repositorio del servidor MCP de GitHub](https://github.com/modelcontextprotocol/servers/tree/main/src/github)
