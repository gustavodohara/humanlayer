# Referencia Rápida - Comandos y Agentes Claude Code

## 🚀 Comandos Más Usados

### Workflow de Features Completo

```bash
/ralph_research     # Investiga ticket de mayor prioridad
/ralph_plan        # Crea plan de implementación
/ralph_impl        # Implementa el ticket
/validate_plan     # Valida implementación
/commit            # Crea commits
/describe_pr       # Genera descripción de PR
```

### Planificación e Implementación

| Comando | Propósito | Modelo |
|---------|-----------|--------|
| `/create_plan` | Crear plan detallado con investigación | opus |
| `/iterate_plan` | Actualizar plan existente | opus |
| `/implement_plan` | Ejecutar plan técnico | default |
| `/validate_plan` | Verificar implementación | default |

### Investigación

| Comando | Propósito | Modelo |
|---------|-----------|--------|
| `/research_codebase` | Documentar codebase as-is | opus |
| `/project_knowledge` | Generar docs comprehensivas | default |
| `/debug` | Investigar logs y BD | default |

### Gestión de Tickets

| Comando | Propósito | Modelo |
|---------|-----------|--------|
| `/linear` | Gestionar tickets Linear | default |
| `/ralph_research` | Investigar ticket prioridad | default |
| `/ralph_plan` | Planear ticket prioridad | default |
| `/ralph_impl` | Implementar ticket pequeño | sonnet |

### Control de Versiones

| Comando | Propósito | Modelo |
|---------|-----------|--------|
| `/commit` | Commits con aprobación | default |
| `/ci_commit` | Commits para CI | default |
| `/describe_pr` | Descripción de PR | default |

### Gestión de Sesiones

| Comando | Propósito | Modelo |
|---------|-----------|--------|
| `/create_handoff` | Crear documento de handoff | default |
| `/resume_handoff` | Reanudar desde handoff | default |
| `/local_review` | Setup worktree para review | default |

---

## 🤖 Agentes Especializados

### Agentes de Codebase

| Agente | Cuándo Usar | Herramientas |
|--------|-------------|--------------|
| `codebase-locator` | Encontrar DÓNDE vive el código | Grep, Glob, LS |
| `codebase-analyzer` | Entender CÓMO funciona | Read, Grep, Glob, LS |
| `codebase-pattern-finder` | Encontrar ejemplos similares | Grep, Glob, Read, LS |

### Agentes de Thoughts

| Agente | Cuándo Usar | Herramientas |
|--------|-------------|--------------|
| `thoughts-locator` | Descubrir documentos thoughts/ | Grep, Glob, LS |
| `thoughts-analyzer` | Análisis profundo de docs | Read, Grep, Glob, LS |

### Agentes de Investigación Externa

| Agente | Cuándo Usar | Herramientas |
|--------|-------------|--------------|
| `web-search-researcher` | Buscar info en la web | WebSearch, WebFetch |

---

## 📋 Flujos de Trabajo Comunes

### 1. Nueva Feature desde Ticket Linear

```bash
/ralph_research     # → Investiga el ticket
                   # → Mueve a "Research in Review"

/ralph_plan        # → Crea plan de implementación
                   # → Mueve a "Plan in Review"

/ralph_impl        # → Setup worktree
                   # → Implementa
                   # → Crea commits
                   # → Crea PR
                   # → Mueve a "Code Review"
```

### 2. Investigar Codebase

```bash
/research_codebase                    # Inicia investigación
# → Spawns sub-agents:
#   - codebase-locator (encontrar archivos)
#   - codebase-analyzer (entender código)
#   - thoughts-locator (buscar docs)
# → Genera documento en thoughts/shared/research/
```

### 3. Implementar Feature

```bash
/create_plan <ticket>                 # Crear plan detallado
/implement_plan <plan>                # Implementar fase por fase
/validate_plan <plan>                 # Verificar implementación
/commit                               # Crear commits
/describe_pr                          # Generar descripción PR
```

### 4. Debugging Durante Testing

```bash
/debug                                # Investiga logs, BD, git
# → Analiza logs en ~/.humanlayer/logs/
# → Query BD en ~/.humanlayer/daemon.db
# → Revisa git status y recent commits
# → Presenta report de debugging
```

---

## 🎯 Atajos Mentales

### ¿Necesitas Encontrar Código?
→ `codebase-locator` → `codebase-analyzer`

### ¿Necesitas Ver Cómo Algo Está Hecho?
→ `codebase-pattern-finder`

### ¿Necesitas Buscar Documentación Histórica?
→ `thoughts-locator` → `thoughts-analyzer`

### ¿Necesitas Información Externa?
→ `web-search-researcher`

### ¿Necesitas Crear un Plan?
→ `/create_plan` (usa todos los agentes)

### ¿Necesitas Investigar el Codebase?
→ `/research_codebase` (usa todos los agentes)

---

## 💡 Tips Importantes

### Comandos vs Agentes
- **Comandos** (`/command`): Operaciones de alto nivel que orquestan múltiples pasos
- **Agentes**: Workers especializados invocados por comandos para tareas específicas

### Modelos
- **opus**: Para planificación crítica y análisis profundo
- **sonnet**: Para implementación y tareas de búsqueda
- **default**: Usa el modelo configurado de la sesión

### Thoughts Directory
- `thoughts/shared/` → Documentos del equipo
- `thoughts/allison/` → Notas personales
- `thoughts/global/` → Cross-repo thoughts
- Muchos comandos leen/escriben aquí automáticamente

### Linear Workflow
Estados clave en orden:
1. Triage
2. Research Needed → Research in Progress → Research in Review
3. Ready for Plan → Plan in Progress → Plan in Review
4. Ready for Dev → In Dev → Code Review → Done

### Verificación de Planes
Planes tienen dos tipos de criterios:
- **Automated**: Puede ejecutarse con comandos (`make test`)
- **Manual**: Requiere pruebas humanas (UI/UX)

---

## 📚 Para Más Detalles

- Ver [README.md](README.md) para índice completo
- Ver carpeta `commands/` para detalles de cada comando
- Ver carpeta `agents/` para detalles de cada agente

---

## 🔗 Recursos Externos

- **Linear Workspace**: https://linear.app/humanlayer
- **Thoughts Repo**: https://github.com/humanlayer/thoughts
- **Main Repo**: https://github.com/humanlayer/humanlayer
