# /project_knowledge - Generar Documentación Comprehensiva del Proyecto

## Descripción

Analiza la estructura del proyecto y genera documentación comprehensiva en `thoughts/shared/knowledge/`. Cubre arquitectura, mejores prácticas, estrategias de testing, procesos de QA, features y otro conocimiento crítico del proyecto.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Configuración Inicial

Cuando se invoca, responde con:
```
I'll analyze this project's structure and create comprehensive knowledge documentation.

The analysis will create documents covering:
- Architecture and system design
- Folder structure and directory organization
- Best practices and conventions
- Testing strategies
- QA and quality processes
- Feature documentation (one per major feature)
- Development workflow
- Technology stack
- Dependencies and integrations

I'll save all documents to `thoughts/shared/knowledge/`. Shall I proceed?
```

Luego espera confirmación del usuario, o procede inmediatamente si se invocó con parámetro.

## Pasos del Proceso

### Paso 1: Descubrimiento Inicial del Proyecto

1. **Lee archivos clave del proyecto primero**:
   - Lee `README.md` si existe (COMPLETAMENTE, sin limit/offset)
   - Lee `package.json`, `go.mod`, `Cargo.toml`, u otros archivos de dependencias
   - Lee archivos de configuración (`.gitignore`, `Makefile`, `docker-compose.yml`, etc.)
   - Lee cualquier documentación existente en `docs/` o similar
   - **IMPORTANTE**: Usa herramienta Read SIN parámetros limit/offset
   - **CRÍTICO**: Lee estos archivos tú mismo en contexto principal antes de generar sub-tareas

2. **Analiza estructura del proyecto**:
   - Identifica tipo de proyecto (monorepo, single repo, microservices, etc.)
   - Identifica technology stack (lenguajes, frameworks, herramientas)
   - Identifica sistema de build y dependencias
   - Crea lista TodoWrite para rastrear todas las tareas de análisis

3. **Genera tareas de descubrimiento paralelas**:

   **Task 1 - Project Structure Analysis**:
   Analyze directory structure to understand:
   1. Main directories and their purposes
   2. Entry points (main files, config files)
   3. Build and dependency management
   4. Test directories and patterns
   5. Documentation structure
   Use tools: ListDir, Read, Glob
   Return: Directory structure map with descriptions

   **Task 1b - Detailed Folder Structure Documentation**:
   Create comprehensive folder structure documentation...

   **Task 2 - Technology Stack Identification**:
   Identify all technologies used...

   **Task 3 - Configuration and Environment**:
   Find and document configuration...

### Paso 2: Fase de Análisis Profundo

Después de que el descubrimiento inicial se complete, genera investigación paralela comprehensiva:

1. **Architecture Analysis**: Documenta arquitectura del sistema
2. **Best Practices and Conventions**: Identifica estándares de código
3. **Testing Strategy**: Documenta enfoque de testing
4. **QA and Quality Processes**: Documenta quality assurance
5. **Feature Identification**: Identifica features y capacidades principales
6. **Development Workflow**: Documenta procesos de desarrollo
7. **Dependencies and Integrations**: Documenta dependencias externas

### Paso 3: Espera que Todas las Tareas Completen

- **IMPORTANTE**: Espera que TODAS las tareas de sub-agentes completen antes de proceder
- Compila todos los hallazgos de todas las tareas
- Cross-referencia hallazgos para asegurar consistencia
- Identifica cualquier brecha que necesite investigación adicional

### Paso 4: Genera Metadata

1. **Ejecuta script de metadata**:
   ```bash
   hack/spec_metadata.sh
   ```
   O reúne manualmente: fecha actual, commit git, rama, nombre repositorio

2. **Genera estructura de documento**:
   Todos los documentos se guardarán en `thoughts/shared/knowledge/`:
   ```
   thoughts/shared/knowledge/
   ├── architecture.md
   ├── folder-structure.md
   ├── best-practices.md
   ├── testing.md
   ├── qa.md
   ├── development-workflow.md
   ├── technology-stack.md
   ├── dependencies.md
   ├── features/
   │   ├── feature-1-name.md
   │   ├── feature-2-name.md
   │   └── ...
   └── README.md (index of all knowledge documents)
   ```

### Paso 5: Genera Documentos de Conocimiento

Para cada categoría de documento, crea archivo comprehensivo de markdown con frontmatter YAML:

#### Documentos Incluidos

- **architecture.md**: Visión general de arquitectura del sistema
- **folder-structure.md**: Organización de directorios y estructura
- **best-practices.md**: Estándares de código y convenciones
- **testing.md**: Estrategias de testing
- **qa.md**: Procesos de quality assurance
- **development-workflow.md**: Guía de desarrollo
- **technology-stack.md**: Inventario de tecnologías
- **dependencies.md**: Dependencias e integraciones
- **features/**: Documentación individual de features
- **README.md**: Índice de todos los documentos de conocimiento

Cada documento incluye:
- Frontmatter YAML con metadata
- Secciones comprehensivas
- Referencias file:line específicas
- Ejemplos de código donde sea relevante

### Paso 6: Sincroniza y Presenta

1. **Sincroniza los documentos de conocimiento**:
   ```bash
   git add thoughts/shared/knowledge/ && git commit -m "Add project knowledge base documentation"
   ```

2. **Presenta los resultados**:
   ```
   I've created comprehensive knowledge documentation for [Project Name] in `thoughts/shared/knowledge/`.

   Generated documents:
   - architecture.md - System architecture overview
   - folder-structure.md - Directory organization and folder structure
   - best-practices.md - Coding standards and conventions
   - testing.md - Testing strategies
   - qa.md - Quality assurance processes
   - development-workflow.md - Development guide
   - technology-stack.md - Technology inventory
   - dependencies.md - Dependencies and integrations
   - features/ - Individual feature documentation ([N] features)
   - README.md - Knowledge base index

   All documents are ready for review and can be updated as the project evolves.
   ```

## Guías Importantes

### 1. Ser Comprehensivo
- Documenta todo lo que existe, no lo que debería existir
- Incluye rutas de archivo específicas y números de línea
- Referencia ejemplos de código reales

### 2. Usar Investigación Paralela
- Siempre genera múltiples sub-agentes en paralelo
- Usa agentes especializados para diferentes áreas de investigación
- Espera que todas las tareas completen antes de sintetizar

### 3. Mantener Consistencia
- Usa formato de frontmatter consistente en todos los documentos
- Sigue los mismos patrones de estructura
- Incluye referencias de código con formato file:line

### 4. Documentar As-Is
- Documenta lo que existe, no recomendaciones
- Sé fáctico y descriptivo
- Incluye ejemplos del codebase real

### 5. Lectura de Archivos
- Siempre lee archivos mencionados COMPLETAMENTE (sin limit/offset)
- Lee archivos clave antes de generar sub-tareas
- Asegura contexto completo antes de proceder

### 6. Ordenamiento Crítico
- SIEMPRE lee archivos clave primero (paso 1)
- SIEMPRE espera que todos los sub-agentes completen (paso 3)
- SIEMPRE reúne metadata antes de escribir (paso 4)
- NUNCA escribas documentos con valores placeholder

## Personalización

El usuario puede solicitar enfoques específicos:
- Enfócate en áreas específicas (ej: "focus on backend architecture")
- Omite ciertas categorías (ej: "skip QA documentation")
- Agrega categorías personalizadas (ej: "include deployment documentation")

Adapta el análisis y generación de documentos en consecuencia.

## Flujo de Interacción de Ejemplo

```
User: /project_knowledge
Assistant: I'll analyze this project's structure and create comprehensive knowledge documentation...

[Spawning parallel research tasks]
[Waiting for all tasks to complete]
[Generating documents]

I've created comprehensive knowledge documentation for [Project Name] in `thoughts/shared/knowledge/`.

Generated documents:
- architecture.md
- folder-structure.md
- best-practices.md
- testing.md
- qa.md
- development-workflow.md
- technology-stack.md
- dependencies.md
- features/ - 5 feature documents
- README.md

All documents are ready for review.
```

## Uso

```bash
/project_knowledge
# O con enfoque específico:
/project_knowledge focus on backend architecture
```

## Ver También

- [/research_codebase](research_codebase.md) - Para investigar áreas específicas
- [/create_plan](create_plan.md) - Usa knowledge base para planificación
