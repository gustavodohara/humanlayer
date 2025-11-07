---
date: 2025-11-05T22:02:17-03:00
researcher: Claude (Sonnet 4.5)
git_commit: 15e674c8d4707fefad2452628041cd9bbf247555
branch: main
repository: humanlayer
topic: "Stack Tecnológico"
tags: [tecnología, stack, herramientas]
status: complete
last_updated: 2025-11-05
last_updated_by: Claude (Sonnet 4.5)
---

# Stack Tecnológico: HumanLayer

**Fecha**: 2025-11-05

## Lenguajes de Programación

### TypeScript
- **Versión**: 5.9.2 (raíz), 5.8.3 (hlyr), ~5.6.2 (humanlayer-wui)
- **Uso**: Proyectos frontend, CLI, SDKs

### Go
- **Versión**: 1.24.0 (hld), 1.21 (claudecode-go)
- **Uso**: Daemon backend, SDK Claude Code

### Rust
- **Versión**: 1.83.0 (CI), stable (producción)
- **Uso**: Backend Tauri (humanlayer-wui)
- **Edition**: 2021

### Node.js
- **Versión**: >=20 (raíz), >=16 (hlyr)
- **Uso**: Runtime para proyectos TypeScript

## Frameworks

### Frontend
- **React**: 19 (apps/react), 19.1.0 (humanlayer-wui)
- **Tauri**: 2 - Framework desktop multiplataforma
- **Vite**: 6.3.5 - Build tool y dev server

### Backend
- **Gin**: 1.10.1 - Framework HTTP para Go
- **gin-contrib/cors**: 1.7.6 - CORS middleware

### UI Components
- **Radix UI**: Conjunto completo de componentes React accesibles
- **Tiptap**: Editor de texto rico (v2 y v3)
- **Tailwind CSS**: 4.1.10 - Framework CSS utility-first

## Herramientas de Build

### Package Managers
- **Bun**: 1.2.23 - Package manager principal
- **npm**: Para algunos subproyectos

### Build Orchestration
- **Turbo**: 2.5.8 - Monorepo task orchestration
- **tsup**: 8.5.0 - TypeScript bundler
- **Vite**: 6.3.5 - Frontend build tool
- **Make**: Orquestación a nivel de sistema

## Bases de Datos

### SQLite
- **Driver Go**: github.com/mattn/go-sqlite3 v1.14.28
- **Uso**: Base de datos local del daemon
- **Configuración**: WAL mode para concurrencia

### PostgreSQL
- **Versión**: 17 (imagen Docker)
- **Driver**: pg v8.16.3
- **ORM**: Drizzle ORM v0.44.6

### ElectricSQL
- **Versión**: @electric-sql/react v1.0.14
- **Uso**: Sincronización en tiempo real

## Servicios Externos

### APIs Cloud
- **HumanLayer Cloud API**: https://api.humanlayer.dev/humanlayer/v1
- **Anthropic API**: Claude AI models
- **Linear API**: Gestión de proyectos (@linear/sdk v1.22.0)

### Observabilidad
- **Sentry**: @sentry/react v10.10.0 - Error tracking
- **PostHog**: posthog-js v1.279.3 - Analytics

## Protocolos y Estándares

### Model Context Protocol (MCP)
- **TypeScript**: @modelcontextprotocol/sdk v1.12.0
- **Go**: github.com/mark3labs/mcp-go v0.37.0

### OpenAPI
- **Especificación**: OpenAPI 3.1.0
- **Codegen**: oapi-codegen para Go, @openapitools/openapi-generator-cli para TypeScript

### Server-Sent Events (SSE)
- **Go**: github.com/r3labs/sse/v2 v2.10.0
- **TypeScript**: eventsource v4.0.0

### CRDT
- **Yjs**: v13.6.27 - Estructura de datos CRDT
- **y-protocols**: v1.0.6

## Herramientas de Desarrollo

### Linting y Formateo
- **ESLint**: v8.57.0 / v9.29.0
- **Prettier**: v3.6.2 / v3.5.3
- **golangci-lint**: v2.1.6
- **Biome**: v2.2.6
- **Clippy**: Rust linter (v1.83.0)

### Testing
- **Vitest**: v3.1.4 - Test runner TypeScript
- **Bun Test**: Built-in test framework
- **@testing-library/react**: v16.3.0
- **testify**: v1.11.1 - Testing Go
- **go.uber.org/mock**: v0.5.2 - Mock generation

### Desarrollo UI
- **Storybook**: v9.1.5 - Component development

## State Management
- **Zustand**: v5.0.5 - State management React

## Routing
- **React Router**: v7.6.3

## Styling
- **Tailwind CSS**: v4.1.10
- **class-variance-authority**: v0.7.1
- **clsx**: v2.1.1
- **tailwind-merge**: v3.3.1

## Markdown y Syntax
- **react-markdown**: v9.0.0
- **react-syntax-highlighter**: v15.5.0
- **remark-gfm**: v4.0.0
- **lowlight**: v3.3.0 - Syntax highlighting

## Utilidades
- **date-fns**: v4.1.0 - Manipulación de fechas
- **chalk**: v5.4.1 - Terminal colors
- **commander**: v14.0.0 - CLI framework
- **zod**: v4.1.11/v4.1.12 - Schema validation
- **nanoid**: v5.1.6 - ID generation

## CI/CD
- **GitHub Actions**: Automatización de workflows
- **Docker**: Containerización (PostgreSQL, Electric SQL)

## Deployment
- **Tauri**: Empaquetado desktop
- **Homebrew**: Distribución macOS
- **DMG**: Instaladores macOS

## Resumen por Componente

### hld (Daemon)
Go 1.24.0, Gin, SQLite, MCP-Go, OpenAPI, SSE

### hlyr (CLI)
TypeScript 5.8.3, Node >=16, Bun, Commander, MCP SDK

### humanlayer-wui (Desktop UI)
TypeScript 5.6.2, React 19, Tauri 2, Vite, Radix UI, Zustand

### apps/react (Collaborative App)
React 19, Bun, Tiptap, ElectricSQL, Yjs

### claudecode-go (SDK)
Go 1.21, minimal dependencies
