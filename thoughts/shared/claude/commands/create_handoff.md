# /create_handoff - Crear Documento de Handoff

## Descripción

Crea un documento de handoff para transferir trabajo a otra sesión. Captura el contexto actual, learnings y próximos pasos para que otra sesión pueda continuar el trabajo sin problemas.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Cuándo Usar

Crea un handoff cuando:
- Necesitas pausar trabajo y continuarlo más tarde
- Quieres transferir trabajo a otro desarrollador o sesión
- Has alcanzado límites de contexto y necesitas comenzar sesión fresca
- Has aprendido lecciones importantes que la próxima sesión debe saber
- El trabajo está parcialmente completado y necesita documentación

## Estructura del Documento de Handoff

Los handoffs se guardan en: `thoughts/shared/handoffs/ENG-XXXX/YYYY-MM-DD_HH-MM-SS_ENG-XXXX_description.md`

### Frontmatter
```yaml
---
date: [Fecha y hora actual ISO]
researcher: [Tu nombre]
git_commit: [Commit hash actual]
branch: [Rama actual]
repository: humanlayer
ticket: ENG-XXXX
status: in_progress
---
```

### Secciones del Documento

```markdown
# Handoff: [Feature/Task Name]

## Context
[Explicación de alto nivel de qué se está trabajando y por qué]

## Current Status
[Dónde está el trabajo ahora - qué está hecho, qué no]

## Task(s)
- [ ] Tarea 1 descripción
- [x] Tarea 2 completada
- [ ] Tarea 3 pendiente

## Recent Changes
[Cambios específicos hechos en esta sesión]
- Modified `file.ts:123` - [qué se cambió y por qué]
- Added `newfile.ts` - [propósito del archivo]
- Updated tests in `test.ts:45-67`

## Learnings
[Insights críticos descubiertos durante el trabajo]
- `important/file.ts:89` - Descubrimos que X se comporta de manera Y
- El patrón usado en `example.ts:123` debe seguirse para Z
- Evitar enfoque A porque B

## Artifacts
[Referencias a documentos importantes]
- Feature spec: `thoughts/shared/research/feature.md`
- Implementation plan: `thoughts/shared/plans/plan.md`
- Related tickets: ENG-1234, ENG-5678

## Next Steps
[Acciones específicas que la próxima sesión debe tomar]
1. Completar implementación de X en `file.ts`
2. Agregar tests para escenario Y
3. Actualizar documentación en Z
4. Verificar integración con componente A

## Blockers/Questions
[Cualquier blocker o pregunta sin resolver]
- Esperando decisión sobre X
- No claro si debemos usar enfoque A o B para Y
- Necesita investigación adicional sobre Z

## Code References
- `path/to/file.ts:123-145` - Main implementation
- `another/file.ts:67` - Helper function
- `test/file.spec.ts:89` - Test coverage

## Git State
[Estado de git al momento del handoff]
- Branch: [nombre de rama]
- Commits ahead of main: [número]
- Uncommitted changes: [descripción]
```

## Proceso de Creación

### 1. Reúne Contexto de la Sesión

- Revisa el historial de conversación
- Identifica qué tareas estaban siendo trabajadas
- Nota qué está completado vs incompleto
- Captura cualquier decisión o learning importante

### 2. Verifica Estado del Código

```bash
# Estado actual de git
git status
git diff
git log --oneline -10

# Rama actual y relación con main
git rev-parse --abbrev-ref HEAD
git rev-list --count HEAD ^main
```

### 3. Identifica Artifacts Clave

- Encuentra planes de implementación referenciados
- Localiza documentos de investigación relacionados
- Nota números de tickets de Linear
- Identifica archivos de código críticos

### 4. Captura Learnings

Los learnings son críticos - estos son los insights que ahorran tiempo:
- Patrones de código que deben seguirse
- Enfoques que no funcionaron y por qué
- Descubrimientos sobre cómo funciona el sistema
- Gotchas o casos extremos encontrados
- Decisiones hechas y su razonamiento

### 5. Define Próximos Pasos Claros

Los próximos pasos deben ser:
- Específicos y accionables
- Ordenados por prioridad/dependencia
- Referenciados a archivos y líneas específicas cuando sea relevante
- Lo suficientemente claros para que alguien sin contexto pueda continuar

### 6. Documenta Blockers

Si hay blockers o preguntas sin resolver:
- Declara qué está bloqueando el progreso
- Proporciona contexto sobre por qué es un blocker
- Sugiere posibles rutas hacia adelante
- Nota a quién preguntar o dónde buscar respuestas

### 7. Guarda y Comunica

```bash
# Guarda el handoff
# Location: thoughts/shared/handoffs/ENG-XXXX/YYYY-MM-DD_HH-MM-SS_ENG-XXXX_description.md

# Sincroniza thoughts
humanlayer thoughts sync

# Informa al usuario
```

## Mejores Prácticas

### 1. Sé Específico
- No "arreglar el bug" sino "arreglar el bug de rate limiting en `handler.ts:123` cuando el usuario excede quota"
- Incluye referencias file:line para todo

### 2. Captura el "Por Qué"
- No solo qué hiciste, sino por qué
- No solo qué queda, sino por qué importa
- Documenta decisiones y su razonamiento

### 3. Haz que sea Accionable
- La próxima sesión debe poder comenzar inmediatamente
- Proporciona todos los enlaces y referencias necesarias
- Explica cualquier contexto no obvio

### 4. Sé Honesto
- Si algo está medio hecho, dilo
- Si tomaste atajos, docum éntalos
- Si no estás seguro de algo, anótalo

### 5. Piensa en tu "Yo Futuro"
- ¿Qué desearías saber si retomaras esto en una semana?
- ¿Qué contexto se perdería de la conversación?
- ¿Qué te tomaría tiempo redescubrir?

## Salida del Comando

```
Created handoff document at:
thoughts/shared/handoffs/ENG-1234/2025-01-08_14-30-00_ENG-1234_feature-name.md

The handoff includes:
- Current status and recent changes
- 3 completed tasks, 2 remaining
- 5 key learnings about the implementation
- Clear next steps for continuation
- Git state and code references

Synced to thoughts repository.

To resume this work later, use:
/resume_handoff thoughts/shared/handoffs/ENG-1234/2025-01-08_14-30-00_ENG-1234_feature-name.md
```

## Uso

```bash
/create_handoff
# O con ticket específico:
/create_handoff ENG-1234
```

## Ver También

- [/resume_handoff](resume_handoff.md) - Para retomar trabajo desde un handoff
- [/implement_plan](implement_plan.md) - Para continuar implementación después de retomar
