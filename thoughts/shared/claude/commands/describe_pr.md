# /describe_pr - Generar Descripción de Pull Request

## Descripción

Genera descripciones comprehensivas de Pull Requests siguiendo el template del repositorio. Lee el template, analiza los cambios, ejecuta verificaciones y actualiza el PR con una descripción completa.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Proceso

### 1. Lee el Template de PR

- Verifica si existe `thoughts/shared/pr_description.md`
- Si no existe, informa al usuario que necesita crear el template
- Lee el template cuidadosamente para entender todas las secciones y requisitos

### 2. Identifica el PR a Describir

- Verifica si la rama actual tiene un PR asociado: `gh pr view --json url,number,title,state`
- Si no hay PR o estás en main/master, lista PRs abiertos
- Pregunta al usuario qué PR quiere describir

### 3. Verifica Descripción Existente

- Verifica si `thoughts/shared/prs/{number}_description.md` ya existe
- Si existe, léela e informa al usuario que la actualizarás
- Considera qué ha cambiado desde la última descripción

### 4. Recopila Información Comprehensiva del PR

```bash
gh pr diff {number}                    # Obtén el diff completo
gh pr view {number} --json commits     # Historial de commits
gh pr view {number} --json baseRefName # Rama base
gh pr view {number} --json url,title,number,state  # Metadata
```

### 5. Analiza los Cambios Exhaustivamente

**Ultrathink** sobre los cambios de código, sus implicaciones arquitectónicas e impactos potenciales:

- Lee todo el diff cuidadosamente
- Para contexto, lee archivos referenciados pero no mostrados en el diff
- Entiende el propósito e impacto de cada cambio
- Identifica cambios user-facing vs detalles de implementación interna
- Busca breaking changes o requisitos de migración

### 6. Maneja Requisitos de Verificación

- Busca items de checklist en la sección "How to verify it" del template
- Para cada paso de verificación:
  - Si es un comando que puedes ejecutar (`make check test`, `npm test`), ejecútalo
  - Si pasa, marca el checkbox como marcado: `- [x]`
  - Si falla, déjalo sin marcar y nota qué falló: `- [ ]` con explicación
  - Si requiere pruebas manuales (UI, servicios externos), déjalo sin marcar y nota para el usuario
- Documenta cualquier paso de verificación que no pudiste completar

### 7. Genera la Descripción

Llena cada sección del template exhaustivamente:
- Responde cada pregunta/sección basándote en tu análisis
- Sé específico sobre problemas resueltos y cambios realizados
- Enfócate en impacto al usuario donde sea relevante
- Incluye detalles técnicos en secciones apropiadas
- Escribe una entrada de changelog concisa
- Asegúrate de que todos los items del checklist estén abordados

### 8. Guarda y Sincroniza la Descripción

```bash
# Guarda el documento
thoughts/shared/prs/{number}_description.md

# Sincroniza thoughts
humanlayer thoughts sync

# Muestra al usuario la descripción generada
```

### 9. Actualiza el PR

```bash
# Actualiza descripción del PR directamente
gh pr edit {number} --body-file thoughts/shared/prs/{number}_description.md

# Confirma que la actualización fue exitosa
# Si quedan pasos de verificación sin marcar, recuerda al usuario completarlos
```

## Notas Importantes

- **Funciona cross-repo**: Siempre lee el template local
- **Sé exhaustivo pero conciso**: Las descripciones deben ser escaneables
- **Enfócate en el "por qué"**: Tanto como en el "qué"
- **Breaking changes prominentes**: Incluye cambios que rompen compatibilidad o notas de migración
- **PRs multi-componente**: Organiza la descripción en consecuencia
- **Ejecuta verificaciones**: Intenta siempre ejecutar comandos de verificación cuando sea posible
- **Comunica claramente**: Qué pasos de verificación necesitan pruebas manuales

## Uso

```bash
/describe_pr
# Se te pedirá seleccionar un PR o usará el PR de la rama actual
```

## Ver También

- [/ci_describe_pr](ci_describe_pr.md) - Variante para CI
- [/commit](commit.md) - Para crear commits antes del PR
- [/validate_plan](validate_plan.md) - Para verificar implementación
