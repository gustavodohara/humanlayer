# /debug - Debug de Issues Durante Testing Manual

## Descripción

Ayuda a debuggear issues durante testing manual o implementación investigando logs, estado de base de datos e historial de git sin editar archivos. Piensa en esto como una forma de bootstrap una sesión de debugging sin usar el contexto de la ventana principal.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Respuesta Inicial

### Cuando se invoca CON archivo de plan/ticket:
```
I'll help debug issues with [file name]. Let me understand the current state.

What specific problem are you encountering?
- What were you trying to test/implement?
- What went wrong?
- Any error messages?

I'll investigate the logs, database, and git state to help figure out what's happening.
```

### Cuando se invoca SIN parámetros:
```
I'll help debug your current issue.

Please describe what's going wrong:
- What are you working on?
- What specific problem occurred?
- When did it last work?

I can investigate logs, database state, and recent changes to help identify the issue.
```

## Información del Entorno

Tienes acceso a estas ubicaciones y herramientas clave:

### Logs (creados automáticamente por `make daemon` y `make wui`):
- Logs MCP: `~/.humanlayer/logs/mcp-claude-approvals-*.log`
- Logs combinados WUI/Daemon: `~/.humanlayer/logs/wui-${BRANCH_NAME}/codelayer.log`
- Primera línea muestra: `[timestamp] starting [service] in [directory]`

### Base de Datos:
- Ubicación: `~/.humanlayer/daemon-{BRANCH_NAME}.db`
- Base de datos SQLite con sesiones, eventos, aprobaciones, etc.
- Puede hacer query directamente con `sqlite3`

### Estado de Git:
- Verifica rama actual, commits recientes, cambios sin commit
- Similar a cómo funcionan los comandos `commit` y `describe_pr`

### Estado del Servicio:
- Verifica si daemon está corriendo: `ps aux | grep hld`
- Verifica si WUI está corriendo: `ps aux | grep wui`
- Socket existe: `~/.humanlayer/daemon.sock`

## Pasos del Proceso

### Paso 1: Entiende el Problema

Después de que el usuario describe el issue:

1. **Lee cualquier contexto proporcionado** (archivo de plan o ticket):
   - Entiende qué están implementando/testeando
   - Nota en qué fase o paso están
   - Identifica comportamiento esperado vs actual

2. **Verificación rápida de estado**:
   - Rama git actual y commits recientes
   - Cualquier cambio sin commit
   - Cuándo comenzó a ocurrir el issue

### Paso 2: Investiga el Issue

Genera agentes Task paralelos para investigación eficiente:

```
Task 1 - Check Recent Logs:
Find and analyze the most recent logs for errors:
1. Find latest daemon log: ls -t ~/.humanlayer/logs/daemon-*.log | head -1
2. Find latest WUI log: ls -t ~/.humanlayer/logs/wui-*.log | head -1
3. Search for errors, warnings, or issues around the problem timeframe
4. Note the working directory (first line of log)
5. Look for stack traces or repeated errors
Return: Key errors/warnings with timestamps
```

```
Task 2 - Database State:
Check the current database state:
1. Connect to database: sqlite3 ~/.humanlayer/daemon.db
2. Check schema: .tables and .schema for relevant tables
3. Query recent data:
   - SELECT * FROM sessions ORDER BY created_at DESC LIMIT 5;
   - SELECT * FROM conversation_events WHERE created_at > datetime('now', '-1 hour');
   - Other queries based on the issue
4. Look for stuck states or anomalies
Return: Relevant database findings
```

```
Task 3 - Git and File State:
Understand what changed recently:
1. Check git status and current branch
2. Look at recent commits: git log --oneline -10
3. Check uncommitted changes: git diff
4. Verify expected files exist
5. Look for any file permission issues
Return: Git state and any file issues
```

### Paso 3: Presenta Hallazgos

Basándote en la investigación, presenta reporte de debugging enfocado:

```markdown
## Debug Report

### What's Wrong
[Declaración clara del issue basada en evidencia]

### Evidence Found

**From Logs** (`~/.humanlayer/logs/`):
- [Error/warning con timestamp]
- [Patrón o issue repetido]

**From Database**:
```sql
-- Query relevante y resultado
[Hallazgo de base de datos]
```

**From Git/Files**:
- [Cambios recientes que podrían estar relacionados]
- [Issues de estado de archivo]

### Root Cause
[Explicación más probable basada en evidencia]

### Next Steps

1. **Try This First**:
   ```bash
   [Comando o acción específica]
   ```

2. **If That Doesn't Work**:
   - Restart services: `make daemon` and `make wui`
   - Check browser console for WUI errors
   - Run with debug: `HUMANLAYER_DEBUG=true make daemon`

### Can't Access?
Some issues might be outside my reach:
- Browser console errors (F12 in browser)
- MCP server internal state
- System-level issues

Would you like me to investigate something specific further?
```

## Notas Importantes

- **Enfócate en escenarios de testing manual** - Esto es para debugging durante implementación
- **Siempre requiere descripción del problema** - No puede debuggear sin saber qué está mal
- **Lee archivos completamente** - Sin limit/offset cuando leas contexto
- **Piensa como `commit` o `describe_pr`** - Entiende estado de git y cambios
- **Guía de vuelta al usuario** - Algunos issues (consola del browser, MCP internals) están fuera de alcance
- **Sin edición de archivos** - Solo investigación pura

## Referencia Rápida

### Encontrar Logs Más Recientes:
```bash
ls -t ~/.humanlayer/logs/daemon-*.log | head -1
ls -t ~/.humanlayer/logs/wui-*.log | head -1
```

### Queries de Base de Datos:
```bash
sqlite3 ~/.humanlayer/daemon.db ".tables"
sqlite3 ~/.humanlayer/daemon.db ".schema sessions"
sqlite3 ~/.humanlayer/daemon.db "SELECT * FROM sessions ORDER BY created_at DESC LIMIT 5;"
```

### Verificación de Servicio:
```bash
ps aux | grep hld     # ¿Está corriendo daemon?
ps aux | grep wui     # ¿Está corriendo WUI?
```

### Estado de Git:
```bash
git status
git log --oneline -10
git diff
```

## Recordatorio

Este comando te ayuda a investigar sin quemar el contexto de la ventana principal. Perfecto para cuando encuentras un issue durante testing manual y necesitas investigar logs, base de datos o estado de git.

## Uso

```bash
/debug
# O con contexto:
/debug thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-feature.md
```

## Ver También

- [/implement_plan](implement_plan.md) - Para implementación que estás debuggeando
- [/validate_plan](validate_plan.md) - Para validar después de arreglar issues
