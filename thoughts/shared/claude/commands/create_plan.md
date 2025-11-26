# /create_plan - Crear Plan de Implementación Detallado

## Descripción

Crea planes de implementación detallados a través de un proceso interactivo e iterativo. Este comando es **escéptico**, **exhaustivo** y trabaja colaborativamente con el usuario para producir especificaciones técnicas de alta calidad.

## Modelo

`opus` - Usa el modelo más potente para planificación crítica

## Flujo de Trabajo

### 1. Recopilación de Contexto e Análisis Inicial

- Lee todos los archivos mencionados COMPLETAMENTE (sin limit/offset)
- Genera tareas de investigación para recopilar contexto usando agentes especializados:
  - `codebase-locator`: Encuentra archivos relacionados
  - `codebase-analyzer`: Entiende la implementación actual
  - `thoughts-locator`: Encuentra documentos existentes
- Lee TODOS los archivos identificados en contexto principal
- Analiza y verifica la comprensión
- Presenta comprensión informada con preguntas enfocadas

### 2. Investigación y Descubrimiento

- Crea una lista de tareas de investigación con TodoWrite
- Genera múltiples sub-tareas en paralelo usando agentes especializados
- Espera a que TODAS las sub-tareas se completen
- Presenta hallazgos y opciones de diseño

### 3. Desarrollo de Estructura del Plan

- Crea esquema inicial del plan
- Obtiene feedback sobre la estructura antes de escribir detalles

### 4. Escritura Detallada del Plan

- **Ubicación**: `thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-description.md`
- **Formato del Nombre**:
  - Con ticket: `2025-01-08-ENG-1478-parent-child-tracking.md`
  - Sin ticket: `2025-01-08-improve-error-handling.md`

#### Estructura del Plan

```markdown
# [Feature/Task Name] Implementation Plan

## Overview
[Breve descripción]

## Current State Analysis
[Qué existe ahora, qué falta, restricciones descubiertas]

## Desired End State
[Especificación del estado final deseado y cómo verificarlo]

### Key Discoveries:
- [Hallazgo importante con referencia file:line]

## What We're NOT Doing
[Items explícitamente fuera del alcance]

## Implementation Approach
[Estrategia de alto nivel y razonamiento]

## Phase 1: [Descriptive Name]

### Overview
[Qué logra esta fase]

### Changes Required:
#### 1. [Component/File Group]
**File**: `path/to/file.ext`
**Changes**: [Resumen de cambios]

### Success Criteria:

#### Automated Verification:
- [ ] Comando verificable: `make test`
- [ ] Build pasa: `make build`

#### Manual Verification:
- [ ] Feature funciona en UI
- [ ] Performance aceptable

**Implementation Note**: Después de completar esta fase y toda la verificación automatizada pasa, pausar aquí para confirmación manual del humano de que las pruebas manuales fueron exitosas antes de proceder a la siguiente fase.
```

### 5. Sincronizar y Revisar

- Ejecuta `humanlayer thoughts sync`
- Presenta ubicación del borrador del plan
- Itera basándose en feedback

## Principios Importantes

### 1. Ser Escéptico
- Cuestiona requisitos vagos
- Identifica problemas potenciales temprano
- Pregunta "por qué" y "qué pasa si"
- No asumas - verifica con código

### 2. Ser Interactivo
- No escribas el plan completo de una vez
- Obtén aprobación en cada paso importante
- Permite correcciones de rumbo
- Trabaja colaborativamente

### 3. Ser Exhaustivo
- Lee TODOS los archivos de contexto COMPLETAMENTE
- Investiga patrones de código reales usando sub-tareas paralelas
- Incluye rutas de archivo específicas y números de línea
- Escribe criterios de éxito medibles con distinción clara entre automatizado vs manual

### 4. Ser Práctico
- Enfócate en cambios incrementales y probables
- Considera migración y rollback
- Piensa en casos extremos
- Incluye "lo que NO estamos haciendo"

### 5. Sin Preguntas Abiertas en el Plan Final
- Si encuentras preguntas abiertas durante la planificación, DETENTE
- Investiga o pide aclaraciones inmediatamente
- NO escribas el plan con preguntas sin resolver
- El plan debe ser completo y accionable
- Cada decisión debe estar tomada antes de finalizar

## Criterios de Éxito

**Siempre separa los criterios de éxito en dos categorías:**

### 1. Verificación Automatizada (puede ejecutarse por agentes)
- Comandos que se pueden ejecutar: `make test`, `npm run lint`
- Archivos específicos que deben existir
- Compilación de código/verificación de tipos
- Suites de pruebas automatizadas
- **Preferir comandos `make`**: `make -C humanlayer-wui check` en lugar de `cd humanlayer-wui && bun run fmt`

### 2. Verificación Manual (requiere pruebas humanas)
- Funcionalidad UI/UX
- Performance bajo condiciones reales
- Casos extremos difíciles de automatizar
- Criterios de aceptación del usuario

## Patrones Comunes

### Para Cambios de Base de Datos
1. Comenzar con schema/migration
2. Agregar métodos de store
3. Actualizar lógica de negocio
4. Exponer vía API
5. Actualizar clientes

### Para Nuevas Features
1. Investigar patrones existentes primero
2. Comenzar con modelo de datos
3. Construir lógica de backend
4. Agregar endpoints de API
5. Implementar UI al final

### Para Refactoring
1. Documentar comportamiento actual
2. Planear cambios incrementales
3. Mantener compatibilidad hacia atrás
4. Incluir estrategia de migración

## Mejores Prácticas para Sub-tareas

- Genera múltiples tareas en paralelo para eficiencia
- Cada tarea debe enfocarse en un área específica
- Proporciona instrucciones detalladas
- **Sé EXTREMADAMENTE específico sobre directorios**: Incluye el contexto de ruta completa en tus prompts
- Especifica herramientas de solo lectura a usar
- Solicita referencias específicas file:line en respuestas
- Espera a que todas las tareas se completen antes de sintetizar
- **Verifica resultados de sub-tareas**: Si una sub-tarea devuelve resultados inesperados, genera tareas de seguimiento

## Uso

```bash
/create_plan
# O con archivo de ticket:
/create_plan thoughts/shared/tickets/eng_1234.md
# O con análisis profundo:
/create_plan think deeply about thoughts/shared/tickets/eng_1234.md
```

## Ver También

- [/iterate_plan](iterate_plan.md) - Para actualizar planes existentes
- [/implement_plan](implement_plan.md) - Para implementar el plan creado
- [/validate_plan](validate_plan.md) - Para validar la implementación
