# Feature de Creación de Sesiones - Documentación del Flujo Completo

**Creado**: 2025-11-08
**Repositorio**: humanlayer
**Estado**: Implementación Activa

---

## Descripción General

El feature de creación de sesiones en HumanLayer permite a los usuarios crear y lanzar sesiones de Claude Code mediante un flujo de trabajo basado en drafts (borradores). Las sesiones se crean primero como "drafts" que pueden ser editados, guardados y sincronizados en tiempo real antes de ser lanzados para iniciar la ejecución real.

### Características Clave

- **Diseño Draft-First**: Las sesiones se crean con estado "Draft" y se lanzan explícitamente
- **Sincronización en Tiempo Real**: Los cambios se auto-guardan al daemon cada 500ms mientras se edita
- **Creación Lazy**: Las sesiones draft solo se crean cuando el usuario comienza a escribir
- **Arquitectura Multi-Capa**: UI → HTTP API → Session Manager → Base de Datos SQLite → Proceso Claude Code
- **Ciclo de Vida con Estados**: Draft → Starting → Running → Completed/Failed/Interrupted

---

## Visión General de la Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                 Capa de Interacción del Usuario              │
│  (Componentes React: DraftSessionPage, DraftLauncherForm)   │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP/REST
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                      Capa HTTP API                           │
│         (hld/api/handlers/sessions.go)                      │
│  POST /api/v1/sessions, PATCH /api/v1/sessions/:id         │
│  POST /api/v1/sessions/:id/launch                           │
└────────────────────────┬────────────────────────────────────┘
                         │ Interfaz Go
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                 Capa Session Manager                         │
│              (hld/session/manager.go)                       │
│  LaunchSession(), LaunchDraftSession(),                     │
│  UpdateSessionSettings()                                     │
└────────────────────────┬────────────────────────────────────┘
                         │ Operaciones de BD
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Capa de Base de Datos                     │
│               (hld/store/sqlite.go)                         │
│  CreateSession(), UpdateSession(), tabla sessions           │
└─────────────────────────────────────────────────────────────┘
```

---

## Flujo Completo de Creación de Sesiones

### 1. Creación de Draft (UI → Base de Datos)

#### Puntos de Entrada

**Acciones del Usuario** (`humanlayer-wui/src/hooks/useSessionLauncher.ts:464-477`):
- Presionar tecla 'C' (hotkey global) → Navega a `/sessions/draft`
- Click en botón "Create" → Navega a `/sessions/draft`
- Navegación directa a URL `/sessions/draft?id={draft-id}` (para drafts existentes)

**Inicialización de Página** (`humanlayer-wui/src/pages/DraftSessionPage.tsx:25-73`):
- Si existe parámetro `?id`:
  - Obtiene draft existente del daemon vía `daemonClient.listSessions()`
  - Puebla `activeSessionDetail` en el store Zustand
  - Navega de vuelta a página de nuevo draft si no se encuentra el draft
- Si no hay `?id`:
  - Espera a que el usuario comience a escribir (creación lazy)
  - Registra evento PostHog `DRAFT_LAUNCHER_OPENED`

#### Trigger de Creación Lazy del Draft

**Cuando el Usuario Escribe** (`humanlayer-wui/src/components/internal/SessionDetail/components/DraftLauncherForm.tsx:208-240`):

Disparado por:
- Escribir en input de título (línea 370-371)
- Escribir en input de directorio de trabajo (línea 385-386)
- Escribir en editor de prompt (línea 243-259)

**Flujo de Creación**:
1. Verifica si el draft ya existe vía `draftCreatingRef` para prevenir race conditions (línea 210)
2. Establece flag de creación (línea 213)
3. **Llamada API** al daemon (líneas 217-221):
   ```typescript
   const response = await daemonClient.launchSession({
     query: '',
     working_dir: workingDirectoryRef.current,
     draft: true,
   })
   ```
4. Actualiza estado `sessionId` con el ID retornado (líneas 224-225)
5. Notifica al padre vía callback `onSessionUpdated` (líneas 228-230)

#### Procesamiento HTTP API

**Endpoint**: `POST /api/v1/sessions` (`hld/api/handlers/sessions.go:161-286`)

**Request Body**:
```json
{
  "query": "",
  "working_dir": "/path/to/project",
  "draft": true
}
```

**Flujo del Handler**:
1. Construye struct `session.LaunchSessionConfig` (líneas 163-248)
   - Establece `OutputFormat` a `claudecode.OutputStreamJSON`
   - Mapea enum de modelo: "opus" → `claudecode.ModelOpus`, etc.
   - Maneja configuración de proxy si está habilitado
   - Convierte tilde (`~`) a directorio home vía `expandTilde()` (líneas 61-76)

2. Llama al Session Manager (línea 253):
   ```go
   sess, err := h.manager.LaunchSession(ctx, config, isDraft)
   ```

3. Retorna respuesta (líneas 282-285):
   ```json
   {
     "data": {
       "session_id": "uuid-v4-string",
       "run_id": "uuid-v4-string"
     }
   }
   ```

#### Procesamiento del Session Manager

**Método**: `LaunchSession()` (`hld/session/manager.go:182-544`)

**Ejecución Paso a Paso**:

1. **Inicialización del Cliente Claude** (líneas 184-187)
   - Llama a `initializeClaudeClient()` que verifica path configurado o auto-detecta
   - Valida que el binario es ejecutable
   - Retorna error si Claude no está disponible

2. **Generación de IDs** (líneas 189-190)
   ```go
   sessionID := uuid.New().String()
   runID := uuid.New().String()
   ```

3. **Inyección de Configuración MCP** (líneas 193-263)
   - Inyecta servidor MCP CodeLayer:
     - Command: `hldconfig.DefaultCLICommand`
     - Args: `["mcp", "claude_approvals"]`
     - Env: `HUMANLAYER_SESSION_ID`, `HUMANLAYER_DAEMON_SOCKET`
   - Configura servidores MCP del usuario:
     - Servidores HTTP: Inyecta header `X-Session-ID`
     - Servidores Stdio: Inyecta variable env `HUMANLAYER_RUN_ID`
   - Establece tool de permission prompt a `"mcp__codelayer__request_permission"` si está vacío

4. **Validación de Directorio de Trabajo** (líneas 266-325)
   - **Para Drafts**: Omite validación (línea 278)
   - **Para No-Drafts**:
     - Expande `~` a directorio home
     - Verifica que el directorio existe
     - Crea si `CreateDirectoryIfNotExists=true`
     - Retorna `DirectoryNotFoundError` si falta y la creación está deshabilitada

5. **Creación de Sesión en Base de Datos** (líneas 328-379)
   - Crea `store.Session` vía `store.NewSessionFromConfig()`
   - Calcula resumen (normaliza espacios en blanco, trunca a 50 caracteres)
   - Establece estado inicial:
     - Draft: `store.SessionStatusDraft` (línea 336)
     - No-draft: `store.SessionStatusStarting` (línea 338)
   - Guarda sesión con `m.store.CreateSession()`
   - Guarda servidores MCP en base de datos

6. **Retorno Temprano de Draft** (líneas 425-441)
   - Si `isDraft=true`:
     - Registra creación de draft
     - Retorna struct `Session` con estado `StatusDraft`
     - **NO lanza proceso Claude**

#### Persistencia en Base de Datos

**Método**: `CreateSession()` (`hld/store/sqlite.go:1175-1202`)

**SQL Insert**: Inserta 31 campos en la tabla sessions:
```sql
INSERT INTO sessions (
  id, run_id, claude_session_id, parent_session_id,
  query, summary, title, model, model_id, working_dir,
  max_turns, system_prompt, append_system_prompt,
  custom_instructions, permission_prompt_tool,
  allowed_tools, disallowed_tools, status,
  created_at, last_activity_at, auto_accept_edits,
  archived, dangerously_skip_permissions,
  dangerously_skip_permissions_expires_at,
  dangerously_skip_permissions_timeout_ms,
  proxy_enabled, proxy_base_url, proxy_model_override,
  proxy_api_key, additional_directories, editor_state
) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
```

**Valores Iniciales para Draft**:
- `id`: UUID generado
- `run_id`: UUID generado
- `query`: String vacío (`""`)
- `status`: `"draft"`
- `working_dir`: Path proporcionado por usuario
- `created_at`: `CURRENT_TIMESTAMP`
- `last_activity_at`: `CURRENT_TIMESTAMP`
- Todos los demás campos: NULL o valores por defecto

**Resultado**: La sesión draft ahora existe en la base de datos y puede ser editada.

---

### 2. Edición de Draft y Auto-Sincronización

#### Flujo de Edición en Tiempo Real

**Componente Editor** (`humanlayer-wui/src/components/internal/SessionDetail/components/DraftLauncherInput.tsx`)

**Inicialización del Editor TipTap** (líneas 74-84):
- Carga estado inicial desde `session.editorState` si está disponible
- Parsea estado JSON del editor
- Usa editor vacío si hay error de parseo

**Cambios de Contenido** (líneas 87-111):
- `handleChange` recibe actualizaciones de contenido TipTap
- Para drafts existentes, guarda directamente al daemon:
  ```typescript
  await daemonClient.updateSession(session.id, {
    editorState: valueStr,
  })
  ```

#### Estrategia de Sincronización con Debounce

**Implementación de Sync** (`humanlayer-wui/src/components/internal/SessionDetail/components/DraftLauncherForm.tsx:264-338`)

**Sync con Debounce** (líneas 307-322):
- Timer de debounce de 500ms (línea 321)
- Limpia timer previo y establece uno nuevo en cada cambio
- Solo sincroniza si existe `sessionId`

**Sync Inmediato** (líneas 264-304):
- Usado cuando se navega fuera (tecla escape)
- Recopila todos los valores del formulario:
  - `title`, `workingDir`, `editorState`
  - `autoAcceptEdits`, `dangerouslySkipPermissions`
  - `model`, `proxyEnabled`, `proxyBaseUrl`, `proxyModelOverride`

**Llamada API**:
```typescript
await daemonClient.updateSession(currentSessionId, {
  title: titleRef.current,
  working_dir: workingDirectoryRef.current,
  auto_accept_edits: defaultAutoAcceptEditsSetting,
  dangerously_skip_permissions: defaultDangerouslyBypassPermissionsSetting,
  editor_state: editorStateJson,
  model,
  proxy_enabled: proxyEnabled,
  proxy_base_url: proxyBaseUrl,
  proxy_model_override: proxyModelOverride,
})
```

**Triggers de Auto-Sync** (líneas 325-338):
- Cualquier cambio a variables de estado tracked dispara sync con debounce:
  - `sessionId`, `title`, `workingDirectory`
  - `defaultAutoAcceptEditsSetting`, `defaultDangerouslyBypassPermissionsSetting`
  - `model`, `proxyEnabled`, `proxyBaseUrl`, `proxyModelOverride`

#### Procesamiento de Actualización HTTP API

**Endpoint**: `PATCH /api/v1/sessions/:id` (`hld/api/handlers/sessions.go:467-697`)

**Flujo del Handler**:

1. **Construir Struct SessionUpdate** (líneas 471-613)
   - Crea `store.SessionUpdate` con solo los campos especificados
   - Valida transiciones de estado (draft ↔ discarded solamente)
   - Serializa `additional_directories` a string JSON

2. **Llamar al Session Manager** (línea 615):
   ```go
   err := h.manager.UpdateSessionSettings(ctx, sessionID, update)
   ```

3. **Obtener Sesión Actualizada** (líneas 676-691)
   - Recupera sesión actualizada de la base de datos
   - Usa mapper para convertir a tipo API
   - Retorna detalles completos de sesión

#### Actualización del Session Manager

**Método**: `UpdateSessionSettings()` (referenciado pero delega al store)

**Actualización en Base de Datos** (`hld/store/sqlite.go:1205-1363`):

**Constructor de Query Dinámico**:
- Solo actualiza campos especificados en struct `SessionUpdate`
- Construye cláusulas SET condicionalmente (líneas 1210-1338)
- Ejemplo para título:
  ```go
  if updates.Title != nil {
      setParts = append(setParts, "title = ?")
      args = append(args, *updates.Title)
  }
  ```
- No-op si no se proporcionan campos (líneas 1340-1343)

**Ejecución SQL**:
```sql
UPDATE sessions SET
  title = ?,
  working_dir = ?,
  editor_state = ?,
  auto_accept_edits = ?,
  model = ?,
  last_activity_at = ?
WHERE id = ?
```

**Resultado**: Sesión draft continuamente persistida a base de datos mientras el usuario edita.

---

### 3. Lanzamiento de Draft (Ejecutar Sesión)

#### Triggers de Lanzamiento

**Acciones del Usuario** (`humanlayer-wui/src/components/internal/SessionDetail/components/DraftLauncherForm.tsx`):
- Click en botón "Launch" (`DraftLauncherInput.tsx:449-460`)
- Presionar hotkey `Cmd+Enter` o `Ctrl+Enter` (`DraftLauncherInput.tsx:332-344`)

Ambos llaman a `handleSubmit` → callback `onLaunchDraft`

#### Flujo de Lanzamiento (UI)

**Método**: `handleLaunchDraft()` (`DraftLauncherForm.tsx:442-558`)

**Paso a Paso**:

1. **Validación** (líneas 447-470)
   - Verifica que existe la sesión
   - Valida que directorio de trabajo no está vacío
   - Valida que el prompt no está vacío

2. **Preparación de Configuraciones** (líneas 473-499)
   - Construye payload de actualización con:
     - `autoAcceptEdits`
     - `dangerouslySkipPermissions`
     - `dangerouslySkipPermissionsTimeoutMs` (si bypass habilitado)
     - API keys específicas del proveedor (Baseten, OpenRouter)

3. **Actualización Pre-Lanzamiento** (línea 501):
   ```typescript
   await daemonClient.updateSession(sessionId, updatePayload)
   ```

4. **Request de Lanzamiento** (línea 504):
   ```typescript
   await daemonClient.launchDraftSession(
     sessionId,
     currentPrompt,
     createDirectoryIfNotExists
   )
   ```

5. **Acciones Post-Lanzamiento** (líneas 506-529)
   - Registra evento PostHog `SESSION_CREATED`
   - Guarda directorio de trabajo a localStorage
   - Limpia contenido del editor y localStorage
   - Refresca lista de sesiones
   - Establece modo de vista a `ViewMode.Normal`
   - **Navega a `/sessions/{sessionId}`**

6. **Manejo de Errores** (líneas 530-544)
   - Captura error 422 de directorio no encontrado
   - Muestra diálogo de creación de directorio
   - Otros errores muestran notificación toast

#### Procesamiento de Lanzamiento HTTP API

**Endpoint**: `POST /api/v1/sessions/:id/launch` (`hld/api/handlers/sessions.go:886-1008`)

**Request Body**:
```json
{
  "prompt": "Texto del prompt del usuario",
  "create_directory_if_not_exists": false
}
```

**Flujo del Handler**:

1. **Obtener y Validar Sesión** (líneas 888-923)
   - Recupera sesión del store
   - Valida que el estado es "draft"
   - Retorna error 400 si no está en estado draft

2. **Recálculo de Timer de Bypass Permissions** (líneas 925-950)
   - Si bypass permissions habilitado con timeout:
     - Recalcula expiración desde **tiempo actual** (no timestamp original)
     - Actualiza sesión vía `store.UpdateSession()`
   - Asegura que el timer comienza cuando la sesión realmente se lanza

3. **Llamar al Session Manager** (línea 959):
   ```go
   err = h.manager.LaunchDraftSession(
     ctx,
     sessionID,
     req.Body.Prompt,
     createDirectoryIfNotExists
   )
   ```

4. **Manejo de Errores** (líneas 961-983)
   - Verifica `DirectoryNotFoundError` → Retorna 422
   - Otros errores → Retorna 500 con código "HLD-4005"

5. **Obtener Sesión Actualizada** (líneas 987-1002)
   - Recupera sesión después del lanzamiento
   - Retorna sesión completa con estado actualizado

#### Procesamiento de Lanzamiento del Session Manager

**Método**: `LaunchDraftSession()` (`hld/session/manager.go:2077-2154`)

**Ejecución Paso a Paso**:

1. **Recuperar Sesión** (líneas 2078-2082)
   - Llama a `m.store.GetSession(ctx, sessionID)`

2. **Validar Estado Draft** (líneas 2084-2087)
   - Verifica `sess.Status == store.SessionStatusDraft`
   - Retorna error si no está en estado draft

3. **Validación de Directorio de Trabajo** (líneas 2089-2137)
   - **Ocurre ANTES de actualización de estado** (mantiene sesión en draft si falla)
   - Expande `~` a directorio home
   - Verifica existencia del directorio
   - Crea si `createDirectoryIfNotExists=true`
   - Retorna `DirectoryNotFoundError` si falta y la creación está deshabilitada

4. **Actualizar Query y Estado de Sesión** (líneas 2139-2154)
   - Actualiza vía `store.SessionUpdate`:
     - `Status`: `StatusStarting`
     - `Query`: Nuevo parámetro prompt
     - `Summary`: Calculado desde el prompt
     - `LastActivityAt`: Tiempo actual
     - `EditorState`: String vacío (limpia estado del editor draft)

5. **Reconstruir Configuración Claude** (líneas 2159-2237)
   - Crea `claudecode.SessionConfig` desde sesión almacenada
   - Deserializa arrays JSON (allowed_tools, disallowed_tools, additional_directories)
   - Recupera servidores MCP de base de datos
   - Reconstruye configuraciones de servidores HTTP y stdio

6. **Construir Configuración de Lanzamiento** (líneas 2239-2258)
   - Crea `LaunchSessionConfig` con:
     - `SessionConfig` reconstruido
     - Title, AutoAcceptEdits, DangerouslySkipPermissions
     - Configuraciones de proxy
     - Timeout restante de bypass permissions

7. **Lanzar con IDs Existentes** (línea 2262)
   - Llama a `m.launchDraftWithConfig(ctx, sessionID, sess.RunID, launchConfig)`
   - Usa session ID y run ID **existentes** (no UUIDs nuevos)

#### Método de Lanzamiento Interno

**Método**: `launchDraftWithConfig()` (`hld/session/manager.go:1917-2073`)

**Ejecución Paso a Paso**:

1. **Inicialización del Cliente Claude** (líneas 1918-1922)
   - Igual que paso 1 de `LaunchSession()`

2. **Inyección de Configuración MCP** (líneas 1924-1974)
   - Inyecta servidor MCP CodeLayer
   - Configura servidores MCP HTTP/stdio
   - Establece tool de permission prompt

3. **Configuración de Proxy** (líneas 1976-1990)
   - Establece `ANTHROPIC_BASE_URL` al endpoint proxy del daemon
   - Formato: `http://localhost:{httpPort}/api/v1/anthropic_proxy/{sessionID}`
   - Establece `ANTHROPIC_API_KEY="proxy-handled"` dummy

4. **Lanzamiento de Sesión Claude** (líneas 1992-2006)
   - Registra configuración de lanzamiento
   - Llama a `client.Launch(claudeConfig)`
   - En caso de falla:
     - Registra error
     - Actualiza estado a `StatusFailed`
     - Retorna error

5. **Almacenamiento de Proceso y Actualización de Estado** (líneas 2008-2040)
   - Envuelve sesión en `ClaudeSessionWrapper`
   - Almacena en mapa `m.activeProcesses` con clave session ID
   - Actualiza estado de base de datos a `StatusRunning`
   - Publica evento `EventSessionStatusChanged`:
     - Estado anterior: `StatusStarting`
     - Estado nuevo: `StatusRunning`

6. **Setup de Query y Monitoreo** (líneas 2042-2066)
   - Almacena query en `m.pendingQueries` para inyección posterior
   - Lanza goroutine `monitorSession()`
   - Programa reconciliación de aprobaciones después de 2 segundos

7. **Retornar Éxito** (líneas 2068-2073)
   - Registra éxito
   - Retorna `nil`

---

### 4. Monitoreo y Ciclo de Vida de Sesión

#### Goroutine de Monitoreo

**Método**: `monitorSession()` (`hld/session/manager.go:548-757`)

**Loop de Procesamiento de Eventos** (líneas 552-633):
- Escucha en canal `claudeSession.GetEvents()`
- Verifica cancelación de contexto antes de cada operación
- Serializa y almacena JSON de evento raw para debugging
- Captura ID de sesión Claude desde eventos
- Actualiza base de datos con ID de sesión Claude
- Inyecta query pendiente como primer evento de conversación
- Procesa cada evento vía `processStreamEvent()`

**Inyección de Query** (líneas 616-625):
- Llama a `injectQueryAsFirstEvent()` cuando se captura ID de sesión Claude
- Verifica deduplicación:
  - Si primer evento ya es mensaje de usuario: omite inyección
- Crea `ConversationEvent`:
  - Sequence: 1
  - EventType: `EventTypeMessage`
  - Role: "user"
  - Content: query

**Manejo de Completitud de Sesión** (líneas 635-757):
- Espera a que `claudeSession.Wait()` retorne resultado
- Verifica interrupción intencional:
  - Si estado es `StatusInterrupting`: Actualiza a `StatusInterrupted`
- Maneja casos de falla:
  - Error de Wait() o `result.IsError=true`
  - Actualiza a `StatusFailed` vía `updateSessionStatus()`
- Maneja completitud exitosa:
  - Actualiza a `StatusCompleted`
  - Almacena datos de resultado: `CostUSD`, `DurationMS`, `NumTurns`, `ResultContent`
  - Publica evento de completitud
- Limpieza:
  - Remueve de mapa `activeProcesses`
  - Elimina de `pendingQueries`

#### Resumen de Transiciones de Estado

**Ciclo de Vida Completo**:
```
Draft → Starting → Running → Completed/Failed/Interrupted
                              ↑
                              └─ Interrupting (intermedio)
```

**Flujos Alternativos**:
- Draft → Discarded (usuario cancela sin lanzar)
- Starting → Failed (error de lanzamiento)
- Running → Interrupted (usuario envía SIGINT)

---

## Esquema de Base de Datos

### Tabla Sessions

**Ubicación**: `hld/store/sqlite.go:83-125` (esquema inicial) + migraciones

**Columnas Centrales**:

**Identidad y Relaciones**:
- `id TEXT PRIMARY KEY` - Identificador único de sesión
- `run_id TEXT NOT NULL UNIQUE` - HumanLayer run ID
- `claude_session_id TEXT` - ID de sesión Claude API (poblado después del lanzamiento)
- `parent_session_id TEXT` - Para cadenas/reanudación de sesiones

**Configuración de Lanzamiento**:
- `query TEXT NOT NULL` - Query/prompt inicial del usuario
- `summary TEXT` - Resumen de sesión generado (máx 50 caracteres)
- `title TEXT DEFAULT ''` - Título de sesión editable por usuario (Migración 10)
- `model TEXT` - Nombre de display del modelo ("opus", "sonnet", "haiku")
- `model_id TEXT` - Identificador completo del modelo (Migración 12)
- `working_dir TEXT` - Path del directorio de trabajo
- `max_turns INTEGER` - Turnos máximos de conversación
- `system_prompt TEXT`, `append_system_prompt TEXT` - Prompts del sistema
- `custom_instructions TEXT` - Instrucciones proporcionadas por usuario
- `permission_prompt_tool TEXT` - Tool MCP para permisos
- `allowed_tools TEXT`, `disallowed_tools TEXT` - Arrays JSON
- `additional_directories TEXT` - Array JSON para --add-dir (Migración 11)

**Estado de Runtime**:
- `status TEXT NOT NULL DEFAULT 'starting'` - Estado del ciclo de vida
- `created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP`
- `last_activity_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP`
- `completed_at TIMESTAMP` - Timestamp de completitud

**Resultados y Métricas**:
- `cost_usd REAL` - Costo total en USD
- `duration_ms INTEGER` - Duración total de sesión
- `num_turns INTEGER` - Número de turnos de conversación
- `result_content TEXT` - Resultado/salida final
- `error_message TEXT` - Mensaje de error si falló
- `input_tokens INTEGER`, `output_tokens INTEGER` (Migración 13)
- `cache_creation_input_tokens INTEGER`, `cache_read_input_tokens INTEGER` (Migración 13)
- `effective_context_tokens INTEGER` (Migración 13)

**Configuraciones de Sesión**:
- `auto_accept_edits BOOLEAN DEFAULT 0` - Auto-aprobar ediciones de archivos
- `dangerously_skip_permissions BOOLEAN DEFAULT 0` - Bypass de todos los permisos
- `dangerously_skip_permissions_expires_at TIMESTAMP` - Expiración de bypass de permisos
- `dangerously_skip_permissions_timeout_ms INTEGER` - Duración para bypass (Migración 21)

**Configuración de Proxy** (Migración 15):
- `proxy_enabled BOOLEAN DEFAULT 0`
- `proxy_base_url TEXT DEFAULT ''`
- `proxy_model_override TEXT DEFAULT ''`
- `proxy_api_key TEXT DEFAULT ''`

**Estado de UI**:
- `archived BOOLEAN DEFAULT FALSE` - Flag de soft delete
- `editor_state TEXT` - Blob JSON para persistencia de editor draft (Migración 20)

**Índices**:
- `PRIMARY KEY` en `id`
- Constraint `UNIQUE` en `run_id`
- `idx_sessions_claude` - En `claude_session_id`
- `idx_sessions_status` - En `status`
- `idx_sessions_run_id` - En `run_id`
- `idx_sessions_parent` - En `parent_session_id` (Migración 5)
- `idx_sessions_archived` - En `archived` (Migración 9)
- `idx_sessions_title` - Índice B-tree para queries LIKE (Migración 22)
- `idx_sessions_last_activity` - Índice descendente para ordenamiento (Migración 22)

### Tablas Relacionadas

#### conversation_events (`sqlite.go:132-167`)
- Almacena todos los mensajes de conversación, llamadas a tools y resultados de tools
- Foreign key: `session_id` → `sessions(id)`
- Columnas: `sequence`, `event_type`, `role`, `content`, `tool_id`, `tool_name`, `tool_input_json`, `approval_status`, `approval_id`

#### approvals (`sqlite.go:201-220`)
- Rastrea solicitudes de aprobación locales para tools
- Foreign key: `session_id` → `sessions(id)`
- Columnas: `id`, `run_id`, `status`, `tool_name`, `tool_input`, `comment`, `tool_use_id` (Migración 14)

#### mcp_servers (`sqlite.go:169-180`)
- Almacena configuraciones de servidor MCP por sesión
- Foreign key: `session_id` → `sessions(id)`
- Columnas: `name`, `command`, `args_json`, `env_json`

#### raw_events (`sqlite.go:182-191`)
- Almacenamiento de debug para JSON de eventos raw
- Foreign key: `session_id` → `sessions(id)`
- Columnas: `event_json`, `created_at`

#### file_snapshots (Migración 7, `sqlite.go:425-439`)
- Rastrea lecturas de archivos para generación de diffs
- Foreign key: `session_id` → `sessions(id)`
- Columnas: `tool_id`, `file_path`, `content`, `created_at`

---

## Patrones de Diseño Clave

### 1. Arquitectura Draft-First

**Justificación**: Permite a los usuarios preparar sesiones sin comprometerse a la ejecución.

**Beneficios**:
- Usuario puede configurar todas las opciones antes del lanzamiento
- Estado del editor persistido a través de refrescos de página
- Múltiples drafts pueden existir simultáneamente
- Lanzamientos fallidos no pierden input del usuario

**Implementación**:
- Flag draft previene lanzamiento de proceso Claude en `LaunchSession()`
- Estado "draft" de base de datos distingue de sesiones activas
- Método separado `LaunchDraftSession()` para lanzamiento diferido

### 2. Creación Lazy con Sync con Debounce

**Justificación**: Minimizar escrituras innecesarias a base de datos mientras se asegura seguridad de datos.

**Beneficios**:
- No hay registro en BD hasta que usuario muestra intención (escribir)
- Reduce carga en daemon para navegaciones accidentales
- Debounce de 500ms reduce frecuencia de escritura
- Sync inmediato al navegar fuera asegura no pérdida de datos

**Implementación**:
- `handleCreateDraft()` solo llamado en primer input
- `syncToDaemon()` usa timer con delay de 500ms
- `syncImmediately()` bypasea timer para eventos de navegación

### 3. Validación en Múltiples Capas

**Justificación**: Fallar rápido con mensajes de error claros en la capa apropiada.

**Capas**:
- **Capa UI**: Validación básica (prompt no vacío, directorio de trabajo)
- **Capa API**: Validación de esquema, verificaciones de transición de estado
- **Capa Manager**: Existencia de directorio, disponibilidad de Claude
- **Capa Base de Datos**: Constraints de foreign key, constraints CHECK

**Estrategia de Validación de Directorio**:
- Drafts: Omitir validación (usuario puede crear directorio después)
- Lanzamiento de Draft: Validar antes de actualización de estado (mantiene sesión en draft si falla)
- Lanzamiento Directo: Validar antes de creación en base de datos

### 4. Actualizaciones de Estado Manejadas por Eventos

**Justificación**: Mantener UI en sincronía con backend sin polling.

**Implementación**:
- Session Manager publica `EventSessionStatusChanged` al event bus
- UI se suscribe a eventos para actualizaciones en tiempo real
- Cambios de estado: Draft → Starting → Running → Completed/Failed/Interrupted
- Base de datos actualizada antes de que el evento sea publicado para consistencia

### 5. Inyección de Configuración de Servidor MCP

**Justificación**: Habilitar flujo de aprobación sin setup manual de MCP.

**Implementación**:
- Servidor MCP CodeLayer siempre inyectado (sobrescribe si existe)
- Variables de entorno inyectadas:
  - `HUMANLAYER_SESSION_ID` para correlación de sesión
  - `HUMANLAYER_DAEMON_SOCKET` para comunicación RPC
- Tool de permission prompt auto-configurado a `mcp__codelayer__request_permission`

### 6. Preservación de Session ID

**Justificación**: Draft y sesión lanzada son la misma entidad.

**Implementación**:
- Creación de draft genera UUID inmediatamente
- `LaunchDraftSession()` reutiliza session ID y run ID existentes
- Registro de base de datos actualizado in-place (estado, query, resumen)
- UI navega al mismo session ID después del lanzamiento

### 7. Configuración de Proxy

**Justificación**: Habilitar tracking de costos y override de modelo sin cambios de código.

**Implementación**:
- Proxy opcional habilitado vía checkbox de UI
- Daemon intercepta llamadas API de Claude vía override de `ANTHROPIC_BASE_URL`
- Session ID incluido en URL de proxy para correlación de request
- API key dummy establecida (key real manejada por daemon)

---

## Referencias de Código

### Capa UI
- Entrada: `humanlayer-wui/src/pages/DraftSessionPage.tsx:25-73`
- Formulario: `humanlayer-wui/src/components/internal/SessionDetail/components/DraftLauncherForm.tsx:208-240` (creación), `442-558` (lanzamiento)
- Editor: `humanlayer-wui/src/components/internal/SessionDetail/components/DraftLauncherInput.tsx:74-84` (init), `87-111` (cambios)
- Sync: `humanlayer-wui/src/components/internal/SessionDetail/components/DraftLauncherForm.tsx:264-338`

### Capa HTTP API
- Crear: `hld/api/handlers/sessions.go:161-286`
- Actualizar: `hld/api/handlers/sessions.go:467-697`
- Lanzar: `hld/api/handlers/sessions.go:886-1008`
- Eliminar: `hld/api/handlers/sessions.go:717-784` (soft), `787-866` (hard)

### Capa Session Manager
- Lanzar: `hld/session/manager.go:182-544` (directo), `2077-2154` (draft)
- Interno: `hld/session/manager.go:1917-2073` (impl de lanzamiento de draft)
- Monitorear: `hld/session/manager.go:548-757`
- Actualizar: Delega al store

### Capa Base de Datos
- Crear: `hld/store/sqlite.go:1175-1202`
- Actualizar: `hld/store/sqlite.go:1205-1363`
- Esquema: `hld/store/sqlite.go:83-125` (inicial), migraciones 3-22 (evolución)

---

## Manejo de Errores

### Directorio No Encontrado (422)

**Trigger**: El directorio de trabajo no existe al lanzar.

**Flujo**:
1. Session Manager verifica directorio con `os.Stat()`
2. Retorna `DirectoryNotFoundError` si falta
3. Handler API detecta tipo de error con `errors.As()`
4. Retorna respuesta 422:
   ```json
   {
     "error": "directory_not_found",
     "message": "Directory does not exist",
     "path": "/path/to/directory",
     "requires_creation": true
   }
   ```
5. UI muestra componente `CreateDirectoryDialog`
6. Usuario puede reintentar lanzamiento con `createDirectoryIfNotExists: true`

### Sesión No en Estado Draft (400)

**Trigger**: Intentar lanzar una sesión que no es draft.

**Flujo**:
1. `LaunchDraftSession()` verifica `sess.Status == store.SessionStatusDraft`
2. Retorna error si la verificación falla
3. API retorna 400 con código "HLD-4001"
4. UI muestra mensaje de error toast

### Claude No Disponible (500)

**Trigger**: Binario de Claude no encontrado o no ejecutable.

**Flujo**:
1. `initializeClaudeClient()` intenta auto-detección
2. Verifica `$PATH` para ejecutable `claude`
3. Valida permisos con `IsExecutable()`
4. Almacena error en `m.claudeClientErr`
5. Retorna error en llamada a `getClaudeClient()`
6. API retorna 500 con código "HLD-1001"
7. Endpoint de health retorna estado "degraded"

---

## Consideraciones de Testing

### Testing Unitario

**Session Manager**:
- Mock de interfaz `ClaudeSession` para tests de lanzamiento
- Mock de interfaz `store.ConversationStore` para tests de base de datos
- Testear transiciones de estado independientemente
- Testear lógica de inyección MCP
- Testear casos edge de validación de directorio

**Handlers API**:
- Usar `httptest.NewRecorder()` para testing de respuestas
- Mock de interfaz `SessionManager`
- Testear detección de tipo de error (`DirectoryNotFoundError`)
- Testear validación de requests

**Base de Datos**:
- Usar SQLite en memoria (`:memory:`)
- Testear migraciones independientemente
- Testear constraints de FOREIGN KEY
- Testear que índices mejoran performance de queries

### Testing de Integración

**UI → API**:
- Usar Playwright o Cypress
- Testear flujo de creación de draft end-to-end
- Testear comportamiento de auto-sync con delays de red
- Testear diálogo de creación de directorio

**API → Manager**:
- Base de datos SQLite real con test fixtures
- Mock de cliente Claude para evitar spawning de procesos
- Testear ciclo de vida completo de draft
- Testear creación concurrente de sesiones

**Full Stack**:
- Daemon real con modo test
- Mock de respuestas Claude o usar test fixtures
- Testear monitoreo de sesión y publicación de eventos
- Testear limpieza al completar sesión

---

## Consideraciones de Performance

### Índices de Base de Datos

**Críticos para Performance**:
- `idx_sessions_status` - Filtrar sesiones activas vs completadas
- `idx_sessions_last_activity` - Ordenar por actividad reciente (descendente)
- `idx_sessions_title` - Búsqueda fuzzy en títulos
- Índices parciales en `approvals` - Filtrar aprobaciones pendientes eficientemente

### Estrategia de Debouncing

**Sync del Editor**:
- Debounce de 500ms reduce escrituras a BD en ~80%
- Sync inmediato en navegación asegura no pérdida de datos
- Refs previenen re-renders innecesarios durante tipeo

### Event Bus

**Publish-Subscribe**:
- Cambios de estado publicados una vez al event bus
- Múltiples suscriptores reciben evento simultáneamente
- UI actualiza sin polling (conexión WebSocket en producción)

### Optimización de Queries

**Lista de Sesiones**:
- Usa índice en `archived` para filtrado
- Usa índice en `last_activity_at` para ordenamiento
- Limita query a sesiones recientes (paginación)

---

## Mejoras Futuras

### Discutidas pero No Implementadas

1. **Templates de Sesión**
   - Guardar configuraciones comunes como templates
   - Lanzamiento rápido desde biblioteca de templates

2. **Expiración de Drafts**
   - Auto-eliminar drafts después de N días de inactividad
   - Política de retención configurable

3. **Drafts Colaborativos**
   - Compartir drafts entre miembros del equipo
   - Edición colaborativa en tiempo real

4. **Versionado de Drafts**
   - Rastrear cambios a drafts a lo largo del tiempo
   - Restaurar versiones previas

5. **Clonación de Sesiones**
   - Duplicar configuración de sesión existente
   - Lanzar nueva sesión con mismas configuraciones

---

## Glosario

**Draft**: Sesión creada pero no lanzada aún. Estado: "draft", sin proceso Claude.

**Launch**: Transición de estado draft a running. Inicia proceso Claude Code.

**Run ID**: Identificador HumanLayer para rastrear aprobaciones y eventos.

**Session ID**: Identificador del daemon para la entidad de sesión.

**Claude Session ID**: Identificador interno de sesión de Claude (poblado después del lanzamiento).

**MCP**: Model Context Protocol - Interfaz para que Claude interactúe con tools externos.

**CodeLayer**: Servidor MCP interno para integración de flujo de aprobación.

**Proxy Mode**: Daemon intercepta llamadas API de Claude para tracking de costos y override de modelo.

**Auto-Accept Edits**: Configuración para aprobar automáticamente tools de edición de archivos sin revisión humana.

**Bypass Permissions**: Configuración para aprobar automáticamente TODOS los tools sin revisión humana (peligroso).

**Editor State**: Serialización JSON de contenido del editor TipTap para persistencia de draft.

---

## Ver También

- `thoughts/shared/knowledge/architecture.md` - Arquitectura general del sistema
- `thoughts/shared/knowledge/features/approval-workflow.md` - Flujo de aprobación human-in-the-loop
- `thoughts/shared/knowledge/testing.md` - Estrategias y patrones de testing
- `thoughts/shared/knowledge/database.md` - Esquema de base de datos y migraciones
