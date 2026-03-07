# Engineering Brain

Sistema de conocimiento profesional para ingeniería de software. Este repositorio centraliza toda la documentación, conocimiento y aprendizaje profesional a lo largo de la carrera.

## 🎯 Objetivo

Este proyecto es un **sistema de conocimiento independiente** que documenta:
- Proyectos profesionales (freelance, outsourcing, trabajo principal)
- Conocimiento técnico transversal y reutilizable
- Decisiones arquitectónicas (ADRs)
- Análisis y aprendizajes
- Bitácora profesional diaria

**IMPORTANTE**: Este directorio es el **único lugar** donde debe generarse y almacenarse toda la documentación profesional relacionada con proyectos, conocimiento técnico, análisis y aprendizaje.

---

## 📋 Instrucciones para IDEs con IA (Cursor, GitHub Copilot, etc.)

### Regla Principal

**SIEMPRE generar documentación dentro de este directorio** (`C:\develop\engineering-brain` o la ruta donde esté clonado). Este proyecto es **independiente** de otros proyectos en los que se esté trabajando.

### Cómo Manejar la Generación de Documentación

Cuando un IDE con IA necesite generar documentación, debe seguir estas reglas:

#### 1. **Identificar el Tipo de Contenido**
- **Documentación de Proyectos**: `01-projects/[categoria]/[proyecto]/[tipo]/`
- **Conocimiento Transversal**: `02-knowledge/[categoria]/[tema].md`
- **Registro Diario**: `03-journal/YYYY/YYYY-MM-DD.md`
- **Análisis de IA**: `01-projects/[proyecto]/ai-analysis/[nombre].md`
- **Decisiones Arquitectónicas**: `01-projects/[proyecto]/decisions/[nombre].md`
- **Documentación de Arquitectura**: `01-projects/[proyecto]/architecture/[nombre].md`

#### 2. **Usar Templates Disponibles**
- Consultar `05-templates/` antes de crear nuevos documentos
- Usar los templates como base para mantener consistencia

#### 3. **Seguir Convenciones**
- **Nombres de archivos**: `kebab-case.md` (ej: `payment-service.md`)
- **Enlaces Obsidian**: Usar `[[nombre-documento]]` para conectar conocimiento
- **Formato**: Markdown estándar
- **Fechas**: Formato `YYYY-MM-DD` cuando sea relevante

#### 4. **Estructura de Proyectos**
Cada proyecto debe tener:
```
[proyecto]/
├── index.md              # Dashboard del proyecto
├── architecture/         # Documentación de arquitectura
├── implementations/      # Detalles de implementación
├── api/                  # Documentación de API
├── decisions/            # ADRs (Architecture Decision Records)
├── ai-analysis/          # Análisis generados por IA
├── issues/               # Problemas y bugs
├── changelog/            # Historial de cambios
└── notes/                # Notas misceláneas
```

#### 5. **Categorías de Proyectos**
- `freelance/` - Proyectos freelance
- `outsourcing/` - Proyectos de outsourcing
- `work-main-company/` - Proyectos de la empresa principal
- `personal/` - Proyectos personales
- `second-company/` - Proyectos de segunda empresa

### Workflow Recomendado para IA

1. **Identificar** el tipo de contenido a documentar
2. **Seleccionar** la ubicación correcta según la estructura
3. **Usar** el template apropiado de `05-templates/`
4. **Incluir** enlaces a documentos relacionados usando `[[nombre]]`
5. **Guardar** en el directorio correcto dentro de `engineering-brain`
6. **NUNCA** generar documentación fuera de este directorio

### Recordatorios Importantes para IA

- ✅ **SIEMPRE** generar documentación dentro de `engineering-brain`
- ✅ **SIEMPRE** seguir la estructura de directorios definida
- ✅ **SIEMPRE** usar templates cuando estén disponibles
- ✅ **SIEMPRE** incluir enlaces Obsidian a documentos relacionados
- ✅ **NUNCA** generar documentación fuera de este directorio
- ✅ **NUNCA** crear archivos sin seguir la estructura definida
- ✅ Este proyecto es **independiente** - no requiere configuración en otros proyectos

---

## 📁 Estructura

```
engineering-brain/
├── 01-projects/          # Proyectos profesionales
│   ├── freelance/        # Proyectos freelance
│   ├── outsourcing/      # Proyectos de outsourcing
│   ├── work-main-company/ # Proyectos empresa principal
│   ├── personal/         # Proyectos personales
│   └── second-company/    # Proyectos segunda empresa
├── 02-knowledge/         # Conocimiento transversal
│   ├── ai/               # Conocimiento sobre IA
│   ├── architecture/     # Patrones y arquitecturas
│   ├── backend/          # Conocimiento backend
│   ├── devops/           # DevOps y CI/CD
│   ├── microservices/    # Microservicios
│   └── patterns/          # Patrones de diseño
├── 03-journal/           # Bitácora profesional diaria
│   └── YYYY/            # Organizado por año
│       └── YYYY-MM-DD.md
├── 04-resources/         # Material de referencia
│   ├── articles/        # Artículos guardados
│   ├── books/           # Resúmenes de libros
│   ├── research/        # Investigaciones
│   └── tools/           # Documentación de herramientas
├── 05-templates/         # Plantillas para documentación
│   ├── adr-template.md
│   ├── analysis-template.md
│   ├── architecture-template.md
│   └── implementation-template.md
├── 06-inbox/            # Notas sin clasificar (temporal)
└── 07-dashboards/       # Paneles de navegación
    ├── knowledge-map.md
    ├── learning-roadmap.md
    └── projects.md
```

## 📖 Uso

1. **Proyectos**: Cada proyecto tiene su propia carpeta con estructura consistente
2. **Knowledge**: Conocimiento reutilizable entre proyectos
3. **Journal**: Registro diario de trabajo y aprendizaje
4. **Templates**: Plantillas para generar documentación consistente

## 🗺️ Dashboards

- [[projects]] - Vista general de todos los proyectos
- [[knowledge-map]] - Mapa de conocimiento transversal
- [[learning-roadmap]] - Roadmap de aprendizaje

## 🔄 Convenciones de Commits Git

```
docs: add login flow documentation
docs: update architecture diagrams
notes: learning kafka streams
analysis: review payment service
feat: add new project structure
fix: correct documentation links
```

## 🔧 Herramientas

### Git
Este vault está versionado con Git para mantener historial de conocimiento y decisiones.

### Obsidian (Opcional)
Para visualización con enlaces `[[documento]]`:
- Dataview
- Templater
- Calendar
- Excalidraw
- Mermaid

## 📚 Más Información

Ver `QUICK-START.md` para guía de inicio rápido y uso diario.
