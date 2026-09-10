# Git — MOC

> [!info] Como trabajar este MOC
> Git debe conservar decisiones y evidencia, no convertirse en una coleccion de commits sin contexto. Cada practica debe incluir estado inicial, cambio, verificacion y forma de deshacerlo.

## Fundamentos

- [[00 Control de Versiones]]
- [[02 Instalación y Configuración de Git]]
- Commits, branches, remotes y tags
- [[Templates/Concept]]

## Flujo profesional

- [ ] Crear un commit pequeno y explicativo
- [ ] Comparar cambios con `git diff`
- [ ] Crear y fusionar una branch
- [ ] Recuperar un cambio con `git revert`
- [ ] Resolver un conflicto y documentar la decision
- [ ] Enlazar commit y laboratorio

## Ejemplo minimo

```bash
git status
git diff
git add archivo.md
git commit -m "Documenta el experimento"
git log --oneline -3
```

## Diagnostico

- Pregunta: ¿que cambio hice y por que?
- Evidencia: `git show --stat`
- Recuperacion: `git revert <commit>`
- Trade-off: commits pequenos facilitan revisar y revertir, pero exigen disciplina.

## Practica

- [ ] Completar [[Templates/Lab]] para un conflicto
- [ ] Enlazar el resultado desde un proyecto
