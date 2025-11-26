# /resume_handoff - Reanudar Trabajo desde Handoff

## Descripción

Reanuda trabajo desde un documento de handoff a través de un proceso interactivo. Los handoffs contienen contexto crítico, learnings y próximos pasos de sesiones de trabajo previas que necesitan ser entendidos y continuados.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Respuesta Inicial

### 1. Si se proporcionó ruta del documento de handoff

- Omite mensaje por defecto
- Lee el documento de handoff COMPLETAMENTE inmediatamente
- Lee inmediatamente cualquier documento de investigación o plan que enlace bajo `thoughts/shared/plans` o `thoughts/shared/research` - NO uses sub-agente para leer estos archivos críticos
- Comienza proceso de análisis ingiriendo contexto relevante del handoff, leyendo archivos adicionales que mencione
- Luego propone curso de acción al usuario y confirma, o pide aclaración sobre dirección

### 2. Si se proporcionó número de ticket (como ENG-XXXX)

```bash
# Sincroniza thoughts para asegurar actualización
humanlayer thoughts sync

# Localiza el handoff más reciente para el ticket
# Los tickets estarán en: thoughts/shared/handoffs/ENG-XXXX/
# Lista contenido del directorio
```

- **Si hay cero archivos o el directorio no existe**: "I'm sorry, I can't seem to find that handoff document. Can you please provide me with a path to it?"
- **Si hay solo un archivo**: Procede con ese handoff
- **Si hay múltiples archivos**: Usando la fecha y hora en el nombre del archivo (formato `YYYY-MM-DD_HH-MM-SS` en formato 24 horas), procede con el handoff más reciente

Luego:
- Lee el handoff COMPLETAMENTE
- Lee inmediatamente cualquier documento de investigación o plan que enlace
- Comienza análisis
- Propone curso de acción

### 3. Si no se proporcionaron parámetros

```
I'll help you resume work from a handoff document. Let me find the available handoffs.

Which handoff would you like to resume from?

Tip: You can invoke this command directly with a handoff path:
/resume_handoff thoughts/shared/handoffs/ENG-XXXX/YYYY-MM-DD_HH-MM-SS_ENG-XXXX_description.md

or using a ticket number to resume from the most recent handoff for that ticket:
/resume_handoff ENG-XXXX
```

## Pasos del Proceso

### Paso 1: Lee y Analiza el Handoff

1. **Lee el documento de handoff completamente**:
   - Usa herramienta Read SIN parámetros limit/offset
   - Extrae todas las secciones:
     - Task(s) y sus estados
     - Recent changes
     - Learnings
     - Artifacts
     - Action items y next steps
     - Other notes

2. **Genera tareas de investigación enfocadas**:
   Basándote en el contenido del handoff, genera tareas de investigación paralelas para verificar estado actual:

   ```
   Task 1 - Gather artifact context:
   Read all artifacts mentioned in the handoff.
   1. Read feature documents listed in "Artifacts"
   2. Read implementation plans referenced
   3. Read any research documents mentioned
   4. Extract key requirements and decisions
   Use tools: Read
   Return: Summary of artifact contents and key decisions
   ```

3. **Espera a que TODAS las sub-tareas completen** antes de proceder

4. **Lee archivos críticos identificados**:
   - Lee archivos de la sección "Learnings" completamente
   - Lee archivos de "Recent changes" para entender modificaciones
   - Lee cualquier archivo nuevo relacionado descubierto durante investigación

### Paso 2: Sintetiza y Presenta Análisis

1. **Presenta análisis comprehensivo**:
   ```
   I've analyzed the handoff from [date] by [researcher]. Here's the current situation:

   **Original Tasks:**
   - [Task 1]: [Status from handoff] → [Current verification]
   - [Task 2]: [Status from handoff] → [Current verification]

   **Key Learnings Validated:**
   - [Learning with file:line reference] - [Still valid/Changed]
   - [Pattern discovered] - [Still applicable/Modified]

   **Recent Changes Status:**
   - [Change 1] - [Verified present/Missing/Modified]
   - [Change 2] - [Verified present/Missing/Modified]

   **Artifacts Reviewed:**
   - [Document 1]: [Key takeaway]
   - [Document 2]: [Key takeaway]

   **Recommended Next Actions:**
   Based on the handoff's action items and current state:
   1. [Most logical next step based on handoff]
   2. [Second priority action]
   3. [Additional tasks discovered]

   **Potential Issues Identified:**
   - [Any conflicts or regressions found]
   - [Missing dependencies or broken code]

   Shall I proceed with [recommended action 1], or would you like to adjust the approach?
   ```

2. **Obtén confirmación** antes de proceder

### Paso 3: Crea Plan de Acción

1. **Usa TodoWrite para crear lista de tareas**:
   - Convierte action items del handoff en todos
   - Agrega cualquier nueva tarea descubierta durante análisis
   - Prioriza basándote en dependencias y guía del handoff

2. **Presenta el plan**:
   ```
   I've created a task list based on the handoff and current analysis:

   [Show todo list]

   Ready to begin with the first task: [task description]?
   ```

### Paso 4: Comienza Implementación

1. **Comienza con la primera tarea aprobada**
2. **Referencia learnings del handoff** durante la implementación
3. **Aplica patrones y enfoques documentados** en el handoff
4. **Actualiza progreso** a medida que se completen tareas

## Guías

### 1. Sé Exhaustivo en Análisis

- Lee el documento de handoff completo primero
- Verifica que TODOS los cambios mencionados aún existen
- Verifica regresiones o conflictos
- Lee todos los artifacts referenciados

### 2. Sé Interactivo

- Presenta hallazgos antes de comenzar trabajo
- Obtén buy-in en el enfoque
- Permite correcciones de curso
- Adáptate basándote en estado actual vs estado del handoff

### 3. Aprovecha la Sabiduría del Handoff

- Presta atención especial a la sección "Learnings"
- Aplica patrones y enfoques documentados
- Evita repetir errores mencionados
- Construye sobre soluciones descubiertas

### 4. Rastrea Continuidad

- Usa TodoWrite para mantener continuidad de tareas
- Referencia el documento de handoff en commits
- Documenta cualquier desviación del plan original
- Considera crear nuevo handoff cuando termines

### 5. Valida Antes de Actuar

- Nunca asumas que el estado del handoff coincide con el estado actual
- Verifica que todas las referencias de archivo aún existen
- Verifica breaking changes desde el handoff
- Confirma que los patrones siguen siendo válidos

## Escenarios Comunes

### Escenario 1: Continuación Limpia
- Todos los cambios del handoff están presentes
- Sin conflictos o regresiones
- Próximos pasos claros en action items
- Procede con acciones recomendadas

### Escenario 2: Codebase Divergido
- Algunos cambios faltantes o modificados
- Nuevo código relacionado agregado desde handoff
- Necesidad de reconciliar diferencias
- Adapta plan basándote en estado actual

### Escenario 3: Trabajo de Handoff Incompleto
- Tareas marcadas como "in_progress" en handoff
- Necesidad de completar trabajo inacabado primero
- Puede necesitar re-entender implementaciones parciales
- Enfócate en completar antes de nuevo trabajo

### Escenario 4: Handoff Obsoleto
- Ha pasado tiempo significativo
- Ha ocurrido refactoring mayor
- El enfoque original puede no aplicar más
- Necesidad de re-evaluar estrategia

## Flujo de Interacción de Ejemplo

```
User: /resume_handoff specification/feature/handoffs/handoff-0.md
Assistant: Let me read and analyze that handoff document...

[Reads handoff completely]
[Spawns research tasks]
[Waits for completion]
[Reads identified files]

I've analyzed the handoff from [date]. Here's the current situation...

[Presents analysis]

Shall I proceed with implementing the webhook validation fix, or would you like to adjust the approach?

User: Yes, proceed with the webhook validation
Assistant: [Creates todo list and begins implementation]
```

## Uso

```bash
/resume_handoff thoughts/shared/handoffs/ENG-1234/2025-01-08_14-30-00_ENG-1234_feature.md
# O con número de ticket:
/resume_handoff ENG-1234
# O sin parámetros para listar:
/resume_handoff
```

## Ver También

- [/create_handoff](create_handoff.md) - Para crear documentos de handoff
- [/implement_plan](implement_plan.md) - Para continuar implementación después de reanudar
