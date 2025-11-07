---
date: 2025-11-05T22:02:17-03:00
researcher: Claude (Sonnet 4.5)
git_commit: 15e674c8d4707fefad2452628041cd9bbf247555
branch: main
repository: humanlayer
topic: "Flujo de Desarrollo"
tags: [desarrollo, workflow, guía]
status: complete
last_updated: 2025-11-05
last_updated_by: Claude (Sonnet 4.5)
---

# Flujo de Desarrollo: HumanLayer

## Setup Inicial del Entorno

### Prerrequisitos
- **Go**: 1.21+ (1.24.0 recomendado)
- **Node.js**: >=20
- **Bun**: 1.2.23+
- **Rust**: Para Tauri (1.83.0+)

### Comando de Setup
```bash
make setup
```

Esto ejecuta:
1. Instalación de dependencias de plataforma
2. Instalación de herramientas (mockgen, golangci-lint)
3. Generación de mocks de HLD
4. Build de SDK TypeScript
5. Instalación de dependencias WUI
6. Build de hlyr CLI

## Comandos de Build

### Por Componente

```bash
# HLD (Daemon)
cd hld && go build -o hld ./cmd/hld
make -C hld build

# hlyr (CLI)
cd hlyr && npm install && npm run build

# WUI (Desktop)
cd humanlayer-wui && bun run build

# claudecode-go
cd claudecode-go && go build
```

### Builds Especiales

```bash
# Build nightly completo
make codelayer-nightly-bundle

# Build bundle para macOS
make codelayer-bundle
```

## Ejecutar Localmente

### Modo Desarrollo Estándar

```bash
# Opción recomendada (auto-lanza daemon)
make codelayer-dev

# Con analytics PostHog
POSTHOG=true make codelayer-dev
```

### Modo Desarrollo con Aislamiento por Ticket

```bash
# Para trabajar en ticket específico
make codelayer-dev TICKET=ENG-2114

# Esto crea:
# - DB: ~/.humanlayer/daemon-ENG-2114.db
# - Socket: ~/.humanlayer/daemon-{port}.sock
# - Puerto único calculado desde número de ticket
```

### Ejecutar Componentes por Separado

```bash
# Terminal 1 - Daemon
make daemon-dev

# Terminal 2 - WUI
make wui-dev

# Storybook (desarrollo de componentes)
make storybook
```

## Workflow de Git

### Crear Worktree

```bash
# Nombre generado automáticamente
make worktree

# O manualmente
hack/create_worktree.sh [nombre] [base_branch]
```

### Branching
- **Main branch**: `main`
- **Feature branches**: Crear desde `main`
- **Worktrees**: Ubicados en `~/wt/humanlayer/`

## Verificación y Testing

### Ejecutar Checks

```bash
# Todos los checks (linting, formato, typecheck)
make check

# Checks verbose
make check-verbose

# Por componente
make check-hlyr
make check-wui
make check-hld
make check-claudecode-go
```

### Ejecutar Tests

```bash
# Todos los tests
make test

# Tests verbose
make test-verbose

# Por componente
make test-hlyr    # Vitest
make test-wui     # Bun test
make test-hld     # Go unit + integration
make test-claudecode-go
```

### Tests de HLD Específicos

```bash
cd hld

# Unit tests
make test-unit

# Integration tests
make test-integration

# Con race detection
make test-unit-race
make test-integration-race

# E2E tests
make e2e-test
```

## Contribución

### Proceso de Pull Request

1. Fork del repositorio
2. Crear rama nueva
3. **Ejecutar antes de PR**:
```bash
make check test
```
4. Commit y push
5. Crear PR

### Comandos de Claude Code

Workflow recomendado:
```bash
/research_codebase
/create_plan
/implement_plan
/commit
gh pr create --fill
/describe_pr
```

### Instalar Git Hooks

```bash
make githooks
# Instala pre-push hook que ejecuta 'make check test'
```

## Tareas Comunes

### Regenerar SDKs

```bash
make generate-sdks
# Regenera TypeScript SDK desde OpenAPI spec
```

### Regenerar Mocks (Go)

```bash
make -C hld mocks
```

### Gestión de Base de Datos

```bash
# Clonar DB nightly a dev
make clone-nightly-db-to-dev-db

# Copiar DB producción a timestamped dev
make copy-db-to-dev

# Limpiar DBs antiguos (>10 días)
make cleanup-dev
```

### Ver Estado de Desarrollo

```bash
make dev-status
# Muestra: sockets, databases, daemons activos
```

## Debugging

### Ubicación de Logs

```bash
# Daemon desarrollo
~/.humanlayer/logs/daemon-dev-{timestamp}.log

# WUI desarrollo
~/.humanlayer/logs/wui-{branch}/codelayer.log

# Producción macOS
~/Library/Logs/dev.humanlayer.wui/CodeLayer.log
```

### Inspeccionar Base de Datos

```bash
# Ver sesiones recientes
sqlite3 ~/.humanlayer/daemon-dev.db \
  "SELECT * FROM sessions ORDER BY created_at DESC LIMIT 5;"

# Ver schema
sqlite3 ~/.humanlayer/daemon-dev.db ".schema"
```

### Testing RPC Manual

```bash
echo '{"jsonrpc":"2.0","method":"getSessionLeaves","params":{},"id":1}' | \
  nc -U ~/.humanlayer/daemon-dev.sock | jq '.'
```

## Comandos Rápidos de Referencia

```bash
# Setup completo
make setup

# Desarrollo
make codelayer-dev

# Desarrollo con ticket
make codelayer-dev TICKET=ENG-2114

# Checks + tests
make check-test

# Generar SDKs
make generate-sdks

# Estado de desarrollo
make dev-status

# Crear worktree
make worktree

# Storybook
make storybook

# Limpiar artefactos antiguos
make cleanup-dev
```

## Convenciones de Código

### Sistema de TODO

```
TODO(0): Crítico - nunca hacer merge
TODO(1): Alto - fallas arquitectónicas, bugs mayores
TODO(2): Medio - bugs menores, features faltantes
TODO(3): Bajo - polish, tests, documentación
TODO(4): Preguntas/investigaciones
PERF: Optimizaciones de rendimiento
```

### Estilo Go
- Context como primer parámetro
- No almacenar context en structs
- Manejo graceful de cancelación

### Estilo React (WUI)
- React 19: `ref` como prop estándar (NO usar `forwardRef`)
- Preferir componentes ShadCN
- Tailwind CSS para estilos
- Zustand para state global
- Tests primero para cambios complejos (TDD)

## Recursos Adicionales

- **README.md**: Documentación principal
- **CONTRIBUTING.md**: Guía detallada de contribución
- **CLAUDE.md**: Instrucciones específicas del proyecto
- **docs/**: Documentación de usuario
- **READMEs de componentes**: En cada subdirectorio
