# /ci_describe_pr - Generar Descripción de PR (Variante CI)

## Descripción

Variante de `/describe_pr` diseñada para uso en CI/CD. Genera descripciones comprehensivas de Pull Requests siguiendo el template del repositorio, similar a `/describe_pr` pero adaptada para contextos de automatización.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Diferencias con `/describe_pr`

Esta es una variante del comando `/describe_pr` principal. Las diferencias clave:

- Optimizada para entornos de CI/CD
- Puede tener diferentes timeouts o configuraciones de ejecución
- El flujo es similar pero adaptado para automatización

## Proceso

El proceso es esencialmente el mismo que `/describe_pr`:

1. Lee el template de PR
2. Identifica el PR a describir
3. Verifica descripción existente
4. Recopila información comprehensiva del PR
5. Analiza los cambios exhaustivamente
6. Maneja requisitos de verificación
7. Genera la descripción
8. Guarda y sincroniza
9. Actualiza el PR

Ver [/describe_pr](describe_pr.md) para detalles completos del proceso.

## Consideraciones de CI

Cuando se ejecuta en CI:
- Puede tener acceso limitado a contexto de conversación
- Debe ser más autónomo en sus decisiones
- Puede tener timeouts diferentes
- Necesita manejar fallos de forma elegante

## Uso

```bash
/ci_describe_pr
```

## Ver También

- [/describe_pr](describe_pr.md) - Versión principal para uso interactivo
- [/ci_commit](ci_commit.md) - Variante CI del comando commit
