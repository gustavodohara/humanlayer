# /ralph_impl - Implementar Ticket Pequeño de Mayor Prioridad

## Descripción

Implementa automáticamente el ticket Linear de mayor prioridad que está en estado "ready for dev". Configura worktree, implementa el plan, crea commits y PR.

## Modelo

`sonnet` - Modelo eficiente para implementación

## Parte I - Si se Menciona un Ticket

Si se proporciona número de ticket (ej: `ENG-1234`):
- Usa `linear` CLI para fetch el ticket: `./thoughts/shared/tickets/ENG-XXXX.md`
- Lee el ticket y todos los comentarios para entender el plan de implementación y concerns

## Parte I - Si NO se Menciona Ticket

Si no se proporciona número de ticket:
1. Lee `.claude/commands/linear.md`
2. Fetch los 10 tickets de mayor prioridad en estado "ready for dev" usando MCP tools, notando todos los items en la sección `links`
3. Selecciona el issue de mayor prioridad SMALL o XS de la lista
   - Si no existen issues SMALL o XS, **SALIR INMEDIATAMENTE** e informar al usuario
4. Usa `linear` CLI para fetch el ticket: `./thoughts/shared/tickets/ENG-XXXX.md`
5. Lee el ticket y todos los comentarios

## Parte II - Próximos Pasos

**Piensa profundamente.**

1. Mueve el item a "in dev" usando MCP tools
2. Identifica el documento de plan de implementación linkedado desde la sección `links`
3. Si no existe plan, mueve el ticket de vuelta a "ready for spec" y SALIR con explicación

**Piensa profundamente sobre la implementación.**

## Parte III - Setup de Worktree e Implementación

1. **Configura worktree para implementación**:
   ```bash
   # Lee el script
   ./hack/create_worktree.sh

   # Crea nuevo worktree con nombre de rama Linear
   ./hack/create_worktree.sh ENG-XXXX BRANCH_NAME
   ```

2. **Lanza sesión de implementación**:
   ```bash
   humanlayer-nightly launch \
     --model opus \
     --dangerously-skip-permissions \
     --dangerously-skip-permissions-timeout 15m \
     --title "implement ENG-XXXX" \
     -w ~/wt/humanlayer/ENG-XXXX \
     "/implement_plan and when you are done implementing and all tests pass, read ./claude/commands/commit.md and create a commit, then read ./claude/commands/describe_pr.md and create a PR, then add a comment to the Linear ticket with the PR link"
   ```

Esta sesión:
- Implementa el plan fase por fase
- Crea commits cuando la implementación esté completa
- Genera descripción de PR
- Actualiza el ticket Linear con link al PR

**Piensa profundamente, usa TodoWrite para rastrear tus tareas.**

Cuando obtengas datos de Linear, obtén los 10 items principales por prioridad pero solo trabaja en UNO - específicamente el issue de mayor prioridad de tamaño SMALL o XS.

## Notas Importantes

- **Solo tickets pequeños**: Solo trabaja en tickets SMALL o XS
- **Plan requerido**: El ticket debe tener plan de implementación linkedado
- **Worktree aislado**: Cada implementación obtiene su propio worktree
- **Sesión automatizada**: La sesión lanzada maneja implementación completa
- **Flujo completo**: Implementación → Commits → PR → Update ticket
- **Un ticket a la vez**: Solo trabaja en el ticket de mayor prioridad

## Flujo Completo

```
1. ralph_impl se invoca
   ↓
2. Encuentra ticket de mayor prioridad en "ready for dev"
   ↓
3. Mueve ticket a "in dev"
   ↓
4. Configura worktree con rama de feature
   ↓
5. Lanza nueva sesión Claude Code en worktree
   ↓
6. Sesión implementa plan
   ↓
7. Sesión crea commits
   ↓
8. Sesión crea PR
   ↓
9. Sesión actualiza ticket Linear con link al PR
   ↓
10. Ticket se mueve automáticamente a "code review"
```

## Uso

```bash
/ralph_impl
# O con ticket específico:
/ralph_impl ENG-1234
```

## Ver También

- [/ralph_plan](ralph_plan.md) - Para crear plan antes de implementar
- [/implement_plan](implement_plan.md) - El comando usado en la sesión lanzada
- [/commit](commit.md) - Para crear commits
- [/describe_pr](describe_pr.md) - Para generar descripción de PR
- [/linear](linear.md) - Para gestión de tickets Linear
