# /iterate_plan - Iterar Sobre Plan Existente

## Descripción

Actualiza planes de implementación existentes basándose en feedback del usuario. Es escéptico, exhaustivo y asegura que los cambios estén fundamentados en la realidad del codebase.

## Modelo

`opus` - Usa el modelo más potente para actualizar planes críticos

## Respuesta Inicial

Cuando se invoca el comando, maneja diferentes escenarios de entrada:

### Si NO se proporciona archivo de plan:
```
I'll help you iterate on an existing implementation plan.

Which plan would you like to update? Please provide the path to the plan file
(e.g., `thoughts/shared/plans/2025-10-16-feature.md`).

Tip: You can list recent plans with `ls -lt thoughts/shared/plans/ | head`
```
Espera entrada del usuario, luego verifica de nuevo por feedback.

### Si se proporciona plan pero NO feedback:
```
I've found the plan at [path]. What changes would you like to make?

For example:
- "Add a phase for migration handling"
- "Update the success criteria to include performance tests"
- "Adjust the scope to exclude feature X"
- "Split Phase 2 into two separate phases"
```
Espera entrada del usuario.

### Si se proporcionan TANTO plan COMO feedback:
- Procede inmediatamente al Paso 1
- No se necesitan preguntas preliminares

## Pasos del Proceso

### Paso 1: Lee y Entiende el Plan Actual

1. **Lee el archivo del plan existente COMPLETAMENTE**:
   - Usa la herramienta Read SIN parámetros limit/offset
   - Entiende la estructura actual, fases y alcance
   - Nota los criterios de éxito y enfoque de implementación

2. **Entiende los cambios solicitados**:
   - Analiza qué quiere agregar/modificar/eliminar el usuario
   - Identifica si los cambios requieren investigación del codebase
   - Determina el alcance de la actualización

### Paso 2: Investiga Si es Necesario

**Solo genera tareas de investigación si los cambios requieren nueva comprensión técnica.**

Si el feedback del usuario requiere entender nuevos patrones de código o validar suposiciones:

1. **Crea una lista de tareas de investigación** usando TodoWrite

2. **Genera sub-tareas paralelas para investigación**:
   Usa el agente correcto para cada tipo de investigación:

   **Para investigación de código:**
   - `codebase-locator` - Para encontrar archivos relevantes
   - `codebase-analyzer` - Para entender detalles de implementación
   - `codebase-pattern-finder` - Para encontrar patrones similares

   **Para contexto histórico:**
   - `thoughts-locator` - Para encontrar investigación o decisiones relacionadas
   - `thoughts-analyzer` - Para extraer insights de documentos

   **Sé EXTREMADAMENTE específico sobre directorios**:
   - Si el cambio involucra "WUI", especifica directorio `humanlayer-wui/`
   - Si involucra "daemon", especifica directorio `hld/`
   - Incluye contexto de ruta completa en prompts

3. **Lee cualquier archivo nuevo identificado por investigación**:
   - Léelos COMPLETAMENTE en el contexto principal
   - Compara con los requisitos del plan

4. **Espera a que TODAS las sub-tareas se completen** antes de proceder

### Paso 3: Presenta Comprensión y Enfoque

Antes de hacer cambios, confirma tu comprensión:

```
Based on your feedback, I understand you want to:
- [Change 1 with specific detail]
- [Change 2 with specific detail]

My research found:
- [Relevant code pattern or constraint]
- [Important discovery that affects the change]

I plan to update the plan by:
1. [Specific modification to make]
2. [Another modification]

Does this align with your intent?
```

Obtén confirmación del usuario antes de proceder.

### Paso 4: Actualiza el Plan

1. **Haz ediciones enfocadas y precisas** al plan existente:
   - Usa la herramienta Edit para cambios quirúrgicos
   - Mantén la estructura existente a menos que se cambie explícitamente
   - Mantén todas las referencias file:line precisas
   - Actualiza criterios de éxito si es necesario

2. **Asegura consistencia**:
   - Si agregas una nueva fase, asegúrate de que siga el patrón existente
   - Si modificas alcance, actualiza sección "What We're NOT Doing"
   - Si cambias enfoque, actualiza sección "Implementation Approach"
   - Mantén la distinción entre criterios de éxito automatizados vs manuales

3. **Preserva estándares de calidad**:
   - Incluye rutas de archivo específicas y números de línea para contenido nuevo
   - Escribe criterios de éxito medibles
   - Usa comandos `make` para verificación automatizada
   - Mantén lenguaje claro y accionable

### Paso 5: Sincroniza y Revisa

1. **Sincroniza el plan actualizado**:
   ```bash
   humanlayer thoughts sync
   ```
   Esto asegura que los cambios estén apropiadamente indexados

2. **Presenta los cambios realizados**:
   ```
   I've updated the plan at `thoughts/shared/plans/[filename].md`

   Changes made:
   - [Specific change 1]
   - [Specific change 2]

   The updated plan now:
   - [Key improvement]
   - [Another improvement]

   Would you like any further adjustments?
   ```

3. **Prepárate para iterar más** basándose en feedback

## Guías Importantes

### 1. Ser Escéptico
- No aceptes ciegamente solicitudes de cambio que parezcan problemáticas
- Cuestiona feedback vago - pide aclaración
- Verifica viabilidad técnica con investigación de código
- Señala conflictos potenciales con fases del plan existentes

### 2. Ser Quirúrgico
- Haz ediciones precisas, no reescrituras completas
- Preserva buen contenido que no necesita cambiar
- Solo investiga lo necesario para los cambios específicos
- No sobre-ingenierices las actualizaciones

### 3. Ser Exhaustivo
- Lee el plan existente completo antes de hacer cambios
- Investiga patrones de código si los cambios requieren nueva comprensión técnica
- Asegura que las secciones actualizadas mantengan estándares de calidad
- Verifica que los criterios de éxito sigan siendo medibles

### 4. Ser Interactivo
- Confirma comprensión antes de hacer cambios
- Muestra qué planeas cambiar antes de hacerlo
- Permite correcciones de rumbo
- No desaparezcas en investigación sin comunicar

### 5. Rastrear Progreso
- Usa TodoWrite para rastrear tareas de actualización si son complejas
- Actualiza todos a medida que completes investigación
- Marca tareas como completadas cuando termines

### 6. Sin Preguntas Abiertas
- Si el cambio solicitado plantea preguntas, PREGUNTA
- Investiga u obtén aclaración inmediatamente
- NO actualices el plan con preguntas sin resolver
- Cada cambio debe ser completo y accionable

## Guías de Criterios de Éxito

Cuando actualices criterios de éxito, siempre mantén la estructura de dos categorías:

### 1. Verificación Automatizada (puede ejecutarse por agentes de ejecución):
- Comandos que pueden ejecutarse: `make test`, `npm run lint`, etc.
- Preferir comandos `make`: `make -C humanlayer-wui check` en lugar de `cd humanlayer-wui && bun run fmt`
- Archivos específicos que deben existir
- Compilación de código/verificación de tipos

### 2. Verificación Manual (requiere pruebas humanas):
- Funcionalidad UI/UX
- Performance bajo condiciones reales
- Casos extremos difíciles de automatizar
- Criterios de aceptación del usuario

## Mejores Prácticas para Generar Sub-tareas

Cuando generes sub-tareas de investigación:

1. **Solo genera si es realmente necesario** - no investigues para cambios simples
2. **Genera múltiples tareas en paralelo** para eficiencia
3. **Cada tarea debe estar enfocada** en un área específica
4. **Proporciona instrucciones detalladas** incluyendo:
   - Exactamente qué buscar
   - En qué directorios enfocarse
   - Qué información extraer
   - Formato de salida esperado
5. **Solicita referencias específicas file:line** en respuestas
6. **Espera a que todas las tareas se completen** antes de sintetizar
7. **Verifica resultados de sub-tareas** - si algo parece mal, genera tareas de seguimiento

## Flujos de Interacción de Ejemplo

**Escenario 1: Usuario proporciona todo por adelantado**
```
User: /iterate_plan thoughts/shared/plans/2025-10-16-feature.md - add phase for error handling
Assistant: [Lee plan, investiga patrones de error handling, actualiza plan]
```

**Escenario 2: Usuario proporciona solo archivo de plan**
```
User: /iterate_plan thoughts/shared/plans/2025-10-16-feature.md
Assistant: I've found the plan. What changes would you like to make?
User: Split Phase 2 into two phases - one for backend, one for frontend
Assistant: [Procede con actualización]
```

**Escenario 3: Usuario no proporciona argumentos**
```
User: /iterate_plan
Assistant: Which plan would you like to update? Please provide the path...
User: thoughts/shared/plans/2025-10-16-feature.md
Assistant: I've found the plan. What changes would you like to make?
User: Add more specific success criteria
Assistant: [Procede con actualización]
```

## Uso

```bash
/iterate_plan thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-feature.md - add phase for X
# O sin feedback para que te pregunte:
/iterate_plan thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-feature.md
```

## Ver También

- [/create_plan](create_plan.md) - Para crear planes iniciales
- [/validate_plan](validate_plan.md) - Para validar implementación
- [/implement_plan](implement_plan.md) - Para ejecutar el plan
