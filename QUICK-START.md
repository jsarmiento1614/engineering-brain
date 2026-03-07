# Quick Start Guide

## Configuración Inicial

### 1. Abrir en Obsidian

1. Abre Obsidian
2. Abre vault existente
3. Selecciona la carpeta `C:\develop\engineering-brain`

### 2. Plugins Recomendados

Instala estos plugins desde Community Plugins:

- **Dataview**: Para consultar notas como base de datos
- **Templater**: Para automatizar creación de documentos
- **Calendar**: Para gestión del journal diario
- **Excalidraw**: Para diagramas visuales
- **Mermaid**: Ya incluido, para diagramas en Markdown

### 3. Configurar Templater

1. Instala Templater plugin
2. Configura la carpeta de templates: `05-templates`
3. Configura el template folder en Settings > Templater

## Uso Diario

### Crear Nueva Nota de Proyecto

1. Navega al proyecto correspondiente
2. Usa el template apropiado desde `05-templates`
3. Guarda en la carpeta correcta según el tipo

### Registrar Trabajo Diario

1. Abre `03-journal/2026/`
2. Crea o edita el archivo del día: `YYYY-MM-DD.md`
3. Documenta trabajo realizado

### Agregar Conocimiento Transversal

1. Navega a `02-knowledge/`
2. Crea documento en la categoría apropiada
3. Enlaza desde proyectos relacionados usando `[[nombre-documento]]`

## Workflow con IA

1. Implementa código
2. Pide a IA que analice cambios
3. IA genera documentación usando templates
4. Guarda en el vault
5. Enlaza a documentos relacionados

## Convenciones de Commits Git

```
docs: add login flow documentation
docs: update architecture diagrams
notes: learning kafka streams
analysis: review payment service
feat: add new project structure
fix: correct documentation links
```

## Estructura de Proyecto

Cada proyecto debe tener:

- `index.md` - Dashboard del proyecto
- `architecture/` - Documentación de arquitectura
- `implementations/` - Detalles de implementación
- `api/` - Documentación de API
- `decisions/` - ADRs (Architecture Decision Records)
- `ai-analysis/` - Análisis generados por IA
- `issues/` - Problemas y bugs
- `changelog/` - Historial de cambios
- `notes/` - Notas misceláneas

## Enlaces

Usa enlaces de Obsidian `[[nombre-documento]]` para conectar conocimiento.

## Dashboards

- [[projects]] - Vista de todos los proyectos
- [[knowledge-map]] - Mapa de conocimiento
- [[learning-roadmap]] - Roadmap de aprendizaje

## Próximos Pasos

1. Configura Obsidian con plugins recomendados
2. Personaliza templates según tus necesidades
3. Comienza a documentar tus proyectos actuales
4. Crea entradas diarias en el journal
5. Conecta conocimiento usando enlaces
