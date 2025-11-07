---
date: 2025-11-05T22:02:17-03:00
researcher: Claude (Sonnet 4.5)
git_commit: 15e674c8d4707fefad2452628041cd9bbf247555
branch: main
repository: humanlayer
topic: "Estructura de Carpetas y Organización"
tags: [estructura-carpetas, organización, conocimiento]
status: complete
last_updated: 2025-11-05
last_updated_by: Claude (Sonnet 4.5)
---

# Estructura de Carpetas: HumanLayer

**Fecha**: 2025-11-05
**Repositorio**: humanlayer
**Git Commit**: 15e674c8d4707fefad2452628041cd9bbf247555

## Visión General

HumanLayer es un **monorepo** que contiene dos grupos principales de proyectos interconectados. La organización sigue una estructura de workspaces con Bun y Turborepo para gestión de builds.

## Tipo de Proyecto

**Monorepo** con dos grupos:
- **Proyecto 1**: HumanLayer SDK & Platform (legado, removido en PR #646)
- **Proyecto 2**: Local Tools Suite (CodeLayer - enfoque actual)

## Árbol de Directorios Raíz

```
humanlayer/
├── .claude/                 # Configuración de Claude Code
├── .github/                 # GitHub Actions workflows
├── apps/                    # Aplicaciones del monorepo
├── docs/                    # Documentación Mintlify
├── hack/                    # Scripts de desarrollo y utilidades
├── hld/                     # HumanLayer Daemon (Go)
├── hlyr/                    # HumanLayer CLI (TypeScript)
├── humanlayer-wui/          # CodeLayer Desktop UI (Tauri + React)
├── claudecode-go/           # Claude Code Go SDK
├── packages/                # Paquetes compartidos TypeScript
├── scripts/                 # Scripts de build
├── thoughts/                # Knowledge base (este directorio)
├── Makefile                 # Orquestación de builds principal
├── package.json             # Workspace raíz
├── turbo.json               # Configuración Turborepo
├── CLAUDE.md                # Instrucciones para Claude Code
├── CONTRIBUTING.md          # Guía de contribución
└── README.md                # Documentación principal
```

## Directorios de Nivel Raíz

### Configuración Raíz

**Package Manager**: Bun v1.2.23
**Node Version**: >= 20
**Estructura**: Workspaces `apps/*` y `packages/*`

**Archivos de configuración clave**:
- `package.json` - Workspace raíz y scripts
- `turbo.json` - Orquestación de builds
- `tsconfig.json` - Configuración TypeScript base
- `biome.jsonc` - Linter/formatter
- `docker-compose.yml` - PostgreSQL y Electric SQL (dev)
- `.pre-commit-config.yaml` - Pre-commit hooks

**Documentación raíz**:
- `README.md` - Documentación principal del proyecto
- `CLAUDE.md` - Instrucciones para Claude Code
- `DEVELOPMENT.md` - Guía de desarrollo
- `CONTRIBUTING.md` - Guía de contribución
- `humanlayer.md` - Documentación adicional

## Proyecto 2: Local Tools Suite (Enfoque Actual)

### 1. hld/ - Go Daemon

**Ubicación**: `/home/gustavo/workspace/humanlayer/hld/`
**Propósito**: Daemon backend que coordina aprobaciones y gestiona sesiones Claude Code

**Estructura**:
```
hld/
├── cmd/hld/              # Punto de entrada (main.go)
├── api/                  # HTTP API layer
│   ├── handlers/         # Request handlers
│   └── openapi.yaml      # Especificación OpenAPI
├── daemon/               # Implementación daemon core
├── session/              # Gestión de sesiones (15+ archivos)
├── store/                # Capa de persistencia SQLite (8+ archivos)
├── approval/             # Gestión de aprobaciones
├── rpc/                  # Servidor JSON-RPC (7+ archivos)
├── mcp/                  # Servidor Model Context Protocol
├── client/               # Biblioteca cliente para daemon
├── bus/                  # Event bus pub/sub
├── config/               # Gestión de configuración
├── internal/             # Paquetes internos
│   ├── filescan/         # Utilidades de escaneo
│   ├── testutil/         # Utilidades de testing
│   └── version/          # Información de versión
├── sdk/typescript/       # SDK TypeScript generado
│   └── src/generated/    # Código OpenAPI generado
├── e2e/                  # Tests end-to-end
├── go.mod                # Módulo Go
├── Makefile              # Comandos de build
├── README.md
└── CLAUDE.md
```

**Tests**:
- 32+ archivos `*_integration_test.go`
- 25+ archivos `*_test.go`
- E2E tests en TypeScript

### 2. hlyr/ - TypeScript CLI

**Ubicación**: `/home/gustavo/workspace/humanlayer/hlyr/`
**Propósito**: CLI con servidor MCP para integración Claude Code

**Estructura**:
```
hlyr/
├── src/
│   ├── index.ts          # Entry point CLI
│   ├── mcp.ts            # Implementación servidor MCP
│   ├── commands/         # Implementaciones de comandos
│   │   ├── claude.ts
│   │   ├── thoughts.ts
│   │   └── configShow.ts
│   ├── daemonClient.ts   # Cliente JSON-RPC del daemon
│   ├── config.ts         # Gestión de configuración
│   └── utils/
├── tests/                # Tests E2E con Vitest
│   ├── claudeInit.e2e.test.ts
│   └── configShow.e2e.test.ts
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── Makefile
└── README.md
```

### 3. humanlayer-wui/ - Desktop/Web UI (CodeLayer)

**Ubicación**: `/home/gustavo/workspace/humanlayer/humanlayer-wui/`
**Propósito**: Interfaz gráfica desktop (Tauri + React)

**Estructura React (src/)**:
```
humanlayer-wui/
├── src/
│   ├── router.tsx         # Configuración React Router
│   ├── AppStore.ts        # Store Zustand principal
│   ├── Layout.tsx         # Layout principal
│   ├── components/
│   │   ├── ui/            # Componentes reutilizables (50+ archivos)
│   │   │   ├── button.tsx
│   │   │   ├── dialog.tsx
│   │   │   └── ...
│   │   └── internal/      # Componentes específicos del proyecto
│   │       ├── SessionTable.tsx
│   │       ├── SessionDetail/
│   │       │   ├── SessionDetail.tsx
│   │       │   ├── components/
│   │       │   ├── hooks/
│   │       │   └── __tests__/
│   │       └── ConversationStream/
│   ├── stores/            # State management Zustand
│   │   └── demo/          # Funcionalidad demo mode
│   ├── services/          # Servicios de negocio
│   │   ├── daemon-service.ts
│   │   └── NotificationService.tsx
│   ├── hooks/             # React hooks personalizados
│   ├── lib/               # Bibliotecas y utilidades
│   │   ├── daemon/        # Cliente del daemon
│   │   ├── telemetry/     # PostHog, Sentry
│   │   └── utils.ts
│   ├── utils/             # Funciones utilitarias
│   └── styles/            # Estilos globales
├── src-tauri/             # Backend Rust (Tauri)
│   ├── src/
│   │   ├── main.rs        # Entry point Tauri
│   │   └── daemon.rs      # Gestión daemon lifecycle
│   ├── Cargo.toml
│   └── tauri.conf.json
├── .storybook/            # Configuración Storybook
├── package.json
├── vite.config.ts
├── Makefile
└── README.md
```

**Tests**: 21+ archivos `.test.ts` y `.test.tsx`

### 4. claudecode-go/ - Go SDK

**Ubicación**: `/home/gustavo/workspace/humanlayer/claudecode-go/`
**Propósito**: SDK Go para lanzar sesiones Claude Code

**Estructura**:
```
claudecode-go/
├── client.go             # Implementación cliente principal
├── types.go              # Definiciones de tipos
├── doc.go                # Documentación del paquete
├── *_test.go             # Tests (7 archivos)
├── go.mod
├── Makefile
└── README.md
```

## Infraestructura Compartida

### 5. docs/ - Sitio de Documentación Mintlify

**Ubicación**: `/home/gustavo/workspace/humanlayer/docs/`

**Estructura**:
```
docs/
├── mint.json             # Configuración Mintlify
├── readme.md
├── introduction.mdx
├── quickstart-typescript.mdx
├── development.mdx
├── images/               # Logos y branding
├── case-studies/
├── Dockerfile
└── package-lock.json
```

### 6. packages/ - Paquetes TypeScript Compartidos

#### packages/contracts/
**Propósito**: Contratos TypeScript compartidos

```
packages/contracts/
├── src/
│   ├── index.ts
│   └── daemon/
│       ├── index.ts
│       └── openapi.ts
├── package.json
├── tsconfig.json
└── README.md
```

#### packages/database/
**Propósito**: Esquemas de base de datos (Drizzle ORM)

```
packages/database/
├── schema/               # Esquemas de DB
│   ├── index.ts
│   ├── thoughts.ts
│   └── scores.ts
├── drizzle/              # Migraciones (3 archivos)
├── scripts/
│   └── test.ts
├── drizzle.config.ts
├── package.json
└── README.md
```

### 7. apps/ - Aplicaciones

#### apps/daemon/
**Propósito**: Daemon TypeScript

```
apps/daemon/
├── src/
│   ├── index.ts
│   ├── router/
│   │   ├── index.ts
│   │   ├── server.ts
│   │   └── sessions.ts
│   └── swagger.ts
├── package.json
└── README.md
```

#### apps/react/
**Propósito**: Aplicación React con ElectricSQL

```
apps/react/
├── src/
│   ├── index.tsx
│   ├── frontend.tsx
│   ├── y-electric/       # Integración Electric SQL
│   ├── lib/              # Bibliotecas compartidas
│   ├── components/ui/    # Componentes UI (8 archivos)
│   └── index.html
├── styles/
│   └── globals.css
├── package.json
└── README.md
```

### 8. scripts/ - Scripts de Build

```
scripts/
├── test.ts
├── dev-with-posthog.sh
├── package.json
└── README.md
```

### 9. hack/ - Utilidades de Desarrollo

**Propósito**: Scripts shell y utilidades para desarrollo

```
hack/
├── setup_repo.sh                # Setup del repositorio
├── install_platform_deps.sh     # Dependencias de plataforma
├── generate_*_icons.sh          # Generación de íconos (varios)
├── port-utils.sh                # Utilidades de puertos
├── spec_metadata.sh             # Metadatos OpenAPI
├── create_worktree.sh           # Gestión de worktrees
├── cleanup_worktree.sh
├── linear/                      # Integración Linear
│   ├── package.json
│   ├── linear-cli.ts
│   └── README.md
└── dex/                         # Flow de desarrollo
    └── flow-tmux.sh
```

### 10. .github/ - Configuración GitHub

```
.github/
├── workflows/               # GitHub Actions
│   ├── main.yml             # CI/CD principal
│   ├── claude.yml           # Integración Claude
│   ├── release-macos.yml    # Build releases macOS
│   ├── linear-*.yml         # Automatización Linear (3 archivos)
│   └── CLAUDE.md
├── ISSUE_TEMPLATE/          # Templates de issues
│   ├── bug_report.md
│   ├── feature_request.md
│   └── feedback.md
└── PULL_REQUEST_TEMPLATE.md
```

### 11. .claude/ - Configuración Claude Code

```
.claude/
├── settings.local.json
├── commands/                # Comandos slash personalizados (30+)
│   ├── project_knowledge.md
│   ├── create_plan.md
│   ├── implement_plan.md
│   ├── research_codebase.md
│   ├── linear.md
│   ├── debug.md
│   └── ...
└── agents/                  # Agentes personalizados
    ├── thoughts-locator.md
    ├── thoughts-analyzer.md
    ├── codebase-pattern-finder.md
    └── web-search-researcher.md
```

## Organización de Tests

### TypeScript
- **Co-ubicados**: Tests junto a código (`.test.ts`, `.test.tsx`)
- **Directorios __tests__/**: Para componentes React

### Go
- **Mismo paquete**: `*_test.go`
- **Integración**: `*_integration_test.go` con build tags
- **E2E**: Directorio `e2e/` separado

## Convenciones de Nombrado

### Archivos TypeScript
- **Componentes**: PascalCase (`SessionTable.tsx`)
- **Hooks**: camelCase con `use` (`useSessions.ts`)
- **Utilidades**: camelCase (`logging.ts`)
- **Tests**: sufijo `.test.tsx`
- **Stories**: sufijo `.stories.tsx`

### Archivos Go
- **Archivos**: snake_case (`session_manager.go`)
- **Tests**: sufijo `_test.go`
- **Integración**: sufijo `_integration_test.go`
- **Mocks**: prefijo `mock_` (`mock_session.go`)

## Patrones de Organización

### Módulos por Funcionalidad
- Cada directorio agrupa funcionalidad relacionada
- Subdirectorios para componentes grandes

### Separación Concerns
- `src/components/` - UI components
- `src/lib/` - Lógica de negocio
- `src/hooks/` - React hooks
- `src/stores/` - State management
- `src/utils/` - Utilidades puras

### Tests Cerca del Código
- Tests co-ubicados facilitan mantenimiento
- `__tests__/` para múltiples tests de un componente

## Archivos de Salida de Build

**Nota**: Ningún directorio `dist/`, `build/`, o `node_modules/` está commiteado (excluidos via .gitignore)

## Resumen

El monorepo HumanLayer está organizado en:
- **2 proyectos Go**: hld (daemon), claudecode-go (SDK)
- **11 proyectos TypeScript**: CLI, WUI, apps, packages
- **1 proyecto Rust**: Tauri backend
- **Infraestructura**: docs, scripts, hack/, .github/, .claude/

Estructura optimizada para:
- Development paralelo con worktrees
- Build orchestration con Turborepo
- Separation of concerns clara
- Testing comprehensivo
