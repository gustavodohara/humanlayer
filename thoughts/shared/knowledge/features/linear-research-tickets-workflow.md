# Workflow de Automatización de Tickets de Investigación en Linear

## Descripción General

El workflow de GitHub Actions `linear-research-tickets.yml` proporciona capacidades automatizadas de investigación para tickets de Linear utilizando Claude Code. Este workflow permite la investigación autónoma del código base, generación de documentación y gestión de tickets de Linear sin intervención humana.

**Ubicación del archivo**: `.github/workflows/linear-research-tickets.yml`

## Propósito

Este workflow automatiza el proceso de investigación y documentación para tickets de ingeniería mediante:
- Obtención automática de tickets de Linear que necesitan investigación
- Uso de Claude Code para investigar el código base según los requisitos del ticket
- Generación de documentos de investigación comprehensivos
- Sincronización de documentación a un repositorio separado de "thoughts"
- Gestión de transiciones de estado de tickets a lo largo del ciclo de vida de la investigación
- Respeto de límites de Trabajo en Progreso (WIP) para prevenir sobrecarga

## Mecanismos de Activación

El workflow puede activarse de tres formas:

### 1. Activación por Push (Desarrollo/Pruebas)
```yaml
push:
  branches:
    - kyle/eng-2175-auto-create-worktree-and-checkout-for-linear
```
Actualmente configurado para una rama de desarrollo específica.

### 2. Activación Programada (ACTUALMENTE DESHABILITADA)
```yaml
# schedule:
#   - cron: "*/15 * * * *"  # Cada 15 minutos
```
La activación programada está comentada, indicando que el workflow de investigación automatizada no se ejecuta periódicamente en este momento (como se nota en el commit 8b857101).

### 3. Ejecución Manual
```yaml
workflow_dispatch:
  inputs:
    num_tickets:
      description: "Número máximo de tickets a intentar obtener"
      required: false
      type: string
      default: "10"
```
Puede activarse manualmente con un número configurable de tickets a procesar.

## Arquitectura y Flujo de Jobs

El workflow consiste en dos jobs que se ejecutan secuencialmente:

### Job 1: fetch-tickets

**Propósito**: Determinar la capacidad de investigación y obtener los tickets apropiados.

**Pasos**:

1. **Verificar Capacidad WIP** (paso `check-wip`)
   - Consulta Linear por tickets en estado "research in review"
   - Asignados a "LinearLayer (Claude)"
   - Compara el conteo contra `RESEARCH_WIP_LIMIT` (por defecto: 5)
   - Calcula espacios disponibles: `available_slots = WIP_LIMIT - review_count`
   - Salidas: `review_count`, `available_slots`
   - Incluye manejo comprehensivo de errores para fallos de la API de Linear

2. **Obtener Tickets Investigables** (paso `fetch-tickets`)
   - Solo se ejecuta si `available_slots > 0`
   - Obtiene tickets con:
     - Estado: "research needed"
     - Asignado a: "LinearLayer (Claude)"
     - Límite: `min(available_slots, num_tickets)`
   - Salidas: Array JSON de IDs de tickets

**Variables de Entorno**:
- `RESEARCH_WIP_LIMIT`: Máximo de tickets concurrentes en revisión (por defecto: 5)
- `LINEAR_API_KEY`: Autenticación para la API de Linear

**Lógica del Límite WIP**:
```
if available_slots == 0:
    obtener 0 tickets (límite WIP alcanzado)
else:
    obtener min(available_slots, max_tickets_solicitados)
```

### Job 2: research-tickets

**Propósito**: Ejecutar investigación para cada ticket en paralelo.

**Estrategia de Matriz**:
```yaml
strategy:
  fail-fast: false
  matrix:
    ticket_id: ${{ fromJSON(needs.fetch-tickets.outputs.ticket_ids) }}
```
Se ejecuta en paralelo para cada ticket, continúa incluso si tickets individuales fallan.

**Configuración del Entorno**:
- Node.js 20
- Bun (última versión)
- Claude Code CLI (`@anthropic-ai/claude-code`)
- HumanLayer CLI
- Herramienta CLI de Linear (de `hack/linear`)

**Configuración de Repositorios**:
- **Repositorio principal**: `humanlayer/humanlayer` (código a investigar)
- **Repositorio thoughts**: `humanlayer/thoughts` (almacenamiento de documentación de investigación)
- Usa clave de despliegue SSH para acceso de escritura al repositorio thoughts

**Pasos de Ejecución**:

1. **Checkout de Repositorios**
   - Repositorio principal: `humanlayer`
   - Repositorio thoughts: `humanlayer/thoughts`

2. **Configurar Herramienta Thoughts**
   Crea `.humanlayer.json`:
   ```json
   {
     "thoughts": {
       "thoughtsRepo": "${{ github.workspace }}/thoughts",
       "reposDir": "repos",
       "globalDir": "global",
       "user": "claude",
       "repoMappings": {}
     }
   }
   ```

3. **Obtener Detalles del Ticket**
   - Obtiene contenido del ticket y lo guarda en `thoughts/shared/tickets/{ticket_id}.md`
   - Descarga cualquier imagen del ticket a `/thoughts/shared/images/{ticket_id}/`

4. **Actualizar Estado a "Research in Progress"**
   - Señala que el ticket está siendo investigado activamente

5. **Ejecutar Investigación con Claude Code**
   - Modelo: Opus
   - Comando slash: `/research_codebase`
   - Lee el ticket y comentarios (atención especial a menciones de LinearLayer/Claude)
   - Lee cualquier imagen del ticket
   - Investiga el código base de forma autónoma (sin interacción del usuario)
   - Formato de salida esperado:
   ```
   research completed - [thoughts/shared/YYYY-MM-DD-ENG-XXXX-description.md](...)

   ### open questions
   1. [Primera pregunta abierta]
   2. [Segunda pregunta abierta]
   ...
   ```
   - Salida guardada en `CLAUDE_ANSWER.md`

6. **Sincronizar Investigación al Repositorio Thoughts**
   - Hace commit de cambios al repositorio thoughts
   - Hace pull con rebase para manejar actualizaciones concurrentes
   - Reintenta push hasta 5 veces con backoff exponencial
   - Maneja condiciones de carrera del procesamiento paralelo de tickets

7. **Publicar Resultados en Linear**
   - Agrega la respuesta de Claude como comentario en el ticket
   - Actualiza el estado a "research in review"

**Manejo de Errores**:
- En caso de fallo: Resetea el estado del ticket a "research needed"
- En caso de cancelación: Resetea el estado del ticket a "research needed"
- Asegura que los tickets no se queden atascados en "research in progress"

## Componentes Clave

### Herramienta CLI de Linear
Ubicación: `hack/linear/`

Comandos usados:
- `linear list-issues` - Consultar tickets por estado, asignado, filtros
- `linear get-issue {id}` - Obtener detalles completos del ticket
- `linear fetch-images {id}` - Descargar adjuntos del ticket
- `linear update-status {id} {status}` - Actualizar estado del workflow del ticket
- `linear add-comment -i {id} {comment}` - Publicar comentarios

### Integración con Repositorio Thoughts
- Repositorio privado separado para documentos de investigación
- Estructura: `repos/humanlayer/shared/research/`
- Habilita una base de conocimiento persistente a través de ejecuciones de CI
- Autenticación con clave de despliegue SSH para acceso de escritura

### Integración con Claude Code
- Modo de investigación autónoma (sin interacción del usuario)
- Usa el comando slash `/research_codebase`
- Formato de salida estructurado para fácil análisis
- Omite peligrosamente permisos (contexto automatizado)

## Configuración

### Límite WIP
```yaml
env:
  RESEARCH_WIP_LIMIT: 5
```
Previene sobrecarga del sistema limitando tickets concurrentes en revisión.

### Secretos Requeridos
- `LINEAR_API_KEY` - Autenticación de API de Linear
- `ANTHROPIC_API_KEY` - Acceso a API de Claude
- `THOUGHTS_WRITE_DEPLOY_KEY` - Clave SSH para repositorio thoughts

### Configuración de Git
```bash
git config --global user.email "noreply@anthropic.com"
git config --global user.name "Claude"
```

## Flujo de Estados

```
research needed
    ↓
research in progress (durante ejecución de Claude)
    ↓
research in review (después de completarse exitosamente)
    ↓ (en caso de fallo/cancelación)
research needed (reseteo para reintento)
```

## Características y Decisiones de Diseño

### 1. Gestión de Límite WIP
- Previene saturar la cola de revisión
- Degradación elegante cuando la capacidad está llena
- Notificaciones de GitHub cuando se aproxima/alcanza los límites

### 2. Ejecución Paralela
- Estrategia de matriz para procesamiento concurrente de tickets
- `fail-fast: false` asegura que un fallo no detenga a otros
- Estrategia de git rebase maneja commits concurrentes al repositorio thoughts

### 3. Manejo Robusto de Errores
- Fallos de consulta a API de Linear por defecto a capacidad cero
- Lógica de reintento de push (5 intentos) para condiciones de carrera
- Reseteo de estados asegura que ningún ticket se pierda o quede atascado

### 4. Comunicación Estructurada
- Fuerza formato específico de salida desde Claude
- Extrae preguntas abiertas para revisión humana
- Publica contexto completo de vuelta a Linear

### 5. Coordinación Multi-Repositorio
- Investiga código en repositorio principal
- Almacena documentación en repositorio thoughts
- Sincroniza estado vía API de Linear

## Patrones de Uso

### Ejemplo de Activación Manual
```bash
# Investigar hasta 3 tickets
gh workflow run "Linear: Research Task" \
  --repo humanlayer/humanlayer \
  -f num_tickets=3

# Monitorear ejecución
gh run list --workflow=linear-research-tickets.yml
gh run watch <run-id>
```

### Pruebas Basadas en Rama
Según `.github/workflows/CLAUDE.md`:
1. Crear una rama de feature
2. Agregar rama a activadores de push
3. Probar cambios vía push
4. Remover activador de rama antes de merge

## Monitoreo y Observabilidad

### Avisos de GitHub
```bash
::notice title=WIP Status::2/5 tickets en revisión, 3 espacios disponibles
::warning title=Approaching WIP Limit::Solo 1 espacio(s) restante(s)
::warning title=WIP Limit Reached::Sin capacidad para nuevos tickets
```

### Criterios de Éxito
- Estado del ticket actualizado a "research in review"
- Documento de investigación creado en repositorio thoughts
- Comentario publicado en Linear con hallazgos y preguntas abiertas

### Indicadores de Fallo
- Estado del ticket revertido a "research needed"
- Revisar logs del workflow para error específico
- Problemas comunes: errores de API de Linear, formato de salida de Claude, conflictos de git

## Puntos de Integración

### Con Linear
- Consultas: Filtrar por estado, asignado
- Actualizaciones: Transiciones de estado
- Comentarios: Resultados de investigación, preguntas
- Adjuntos: Descargas de imágenes

### Con Claude Code
- Comando slash: `/research_codebase`
- Modo de ejecución autónoma
- Análisis de salida estructurada

### Con Repositorio Thoughts
- Almacenamiento y organización de documentos
- Referencia de investigación histórica
- Compartición de conocimiento entre tickets

## Estado Actual

**Ejecución Programada**: DESHABILITADA (desde commit 8b857101)
- El schedule cron está comentado
- El workflow solo se ejecuta en activación manual o push a ramas específicas
- Esto puede ser temporal durante desarrollo/refinamiento

## Consideraciones Futuras

1. **Re-habilitar Ejecución Programada**: Una vez estable, descomentar el activador cron
2. **Límites WIP Dinámicos**: Podría ajustarse según profundidad de cola o hora del día
3. **Obtención Basada en Prioridad**: Procesar tickets de mayor prioridad primero
4. **Métricas de Calidad de Investigación**: Rastrear tiempo de completado, conteo de preguntas, frecuencia de revisión
5. **Coordinación de Investigación Paralela**: Detectar áreas de investigación superpuestas entre tickets

## Archivos Relacionados

- `.github/workflows/CLAUDE.md` - Guías de iteración de workflow
- `hack/linear/` - Implementación de herramienta CLI de Linear
- `.humanlayer.json` - Configuración de herramienta thoughts
- `thoughts/shared/tickets/` - Detalles de tickets obtenidos
- `thoughts/shared/images/` - Adjuntos de tickets
- `thoughts/shared/research/` - Documentos de investigación generados
