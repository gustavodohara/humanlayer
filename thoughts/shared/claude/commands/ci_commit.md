# /ci_commit - Crear Commits para Sesión (Variante CI)

## Descripción

Crea commits de git para cambios realizados durante la sesión con mensajes claros y atómicos. Variante del comando `/commit` diseñada para contextos de CI/CD.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Diferencias con `/commit`

Esta es una variante del comando `/commit` principal. Las diferencias clave:

- Optimizada para entornos de CI/CD donde pueden no haber conversaciones interactivas
- Puede ser más autónomo en decisiones de agrupación de commits
- El flujo es similar pero adaptado para automatización

## Proceso

El proceso es esencialmente el mismo que `/commit`:

1. Piensa sobre qué cambió
2. Planea tu(s) commit(s)
3. Presenta plan al usuario (si es interactivo)
4. Ejecuta tras confirmación

Ver [/commit](commit.md) para detalles completos del proceso.

## Guías de Commits

Siguiendo las mismas reglas que `/commit`:

### NUNCA Agregar:
- Información de co-autor
- Atribución de Claude
- Mensajes de generación

### Los Commits Deben:
- Ser autoría solo del usuario
- Usar modo imperativo
- Explicar el "por qué" no solo el "qué"
- Ser atómicos y enfocados
- Agrupar cambios relacionados

## Consideraciones de CI

Cuando se ejecuta en CI:
- Puede crear commits automáticamente sin confirmación del usuario
- Debe ser especialmente cuidadoso con la agrupación lógica de cambios
- Necesita manejar conflictos de merge elegantemente
- Debe proporcionar mensajes claros incluso sin contexto de conversación

## Uso

```bash
/ci_commit
```

## Ver También

- [/commit](commit.md) - Versión principal para uso interactivo
- [/ci_describe_pr](ci_describe_pr.md) - Variante CI de describe_pr
