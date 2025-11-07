# Base de Conocimiento: HumanLayer

Esta base de conocimiento contiene documentación comprehensiva del proyecto HumanLayer, un monorepo que incluye CodeLayer (IDE para orquestar agentes de IA) y HumanLayer SDK & Platform.

## Documentos Principales

### 📐 [Arquitectura](arquitectura.md)
Descripción detallada de la arquitectura del sistema, componentes principales, flujos de datos y decisiones arquitectónicas clave.

**Contenido**:
- Patrón arquitectónico (Cliente-Servidor Multi-Canal Event-Driven)
- Componentes principales (hld, hlyr, WUI, claudecode-go)
- Flujo de datos y comunicación
- Decisiones arquitectónicas clave
- Patrones de diseño utilizados

### 📁 [Estructura de Carpetas](estructura-carpetas.md)
Organización completa del monorepo con descripción de cada directorio y su propósito.

**Contenido**:
- Árbol de directorios completo
- Organización por proyecto
- Convenciones de nombrado
- Patrones de organización
- Tests y su ubicación

### 🛠️ [Stack Tecnológico](stack-tecnologico.md)
Inventario completo de tecnologías, frameworks y herramientas utilizadas.

**Contenido**:
- Lenguajes (TypeScript, Go, Rust, Node.js)
- Frameworks (React, Tauri, Gin)
- Herramientas de build (Bun, Turbo, Vite, Make)
- Bases de datos (SQLite, PostgreSQL)
- Servicios externos (APIs, observabilidad)
- Stack por componente

### 🔄 [Flujo de Desarrollo](flujo-desarrollo.md)
Guía completa de procesos de desarrollo, desde setup hasta deployment.

**Contenido**:
- Setup inicial del entorno
- Comandos de build
- Ejecutar localmente (desarrollo estándar, con tickets, componentes separados)
- Workflow de Git y branching
- Verificación y testing
- Proceso de contribución
- Tareas comunes
- Debugging
- Convenciones de código

## Metadatos del Proyecto

- **Repositorio**: humanlayer
- **Branch**: main
- **Commit**: 15e674c8d4707fefad2452628041cd9bbf247555
- **Fecha de Generación**: 2025-11-05
- **Generado por**: Claude (Sonnet 4.5)

## Estado de la Documentación

- ✅ **Arquitectura**: Completa
- ✅ **Estructura de Carpetas**: Completa
- ✅ **Stack Tecnológico**: Completo
- ✅ **Flujo de Desarrollo**: Completo

## Cómo Usar Esta Base de Conocimiento

### Para Nuevos Desarrolladores

1. **Empieza con**: [Estructura de Carpetas](estructura-carpetas.md) para entender la organización
2. **Luego lee**: [Stack Tecnológico](stack-tecnologico.md) para conocer las herramientas
3. **Después**: [Flujo de Desarrollo](flujo-desarrollo.md) para setup y desarrollo
4. **Finalmente**: [Arquitectura](arquitectura.md) para comprensión profunda del sistema

### Para Desarrollo Rápido

**Comandos esenciales**:
```bash
# Setup inicial
make setup

# Desarrollo
make codelayer-dev

# Checks y tests antes de PR
make check-test

# Ver estado
make dev-status
```

### Para Arquitectura y Diseño

Revisa [Arquitectura](arquitectura.md) para:
- Entender flujos de datos
- Conocer decisiones de diseño
- Ver patrones implementados
- Referencias de archivos clave

## Proyectos Principales

### CodeLayer (Local Tools Suite)

1. **hld** - Daemon Go que coordina aprobaciones y gestiona sesiones
2. **hlyr** - CLI TypeScript con servidor MCP
3. **humanlayer-wui** - Desktop UI (Tauri + React)
4. **claudecode-go** - SDK Go para Claude Code

### HumanLayer SDK & Platform (Legado)

SDKs removidos en PR #646. Documentación histórica disponible en `humanlayer.md`.

## Tecnologías Clave

- **Lenguajes**: TypeScript, Go 1.24, Rust 2021
- **Frontend**: React 19, Tauri 2, Vite
- **Backend**: Gin (Go), SQLite, Event Bus
- **Protocolos**: MCP, OpenAPI 3.1, JSON-RPC, SSE
- **Build**: Bun, Turbo, Make

## Flujo Arquitectónico

```
Claude Code → MCP → hlyr → JSON-RPC → hld → HumanLayer Cloud API
                                 ↑         ↑
                            TUI ─┘         └─ WUI
```

## Contacto y Recursos

- **Repositorio**: https://github.com/humanlayer/humanlayer
- **Documentación de Usuario**: `/docs`
- **Guía de Contribución**: `CONTRIBUTING.md`
- **Instrucciones Claude Code**: `CLAUDE.md`

---

**Última Actualización**: 2025-11-05 por Claude (Sonnet 4.5)
