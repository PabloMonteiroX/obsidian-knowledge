# Study System

> [!abstract] Principio central
> El objetivo no es acumular notas. Es construir una red de modelos mentales que puedas recuperar, ejecutar, explicar y aplicar en un proyecto.

## Flujo de una informacion nueva

### 1. Capturar

Guarda solo la pregunta, la fuente y el motivo en `00 Inbox/`. No intentes ordenar mientras capturas.

```text
Pregunta: ¿por que un proceso puede quedar zombie?
Fuente: [enlace]
Motivo: necesito diagnosticar procesos Linux
```

Usa [[Templates/Source]] si la entrada viene de un libro, curso, documentacion o video.

### 2. Convertir en conocimiento

Crea una nota con [[Templates/Concept]]. Rellena siempre:

- definicion en una frase
- dependencias y enlaces
- ejemplo minimo ejecutable
- error y diagnostico
- preguntas de recuerdo

Si la nota solo resume una fuente, aun no es conocimiento propio.

### 3. Convertir en capacidad

Usa [[Templates/Procedure]] para una tarea repetible y [[Templates/Lab]] para experimentar. El laboratorio debe contener una hipotesis, una observacion, un fallo y una verificacion.

### 4. Integrar

Usa [[Templates/Project]] cuando conectes varias areas. Enlaza el proyecto desde el MOC correspondiente y registra decisiones, restricciones y trade-offs.

### 5. Recordar

Usa [[Templates/Study Session]]. Cierra las fuentes antes de responder. Programa revisiones a 1, 3, 7 y 30 dias.

## Regla de calidad de una nota

Una nota esta lista cuando responde:

1. ¿Que es?
2. ¿De que depende?
3. ¿Como se usa?
4. ¿Como falla?
5. ¿Como lo verifico?
6. ¿Con que se conecta?
7. ¿Que puedo construir con ello?

## Como investigar sin perderse

1. Formula una pregunta estrecha y comprobable.
2. Empieza por una fuente primaria o documentacion oficial.
3. Contrasta afirmaciones importantes con una segunda fuente.
4. Prueba una afirmacion en un ejemplo minimo.
5. Guarda la conclusion, no todo el texto de la fuente.
6. Enlaza la conclusion a sus dependencias y a un laboratorio.

## Como estudiar una semana

- Dia 1: fundamentos y mapa de dependencias.
- Dia 2: ejemplo minimo y variacion propia.
- Dia 3: laboratorio con un fallo introducido.
- Dia 4: recuperacion activa sin consultar notas.
- Dia 5: integrar el tema en un mini proyecto.
- Dia 6: explicar el tema y corregir lagunas.
- Dia 7: revision y decision del siguiente prerrequisito.

## Estados

```text
inbox -> learning -> understood -> practiced -> mastered
                         ^              |
                         +-- needs-review
```

`mastered` no significa saberlo todo. Significa poder explicarlo, usarlo, diagnosticar un fallo y reconocer sus limites.

## Convencion de enlaces

- Conceptos: enlaza prerrequisitos y conceptos relacionados.
- Procedimientos: enlaza el concepto que explica cada paso.
- Laboratorios: enlaza conceptos, procedimientos y proyectos.
- Proyectos: enlaza habilidades y evidencia real.
- Fuentes: enlaza las notas derivadas, no solo la URL.

## Definition of done

- [ ] La nota esta enlazada desde un MOC.
- [ ] No quedan marcadores falsos como `[rellenar]`.
- [ ] El ejemplo minimo fue verificado.
- [ ] Existe evidencia de practica.
- [ ] Hay una proxima revision.
