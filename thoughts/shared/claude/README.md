# Claude Code - Comandos y Agentes

Este directorio contiene resúmenes de todos los comandos slash y agentes disponibles en el proyecto HumanLayer.

## 📚 Índice

### Comandos Slash

Los comandos slash son operaciones definidas por el usuario que comienzan con `/`. Se expanden a prompts completos cuando se ejecutan.

#### Gestión de Tickets y Flujo de Trabajo

- [/describe_pr](commands/describe_pr.md) - Genera descripciones comprehensivas de Pull Requests
- [/ci_describe_pr](commands/ci_describe_pr.md) - Genera descripciones de PR (variante CI)
- [/linear](commands/linear.md) - Gestiona tickets de Linear (crear, actualizar, comentar)
- [/ralph_research](commands/ralph_research.md) - Investiga el ticket Linear de mayor prioridad
- [/ralph_plan](commands/ralph_plan.md) - Crea plan de implementación para ticket de mayor prioridad
- [/ralph_impl](commands/ralph_impl.md) - Implementa ticket pequeño de mayor prioridad
- [/oneshot](commands/oneshot.md) - Investiga ticket y lanza sesión de planificación
- [/oneshot_plan](commands/oneshot_plan.md) - Ejecuta ralph_plan y ralph_impl para un ticket

#### Planificación e Implementación

- [/create_plan](commands/create_plan.md) - Crea planes de implementación detallados con investigación
- [/create_plan_nt](commands/create_plan_nt.md) - Crea planes sin directorio thoughts
- [/create_plan_generic](commands/create_plan_generic.md) - Crea planes con investigación exhaustiva
- [/iterate_plan](commands/iterate_plan.md) - Itera sobre planes existentes
- [/validate_plan](commands/validate_plan.md) - Valida implementación contra el plan
- [/implement_plan](commands/implement_plan.md) - Implementa planes técnicos desde thoughts/shared/plans

#### Investigación y Documentación

- [/research_codebase](commands/research_codebase.md) - Documenta el codebase as-is con thoughts directory
- [/research_codebase_nt](commands/research_codebase_nt.md) - Documenta codebase sin evaluaciones
- [/research_codebase_generic](commands/research_codebase_generic.md) - Investiga codebase usando sub-agentes paralelos
- [/project_knowledge](commands/project_knowledge.md) - Analiza estructura del proyecto y genera documentación

#### Control de Versiones

- [/commit](commands/commit.md) - Crea commits de git con aprobación del usuario
- [/ci_commit](commands/ci_commit.md) - Crea commits para sesión con mensajes claros

#### Gestión de Sesiones

- [/create_handoff](commands/create_handoff.md) - Crea documento de handoff para transferir trabajo
- [/resume_handoff](commands/resume_handoff.md) - Reanuda trabajo desde documento de handoff

#### Desarrollo y Debugging

- [/local_review](commands/local_review.md) - Configura worktree para revisar rama de colega
- [/debug](commands/debug.md) - Debug de issues investigando logs y estado de BD
- [/founder_mode](commands/founder_mode.md) - Crea ticket Linear y PR para features experimentales

---

### Agentes

Los agentes son sub-tareas especializadas que se pueden invocar para realizar investigación o análisis específico.

#### Agentes de Codebase

- [codebase-locator](agents/codebase-locator.md) - Localiza archivos, directorios y componentes
- [codebase-analyzer](agents/codebase-analyzer.md) - Analiza detalles de implementación del codebase
- [codebase-pattern-finder](agents/codebase-pattern-finder.md) - Encuentra implementaciones similares y patrones

#### Agentes de Thoughts

- [thoughts-locator](agents/thoughts-locator.md) - Descubre documentos relevantes en thoughts/
- [thoughts-analyzer](agents/thoughts-analyzer.md) - Análisis profundo de documentos de investigación

#### Agentes de Investigación Externa

- [web-search-researcher](agents/web-search-researcher.md) - Investiga información en la web

---

## 🎯 Flujo de Trabajo Recomendado

### Para Nuevas Features

1. `/ralph_research` - Investiga el ticket
2. `/ralph_plan` - Crea plan de implementación
3. `/ralph_impl` - Implementa el ticket
4. `/validate_plan` - Valida la implementación
5. `/commit` - Crea commits
6. `/describe_pr` - Genera descripción de PR

### Para Investigación de Codebase

1. `/research_codebase` - Para investigación general
2. Usa agentes específicos para análisis profundo:
   - `codebase-locator` - Encontrar archivos
   - `codebase-analyzer` - Entender implementación
   - `codebase-pattern-finder` - Encontrar patrones similares

### Para Documentación

1. `/project_knowledge` - Genera documentación comprehensiva del proyecto
2. `/research_codebase` - Documenta áreas específicas

---

## 📝 Notas

- **Comandos vs Agentes**: Los comandos son operaciones de alto nivel que orquestan múltiples pasos. Los agentes son workers especializados que realizan tareas específicas de investigación.

- **Thoughts Directory**: Muchos comandos interactúan con `thoughts/` para almacenar investigación, planes y documentación.

- **Linear Integration**: Los comandos `ralph_*` están diseñados para integrarse con Linear para gestión de tickets.

- **Modo Opus vs Sonnet**: Algunos comandos especifican el modelo a usar (opus para planificación, sonnet para implementación).
