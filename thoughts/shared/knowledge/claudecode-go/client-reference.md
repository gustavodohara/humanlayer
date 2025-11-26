# Cliente claudecode-go - Referencia Completa

**Creado**: 2025-11-08
**Ubicación**: `claudecode-go/client.go`, `claudecode-go/types.go`
**Propósito**: SDK en Go para lanzar y gestionar sesiones de Claude Code programáticamente

---

## Descripción General

El paquete `claudecode-go` proporciona un cliente Go para lanzar e interactuar con Claude Code CLI de forma programática. Permite:

- Lanzar sesiones de Claude Code con configuración completa
- Obtener eventos en tiempo real vía streaming JSON
- Gestionar el ciclo de vida de sesiones (interrumpir, terminar, esperar)
- Configurar servidores MCP para herramientas personalizadas
- Reanudar o bifurcar sesiones existentes

---

## Estructura del Cliente

### Client

**Definición** (`client.go:44-46`):
```go
type Client struct {
    claudePath string
}
```

**Campo**:
- `claudePath`: Ruta al binario ejecutable de Claude Code CLI

---

## Métodos de Inicialización

### NewClient()

**Ubicación**: `client.go:67-102`

**Firma**:
```go
func NewClient() (*Client, error)
```

**Descripción**: Crea un nuevo cliente buscando automáticamente el binario de Claude.

**Estrategia de Búsqueda** (en orden):
1. **PATH del sistema**: Usa `exec.LookPath("claude")`
2. **Rutas comunes**:
   - `~/.claude/local/claude` - Directorio propio de Claude
   - `~/.npm/bin/claude` - Instalación npm
   - `~/.bun/bin/claude` - Instalación bun
   - `~/.local/bin/claude` - Binarios locales
   - `/usr/local/bin/claude` - Binarios globales
   - `/opt/homebrew/bin/claude` - Homebrew en macOS
3. **Login shell**: Intenta `zsh -lc "which claude"` o `bash -lc "which claude"`

**Validaciones**:
- Omite rutas en `node_modules/`
- Omite archivos `.bak`
- Verifica que el archivo es ejecutable (`mode & 0111`)

**Retorna**:
- `*Client`: Cliente inicializado
- `error`: "claude binary not found in PATH or common locations" si no se encuentra

### NewClientWithPath()

**Ubicación**: `client.go:105-109`

**Firma**:
```go
func NewClientWithPath(claudePath string) *Client
```

**Descripción**: Crea un cliente con una ruta específica al binario de Claude.

**Parámetros**:
- `claudePath`: Ruta completa al ejecutable de Claude

**Uso**: Cuando se conoce la ubicación exacta del binario o para testing.

---

## Métodos del Cliente

### GetPath()

**Ubicación**: `client.go:112-114`

**Firma**:
```go
func (c *Client) GetPath() string
```

**Descripción**: Retorna la ruta al binario de Claude configurado.

### GetVersion()

**Ubicación**: `client.go:117-147`

**Firma**:
```go
func (c *Client) GetVersion() (string, error)
```

**Descripción**: Ejecuta `claude --version` y retorna la versión.

**Detalles de Implementación**:
- Timeout de 5 segundos para prevenir bloqueos
- Captura stdout y stderr
- Trim de espacios en blanco
- Validación de salida no vacía

**Errores Posibles**:
- "claude path not set"
- "claude --version timed out after 5 seconds"
- "claude --version failed with exit code X: stderr"
- "claude --version returned empty output"

### Launch()

**Ubicación**: `client.go:314-419`

**Firma**:
```go
func (c *Client) Launch(config SessionConfig) (*Session, error)
```

**Descripción**: Inicia una sesión de Claude de forma asíncrona y retorna inmediatamente.

**Proceso**:
1. Construye argumentos CLI desde `SessionConfig`
2. Configura proceso `exec.Command`
3. Establece variables de entorno
4. Establece directorio de trabajo
5. Crea pipes para stdout/stderr
6. Inicia proceso
7. Lanza goroutines para parsear salida
8. Retorna `Session` inmediatamente

**Formatos de Salida Soportados**:
- `OutputStreamJSON`: Parsea eventos línea por línea (`parseStreamingJSON`)
- `OutputJSON`: Parsea resultado JSON único (`parseSingleJSON`)
- `OutputText` (default): Captura texto plano (`parseTextOutput`)

**Retorna**:
- `*Session`: Sesión activa con canal de eventos
- `error`: Si falla la inicialización

### LaunchAndWait()

**Ubicación**: `client.go:422-429`

**Firma**:
```go
func (c *Client) LaunchAndWait(config SessionConfig) (*Result, error)
```

**Descripción**: Lanza una sesión y espera a que complete, retornando el resultado.

**Equivalente a**:
```go
session, err := client.Launch(config)
if err != nil {
    return nil, err
}
return session.Wait()
```

---

## SessionConfig - Configuración de Sesión

**Definición** (`types.go:50-73`):
```go
type SessionConfig struct {
    // Requerido
    Query string

    // Gestión de sesión
    SessionID   string
    ForkSession bool

    // Opcional
    Model                 Model
    OutputFormat          OutputFormat
    MCPConfig             *MCPConfig
    PermissionPromptTool  string
    WorkingDir            string
    MaxTurns              int
    SystemPrompt          string
    AppendSystemPrompt    string
    AllowedTools          []string
    DisallowedTools       []string
    AdditionalDirectories []string
    CustomInstructions    string
    Verbose               bool
    Env                   map[string]string
}
```

### Campos Requeridos

#### Query
- **Tipo**: `string`
- **Descripción**: El prompt o pregunta para Claude
- **CLI Flag**: Se pasa como último argumento después de `--`
- **Ejemplo**: `"Analiza este código y sugiere mejoras"`

### Campos de Gestión de Sesión

#### SessionID
- **Tipo**: `string`
- **Descripción**: ID de sesión existente para reanudar
- **CLI Flag**: `--resume <session-id>`
- **Comportamiento**: Si está establecido, reanuda la sesión en lugar de crear una nueva

#### ForkSession
- **Tipo**: `bool`
- **Descripción**: Si es `true` con `SessionID`, bifurca en lugar de reanudar
- **CLI Flag**: `--fork-session` (usado junto con `--resume`)
- **Comportamiento**: Crea una nueva sesión basada en el contexto de la sesión padre

### Campos Opcionales - Configuración del Modelo

#### Model
- **Tipo**: `Model` (enum)
- **Valores Posibles**:
  - `ModelOpus`: Claude Opus (más capaz)
  - `ModelSonnet`: Claude Sonnet (equilibrado)
  - `ModelHaiku`: Claude Haiku (más rápido)
- **CLI Flag**: `--model <model>`
- **Default**: Claude decide basado en la tarea
- **Ejemplo**: `Model: ModelOpus`

### Campos de Formato de Salida

#### OutputFormat
- **Tipo**: `OutputFormat` (enum)
- **Valores Posibles**:
  - `OutputText`: Texto plano (default)
  - `OutputJSON`: JSON estructurado único
  - `OutputStreamJSON`: Eventos JSON streaming línea por línea
- **CLI Flag**: `--output-format <format>`
- **Comportamiento Especial**: `stream-json` automáticamente agrega `--verbose`
- **Ejemplo**: `OutputFormat: OutputStreamJSON`

### Campos de Configuración MCP

#### MCPConfig
- **Tipo**: `*MCPConfig`
- **Descripción**: Configuración de servidores Model Context Protocol
- **CLI Flag**: `--mcp-config <temp-file-path>`
- **Proceso**:
  1. Se serializa a JSON
  2. Se escribe en archivo temporal
  3. Se pasa la ruta del archivo temporal al CLI
  4. El archivo temporal se limpia cuando el proceso termina

**Estructura MCPConfig** (`types.go:45-47`):
```go
type MCPConfig struct {
    MCPServers map[string]MCPServer `json:"mcpServers"`
}
```

**Estructura MCPServer** (`types.go:32-42`):
```go
type MCPServer struct {
    // Para servidores stdio
    Command string            `json:"command,omitempty"`
    Args    []string          `json:"args,omitempty"`
    Env     map[string]string `json:"env,omitempty"`

    // Para servidores HTTP
    Type    string            `json:"type,omitempty"`    // "http"
    URL     string            `json:"url,omitempty"`
    Headers map[string]string `json:"headers,omitempty"`
}
```

**Ejemplo de Configuración MCP**:
```go
mcpConfig := &MCPConfig{
    MCPServers: map[string]MCPServer{
        "filesystem": {
            Command: "mcp-server-filesystem",
            Args:    []string{"/path/to/workspace"},
        },
        "api-server": {
            Type: "http",
            URL:  "http://localhost:3000/mcp",
            Headers: map[string]string{
                "Authorization": "Bearer token",
            },
        },
    },
}
```

#### PermissionPromptTool
- **Tipo**: `string`
- **Descripción**: Nombre del tool MCP para prompts de permisos
- **CLI Flag**: `--permission-prompt-tool <tool-name>`
- **Ejemplo**: `"mcp__codelayer__request_permission"`

### Campos de Ejecución

#### WorkingDir
- **Tipo**: `string`
- **Descripción**: Directorio de trabajo para la sesión
- **Procesamiento**:
  - Expande `~` a directorio home del usuario
  - Convierte a ruta absoluta
  - Limpia la ruta con `filepath.Clean()`
- **Ejemplo**: `"~/projects/myapp"` → `"/home/user/projects/myapp"`

#### MaxTurns
- **Tipo**: `int`
- **Descripción**: Número máximo de turnos de conversación
- **CLI Flag**: `--max-turns <n>`
- **Default**: Sin límite (0)
- **Ejemplo**: `MaxTurns: 10`

### Campos de Prompts del Sistema

#### SystemPrompt
- **Tipo**: `string`
- **Descripción**: Reemplaza completamente el prompt del sistema
- **CLI Flag**: `--system-prompt <text>`
- **Uso**: Para control total sobre el comportamiento de Claude

#### AppendSystemPrompt
- **Tipo**: `string`
- **Descripción**: Agrega texto adicional al prompt del sistema default
- **CLI Flag**: `--append-system-prompt <text>`
- **Uso**: Para agregar contexto sin reemplazar el prompt base

#### CustomInstructions
- **Tipo**: `string`
- **Descripción**: Instrucciones personalizadas adicionales
- **Comportamiento**: Similar a `AppendSystemPrompt` pero separado conceptualmente
- **Ejemplo**: `"Siempre responde en español"`

### Campos de Control de Herramientas

#### AllowedTools
- **Tipo**: `[]string`
- **Descripción**: Lista explícita de herramientas permitidas
- **CLI Flag**: `--allowedTools <tool1>,<tool2>,...`
- **Formato**: Separado por comas
- **Ejemplo**: `[]string{"read_file", "write_file", "bash"}`

#### DisallowedTools
- **Tipo**: `[]string`
- **Descripción**: Lista de herramientas prohibidas
- **CLI Flag**: `--disallowedTools <tool1>,<tool2>,...`
- **Formato**: Separado por comas
- **Ejemplo**: `[]string{"bash", "edit_file"}`

### Campos de Directorios Adicionales

#### AdditionalDirectories
- **Tipo**: `[]string`
- **Descripción**: Directorios adicionales para agregar al contexto
- **CLI Flag**: `--add-dir <path>` (uno por directorio)
- **Procesamiento** (`client.go:266-294`):
  1. Expande `~` a home directory
  2. Convierte a ruta absoluta
  3. Agrega flag `--add-dir` para cada directorio
- **Ejemplo**:
  ```go
  AdditionalDirectories: []string{
      "~/project-a",
      "~/project-b/lib",
  }
  ```

### Campos de Debugging

#### Verbose
- **Tipo**: `bool`
- **Descripción**: Activa salida verbose
- **CLI Flag**: `--verbose`
- **Comportamiento**: Automáticamente activado cuando `OutputFormat == OutputStreamJSON`

### Campos de Entorno

#### Env
- **Tipo**: `map[string]string`
- **Descripción**: Variables de entorno para el proceso Claude
- **Implementación** (`client.go:324-329`):
  - Comienza con `os.Environ()` (entorno actual)
  - Agrega/sobrescribe con valores del mapa
- **Ejemplo**:
  ```go
  Env: map[string]string{
      "ANTHROPIC_API_KEY": "sk-ant-...",
      "ANTHROPIC_BASE_URL": "http://localhost:8080/proxy",
  }
  ```

---

## Construcción de Argumentos CLI

### buildArgs()

**Ubicación**: `client.go:183-311`

**Firma**:
```go
func (c *Client) buildArgs(config SessionConfig) ([]string, error)
```

**Descripción**: Convierte `SessionConfig` en argumentos de línea de comandos.

**Orden de Argumentos**:
1. Session management (`--resume`, `--fork-session`)
2. Model (`--model`)
3. Output format (`--output-format`, `--verbose` si es necesario)
4. MCP config (`--mcp-config`)
5. Permission tool (`--permission-prompt-tool`)
6. Max turns (`--max-turns`)
7. System prompts (`--system-prompt`, `--append-system-prompt`)
8. Tools (`--allowedTools`, `--disallowedTools`)
9. Additional directories (`--add-dir` × N)
10. Verbose (`--verbose` si no se agregó antes)
11. **DEBE SER ÚLTIMO**: `--print -- <query>`

**Nota Crítica**: El orden es importante. Los flags deben venir antes de `--`, y el query debe ser el último argumento.

**Ejemplo de Argumentos Generados**:
```bash
claude \
  --model opus \
  --output-format stream-json \
  --verbose \
  --mcp-config /tmp/mcp-config-123.json \
  --max-turns 10 \
  --add-dir /home/user/project-a \
  --add-dir /home/user/project-b \
  --print -- "Analiza este código"
```

---

## Session - Sesión Activa

**Definición** (`types.go:283-299`):
```go
type Session struct {
    ID        string
    Config    SessionConfig
    StartTime time.Time

    // Para streaming
    Events chan StreamEvent

    // Gestión de proceso
    cmd    *exec.Cmd
    done   chan struct{}
    result *Result

    // Manejo thread-safe de errores
    mu  sync.RWMutex
    err error
}
```

### Métodos de Session

#### Wait()

**Ubicación**: `client.go:435-443`

**Firma**:
```go
func (s *Session) Wait() (*Result, error)
```

**Descripción**: Bloquea hasta que la sesión complete y retorna el resultado.

**Comportamiento**:
- Espera en canal `done`
- Retorna error si el proceso falló
- Retorna `Result` si fue exitoso

**TODO**: Agregar soporte para context para permitir cancelación/timeout.

#### Kill()

**Ubicación**: `client.go:446-451`

**Firma**:
```go
func (s *Session) Kill() error
```

**Descripción**: Termina la sesión abruptamente (SIGKILL).

**Uso**: Forzar terminación cuando Interrupt() no es suficiente.

#### Interrupt()

**Ubicación**: `client.go:454-459`

**Firma**:
```go
func (s *Session) Interrupt() error
```

**Descripción**: Envía señal SIGINT al proceso de sesión.

**Comportamiento**: Graceful shutdown, permite a Claude completar operaciones en curso.

#### SetError()

**Ubicación**: `types.go:302-308`

**Firma**:
```go
func (s *Session) SetError(err error)
```

**Descripción**: Establece el error de forma thread-safe.

**Comportamiento**: Solo establece el primer error (no sobrescribe).

#### Error()

**Ubicación**: `types.go:311-315`

**Firma**:
```go
func (s *Session) Error() error
```

**Descripción**: Obtiene el error de forma thread-safe.

---

## Tipos de Eventos y Resultados

### StreamEvent

**Definición** (`types.go:76-105`):
```go
type StreamEvent struct {
    Type       string
    Subtype    string
    SessionID  string
    Message    *Message
    Tools      []string
    MCPServers []MCPStatus

    // Tracking de sub-tareas
    ParentToolUseID string

    // Campos de evento system (type="system", subtype="init")
    CWD            string
    Model          string
    PermissionMode string
    APIKeySource   string

    // Campos de evento result (type="result")
    CostUSD           float64
    IsError           bool
    DurationMS        int
    DurationAPI       int
    NumTurns          int
    Result            string
    Usage             *Usage
    ModelUsage        map[string]ModelUsageDetail
    Error             string
    PermissionDenials *PermissionDenials
    UUID              string
}
```

**Tipos de Eventos Comunes**:
- `type="system"`: Eventos del sistema (inicialización, configuración)
- `type="message"`: Mensajes de Claude (respuestas, thinking)
- `type="tool_use"`: Claude solicita usar una herramienta
- `type="tool_result"`: Resultado de ejecución de herramienta
- `type="result"`: Resultado final de la sesión

### Result

**Definición** (`types.go:265-280`):
```go
type Result struct {
    Type              string
    Subtype           string
    CostUSD           float64
    IsError           bool
    DurationMS        int
    DurationAPI       int
    NumTurns          int
    Result            string
    SessionID         string
    Usage             *Usage
    ModelUsage        map[string]ModelUsageDetail
    Error             string
    PermissionDenials *PermissionDenials
    UUID              string
}
```

**Campos Clave**:
- `CostUSD`: Costo total en dólares
- `IsError`: Indica si la sesión terminó con error
- `DurationMS`: Duración total en milisegundos
- `NumTurns`: Número de turnos de conversación
- `Result`: Resultado textual final
- `SessionID`: ID de sesión para reanudar

### Message

**Definición** (`types.go:114-121`):
```go
type Message struct {
    ID      string
    Type    string
    Role    string
    Model   string
    Content []Content
    Usage   *Usage
}
```

**Roles Comunes**:
- `"user"`: Mensaje del usuario
- `"assistant"`: Mensaje de Claude

### Content

**Definición** (`types.go:163-172`):
```go
type Content struct {
    Type      string
    Text      string
    Thinking  string
    ID        string
    Name      string
    Input     map[string]interface{}
    ToolUseID string
    Content   ContentField
}
```

**Tipos de Contenido**:
- `type="text"`: Texto plano
- `type="thinking"`: Pensamiento interno de Claude
- `type="tool_use"`: Solicitud de uso de herramienta
- `type="tool_result"`: Resultado de herramienta

### Usage

**Definición** (`types.go:186-194`):
```go
type Usage struct {
    InputTokens              int
    OutputTokens             int
    CacheCreationInputTokens int
    CacheReadInputTokens     int
    ServiceTier              string
    ServerToolUse            *ServerToolUse
    CacheCreation            *CacheCreation
}
```

**Métricas**:
- `InputTokens`: Tokens de entrada
- `OutputTokens`: Tokens de salida
- `CacheCreationInputTokens`: Tokens usados para crear cache
- `CacheReadInputTokens`: Tokens leídos desde cache

---

## Parseo de Salida

### parseStreamingJSON()

**Ubicación**: `client.go:462-543`

**Descripción**: Parsea salida JSON streaming línea por línea.

**Proceso**:
1. Usa `bufio.Scanner` con buffer de 10MB para líneas grandes
2. Lee línea por línea desde stdout
3. Deserializa cada línea a `StreamEvent`
4. Envía eventos al canal `session.Events`
5. Almacena resultado final cuando `Type == "result"`
6. Captura stderr en goroutine separada
7. Cierra canal de eventos al terminar

**Manejo de Errores**:
- Líneas inválidas: Se registra warning y continúa
- Buffer overflow (>10MB): Retorna error específico
- Stderr no vacío: Establece error de sesión

### parseSingleJSON()

**Ubicación**: `client.go:546-589`

**Descripción**: Parsea resultado JSON único.

**Proceso**:
1. Lee todo stdout a buffer
2. Lee todo stderr a buffer
3. Deserializa stdout como `Result`
4. Establece `session.result`
5. Si hay stderr, establece error (pero no sobrescribe resultado válido)

### parseTextOutput()

**Ubicación**: `client.go:592-620`

**Descripción**: Captura salida de texto plano.

**Proceso**:
1. Lee todo stdout y stderr
2. Crea `Result` simple con tipo "result" y subtipo "success"
3. El resultado contiene el texto de stdout trimmed
4. Si hay stderr, establece error

---

## Funciones Utilitarias

### isExecutable()

**Ubicación**: `client.go:150-159`

**Firma**:
```go
func isExecutable(path string) error
```

**Descripción**: Verifica si un archivo es ejecutable.

**Implementación**: Verifica que los bits de ejecución están establecidos (`mode & 0111 != 0`).

### shouldSkipPath()

**Ubicación**: `client.go:49-59`

**Firma**:
```go
func shouldSkipPath(path string) bool
```

**Descripción**: Verifica si una ruta debe omitirse durante búsqueda.

**Criterios de Omisión**:
- Contiene `/node_modules/`
- Termina en `.bak`

### isClosedPipeError()

**Ubicación**: `client.go:20-41`

**Firma**:
```go
func isClosedPipeError(err error) bool
```

**Descripción**: Verifica si un error es debido a pipe cerrado (esperado cuando proceso termina).

**Patrones Verificados**:
- "file already closed"
- "broken pipe"
- "use of closed network connection"
- `syscall.EPIPE`
- `syscall.EBADF`
- `io.EOF`

---

## Ejemplos de Uso

### Ejemplo Básico

```go
package main

import (
    "fmt"
    "log"

    "github.com/humanlayer/humanlayer/claudecode-go"
)

func main() {
    // Crear cliente
    client, err := claudecode.NewClient()
    if err != nil {
        log.Fatal(err)
    }

    // Configurar sesión
    config := claudecode.SessionConfig{
        Query:        "¿Cuál es la capital de Francia?",
        Model:        claudecode.ModelSonnet,
        OutputFormat: claudecode.OutputJSON,
    }

    // Lanzar y esperar
    result, err := client.LaunchAndWait(config)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Resultado: %s\n", result.Result)
    fmt.Printf("Costo: $%.4f\n", result.CostUSD)
}
```

### Ejemplo con Streaming

```go
package main

import (
    "fmt"
    "log"

    "github.com/humanlayer/humanlayer/claudecode-go"
)

func main() {
    client, err := claudecode.NewClient()
    if err != nil {
        log.Fatal(err)
    }

    config := claudecode.SessionConfig{
        Query:        "Explica la programación concurrente en Go",
        Model:        claudecode.ModelSonnet,
        OutputFormat: claudecode.OutputStreamJSON,
    }

    // Lanzar sesión
    session, err := client.Launch(config)
    if err != nil {
        log.Fatal(err)
    }

    // Procesar eventos en tiempo real
    go func() {
        for event := range session.Events {
            if event.Type == "message" && event.Message != nil {
                fmt.Printf("[%s]: ", event.Message.Role)
                for _, content := range event.Message.Content {
                    if content.Type == "text" {
                        fmt.Println(content.Text)
                    }
                }
            }
        }
    }()

    // Esperar resultado final
    result, err := session.Wait()
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("\nCosto total: $%.4f\n", result.CostUSD)
    fmt.Printf("Turnos: %d\n", result.NumTurns)
}
```

### Ejemplo con MCP Server

```go
package main

import (
    "log"

    "github.com/humanlayer/humanlayer/claudecode-go"
)

func main() {
    client, err := claudecode.NewClient()
    if err != nil {
        log.Fatal(err)
    }

    // Configurar servidor MCP
    mcpConfig := &claudecode.MCPConfig{
        MCPServers: map[string]claudecode.MCPServer{
            "filesystem": {
                Command: "mcp-server-filesystem",
                Args:    []string{"/home/user/workspace"},
            },
            "git": {
                Command: "mcp-server-git",
                Args:    []string{"--repo", "/home/user/workspace"},
            },
        },
    }

    config := claudecode.SessionConfig{
        Query:                "Lee el archivo README.md y analízalo",
        Model:                claudecode.ModelSonnet,
        OutputFormat:         claudecode.OutputStreamJSON,
        MCPConfig:            mcpConfig,
        WorkingDir:           "/home/user/workspace",
        PermissionPromptTool: "mcp__codelayer__request_permission",
    }

    result, err := client.LaunchAndWait(config)
    if err != nil {
        log.Fatal(err)
    }

    log.Printf("Resultado: %s", result.Result)
}
```

### Ejemplo con Directorios Adicionales

```go
package main

import (
    "log"

    "github.com/humanlayer/humanlayer/claudecode-go"
)

func main() {
    client, err := claudecode.NewClient()
    if err != nil {
        log.Fatal(err)
    }

    config := claudecode.SessionConfig{
        Query:  "Compara la arquitectura de estos dos proyectos",
        Model:  claudecode.ModelOpus,
        WorkingDir: "~/project-main",
        AdditionalDirectories: []string{
            "~/project-a/src",
            "~/project-b/lib",
        },
    }

    result, err := client.LaunchAndWait(config)
    if err != nil {
        log.Fatal(err)
    }

    log.Printf("Análisis: %s", result.Result)
}
```

### Ejemplo con Control de Herramientas

```go
package main

import (
    "log"

    "github.com/humanlayer/humanlayer/claudecode-go"
)

func main() {
    client, err := claudecode.NewClient()
    if err != nil {
        log.Fatal(err)
    }

    config := claudecode.SessionConfig{
        Query: "Analiza el código pero no ejecutes nada",
        Model: claudecode.ModelSonnet,

        // Solo permitir lectura
        AllowedTools: []string{
            "read_file",
            "list_directory",
            "search_files",
        },

        // Prohibir ejecución
        DisallowedTools: []string{
            "bash",
            "write_file",
            "edit_file",
        },
    }

    result, err := client.LaunchAndWait(config)
    if err != nil {
        log.Fatal(err)
    }

    log.Printf("Análisis: %s", result.Result)
}
```

### Ejemplo con Variables de Entorno

```go
package main

import (
    "log"

    "github.com/humanlayer/humanlayer/claudecode-go"
)

func main() {
    client, err := claudecode.NewClient()
    if err != nil {
        log.Fatal(err)
    }

    config := claudecode.SessionConfig{
        Query: "Analiza la configuración del proyecto",
        Model: claudecode.ModelSonnet,

        // Variables de entorno personalizadas
        Env: map[string]string{
            "ANTHROPIC_API_KEY":  "sk-ant-custom-key",
            "ANTHROPIC_BASE_URL": "http://localhost:8080/proxy",
            "DEBUG": "true",
        },
    }

    result, err := client.LaunchAndWait(config)
    if err != nil {
        log.Fatal(err)
    }

    log.Printf("Resultado: %s", result.Result)
}
```

### Ejemplo con Interrupción

```go
package main

import (
    "context"
    "log"
    "time"

    "github.com/humanlayer/humanlayer/claudecode-go"
)

func main() {
    client, err := claudecode.NewClient()
    if err != nil {
        log.Fatal(err)
    }

    config := claudecode.SessionConfig{
        Query:        "Tarea muy larga...",
        Model:        claudecode.ModelOpus,
        OutputFormat: claudecode.OutputStreamJSON,
    }

    session, err := client.Launch(config)
    if err != nil {
        log.Fatal(err)
    }

    // Timeout de 30 segundos
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    // Esperar resultado o timeout
    done := make(chan struct{})
    go func() {
        _, err := session.Wait()
        if err != nil {
            log.Printf("Error: %v", err)
        }
        close(done)
    }()

    select {
    case <-done:
        log.Println("Sesión completada")
    case <-ctx.Done():
        log.Println("Timeout, interrumpiendo sesión...")
        if err := session.Interrupt(); err != nil {
            log.Printf("Error al interrumpir: %v", err)
        }
        // Esperar limpieza graceful
        <-done
    }
}
```

---

## Consideraciones de Implementación

### Thread Safety

**Session.SetError() y Session.Error()**:
- Usan `sync.RWMutex` para acceso thread-safe
- `SetError()` solo establece el primer error
- Múltiples goroutines pueden llamar estos métodos de forma segura

### Manejo de Pipes

**Cierre de Pipes**:
- Se esperan errores de pipe cerrado cuando el proceso termina
- `isClosedPipeError()` distingue errores esperados vs inesperados
- Previene logging de ruido en terminaciones normales

### Buffer de Parsing

**Scanner Buffer**:
- Buffer de 10MB para `parseStreamingJSON()`
- Previene overflow cuando Claude retorna contenidos de archivos grandes
- Error específico `ErrTooLong` si se excede

### Sincronización de Parseo

**parseDone Channel**:
- Asegura que todo el output se lea antes de que `Wait()` retorne
- Previene race condition entre parseo y completitud de proceso
- Crítico para `OutputStreamJSON` donde eventos se procesan de forma asíncrona

### Archivos Temporales MCP

**Gestión de Archivos Temporales**:
- MCP config se escribe a archivo temporal
- No se limpia explícitamente (OS lo limpia cuando proceso termina)
- Se registra la ruta para debugging

### Expansión de Tilde

**Paths con ~**:
- Se expande automáticamente en `WorkingDir` y `AdditionalDirectories`
- `~/` se convierte a home directory completo
- `~` solo se convierte a home directory
- Fallback a ruta original si la expansión falla

---

## Limitaciones Conocidas

### 1. Falta de Soporte para Context

**Ubicación**: `client.go:432-434` (TODO comment)

**Problema**: `Session.Wait()` no acepta context para cancelación/timeout.

**Workaround Actual**: Usar goroutines con `select` y `session.Interrupt()` (ver ejemplo de interrupción).

**Mejora Propuesta**: Agregar método `WaitContext(ctx context.Context)`.

### 2. Limpieza de Archivos Temporales MCP

**Ubicación**: `client.go:236` (comment)

**Comportamiento**: Los archivos temporales de MCP config no se eliminan explícitamente.

**Justificación**: El OS los limpia cuando el proceso termina.

**Riesgo**: Si se lanzan muchas sesiones, los archivos temporales se acumulan hasta que el daemon se reinicie.

### 3. Scanner Buffer Fijo

**Ubicación**: `client.go:466`

**Límite**: Líneas JSON de máximo 10MB.

**Problema**: Si Claude retorna contenido > 10MB en una sola línea, falla.

**Mitigación Actual**: 10MB es suficiente para la mayoría de casos (archivos grandes se dividen en chunks).

---

## Debugging

### Logging

**Ubicación de Logs**:
- `client.go:219`: MCP config JSON
- `client.go:233`: Ruta de archivo temporal MCP
- `client.go:267-293`: Procesamiento de directorios adicionales
- `client.go:320`: Comando Claude completo
- `client.go:492`: Errores de parsing de eventos

**Formato**:
```go
log.Printf("MCP config JSON: %s", string(mcpJSON))
log.Printf("Executing Claude command: %s %v", c.claudePath, args)
```

### Flags Verbose

**Activación Automática**: `OutputStreamJSON` activa `--verbose` automáticamente.

**Activación Manual**: Establecer `Verbose: true` en `SessionConfig`.

---

## Mejores Prácticas

### 1. Usar OutputStreamJSON para Sesiones Largas

**Justificación**: Permite procesar eventos en tiempo real y mostrar progreso.

```go
config := claudecode.SessionConfig{
    Query:        "Tarea larga",
    OutputFormat: claudecode.OutputStreamJSON,
}

session, _ := client.Launch(config)
for event := range session.Events {
    // Procesar eventos en tiempo real
}
```

### 2. Manejar Permission Denials

**Verificar en Result**:
```go
result, _ := client.LaunchAndWait(config)
if result.PermissionDenials != nil && len(result.PermissionDenials.Denials) > 0 {
    for _, denial := range result.PermissionDenials.Denials {
        log.Printf("Permiso denegado: %s (tool_use_id: %s)",
            denial.ToolName, denial.ToolUseID)
    }
}
```

### 3. Verificar Errores de Sesión

**Error() vs Result**:
```go
session, _ := client.Launch(config)
result, err := session.Wait()

if err != nil {
    // Error de proceso (falló el launch o Wait)
    log.Fatal(err)
}

if result.IsError {
    // Error de Claude (sesión completó pero con error)
    log.Printf("Error de Claude: %s", result.Error)
}
```

### 4. Establecer MaxTurns para Prevenir Loops

**Protección contra loops infinitos**:
```go
config := claudecode.SessionConfig{
    Query:    "Tarea iterativa",
    MaxTurns: 20, // Límite de seguridad
}
```

### 5. Usar AllowedTools para Sandboxing

**Restricción de herramientas en producción**:
```go
config := claudecode.SessionConfig{
    Query: "Analiza código",
    AllowedTools: []string{
        "read_file",
        "list_directory",
    },
    DisallowedTools: []string{
        "bash",
        "write_file",
    },
}
```

---

## Referencias

### Archivos Relacionados
- `claudecode-go/client.go`: Implementación del cliente
- `claudecode-go/types.go`: Definiciones de tipos
- `hld/session/manager.go`: Integración en daemon HLD

### Documentación Relacionada
- `thoughts/shared/knowledge/features/session-creation.md`: Flujo de creación de sesiones
- `thoughts/shared/claude/commands/`: Comandos slash de Claude Code

---

## Changelog

**Versión Actual**: Basado en análisis del 2025-11-08

**Cambios Recientes**:
- Soporte para `AdditionalDirectories` con expansión de tilde
- Buffer de 10MB para líneas JSON grandes
- Manejo de `PermissionDenials` con formato flexible
- Thread-safe error handling con `sync.RWMutex`
- Timeout de 5 segundos en `GetVersion()`
