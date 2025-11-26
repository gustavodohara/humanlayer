# /implement_plan - Implementar Plan Técnico

## Descripción

Implementa planes técnicos aprobados desde `thoughts/shared/plans/`. Estos planes contienen fases con cambios específicos y criterios de éxito.

## Modelo

Por defecto (no especificado, usa el modelo estándar de la sesión)

## Comenzando

Cuando se proporciona una ruta de plan:
- Lee el plan completamente y verifica checkmarks existentes (- [x])
- Lee el ticket original y todos los archivos mencionados en el plan
- **Lee archivos completamente** - nunca uses parámetros limit/offset, necesitas contexto completo
- Piensa profundamente sobre cómo las piezas encajan
- Crea una lista de tareas para seguir tu progreso
- Comienza a implementar si entiendes lo que necesita hacerse

Si no se proporciona ruta de plan, pide una.

## Filosofía de Implementación

Los planes están cuidadosamente diseñados, pero la realidad puede ser desordenada. Tu trabajo es:
- Seguir la intención del plan mientras te adaptas a lo que encuentras
- Implementar cada fase completamente antes de pasar a la siguiente
- Verificar que tu trabajo tiene sentido en el contexto más amplio del codebase
- Actualizar checkboxes en el plan a medida que completes secciones

Cuando las cosas no coinciden con el plan exactamente, piensa sobre por qué y comunícalo claramente. El plan es tu guía, pero tu juicio también importa.

## Manejo de Desajustes

Si encuentras un desajuste:
- DETENTE y piensa profundamente sobre por qué el plan no se puede seguir
- Presenta el problema claramente:
  ```
  Issue in Phase [N]:
  Expected: [lo que dice el plan]
  Found: [situación actual]
  Why this matters: [explicación]

  How should I proceed?
  ```

## Enfoque de Verificación

Después de implementar una fase:
- Ejecuta las verificaciones de criterios de éxito (usualmente `make check test` cubre todo)
- Corrige cualquier issue antes de proceder
- Actualiza tu progreso tanto en el plan como en tus todos
- **Marca items completados en el archivo del plan** usando Edit
- **Pausa para verificación humana**: Después de completar toda la verificación automatizada para una fase, pausa e informa al humano que la fase está lista para pruebas manuales

### Formato de Pausa para Verificación Manual

```
Phase [N] Complete - Ready for Manual Verification

Automated verification passed:
- [Lista de verificaciones automatizadas que pasaron]

Please perform the manual verification steps listed in the plan:
- [Lista de items de verificación manual del plan]

Let me know when manual testing is complete so I can proceed to Phase [N+1].
```

**Nota Importante**: Si se te indica ejecutar múltiples fases consecutivamente, omite la pausa hasta la última fase. De lo contrario, asume que solo estás haciendo una fase.

**No marques items en los pasos de pruebas manuales hasta que sean confirmados por el usuario.**

## Si Te Atascas

Cuando algo no está funcionando como se esperaba:
- Primero, asegúrate de haber leído y entendido todo el código relevante
- Considera si el codebase ha evolucionado desde que se escribió el plan
- Presenta el desajuste claramente y pide orientación

Usa sub-tareas con moderación - principalmente para debugging dirigido o explorar territorio desconocido.

## Reanudando Trabajo

Si el plan tiene checkmarks existentes:
- Confía en que el trabajo completado está hecho
- Continúa desde el primer item sin marcar
- Verifica trabajo previo solo si algo parece mal

## Principio Rector

Recuerda: Estás implementando una solución, no solo marcando checkboxes. Mantén el objetivo final en mente y mantén momentum hacia adelante.

## Uso

```bash
/implement_plan thoughts/shared/plans/2025-01-08-ENG-1234-feature.md
```

## Ver También

- [/create_plan](create_plan.md) - Para crear el plan a implementar
- [/validate_plan](validate_plan.md) - Para validar la implementación
- [/commit](commit.md) - Para crear commits después de la implementación
