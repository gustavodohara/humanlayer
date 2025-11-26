# /ralph_plan - Crear Plan para Ticket de Mayor Prioridad

## Descripción

Crea automáticamente un plan de implementación para el ticket Linear de mayor prioridad que está en estado "ready for spec". Fetches el ticket, revisa investigación existente y genera plan detallado.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Parte I - Si se Menciona un Ticket

Si se proporciona número de ticket (ej: `ENG-1234`):
- Usa `linear` CLI para fetch el ticket: `./thoughts/shared/tickets/ENG-XXXX.md`
- Lee el ticket y todos los comentarios para aprender sobre implementaciones pasadas y investigación

## Parte I - Si NO se Menciona Ticket

Si no se proporciona número de ticket:
1. Lee `.claude/commands/linear.md`
2. Fetch los 10 tickets de mayor prioridad en estado "ready for spec" usando MCP tools, notando todos los items en la sección `links`
3. Selecciona el issue de mayor prioridad SMALL o XS de la lista
   - Si no existen issues SMALL o XS, **SALIR INMEDIATAMENTE** e informar al usuario
4. Usa `linear` CLI para fetch el ticket: `./thoughts/shared/tickets/ENG-XXXX.md`
5. Lee el ticket y todos los comentarios

## Parte II - Próximos Pasos

**Piensa profundamente.**

1. Mueve el item a "plan in progress" usando MCP tools
2. Lee `.claude/commands/create_plan.md`
3. Determina si el item tiene un documento de plan de implementación linkedado basándose en la sección `links`
4. Si el plan existe, estás hecho, responde con link al ticket
5. Si la investigación es insuficiente o tiene preguntas sin responder, crea un nuevo documento de plan siguiendo las instrucciones en `.claude/commands/create_plan.md`

**Piensa profundamente.**

## Parte III - Cuando Terminas

1. Sincroniza y adjunta el documento al ticket:
   ```bash
   humanlayer thoughts sync
   ```
2. Adjunta el documento al ticket usando MCP tools
3. Crea comentario conciso con link al documento
4. Mueve el item a "plan in review" usando MCP tools

**Piensa profundamente, usa TodoWrite para rastrear tus tareas.**

Cuando obtengas datos de Linear, obtén los 10 items principales por prioridad pero solo trabaja en UNO - específicamente el issue de mayor prioridad de tamaño SMALL o XS.

## Salida del Comando

Al completar, imprime mensaje para el usuario:

```
✅ Completed implementation plan for ENG-XXXX: [ticket title]

Approach: [selected approach description]

The plan has been:
- Created at thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-description.md
- Synced to thoughts repository
- Attached to the Linear ticket
- Ticket moved to "plan in review" status

Implementation phases:
- Phase 1: [phase 1 description]
- Phase 2: [phase 2 description]
- Phase 3: [phase 3 description if applicable]

View the ticket: https://linear.app/humanlayer/issue/ENG-XXXX/[ticket-slug]
```

## Notas Importantes

- **Solo tickets pequeños**: Solo trabaja en tickets SMALL o XS
- **Pensamiento profundo**: Usa TodoWrite y piensa profundamente en cada paso
- **Investigación requerida**: Si la investigación es insuficiente, crea nueva investigación primero
- **Plan detallado**: Sigue todos los estándares de `/create_plan`
- **Un ticket a la vez**: Solo trabaja en el ticket de mayor prioridad

## Uso

```bash
/ralph_plan
# O con ticket específico:
/ralph_plan ENG-1234
```

## Ver También

- [/ralph_research](ralph_research.md) - Para investigar antes de planear
- [/ralph_impl](ralph_impl.md) - Para implementar después de planear
- [/create_plan](create_plan.md) - El comando subyacente usado para planificación
- [/linear](linear.md) - Para gestión de tickets Linear
