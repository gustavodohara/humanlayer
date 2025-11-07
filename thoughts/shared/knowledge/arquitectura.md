---
date: 2025-11-05T22:02:17-03:00
researcher: Claude (Sonnet 4.5)
git_commit: 15e674c8d4707fefad2452628041cd9bbf247555
branch: main
repository: humanlayer
topic: "Arquitectura del Proyecto"
tags: [arquitectura, conocimiento, diseño-de-sistemas]
status: complete
last_updated: 2025-11-05
last_updated_by: Claude (Sonnet 4.5)
---

# Arquitectura: HumanLayer

**Fecha**: 2025-11-05
**Repositorio**: humanlayer
**Git Commit**: 15e674c8d4707fefad2452628041cd9bbf247555

## Resumen General

HumanLayer es un monorepo que contiene dos grupos principales de proyectos interconectados:

**Proyecto 1: HumanLayer SDK & Platform** - Producto core que proporciona capacidades human-in-the-loop para agentes AI (SDKs removidos en PR #646, legado)

**Proyecto 2: CodeLayer - Local Tools Suite** - Herramientas que aprovechan el SDK de HumanLayer para proporcionar experiencias de aprobación ricas

## Patrón Arquitectónico

**Arquitectura Cliente-Servidor Multi-Canal con Backend Event-Driven**

```
┌─────────────────┐     MCP Protocol      ┌──────────────┐
│   Claude Code   │◄──────────────────────►│     hlyr     │
└─────────────────┘                        │  (MCP Server)│
                                           └───────┬──────┘
                                                   │
                                            JSON-RPC over
                                            Unix Socket
┌─────────────────┐     HTTP REST API            │
│      WUI        │◄───────────────────────┐      │
│ (Tauri + React) │                        │      │
└─────────────────┘                        ▼      ▼
                                     ┌─────────────────┐
                                     │       hld       │
                                     │    (Daemon)     │
                                     └─────────────────┘
                                            │
                                     ┌──────┴──────┐
                                     │   SQLite    │
                                     │  Database   │
                                     └─────────────┘
```

## Componentes Principales del Sistema

### 1. HLD (HumanLayer Daemon)

**Ubicación**: `/home/gustavo/workspace/humanlayer/hld/`
**Lenguaje**: Go 1.24.0
**Función**: Backend core que coordina todas las operaciones

#### Punto de Entrada
- `/home/gustavo/workspace/humanlayer/hld/cmd/hld/main.go:14` - Inicialización del daemon

#### Estructura del Daemon
```go
type Daemon struct {
    config            *config.Config
    socketPath        string
    listener          net.Listener
    rpcServer         *rpc.Server
    httpServer        *HTTPServer
    sessions          session.SessionManager
    approvals         approval.Manager
    eventBus          bus.EventBus
    store             store.ConversationStore
    permissionMonitor *session.PermissionMonitor
}
```

#### Módulos Clave

**RPC Server** (`/home/gustavo/workspace/humanlayer/hld/rpc/server.go`)
- Implementa JSON-RPC 2.0 sobre Unix Socket
- Socket: `~/.humanlayer/daemon.sock` (configurable)
- Soporta conexiones persistentes para subscripciones

**HTTP Server** (`/home/gustavo/workspace/humanlayer/hld/daemon/http_server.go`)
- Puerto por defecto: 7777
- Basado en OpenAPI 3.1.0
- 16+ endpoints REST

**Session Manager** (`/home/gustavo/workspace/humanlayer/hld/session/manager.go`)
- Gestiona ciclo de vida de sesiones Claude Code
- Mantiene procesos activos por session ID
- Reconcilia aprobaciones con eventos de conversación

**Event Bus** (`/home/gustavo/workspace/humanlayer/hld/bus/events.go`)
- Implementación pub-sub interna
- Buffer de 100 eventos por subscriber
- Tipos de eventos: `new_approval`, `approval_resolved`, `session_status_changed`

**SQLite Store** (`/home/gustavo/workspace/humanlayer/hld/store/sqlite.go`)
- Base de datos: `~/.humanlayer/daemon.db`
- Modo WAL para mejor concurrencia
- Tablas: sessions, conversation_events, approvals, mcp_servers, raw_events

### 2. hlyr (HumanLayer CLI)

**Ubicación**: `/home/gustavo/workspace/humanlayer/hlyr/`
**Lenguaje**: TypeScript/Node.js
**Función**: CLI con servidor MCP para integración con Claude Code

#### Punto de Entrada
- `/home/gustavo/workspace/humanlayer/hlyr/src/index.ts` - Dispatcher de comandos CLI

#### Implementación MCP
- `/home/gustavo/workspace/humanlayer/hlyr/src/mcp.ts` - Servidor MCP para aprobaciones
- Tool: `request_permission` - Solicita aprobación humana
- Polling cada 1 segundo para verificar decisión

#### Cliente del Daemon
- `/home/gustavo/workspace/humanlayer/hlyr/src/daemonClient.ts` - Cliente JSON-RPC
- Comunicación vía Unix socket
- Soporta subscripciones con EventEmitter

### 3. humanlayer-wui (CodeLayer UI)

**Ubicación**: `/home/gustavo/workspace/humanlayer/humanlayer-wui/`
**Stack**: Tauri 2 + React 19 + TypeScript
**Función**: Interfaz gráfica desktop/web

#### Arquitectura Frontend
- **State Management**: Zustand (`AppStore.ts`)
- **Routing**: React Router v7
- **UI**: Radix UI + Tailwind CSS
- **Rich Text**: TipTap editor

#### Arquitectura Backend (Tauri)
- **Rust**: `/home/gustavo/workspace/humanlayer/humanlayer-wui/src-tauri/src/main.rs`
- **Daemon Manager**: Auto-lanza y gestiona ciclo de vida del daemon
- **IPC**: Comandos Tauri para comunicación frontend-backend

### 4. claudecode-go (Claude Code SDK)

**Ubicación**: `/home/gustavo/workspace/humanlayer/claudecode-go/`
**Lenguaje**: Go 1.21
**Función**: SDK para lanzar y controlar sesiones Claude Code

#### API Principal
```go
type Client struct {
    claudePath string
}

func (c *Client) Launch(config SessionConfig) (*Session, error)
func (c *Client) LaunchAndWait(config SessionConfig) (*Result, error)
```

## Flujo de Datos Principal

### Claude Code → Approval → Human

```
1. Claude Code ejecuta → invoca MCP tool `request_permission`
                         ↓
2. hlyr MCP Server → createApproval() via JSON-RPC
                         ↓
3. hld daemon → guarda en SQLite + publica evento
                         ↓
4. WUI (via SSE) → muestra notificación de aprobación
                         ↓
5. Usuario → aprueba/deniega en UI
                         ↓
6. WUI → POST /api/v1/approvals/{id}/respond
                         ↓
7. hld → actualiza DB + publica approval_resolved event
                         ↓
8. hlyr MCP (polling) → detecta cambio
                         ↓
9. hlyr → retorna {behavior: 'allow'/'deny'} a Claude
                         ↓
10. Claude Code → continúa/detiene según respuesta
```

## Comunicación Entre Componentes

### WUI ↔ Daemon (HTTP REST + SSE)
- **Endpoints REST**: 16+ endpoints (crear sesión, listar, actualizar, etc.)
- **SSE**: Streaming en tiempo real en `/stream`

### hlyr ↔ Daemon (JSON-RPC over Unix Socket)
- **Métodos RPC**: health, launchSession, createApproval, getApproval, respondToApproval, Subscribe
- **Socket**: `~/.humanlayer/daemon.sock`

### Daemon Internal (Event Bus)
```go
// Publisher
eventBus.Publish(Event{
    Type: "approval_resolved",
    Data: map[string]interface{}{
        "approval_id": id,
        "approved": true,
    }
})

// Subscriber
subscriber := eventBus.Subscribe(ctx, EventFilter{
    Types: []string{"approval_resolved"},
})
```

### hld ↔ Claude Code (Process Spawning)
- Spawn subprocess con `exec.Command`
- Variables de entorno: `HUMANLAYER_DAEMON_SOCKET`, `HUMANLAYER_SESSION_ID`
- Streaming stdout/stderr

## Decisiones Arquitectónicas Clave

### 1. Monorepo con Turborepo
- **Beneficio**: Build orchestration con dependencias
- **Herramienta**: Turbo + Bun workspaces

### 2. Unix Sockets para IPC Local
- **Seguridad**: Permisos 0600, solo owner
- **Performance**: Sin overhead TCP/HTTP
- **Paths**: `~/.humanlayer/daemon.sock` (prod), `~/.humanlayer/daemon-dev.sock` (dev)

### 3. SQLite como Store Principal
- **Ventajas**: Single-file, WAL mode, ACID, sin servidor externo
- **Configuración**: `PRAGMA journal_mode = WAL`

### 4. Event-Driven con Event Bus Interno
- **Pattern**: Observer/Pub-Sub
- **Beneficio**: Desacoplamiento, múltiples consumidores

### 5. MCP (Model Context Protocol)
- **Estándar**: Protocol para AI tool integration
- **Soporte**: Claude Code native support
- **Transport**: stdio

### 6. OpenAPI-First REST API
- **Spec**: `/home/gustavo/workspace/humanlayer/hld/api/openapi.yaml`
- **Codegen**: `oapi-codegen` genera server stubs
- **Beneficio**: Type safety end-to-end

### 7. Tauri para Desktop App
- **Ventajas**: Native performance, small bundle, OS integration
- **Backend**: Rust con gestión de daemon integrada

### 8. Dual Protocol (JSON-RPC + HTTP REST)
- **JSON-RPC**: hlyr, CLI tools (stateful, Unix socket)
- **HTTP REST**: WUI, external clients (stateless, SSE para real-time)

## Patrones de Diseño

### Repository Pattern
- Interface: `ConversationStore`
- Implementación: `SQLiteStore`

### Manager Pattern
- `SessionManager`, `ApprovalManager`
- Coordina múltiples subsistemas

### Event Sourcing (Parcial)
- Eventos de conversación persisten secuencialmente
- Sequence number para orden garantizado

### Optimistic Updates
- WUI actualiza estado local inmediatamente
- Reconcilia con API asíncrona

### Adapter Pattern
- `ClaudeCodeWrapper` abstrae interacción con Claude CLI

### State Machine
- Session Status: `starting → running → completed`
- Transiciones validadas

## Referencias de Archivos Clave

### Core Components

| Componente | Archivo Principal |
|------------|------------------|
| **hld main** | `/home/gustavo/workspace/humanlayer/hld/cmd/hld/main.go` |
| **Daemon** | `/home/gustavo/workspace/humanlayer/hld/daemon/daemon.go` |
| **RPC Server** | `/home/gustavo/workspace/humanlayer/hld/rpc/server.go` |
| **HTTP API** | `/home/gustavo/workspace/humanlayer/hld/api/handlers/server.go` |
| **Session Manager** | `/home/gustavo/workspace/humanlayer/hld/session/manager.go` |
| **Event Bus** | `/home/gustavo/workspace/humanlayer/hld/bus/events.go` |
| **SQLite Store** | `/home/gustavo/workspace/humanlayer/hld/store/sqlite.go` |
| **CLI Entry** | `/home/gustavo/workspace/humanlayer/hlyr/src/index.ts` |
| **MCP Server** | `/home/gustavo/workspace/humanlayer/hlyr/src/mcp.ts` |
| **Daemon Client** | `/home/gustavo/workspace/humanlayer/hlyr/src/daemonClient.ts` |
| **Tauri Main** | `/home/gustavo/workspace/humanlayer/humanlayer-wui/src-tauri/src/main.rs` |
| **App Store** | `/home/gustavo/workspace/humanlayer/humanlayer-wui/src/AppStore.ts` |
| **ClaudeCode Client** | `/home/gustavo/workspace/humanlayer/claudecode-go/client.go` |

## Resumen Ejecutivo

HumanLayer/CodeLayer es un sistema de orquestación de Claude Code con arquitectura **multi-tier event-driven**:

1. **Backend (hld)**: Daemon Go con SQLite, event bus interno, dual protocol (JSON-RPC + HTTP REST)
2. **CLI (hlyr)**: Tool TypeScript que expone MCP server para workflow de aprobaciones
3. **UI (WUI)**: App desktop Tauri + React con Zustand state management
4. **SDK (claudecode-go)**: Librería Go para control programático de Claude Code

**Fortalezas arquitectónicas**:
- Desacoplamiento vía event bus
- Múltiples interfaces (CLI, GUI, MCP)
- Type-safety end-to-end (OpenAPI codegen)
- Local-first con SQLite
- Extensible vía protocolo MCP

**Patrones clave**: Repository, Manager, Event Sourcing (parcial), Optimistic Updates, Pub-Sub
