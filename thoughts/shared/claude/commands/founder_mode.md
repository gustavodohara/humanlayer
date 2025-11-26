# /founder_mode - Crear Ticket Linear y PR para Features Experimentales

## Descripción

Para features experimentales que se implementaron sin el flujo ticketing/PR adecuado. Crea retroactivamente un ticket Linear y PR para trabajo ya completado.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Contexto

A veces implementas una feature experimental rápidamente sin seguir el flujo normal de:
1. Crear ticket Linear
2. Crear rama de feature
3. Hacer commits
4. Crear PR

Este comando limpia el proceso retroactivamente creando el ticket y PR apropiados.

## Proceso

### 1. Obtén el Commit

```bash
# Si acabas de hacer un commit, obtén su SHA
git rev-parse HEAD

# Si no hiciste commit aún, lee las instrucciones
```

Si no hiciste commit todavía:
- Lee `.claude/commands/commit.md`
- Crea commit apropiado primero
- Luego continúa con los próximos pasos

### 2. Crea Ticket Linear

```bash
# Lee instrucciones de Linear
.claude/commands/linear.md
```

**Piensa profundamente sobre lo que acabas de implementar**, luego:
- Crea ticket Linear describiendo qué hiciste
- Ponlo en estado 'in dev'
- El ticket debe tener headers ### para:
  - "problem to solve" - ¿Qué problema aborda esto?
  - "proposed solution" - ¿Cómo lo resolviste?

### 3. Fetch el Ticket para Nombre de Rama

```bash
# Fetch el ticket para obtener nombre de rama recomendado
# Los tickets tienen nombre de rama sugerido basado en título
```

### 4. Crea Rama de Feature

```bash
# Vuelve a main
git checkout main

# Crea nueva rama con nombre apropiado
git checkout -b 'BRANCHNAME'

# Cherry-pick el commit del trabajo experimental
git cherry-pick 'COMMITHASH'

# Push la rama
git push -u origin 'BRANCHNAME'
```

### 5. Crea PR

```bash
# Crea PR con gh CLI
gh pr create --fill
```

### 6. Genera Descripción de PR

```bash
# Lee instrucciones de describe_pr
.claude/commands/describe_pr.md

# Sigue las instrucciones para generar descripción comprehensiva
```

## Flujo Completo

```
1. Trabajo experimental implementado (posiblemente ya committeado)
   ↓
2. /founder_mode se invoca
   ↓
3. Obtiene/crea commit del trabajo
   ↓
4. Crea ticket Linear describiendo qué se hizo
   ↓
5. Obtiene nombre de rama recomendado del ticket
   ↓
6. Vuelve a main, crea rama de feature
   ↓
7. Cherry-picks commit a rama de feature
   ↓
8. Push rama a remote
   ↓
9. Crea PR con gh CLI
   ↓
10. Genera descripción comprehensiva de PR
```

## Notas Importantes

- **Piensa profundamente**: Al crear el ticket, realmente considera qué problema estabas resolviendo
- **Documentación apropiada**: Aunque fue experimental, documentalo apropiadamente
- **Cherry-pick vs rebase**: Cherry-pick preserva el commit original mientras te permite reorganizar el historial
- **Estado del ticket**: El ticket comienza en 'in dev' ya que el trabajo ya está hecho

## Cuándo Usar

Usa `/founder_mode` cuando:
- Implementaste algo experimentalmente sin ticket
- Quieres limpiar el proceso después del hecho
- El trabajo es lo suficientemente sustancial como para justificar PR apropiado
- Necesitas documentación apropiada para el cambio

## Cuándo NO Usar

No uses `/founder_mode` cuando:
- Puedes seguir el flujo normal (crear ticket primero)
- Los cambios son triviales
- Estás aún experimentando y no listo para committear

## Mejores Prácticas

1. **Documenta honestamente**: El ticket debe describir el problema real que resolviste
2. **Solución propuesta clara**: Explica tu enfoque en la sección de solución
3. **Descripción completa de PR**: Genera descripción apropiada incluso para trabajo experimental
4. **Tests apropiados**: Asegúrate de que el trabajo tenga tests apropiados antes de crear PR

## Ejemplo de Ticket

```markdown
## Problem to solve
Users were experiencing slow response times when loading the dashboard due to
inefficient data fetching. No loading states were shown, leading to poor UX.

## Proposed solution
Implemented optimistic loading with skeleton screens and debounced the API
calls. Added caching layer for frequently accessed data.

Files changed:
- src/components/Dashboard.tsx - Added loading states
- src/api/cache.ts - New caching layer
- src/hooks/useDebounce.ts - Debounce hook
```

## Uso

```bash
/founder_mode
```

## Ver También

- [/commit](commit.md) - Para crear commits apropiados
- [/linear](linear.md) - Para gestión de tickets Linear
- [/describe_pr](describe_pr.md) - Para generar descripción de PR
