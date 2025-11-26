# /ralph_research - Investigar Ticket de Mayor Prioridad

## Descripción

Investiga automáticamente el ticket Linear de mayor prioridad que está en estado "research needed". Fetches el ticket, conduce investigación y genera documento de investigación comprehensivo.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Proceso

### 1. Si se Menciona un Ticket

Si se proporciona número de ticket (ej: `ENG-1234`):
- Fetch el ticket usando `linear` CLI: `./thoughts/shared/tickets/ENG-XXXX.md`
- Lee el ticket y todos los comentarios
- Procede con la investigación

### 2. Si NO se Menciona Ticket

Si no se proporciona número de ticket:
- Lee `.claude/commands/linear.md` para entender el flujo de trabajo
- Fetch los 10 tickets de mayor prioridad en estado "research needed" usando MCP tools
- Selecciona el ticket de mayor prioridad de la lista
- Usa `linear` CLI para fetch el ticket: `./thoughts/shared/tickets/ENG-XXXX.md`
- Lee el ticket y todos los comentarios

### 3. Conduce la Investigación

- Mueve el item a "research in progress" usando MCP tools
- Lee `.claude/commands/research_codebase.md` para instrucciones de investigación
- Conduce investigación exhaustiva del codebase para responder las preguntas del ticket
- Genera documento de investigación siguiendo la estructura estándar

### 4. Sincroniza y Actualiza Linear

```bash
# Sincroniza thoughts
humanlayer thoughts sync

# Adjunta documento de investigación al ticket usando MCP tools
# Crea comentario conciso con link al documento
```

### 5. Actualiza Estado del Ticket

- Mueve el item a "research in review" usando MCP tools
- Informa al usuario sobre la investigación completada

## Salida del Comando

Al completar, imprime mensaje para el usuario:

```
✅ Completed research for ENG-XXXX: [ticket title]

Research findings: [brief summary]

The research has been:
- Created at thoughts/shared/research/YYYY-MM-DD-ENG-XXXX-description.md
- Synced to thoughts repository
- Attached to the Linear ticket
- Ticket moved to "research in review" status

View the ticket: https://linear.app/humanlayer/issue/ENG-XXXX/[ticket-slug]
```

## Notas Importantes

- **Investigación exhaustiva**: Usa todos los agentes necesarios para responder las preguntas
- **Solo documentación**: La investigación describe lo que existe, no propone cambios
- **Contexto histórico**: Revisa thoughts/ para contexto histórico relevante
- **Referencias específicas**: Incluye rutas de archivo y números de línea
- **Pensamiento profundo**: Tómate tiempo para pensar ultra-profundo sobre la tarea

## Uso

```bash
/ralph_research
# O con ticket específico:
/ralph_research ENG-1234
```

## Ver También

- [/ralph_plan](ralph_plan.md) - Para crear plan después de investigación
- [/research_codebase](research_codebase.md) - El comando subyacente usado para investigación
- [/linear](linear.md) - Para gestión de tickets Linear
