# /local_review - Configurar Worktree para Revisar Rama de Colega

## Descripción

Configura un entorno de review local para la rama de un colega. Crea un worktree, configura dependencias y lanza una nueva sesión de Claude Code.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Formato de Entrada

Se invoca con parámetro en formato: `gh_username:branchName`

Ejemplo: `samdickson22:sam/eng-1696-hotkey-for-yolo-mode`

Si no se proporciona parámetro, solicita uno en el formato correcto.

## Proceso

### 1. Parsea la Entrada

- Extrae username de GitHub y nombre de rama del formato `username:branchname`
- Si no se proporciona parámetro, solicita en el formato: `gh_username:branchName`

### 2. Extrae Información del Ticket

- Busca números de ticket en el nombre de rama (ej: `eng-1696`, `ENG-1696`)
- Usa esto para crear nombre corto de directorio del worktree
- Si no se encuentra ticket, usa versión sanitizada del nombre de rama

### 3. Configura Remote y Worktree

```bash
# Verifica si remote ya existe
git remote -v

# Si no existe, agrégalo
git remote add USERNAME git@github.com:USERNAME/humanlayer

# Fetch del remote
git fetch USERNAME

# Crea worktree
git worktree add -b BRANCHNAME ~/wt/humanlayer/SHORT_NAME USERNAME/BRANCHNAME
```

### 4. Configura el Worktree

```bash
# Copia configuración de Claude
cp .claude/settings.local.json WORKTREE/.claude/

# Ejecuta setup
make -C WORKTREE setup

# Inicializa thoughts
cd WORKTREE && humanlayer thoughts init --directory humanlayer
```

## Manejo de Errores

### Si el worktree ya existe:
- Informa al usuario que necesitan eliminarlo primero
- Proporciona comando para eliminar: `git worktree remove ~/wt/humanlayer/SHORT_NAME`

### Si fetch del remote falla:
- Verifica si el username/repo existe
- Proporciona mensaje de error útil

### Si setup falla:
- Proporciona el error pero continúa con el lanzamiento
- El usuario puede arreglar problemas de setup manualmente

## Ejemplo de Uso

```bash
/local_review samdickson22:sam/eng-1696-hotkey-for-yolo-mode
```

Esto:
- Agrega 'samdickson22' como remote (si aún no existe)
- Crea worktree en `~/wt/humanlayer/eng-1696`
- Configura el entorno (copia settings, ejecuta make setup, init thoughts)
- Lanza sesión de Claude Code en el worktree

## Estructura de Directorio Resultante

```
~/wt/humanlayer/
└── eng-1696/                    # Worktree para review
    ├── .claude/
    │   └── settings.local.json  # Copiado desde main
    ├── [todo el código del repo]
    └── thoughts/                # Inicializado para este worktree
```

## Ventajas

- **Aislamiento**: Review en worktree separado sin afectar tu trabajo principal
- **Configuración automática**: Setup completo ejecutado automáticamente
- **Thoughts independiente**: Cada worktree tiene su propio directorio thoughts
- **Fácil limpieza**: Solo elimina el worktree cuando termines

## Flujo de Trabajo de Review

1. **Configurar**: `/local_review username:branch`
2. **Revisar**: Claude Code se lanza en el worktree
3. **Probar**: Ejecuta tests, verifica funcionalidad
4. **Comentar**: Usa Linear o GitHub para proporcionar feedback
5. **Limpiar**: Cuando termines, elimina el worktree

## Limpieza Después del Review

```bash
# Lista worktrees
git worktree list

# Elimina worktree cuando termines
git worktree remove ~/wt/humanlayer/eng-1696

# Opcionalmente, elimina remote si ya no se necesita
git remote remove username
```

## Uso

```bash
/local_review samdickson22:sam/eng-1696-hotkey-for-yolo-mode
# O sin parámetros para que te pregunte:
/local_review
```

## Ver También

- [/describe_pr](describe_pr.md) - Para revisar descripción del PR
- [/validate_plan](validate_plan.md) - Para validar implementación del colega
