# /oneshot_plan - Ejecutar Ralph Plan e Implementación

## Descripción

Comando de conveniencia que ejecuta el flujo completo desde planificación hasta implementación. Combina `/ralph_plan` y `/ralph_impl` en un solo comando.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Proceso

### 1. Crea Plan de Implementación

Usa SlashCommand() para llamar `/ralph_plan` con el número de ticket dado:
```bash
/ralph_plan ENG-XXXX
```

Esto:
- Fetches el ticket de Linear
- Revisa investigación existente
- Crea plan de implementación detallado
- Sincroniza y adjunta a ticket
- Mueve ticket a "plan in review"

### 2. Implementa el Plan

Usa SlashCommand() para llamar `/ralph_impl` con el número de ticket dado:
```bash
/ralph_impl ENG-XXXX
```

Esto:
- Mueve ticket a "in dev"
- Configura worktree
- Lanza sesión de implementación
- Implementa el plan
- Crea commits
- Genera PR
- Actualiza ticket con link al PR

## Flujo Completo

```
1. /oneshot_plan ENG-XXXX se invoca
   ↓
2. /ralph_plan ENG-XXXX ejecuta
   ↓
3. Plan de implementación se crea
   ↓
4. /ralph_impl ENG-XXXX ejecuta
   ↓
5. Worktree se configura
   ↓
6. Nueva sesión implementa plan
   ↓
7. Commits y PR se crean
   ↓
8. Ticket se actualiza con PR link
```

## Ventajas

- **Flujo completo automatizado**: Desde planificación hasta PR
- **Sin pasos manuales**: Todo se ejecuta automáticamente
- **Worktrees aislados**: Cada implementación en su propio worktree
- **Sesión dedicada**: Implementación en sesión fresca con contexto limpio

## Cuando Usar

Usa `/oneshot_plan` cuando:
- Quieres el flujo completo de plan → implementación → PR
- El ticket ya tiene investigación completa
- Es un ticket pequeño (SMALL o XS)
- Confías en el proceso automatizado

## Cuando NO Usar

No uses `/oneshot_plan` cuando:
- El ticket necesita investigación primero (usa `/oneshot` en su lugar)
- Quieres revisar el plan antes de implementar
- Es un ticket grande o complejo
- Prefieres ejecutar pasos manualmente con más control

## Relación con Otros Comandos

- `/oneshot` = `/ralph_research` + lanza `/oneshot_plan`
- `/oneshot_plan` = `/ralph_plan` + `/ralph_impl`

Por lo tanto:
- `/oneshot` = Investigación → Planificación → Implementación
- `/oneshot_plan` = Planificación → Implementación

## Uso

```bash
/oneshot_plan ENG-1234
```

## Ver También

- [/oneshot](oneshot.md) - Incluye investigación antes de planificación
- [/ralph_plan](ralph_plan.md) - Solo planificación
- [/ralph_impl](ralph_impl.md) - Solo implementación
