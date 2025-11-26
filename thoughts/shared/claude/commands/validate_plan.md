# /validate_plan - Validar Implementación Contra Plan

## Descripción

Valida que un plan de implementación fue correctamente ejecutado, verificando todos los criterios de éxito e identificando cualquier desviación o issue.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Configuración Inicial

Cuando se invoca:

1. **Determina contexto** - ¿Estás en una conversación existente o comenzando de nuevo?
   - Si existente: Revisa qué se implementó en esta sesión
   - Si nuevo: Necesitas descubrir qué se hizo a través de git y análisis del codebase

2. **Localiza el plan**:
   - Si se proporciona ruta del plan, úsala
   - De lo contrario, busca en commits recientes referencias al plan o pregunta al usuario

3. **Recopila evidencia de implementación**:
   ```bash
   # Verifica commits recientes
   git log --oneline -n 20
   git diff HEAD~N..HEAD  # Donde N cubre commits de implementación

   # Ejecuta verificaciones comprehensivas
   cd $(git rev-parse --show-toplevel) && make check test
   ```

## Proceso de Validación

### Paso 1: Descubrimiento de Contexto

Si comenzando de nuevo o necesitas más contexto:

1. **Lee el plan de implementación** completamente

2. **Identifica qué debería haber cambiado**:
   - Lista todos los archivos que deberían modificarse
   - Nota todos los criterios de éxito (automatizados y manuales)
   - Identifica funcionalidad clave a verificar

3. **Genera tareas de investigación paralelas** para descubrir implementación:
   ```
   Task 1 - Verify database changes:
   Research if migration [N] was added and schema changes match plan.
   Check: migration files, schema version, table structure
   Return: What was implemented vs what plan specified

   Task 2 - Verify code changes:
   Find all modified files related to [feature].
   Compare actual changes to plan specifications.
   Return: File-by-file comparison of planned vs actual

   Task 3 - Verify test coverage:
   Check if tests were added/modified as specified.
   Run test commands and capture results.
   Return: Test status and any missing coverage
   ```

### Paso 2: Validación Sistemática

Para cada fase en el plan:

1. **Verifica estado de completitud**:
   - Busca checkmarks en el plan (- [x])
   - Verifica que el código real coincida con completitud reclamada

2. **Ejecuta verificación automatizada**:
   - Ejecuta cada comando de "Automated Verification"
   - Documenta estado pass/fail
   - Si hay fallas, investiga causa raíz

3. **Evalúa criterios manuales**:
   - Lista qué necesita pruebas manuales
   - Proporciona pasos claros para verificación del usuario

4. **Piensa profundamente sobre casos extremos**:
   - ¿Se manejaron condiciones de error?
   - ¿Faltan validaciones?
   - ¿Podría la implementación romper funcionalidad existente?

### Paso 3: Genera Reporte de Validación

Crea resumen de validación comprehensivo:

```markdown
## Validation Report: [Plan Name]

### Implementation Status
✓ Phase 1: [Name] - Fully implemented
✓ Phase 2: [Name] - Fully implemented
⚠️ Phase 3: [Name] - Partially implemented (see issues)

### Automated Verification Results
✓ Build passes: `make build`
✓ Tests pass: `make test`
✗ Linting issues: `make lint` (3 warnings)

### Code Review Findings

#### Matches Plan:
- Database migration correctly adds [table]
- API endpoints implement specified methods
- Error handling follows plan

#### Deviations from Plan:
- Used different variable names in [file:line]
- Added extra validation in [file:line] (improvement)

#### Potential Issues:
- Missing index on foreign key could impact performance
- No rollback handling in migration

### Manual Testing Required:
1. UI functionality:
   - [ ] Verify [feature] appears correctly
   - [ ] Test error states with invalid input

2. Integration:
   - [ ] Confirm works with existing [component]
   - [ ] Check performance with large datasets

### Recommendations:
- Address linting warnings before merge
- Consider adding integration test for [scenario]
- Document new API endpoints
```

## Trabajando con Contexto Existente

Si fuiste parte de la implementación:
- Revisa el historial de conversación
- Verifica tu lista de todos para qué se completó
- Enfoca validación en trabajo hecho en esta sesión
- Sé honesto sobre cualquier atajo o item incompleto

## Guías Importantes

1. **Ser exhaustivo pero práctico** - Enfócate en lo que importa
2. **Ejecutar todas las verificaciones automatizadas** - No omitas comandos de verificación
3. **Documentar todo** - Tanto éxitos como issues
4. **Pensar críticamente** - Cuestiona si la implementación realmente resuelve el problema
5. **Considerar mantenimiento** - ¿Será esto mantenible a largo plazo?

## Checklist de Validación

Siempre verifica:
- [ ] Todas las fases marcadas como completas están realmente hechas
- [ ] Tests automatizados pasan
- [ ] El código sigue patrones existentes
- [ ] No se introdujeron regresiones
- [ ] El manejo de errores es robusto
- [ ] Documentación actualizada si es necesario
- [ ] Pasos de pruebas manuales son claros

## Relación con Otros Comandos

Flujo de trabajo recomendado:
1. `/implement_plan` - Ejecuta la implementación
2. `/commit` - Crea commits atómicos para cambios
3. `/validate_plan` - Verifica corrección de implementación
4. `/describe_pr` - Genera descripción de PR

La validación funciona mejor después de que se hacen commits, ya que puede analizar el historial de git para entender qué se implementó.

## Uso

```bash
/validate_plan thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-feature.md
# O si el plan está en contexto:
/validate_plan
```

## Ver También

- [/implement_plan](implement_plan.md) - Para ejecutar el plan
- [/iterate_plan](iterate_plan.md) - Para ajustar el plan si se encuentran issues
- [/describe_pr](describe_pr.md) - Para documentar la implementación validada

**Recuerda**: Buena validación atrapa issues antes de que lleguen a producción. Sé constructivo pero exhaustivo al identificar brechas o mejoras.
