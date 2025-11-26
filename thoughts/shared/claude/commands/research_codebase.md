# /research_codebase - Investigar y Documentar el Codebase

## Descripción

Conduce investigación comprehensiva a través del codebase para responder preguntas del usuario generando sub-agentes paralelos y sintetizando sus hallazgos. Documenta el codebase tal como existe hoy.

## Modelo

`opus` - Usa el modelo más potente para investigación profunda

## **CRÍTICO: Tu Único Trabajo es Documentar y Explicar el Codebase Como Existe Hoy**

- **NO** sugieras mejoras o cambios
- **NO** realices análisis de causa raíz
- **NO** propongas mejoras futuras
- **NO** critiques la implementación o identifiques problemas
- **NO** recomiend refactorings, optimizaciones o cambios arquitectónicos
- **SOLO** describe qué existe, dónde existe, cómo funciona, y cómo interactúan los componentes
- Estás creando un mapa técnico/documentación del sistema existente

## Configuración Inicial

Cuando se invoca el comando, responde con:
```
I'm ready to research the codebase. Please provide your research question or area of interest,
and I'll analyze it thoroughly by exploring relevant components and connections.
```

Luego espera la consulta de investigación del usuario.

## Pasos Después de Recibir la Consulta de Investigación

### 1. Lee Cualquier Archivo Mencionado Directamente Primero

- Si el usuario menciona archivos específicos (tickets, docs, JSON), **léelos COMPLETAMENTE primero**
- **IMPORTANTE**: Usa la herramienta Read SIN parámetros limit/offset
- **CRÍTICO**: Lee estos archivos tú mismo en el contexto principal antes de generar sub-tareas
- Esto asegura que tengas contexto completo antes de descomponer la investigación

### 2. Analiza y Descompone la Pregunta de Investigación

- Desglosa la consulta del usuario en áreas de investigación componibles
- Tómate tiempo para pensar ultra-profundo sobre los patrones subyacentes, conexiones e implicaciones arquitectónicas que el usuario podría estar buscando
- Identifica componentes específicos, patrones o conceptos a investigar
- Crea un plan de investigación usando TodoWrite para rastrear todas las subtareas
- Considera qué directorios, archivos o patrones arquitectónicos son relevantes

### 3. Genera Tareas de Sub-agentes Paralelas para Investigación Comprehensiva

Usa los agentes especializados disponibles:

**Para investigación del codebase:**
- **codebase-locator**: Encuentra DÓNDE viven los archivos y componentes
- **codebase-analyzer**: Entiende CÓMO funciona el código específico (sin criticarlo)
- **codebase-pattern-finder**: Encuentra ejemplos de patrones existentes (sin evaluarlos)

**Para directorio thoughts:**
- **thoughts-locator**: Descubre qué documentos existen sobre el tema
- **thoughts-analyzer**: Extrae insights clave de documentos específicos

**Para investigación web (solo si el usuario lo pide explícitamente):**
- **web-search-researcher**: Para documentación externa y recursos
- **SI usas agentes de investigación web, instrúyelos a devolver ENLACES con sus hallazgos, e INCLUYE esos enlaces en tu reporte final**

### 4. Espera a que Todos los Sub-agentes Completen y Sintetiza Hallazgos

- **IMPORTANTE**: Espera a que TODAS las tareas de sub-agentes se completen antes de proceder
- Compila todos los resultados de sub-agentes
- Prioriza hallazgos del codebase en vivo como fuente primaria de verdad
- Conecta hallazgos a través de diferentes componentes
- Incluye rutas de archivo específicas y números de línea para referencia
- Resalta patrones, conexiones y decisiones arquitectónicas
- Responde las preguntas específicas del usuario con evidencia concreta

### 5. Reúne Metadatos para el Documento de Investigación

- Ejecuta el script `hack/spec_metadata.sh` para generar todos los metadatos relevantes
- **Nombre de archivo**: `thoughts/shared/research/YYYY-MM-DD-ENG-XXXX-description.md`
  - Formato: `YYYY-MM-DD-ENG-XXXX-description.md` donde:
    - YYYY-MM-DD es la fecha de hoy
    - ENG-XXXX es el número de ticket (omite si no hay ticket)
    - description es una breve descripción en kebab-case del tema de investigación
  - Ejemplos:
    - Con ticket: `2025-01-08-ENG-1478-parent-child-tracking.md`
    - Sin ticket: `2025-01-08-authentication-flow.md`

### 6. Genera Documento de Investigación

Usa la metadata recopilada en el paso 5. Estructura el documento con frontmatter YAML seguido de contenido:

```markdown
---
date: [Fecha y hora actual con timezone en formato ISO]
researcher: [Nombre del investigador desde thoughts status]
git_commit: [Hash del commit actual]
branch: [Nombre de la rama actual]
repository: [Nombre del repositorio]
topic: "[Pregunta/Tema del Usuario]"
tags: [research, codebase, relevant-component-names]
status: complete
last_updated: [Fecha actual en formato YYYY-MM-DD]
last_updated_by: [Nombre del investigador]
---

# Research: [Pregunta/Tema del Usuario]

**Date**: [Fecha y hora actual con timezone desde paso 4]
**Researcher**: [Nombre del investigador desde metadata]
**Git Commit**: [Hash del commit actual desde paso 4]
**Branch**: [Nombre de la rama actual desde paso 4]
**Repository**: [Nombre del repositorio]

## Research Question
[Consulta original del usuario]

## Summary
[Documentación de alto nivel de qué se encontró, respondiendo la pregunta del usuario describiendo qué existe]

## Detailed Findings

### [Component/Area 1]
- Descripción de qué existe ([file.ext:line](link))
- Cómo conecta con otros componentes
- Detalles de implementación actuales (sin evaluación)

### [Component/Area 2]
...

## Code References
- `path/to/file.py:123` - Descripción de qué hay ahí
- `another/file.ts:45-67` - Descripción del bloque de código

## Architecture Documentation
[Patrones actuales, convenciones e implementaciones de diseño encontradas en el codebase]

## Historical Context (from thoughts/)
[Insights relevantes desde el directorio thoughts/ con referencias]
- `thoughts/shared/something.md` - Decisión histórica sobre X
- `thoughts/local/notes.md` - Exploración pasada de Y
Nota: Las rutas excluyen "searchable/" incluso si se encuentran ahí

## Related Research
[Enlaces a otros documentos de investigación en thoughts/shared/research/]

## Open Questions
[Cualquier área que necesita investigación adicional]
```

### 7. Agrega Permalinks de GitHub (si aplica)

- Verifica si estás en rama main o si el commit está pushed
- Si está en main/master o pushed, genera permalinks de GitHub
- Reemplaza referencias de archivos locales con permalinks en el documento

### 8. Sincroniza y Presenta Hallazgos

- Ejecuta `humanlayer thoughts sync` para sincronizar el directorio thoughts
- Presenta un resumen conciso de hallazgos al usuario
- Incluye referencias clave de archivo para fácil navegación
- Pregunta si tienen preguntas de seguimiento o necesitan aclaración

## Notas Importantes

- Siempre usa agentes Task paralelos para maximizar eficiencia y minimizar uso de contexto
- Siempre ejecuta investigación del codebase fresca - nunca dependas solo de documentos de investigación existentes
- El directorio thoughts/ proporciona contexto histórico para complementar hallazgos en vivo
- Enfócate en encontrar rutas de archivo concretas y números de línea para referencia del desarrollador
- Los documentos de investigación deben ser auto-contenidos con todo el contexto necesario
- Documenta conexiones entre componentes y cómo los sistemas interactúan
- Incluye contexto temporal (cuándo se condujo la investigación)
- Enlaza a GitHub cuando sea posible para referencias permanentes
- **CRÍTICO**: Tú y todos los sub-agentes son documentalistas, no evaluadores
- **RECUERDA**: Documenta lo que ES, no lo que DEBERÍA SER
- **SIN RECOMENDACIONES**: Solo describe el estado actual del codebase

## Uso

```bash
/research_codebase
# Luego proporciona tu pregunta de investigación cuando se te solicite
```

## Ver También

- [/research_codebase_nt](research_codebase_nt.md) - Variante sin directorio thoughts
- [/project_knowledge](project_knowledge.md) - Para documentación comprehensiva del proyecto
- [codebase-locator](../agents/codebase-locator.md) - Agente para encontrar archivos
- [codebase-analyzer](../agents/codebase-analyzer.md) - Agente para analizar código
