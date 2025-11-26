# /research_codebase_generic - Investigar Codebase con Sub-agentes Paralelos

## Descripción

Variante genérica de `/research_codebase` que conduce investigación comprehensiva usando sub-agentes paralelos. Enfoque más genérico adecuado para diferentes tipos de proyectos.

## Modelo

`opus` - Usa el modelo más potente para investigación profunda

## Diferencias con `/research_codebase`

Esta es una variante genérica del comando `/research_codebase` principal. Las diferencias pueden incluir:

- Enfoque más genérico para diferentes estructuras de proyecto
- Puede tener configuración diferente de thoughts/
- Puede usar proceso de investigación diferente
- Puede tener formato de salida diferente

## Proceso

El proceso es fundamentalmente similar a `/research_codebase`:

### 1. Lee Archivos Mencionados Primero
- Lee COMPLETAMENTE archivos proporcionados
- Sin usar limit/offset
- Lee en contexto principal antes de sub-tareas

### 2. Analiza y Descompone
- Desglosa la consulta en áreas de investigación
- Piensa ultra-profundo sobre patrones e implicaciones
- Crea plan con TodoWrite

### 3. Genera Sub-agentes Paralelas
Usa agentes especializados:
- **codebase-locator**: Encontrar archivos
- **codebase-analyzer**: Entender código
- **codebase-pattern-finder**: Encontrar patrones
- **thoughts-locator**: Descubrir documentos (si aplica)
- **thoughts-analyzer**: Analizar docs (si aplica)

### 4. Espera y Sintetiza
- Espera que TODAS las tareas completen
- Compila resultados
- Conecta hallazgos
- Incluye rutas de archivo y números de línea

### 5. Genera Documento/Presenta
- Puede generar documento de investigación
- O presentar hallazgos directamente
- Dependiendo de configuración del proyecto

## Principios

Mantiene los mismos principios que `/research_codebase`:
- Usa agentes Task paralelos para eficiencia
- Ejecuta investigación fresca del codebase
- Enfócate en rutas de archivo concretas y números de línea
- Documenta qué ES, no qué DEBERÍA SER
- Sin recomendaciones - solo describe estado actual

## Formato de Salida

Similar a `/research_codebase` pero potencialmente adaptado:

```markdown
# Research: [Topic]

## Research Question
[Original query]

## Summary
[High-level findings]

## Detailed Findings
### [Component/Area]
- Finding with references
- Connections to other components

## Code References
- `path/to/file:line` - Description

## Architecture Insights
[Current patterns found]

## Related Research
[Links if applicable]
```

## Cuándo Usar

Usa `/research_codebase_generic` cuando:
- Necesitas enfoque de investigación más genérico
- El proyecto tiene estructura no estándar
- Quieres personalización en el proceso

## Cuándo NO Usar

Usa `/research_codebase` estándar cuando:
- El proyecto sigue estructura estándar
- Quieres el flujo optimizado
- Estás trabajando con tickets Linear

## Uso

```bash
/research_codebase_generic
# Luego proporciona tu pregunta de investigación
```

## Ver También

- [/research_codebase](research_codebase.md) - Versión estándar principal
- [/research_codebase_nt](research_codebase_nt.md) - Variante sin thoughts
- [/project_knowledge](project_knowledge.md) - Para documentación comprehensiva
