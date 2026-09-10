---
type: procedure
domain: systems
level: intermediate
status: inbox
date: {{date}}
prerequisites:
  - "[[Prerequisite 1]]"
related:
  - "[[Related concept]]"
---
# {{title}}

> [!info] Objetivo
> Documenta una tarea repetible. Otra persona debe poder ejecutarla, verificarla y recuperarse de un error sin preguntarte.

## Resultado esperado

Al terminar debe ocurrir: [resultado observable].

## Antes de empezar

- [ ] Tengo permisos y entorno correctos
- [ ] Entiendo [[Prerequisite 1]]
- [ ] Tengo backup o forma de deshacer el cambio

## Pasos

1. Comando o accion: `[comando]`
   - Por que: [razon]
   - Verificacion: `[comando de comprobacion]`
2. [siguiente paso]

## Ejemplo minimo

```bash
# Usa datos de prueba y explica cada comando
[comando]
```

## Fallos y recuperacion

| Sintoma | Diagnostico | Solucion | Como evitarlo |
|---|---|---|---|
| [error] | `[comando]` | [accion] | [control] |

## Seguridad y trade-offs

- Riesgo: [que podria salir mal]
- Permisos minimos: [usuario/capacidad]
- Alternativa: [otra forma]

## Prueba de independencia

- [ ] Ejecutarlo sin mirar la nota
- [ ] Resolver un fallo introducido
- [ ] Explicarselo a otra persona

## Evidencia

- Resultado: [salida, captura, commit o enlace]
- Proxima revision: [fecha]
