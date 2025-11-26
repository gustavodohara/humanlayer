# /create_plan_nt - Crear Plan sin Directorio Thoughts

## Descripción

Variante de `/create_plan` que crea planes de implementación sin usar el directorio thoughts. Útil para proyectos que no tienen sistema thoughts configurado.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Diferencias con `/create_plan`

Esta es una variante del comando `/create_plan` principal con estas diferencias:

- **Sin directorio thoughts**: No lee ni escribe a `thoughts/`
- **Sin investigación histórica**: No busca contexto en documentos thoughts
- **Sin sincronización thoughts**: No ejecuta `humanlayer thoughts sync`
- **Plan inline**: El plan puede guardarse en ubicación diferente o presentarse directamente

## Proceso

El proceso es similar a `/create_plan` pero simplificado:

1. **Recopilación de Contexto**: Lee archivos mencionados y investiga codebase
2. **Investigación**: Usa agentes de codebase (no thoughts-locator/thoughts-analyzer)
3. **Desarrollo de Estructura**: Crea esquema del plan
4. **Escritura Detallada**: Genera plan completo
5. **Presentación**: Muestra plan al usuario (sin guardar en thoughts/)

Ver [/create_plan](create_plan.md) para detalles completos del proceso de planificación.

## Agentes Usados

Solo agentes de codebase:
- `codebase-locator` - Para encontrar archivos relevantes
- `codebase-analyzer` - Para entender implementación actual
- `codebase-pattern-finder` - Para encontrar patrones similares

No usa:
- `thoughts-locator`
- `thoughts-analyzer`

## Estructura del Plan

La estructura del plan es la misma que en `/create_plan`, pero sin metadata thoughts:

```markdown
# [Feature/Task Name] Implementation Plan

## Overview
[Descripción breve]

## Current State Analysis
[Qué existe ahora]

## Desired End State
[Estado final deseado]

## Implementation Approach
[Estrategia de alto nivel]

## Phase 1: [Name]
...

## Phase 2: [Name]
...
```

## Cuándo Usar

Usa `/create_plan_nt` cuando:
- El proyecto no tiene directorio thoughts configurado
- No necesitas contexto histórico de investigación
- Quieres un plan independiente sin dependencies de thoughts
- Estás trabajando en repositorio externo sin thoughts

## Cuándo NO Usar

No uses `/create_plan_nt` si:
- El proyecto tiene sistema thoughts configurado (usa `/create_plan`)
- Necesitas referenciar investigación o decisiones pasadas
- Quieres que el plan sea parte de knowledge base del proyecto

## Uso

```bash
/create_plan_nt
# O con archivo de ticket:
/create_plan_nt path/to/ticket.md
```

## Ver También

- [/create_plan](create_plan.md) - Versión completa con thoughts
- [/research_codebase_nt](research_codebase_nt.md) - Investigación sin thoughts
- [/create_plan_generic](create_plan_generic.md) - Otra variante genérica
