# /linear - Gestión de Tickets Linear

## Descripción

Gestiona tickets de Linear, incluyendo crear tickets desde documentos thoughts, actualizar tickets existentes y seguir los patrones de flujo de trabajo específicos del equipo.

## Flujo de Trabajo del Equipo

El equipo sigue un flujo de trabajo específico para asegurar alineación antes de la implementación de código:

1. **Triage** → Todos los tickets nuevos comienzan aquí para revisión inicial
2. **Spec Needed** → Se necesita más detalle - problema a resolver y esquema de solución necesarios
3. **Research Needed** → El ticket requiere investigación antes de poder escribir el plan
4. **Research in Progress** → Investigación/investigación activa en curso
5. **Research in Review** → Hallazgos de investigación bajo revisión (paso opcional)
6. **Ready for Plan** → Investigación completa, ticket necesita plan de implementación
7. **Plan in Progress** → Escribiendo activamente el plan de implementación
8. **Plan in Review** → Plan escrito y bajo discusión
9. **Ready for Dev** → Plan aprobado, listo para implementación
10. **In Dev** → Desarrollo activo
11. **Code Review** → PR enviado
12. **Done** → Completado

**Principio clave**: La revisión y alineación ocurren en la etapa del plan (no en la etapa del PR) para moverse más rápido y evitar rehacer trabajo.

## Convenciones Importantes

### Mapeo de URLs para Documentos Thoughts

Cuando se referencian documentos thoughts, siempre proporcionar enlaces de GitHub usando el parámetro `links`:
- `thoughts/shared/...` → `https://github.com/humanlayer/thoughts/blob/main/repos/humanlayer/shared/...`
- `thoughts/allison/...` → `https://github.com/humanlayer/thoughts/blob/main/repos/humanlayer/allison/...`
- `thoughts/global/...` → `https://github.com/humanlayer/thoughts/blob/main/global/...`

### Valores por Defecto

- **Status**: Siempre crear tickets nuevos en estado "Triage"
- **Project**: Por defecto "M U L T I C L A U D E" (ID: f11c8d63-9120-4393-bfae-553da0b04fd8) a menos que se indique lo contrario
- **Priority**: Por defecto Medium (3) para la mayoría de tareas
  - Urgent (1): Blockers críticos, issues de seguridad
  - High (2): Features importantes con deadlines, bugs mayores
  - Medium (3): Tareas de implementación estándar (por defecto)
  - Low (4): Nice-to-haves, mejoras menores
- **Links**: Usar el parámetro `links` para adjuntar URLs (no solo links markdown en descripción)

### Asignación Automática de Labels

Aplicar labels automáticamente basándose en el contenido del ticket:
- **hld**: Para tickets sobre el directorio `hld/` (el daemon)
- **wui**: Para tickets sobre `humanlayer-wui/`
- **meta**: Para tickets sobre comandos `hlyr`, herramienta thoughts, o directorio `thoughts/`

Nota: meta es mutuamente exclusivo con hld/wui. Los tickets pueden tener tanto hld como wui, pero no meta con ninguno de los dos.

## Acciones Principales

### 1. Crear Tickets desde Thoughts

#### Proceso:
1. Localizar y leer el documento thoughts
2. Analizar el contenido del documento (extraer problema central, detalles de implementación)
3. Verificar contexto relacionado si se menciona
4. Obtener contexto del workspace de Linear
5. Presentar borrador del ticket al usuario
6. Refinamiento interactivo
7. Crear el ticket Linear
8. Acciones post-creación (actualizar documento thoughts con referencia al ticket)

#### Ejemplo de Ticket:

```markdown
## Draft Linear Ticket

**Title**: Fix resumed sessions to inherit all configuration from parent

**Description**:

## Problem to solve
Currently, resumed sessions only inherit Model and WorkingDir from parent sessions,
causing all other configuration to be lost. Users must re-specify permissions and
settings when resuming.

## Solution
Store all session configuration in the database and automatically inherit it when
resuming sessions, with support for explicit overrides.

**References**:
- Source: `thoughts/shared/research/session-config.md` ([View on GitHub](url))
```

### 2. Agregar Comentarios y Enlaces

**Estructura de comentario recomendada** (~10 líneas):
```markdown
Implemented retry logic in webhook handler to address rate limit issues.

Key insight: The 429 responses were clustered during batch operations,
so exponential backoff alone wasn't sufficient - added request queuing.

Files updated:
- `hld/webhooks/handler.go` ([GitHub](link))
- `thoughts/shared/rate_limit_analysis.md` ([GitHub](link))
```

**Para comentarios con enlaces:**
1. Actualizar el issue con el enlace usando el parámetro `links`
2. Crear el comentario mencionando el enlace

**Para solo enlaces:**
1. Actualizar el issue con el enlace
2. Agregar un comentario breve por posteridad

### 3. Buscar Tickets

Criterios de búsqueda:
- Texto de consulta
- Filtros de Team/Project
- Filtros de estado
- Rangos de fechas (createdAt, updatedAt)

### 4. Actualizar Estado de Tickets

Seguir las progresiones de estado recomendadas:
- Triage → Spec Needed (falta detalle)
- Spec Needed → Research Needed (una vez esbozado problema/solución)
- Research Needed → Research in Progress
- Research in Review → Ready for Plan
- Ready for Plan → Plan in Progress
- Plan in Progress → Plan in Review
- Plan in Review → Ready for Dev
- Ready for Dev → In Dev

## Guías de Calidad de Comentarios

Cuando crees comentarios, enfócate en extraer la **información más valiosa** para un lector humano:

- **Insights clave sobre resúmenes**: ¿Cuál es el momento "aha" o comprensión crítica?
- **Decisiones y trade-offs**: Qué enfoque se eligió y qué permite/previene
- **Blockers resueltos**: Qué estaba previniendo progreso y cómo se abordó
- **Cambios de estado**: Qué es diferente ahora y qué significa para los próximos pasos
- **Sorpresas o descubrimientos**: Hallazgos inesperados que afectan el trabajo

Evita:
- Listas mecánicas de cambios sin contexto
- Reafirmar lo que es obvio desde los diffs de código
- Resúmenes genéricos que no agregan valor

## IDs Comúnmente Usados

### Engineering Team
- **Team ID**: `6b3b2115-efd4-4b83-8463-8160842d2c84`

### Label IDs
- **bug**: `ff23dde3-199b-421e-904c-4b9f9b3d452c`
- **hld**: `d28453c8-e53e-4a06-bea9-b5bbfad5f88a`
- **meta**: `7a5abaae-f343-4f52-98b0-7987048b0cfa`
- **wui**: `996deb94-ba0f-4375-8b01-913e81477c4b`

### Workflow State IDs
- **Triage**: `77da144d-fe13-4c3a-a53a-cfebd06c0cbe`
- **spec needed**: `274beb99-bff8-4d7b-85cf-04d18affbc82`
- **research needed**: `d0b89672-8189-45d6-b705-50afd6c94a91`
- **research in progress**: `c41c5a23-ce25-471f-b70a-eff1dca60ffd`
- **research in review**: `1a9363a7-3fae-42ee-a6c8-1fc714656f09`
- **ready for plan**: `995011dd-3e36-46e5-b776-5a4628d06cc8`
- **plan in progress**: `a52b4793-d1b6-4e5d-be79-b2254185eed0`
- **plan in review**: `15f56065-41ea-4d9a-ab8c-ec8e1a811a7a`
- **ready for dev**: `c25bae2f-856a-4718-aaa8-b469b7822f58`
- **in dev**: `6be18699-18d7-496e-a7c9-37d2ddefe612`
- **Done**: `8159f431-fbc7-495f-a861-1ba12040f672`

### Linear User IDs
- allison: b157f9e4-8faf-4e7e-a598-dae6dec8a584
- dex: 16765c85-2286-4c0f-ab49-0d4d79222ef5
- sundeep: 0062104d-9351-44f5-b64c-d0b59acb516b

## Notas Importantes

- **Problema a resolver requerido**: Todos los tickets deben incluir un claro "problema a resolver" - si el usuario solo proporciona detalles de implementación, DEBES preguntar: "Para escribir un buen ticket, por favor explica el problema que estás intentando resolver desde una perspectiva de usuario"
- Etiquetar usuarios en descripciones y comentarios usando formato `@[name](ID)`, ej: `@[dex](16765c85-2286-4c0f-ab49-0d4d79222ef5)`
- Mantener tickets concisos pero completos - apunta a contenido escaneable
- Enfócate en el "qué" y "por qué", incluye "cómo" solo si está bien definido
- Siempre preservar enlaces al material fuente usando el parámetro `links`
- No crear tickets desde lluvia de ideas en etapa temprana a menos que se solicite

## Uso

```bash
/linear
# O crear ticket desde thoughts:
/linear create ticket from thoughts/shared/research/feature.md
# O agregar comentario:
/linear comment on ENG-1234
# O actualizar estado:
/linear move ENG-1234 to "ready for dev"
```

## Ver También

- [/ralph_research](ralph_research.md) - Investiga ticket de mayor prioridad
- [/ralph_plan](ralph_plan.md) - Crea plan para ticket de mayor prioridad
- [/ralph_impl](ralph_impl.md) - Implementa ticket pequeño
