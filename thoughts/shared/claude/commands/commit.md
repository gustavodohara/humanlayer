# /commit - Crear Commits con Aprobación del Usuario

## Descripción

Crea commits de git para los cambios realizados durante la sesión. Presenta un plan de commits al usuario para aprobación antes de ejecutar. NO incluye atribución de Claude.

## Modelo

Por defecto (usa el modelo configurado de la sesión)

## Proceso

### 1. Piensa Sobre Qué Cambió

- Revisa el historial de conversación y entiende qué se logró
- Ejecuta `git status` para ver cambios actuales
- Ejecuta `git diff` para entender las modificaciones
- Considera si los cambios deberían ser un commit o múltiples commits lógicos

### 2. Planea Tu(s) Commit(s)

- Identifica qué archivos van juntos
- Redacta mensajes de commit claros y descriptivos
- Usa modo imperativo en mensajes de commit (ej: "Add feature" no "Added feature")
- Enfócate en **por qué** se hicieron los cambios, no solo **qué**

### 3. Presenta Tu Plan al Usuario

```
I plan to create [N] commit(s) with these changes:

Commit 1: [message]
Files:
- path/to/file1.js
- path/to/file2.ts

Commit 2: [message]
Files:
- path/to/file3.go

Shall I proceed?
```

### 4. Ejecuta Tras Confirmación

```bash
# Usa git add con archivos específicos (NUNCA uses -A o .)
git add path/to/file1.js path/to/file2.ts

# Crea commit con mensaje planeado
git commit -m "Add feature X to handle Y

This commit adds the ability to process webhooks
by implementing a new handler and validation logic."

# Muestra resultado
git log --oneline -n [number]
```

## Importante

### NUNCA Agregar:
- **Información de co-autor**: No agregues "Co-Authored-By"
- **Atribución de Claude**: No incluyas "Generated with Claude"
- **Mensajes de generación**: Los commits deben parecer escritos por el usuario

### Los Commits Deben:
- Ser autoría solo del usuario
- Usar modo imperativo ("Add" no "Added")
- Explicar el "por qué" no solo el "qué"
- Ser atómicos y enfocados cuando sea posible
- Agrupar cambios relacionados juntos

### Recuerda:
- Tienes el contexto completo de lo que se hizo en esta sesión
- Agrupar cambios relacionados juntos
- Mantener commits enfocados y atómicos cuando sea posible
- El usuario confía en tu juicio - te pidió que hagas commit
- Usar `git add` con archivos específicos, nunca flags globales

## Ejemplo de Buenos Mensajes de Commit

```
Add rate limiting to API endpoints

Implement token bucket algorithm for rate limiting with
configurable limits per endpoint. This prevents abuse and
ensures fair resource usage across clients.

Files changed:
- src/middleware/rate-limit.ts (new)
- src/api/routes.ts (updated)
- config/rate-limits.json (new)
```

## Uso

```bash
/commit
# El comando te preguntará sobre el plan de commits antes de ejecutar
```

## Ver También

- [/ci_commit](ci_commit.md) - Variante para commits en CI
- [/describe_pr](describe_pr.md) - Para crear descripción de PR después
- [/validate_plan](validate_plan.md) - Para verificar implementación antes de commit
