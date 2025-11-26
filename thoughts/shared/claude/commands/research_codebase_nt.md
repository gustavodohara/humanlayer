# /research_codebase_nt - Documentar Codebase Sin Evaluaciones

## Descripción

Variante de `/research_codebase` que documenta el codebase tal como existe sin evaluaciones ni recomendaciones. No usa el directorio thoughts para almacenar resultados.

## Modelo

`opus` - Usa el modelo más potente para investigación profunda

## **CRÍTICO: Solo Documentar, No Evaluar**

- **NO** sugieras mejoras o cambios
- **NO** realices análisis de causa raíz
- **NO** propongas mejoras futuras
- **NO** critiques la implementación o identifiques problemas
- **NO** recomienda refactorings, optimizaciones o cambios arquitectónicos
- **SOLO** describe qué existe, dónde existe, cómo funciona, y cómo interactúan los componentes

## Diferencias con `/research_codebase`

Esta es una variante del comando `/research_codebase` principal con estas diferencias:

- **Sin directorio thoughts**: No lee investigación histórica de `thoughts/`
- **Sin sincronización thoughts**: No guarda resultados en `thoughts/shared/research/`
- **Sin thoughts-locator/thoughts-analyzer**: Solo usa agentes de codebase
- **Presentación directa**: Resultados presentados directamente al usuario
- **Sin contexto histórico**: Solo documenta estado actual del código

## Proceso

Proceso simplificado comparado con `/research_codebase`:

### 1. Lee Archivos Mencionados Primero
- Lee COMPLETAMENTE cualquier archivo proporcionado
- Sin usar limit/offset

### 2. Analiza y Descompone la Pregunta
- Desglosa la consulta en áreas de investigación
- Identifica componentes a investigar
- Crea plan usando TodoWrite

### 3. Genera Tareas de Sub-agentes Paralelas

Solo agentes de codebase:
- **codebase-locator**: Encuentra DÓNDE vive el código
- **codebase-analyzer**: Entiende CÓMO funciona
- **codebase-pattern-finder**: Encuentra ejemplos similares

No usa:
- `thoughts-locator`
- `thoughts-analyzer`

### 4. Espera y Sintetiza
- Espera que TODAS las tareas completen
- Sintetiza hallazgos
- Presenta resultados directamente al usuario

### 5. Presenta Hallazgos
- Resumen conciso de hallazgos
- Referencias clave de archivo para navegación
- Sin guardar en thoughts/

## Formato de Salida

Presenta hallazgos directamente en formato markdown:

```markdown
## Research: [User's Question/Topic]

### Summary
[High-level findings answering the user's question]

### Detailed Findings

#### [Component/Area 1]
- Finding with reference (file.ext:line)
- How it connects to other components
- Implementation details

#### [Component/Area 2]
...

### Code References
- `path/to/file.py:123` - Description
- `another/file.ts:45-67` - Description

### Architecture Documentation
[Current patterns and implementations found]
```

## Agentes Usados

Solo agentes de codebase:
- `codebase-locator` - Localizar archivos
- `codebase-analyzer` - Analizar implementación
- `codebase-pattern-finder` - Encontrar patrones

## Cuándo Usar

Usa `/research_codebase_nt` cuando:
- El proyecto no tiene directorio thoughts configurado
- No necesitas guardar investigación para referencia futura
- Quieres investigación rápida sin overhead de thoughts
- Estás trabajando en repositorio externo

## Cuándo NO Usar

No uses `/research_codebase_nt` si:
- El proyecto tiene sistema thoughts configurado (usa `/research_codebase`)
- Quieres guardar investigación para referencia futura
- Necesitas contexto histórico de decisiones pasadas

## Uso

```bash
/research_codebase_nt
# Luego proporciona tu pregunta de investigación
```

## Ver También

- [/research_codebase](research_codebase.md) - Versión completa con thoughts
- [/research_codebase_generic](research_codebase_generic.md) - Variante genérica
- [/create_plan_nt](create_plan_nt.md) - Planificación sin thoughts
