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
│   └── YYYY-MM-DD/       # Organizado por fecha
├── implementations/      # Detalles de implementación
│   └── YYYY-MM-DD/       # Organizado por fecha
├── api/                  # Documentación de API
│   └── YYYY-MM-DD/       # Organizado por fecha
├── decisions/            # ADRs (Architecture Decision Records)
│   └── YYYY-MM-DD/       # Organizado por fecha
├── ai-analysis/          # Análisis generados por IA
│   └── YYYY-MM-DD/       # Organizado por fecha
├── issues/               # Problemas y bugs
│   └── YYYY-MM-DD/       # Organizado por fecha
├── changelog/            # Historial de cambios
└── notes/                # Notas misceláneas
    └── YYYY-MM-DD/       # Organizado por fecha
```

**IMPORTANTE**: La documentación diaria se organiza por fecha dentro de cada tipo de carpeta. Cada día de trabajo genera una carpeta nueva con formato `YYYY-MM-DD`.

#### 5. **Categorías de Proyectos**
- `freelance/` - Proyectos freelance (incluye proyectos de segunda empresa)
- `outsourcing/` - Proyectos de outsourcing
- `work-main-company/` - Proyectos de la empresa principal
- `personal/` - Proyectos personales

### Documentación Diaria por Proyecto

**REGLA CRÍTICA**: Cuando se está trabajando en un proyecto específico (ej: microservicios de Albatros, frontend backoffice, etc.), **TODA la documentación generada ese día debe agregarse dentro del proyecto correspondiente, organizada por fecha**.

#### Estructura de Documentación Diaria por Proyecto

```
[proyecto]/
├── [tipo-documentacion]/
│   └── YYYY-MM-DD/          # Carpeta con fecha del día
│       ├── documento-1.md
│       ├── documento-2.md
│       └── ...
```

#### Ejemplos de Ubicación por Tipo

- **Arquitectura del día**: `[proyecto]/architecture/YYYY-MM-DD/[nombre].md`
- **Implementaciones del día**: `[proyecto]/implementations/YYYY-MM-DD/[nombre].md`
- **Análisis de IA del día**: `[proyecto]/ai-analysis/YYYY-MM-DD/[nombre].md`
- **Decisiones del día**: `[proyecto]/decisions/YYYY-MM-DD/[nombre].md`
- **API del día**: `[proyecto]/api/YYYY-MM-DD/[nombre].md`
- **Issues del día**: `[proyecto]/issues/YYYY-MM-DD/[nombre].md`
- **Notas del día**: `[proyecto]/notes/YYYY-MM-DD/[nombre].md`

#### Ejemplo Práctico: Microservicio ave-order

Si estás trabajando en el microservicio `ave-order` el día 2026-03-07:

```
01-projects/freelance/albatros/backend/microservicios/ave-order/
├── architecture/
│   └── 2026-03-07/
│       └── order-service-architecture.md
├── implementations/
│   └── 2026-03-07/
│       └── payment-integration.md
├── ai-analysis/
│   └── 2026-03-07/
│       └── code-review-analysis.md
└── notes/
    └── 2026-03-07/
        └── meeting-notes.md
```

#### Reglas para Documentación Diaria

1. **SIEMPRE crear carpeta con fecha** en formato `YYYY-MM-DD` dentro del tipo de documentación correspondiente
2. **SIEMPRE usar la fecha del día actual** cuando se genera documentación
3. **SIEMPRE colocar la documentación en el proyecto específico** donde se está trabajando
4. **NUNCA mezclar documentación de diferentes días** en la misma carpeta
5. **NUNCA crear documentación sin la estructura de fecha** cuando es documentación de trabajo diario

### Workflow Recomendado para IA

1. **Identificar** el proyecto en el que se está trabajando
2. **Identificar** el tipo de contenido a documentar
3. **Crear carpeta con fecha** `YYYY-MM-DD` dentro del tipo correspondiente del proyecto
4. **Seleccionar** la ubicación correcta: `[proyecto]/[tipo]/YYYY-MM-DD/[nombre].md`
5. **Usar** el template apropiado de `05-templates/`
6. **Incluir** enlaces a documentos relacionados usando `[[nombre]]`
7. **Guardar** en el directorio correcto dentro de `engineering-brain`
8. **NUNCA** generar documentación fuera de este directorio

### Recordatorios Importantes para IA

- ✅ **SIEMPRE** generar documentación dentro de `engineering-brain`
- ✅ **SIEMPRE** seguir la estructura de directorios definida
- ✅ **SIEMPRE** usar templates cuando estén disponibles
- ✅ **SIEMPRE** incluir enlaces Obsidian a documentos relacionados
- ✅ **SIEMPRE** crear carpeta con fecha `YYYY-MM-DD` para documentación diaria de proyectos
- ✅ **SIEMPRE** colocar documentación en el proyecto específico donde se está trabajando
- ✅ **NUNCA** generar documentación fuera de este directorio
- ✅ **NUNCA** crear archivos sin seguir la estructura definida
- ✅ **NUNCA** mezclar documentación de diferentes días en la misma carpeta
- ✅ Este proyecto es **independiente** - no requiere configuración en otros proyectos

### Ejemplos de Proyectos con Documentación Diaria

#### Proyecto Albatros - Microservicios
- **Ubicación base**: `01-projects/freelance/albatros/backend/microservicios/[nombre-microservicio]/`
- **Ejemplo**: `01-projects/freelance/albatros/backend/microservicios/ave-order/architecture/2026-03-07/`

#### Proyecto Albatros - Frontend
- **Ubicación base**: `01-projects/freelance/albatros/frontend/backoffice/`
- **Ejemplo**: `01-projects/freelance/albatros/frontend/backoffice/implementations/2026-03-07/`

#### Proyecto Albatros - Packages
- **Ubicación base**: `01-projects/freelance/albatros/packages/[nombre-package]/`
- **Packages disponibles**: 
  - `ave-auth-decorators-package` - Decoradores de autenticación
  - `ave-authorization-package` - Package de autorización
  - `ave-models-package` - Modelos compartidos
  - `ave-utils-npm` - Utilidades compartidas
- **Ejemplo**: `01-projects/freelance/albatros/packages/ave-auth-decorators-package/notes/2026-03-07/`

---

## 📁 Estructura

```
engineering-brain/
├── 01-projects/          # Proyectos profesionales
│   ├── freelance/        # Proyectos freelance (incluye segunda empresa)
│   ├── outsourcing/      # Proyectos de outsourcing
│   ├── work-main-company/ # Proyectos empresa principal
│   └── personal/         # Proyectos personales
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
