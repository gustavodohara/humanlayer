# /create_plan_generic - Crear Plan con Investigación Exhaustiva

## Descripción

Variante genérica de `/create_plan` que crea planes de implementación detallados con investigación exhaustiva e iteración. Similar a `/create_plan` pero potencialmente con enfoque más genérico o configuración diferente.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Diferencias con `/create_plan`

Esta es una variante del comando `/create_plan` principal. Las diferencias pueden incluir:

- Enfoque más genérico para diferentes tipos de proyectos
- Puede tener configuraciones diferentes de thoughts/
- Puede usar diferentes agentes o proceso de investigación
- Puede tener formato de salida diferente

## Proceso

El proceso es fundamentalmente similar a `/create_plan`:

1. **Recopilación de Contexto e Investigación Inicial**
2. **Investigación y Descubrimiento**
3. **Desarrollo de Estructura del Plan**
4. **Escritura Detallada del Plan**
5. **Sincronización y Revisión**

Ver [/create_plan](create_plan.md) para detalles completos del proceso.

## Principios

Mantiene los mismos principios que `/create_plan`:
- Ser escéptico
- Ser interactivo
- Ser exhaustivo
- Ser práctico
- Sin preguntas abiertas en el plan final

## Formato del Plan

Usa estructura similar a `/create_plan`:
- Overview
- Current State Analysis
- Desired End State
- Implementation Approach
- Fases con cambios específicos
- Criterios de éxito (automatizados y manuales)

## Cuándo Usar

Usa `/create_plan_generic` cuando:
- Necesitas planificación exhaustiva con enfoque genérico
- El proyecto tiene estructura no estándar
- Quieres personalización en el proceso de planificación

## Cuándo NO Usar

Usa `/create_plan` estándar cuando:
- El proyecto sigue estructura estándar de HumanLayer
- Quieres el flujo optimizado de planificación
- Estás trabajando con tickets Linear

## Uso

```bash
/create_plan_generic
# O con contexto:
/create_plan_generic path/to/requirements.md
```

## Ver También

- [/create_plan](create_plan.md) - Versión estándar principal
- [/create_plan_nt](create_plan_nt.md) - Variante sin thoughts
- [/iterate_plan](iterate_plan.md) - Para actualizar planes
