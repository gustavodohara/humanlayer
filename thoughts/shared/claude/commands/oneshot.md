# /oneshot - Investigar Ticket y Lanzar Sesión de Planificación

## Descripción

Comando de conveniencia que combina investigación de ticket y lanzamiento de sesión de planificación. Ejecuta `/ralph_research` seguido de lanzar una nueva sesión para `/oneshot_plan`.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Proceso

### 1. Investiga el Ticket

Usa SlashCommand() para llamar `/ralph_research` con el número de ticket dado:
```bash
/ralph_research ENG-XXXX
```

Esto:
- Fetches el ticket de Linear
- Conduce investigación del codebase
- Genera documento de investigación
- Mueve ticket a "research in review"

### 2. Lanza Sesión de Planificación

Lanza una nueva sesión Claude Code para planificación:

```bash
npx humanlayer launch \
  --model opus \
  --dangerously-skip-permissions \
  --dangerously-skip-permissions-timeout 14m \
  --title "plan ENG-XXXX" \
  "/oneshot_plan ENG-XXXX"
```

La nueva sesión:
- Usa modelo opus para planificación crítica
- Tiene permisos omitidos con timeout de 14 minutos
- Ejecuta `/oneshot_plan` con el número de ticket
- Crea el plan de implementación completo

## Flujo Completo

```
1. /oneshot ENG-XXXX se invoca
   ↓
2. /ralph_research ENG-XXXX ejecuta
   ↓
3. Investigación se completa y se guarda
   ↓
4. Nueva sesión se lanza con opus
   ↓
5. /oneshot_plan ENG-XXXX ejecuta en nueva sesión
   ↓
6. Plan de implementación se crea
   ↓
7. Ticket listo para implementación
```

## Ventajas

- **Un comando para investigación + planificación**: No necesitas ejecutar dos comandos manualmente
- **Sesión fresca para planificación**: La nueva sesión tiene contexto limpio para enfocarse en planificación
- **Modelo opus para planificación**: Usa el modelo más potente para la tarea crítica de planificación
- **Permisos manejados**: Configuración de permisos apropiada para flujo automatizado

## Cuando Usar

Usa `/oneshot` cuando:
- Tienes un ticket que necesita tanto investigación como planificación
- Quieres automatizar el flujo de investigación-a-plan
- Quieres asegurar que la planificación use modelo opus
- Prefieres sesión separada para enfoque dedicado en planificación

## Cuando NO Usar

No uses `/oneshot` cuando:
- El ticket ya tiene investigación completa
- Quieres investigar sin planear inmediatamente
- Quieres más control sobre el proceso
- Prefieres ejecutar pasos manualmente

## Uso

```bash
/oneshot ENG-1234
```

## Ver También

- [/ralph_research](ralph_research.md) - Comando de investigación usado
- [/oneshot_plan](oneshot_plan.md) - Comando de planificación lanzado
- [/ralph_plan](ralph_plan.md) - Alternativa que solo planea
