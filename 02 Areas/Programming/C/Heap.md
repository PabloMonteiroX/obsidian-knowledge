# Heap — Memoria dinámica en C

> [!abstract] Idea clave  
> El **heap** es una región de memoria utilizada por el proceso para almacenar objetos cuya gestión dinámica permite controlar su lifetime explícitamente.
> 
> En C, normalmente accedemos al heap mediante:
> 
> ```c
> malloc()
> calloc()
> realloc()
> free()
> ```
> 
> Pero:
> 
> > **Heap y `malloc()` no son sinónimos.**
> 
> `malloc()` es una función de la biblioteca estándar. El heap es parte del modelo de memoria del proceso y de la implementación/runtime que gestiona esa memoria.

---

# 1. ¿Qué es el heap?

El **heap** es memoria utilizada para almacenamiento dinámico.

Permite solicitar memoria durante la ejecución:

```c
int *p;

p = malloc(sizeof(*p));
```

La memoria solicitada pertenece al almacenamiento dinámico del proceso.

Conceptualmente:

```text
                 PROCESO
┌──────────────────────────────────┐
│                                  │
│   CODE / TEXT                    │
│                                  │
│   GLOBAL / STATIC                │
│                                  │
│   HEAP                           │
│       ↓                          │
│   memoria dinámica               │
│                                  │
│                                  │
│   STACK                          │
│                                  │
└──────────────────────────────────┘
```

> [!note]  
> Este esquema es conceptual. La organización real de memoria depende del sistema operativo, arquitectura, ABI y runtime.

---

# 2. ¿Para qué necesitamos el heap?

Porque a veces no conocemos durante la compilación cuánto almacenamiento necesitaremos.

Ejemplo:

```c
size_t count;

scanf("%zu", &count);
```

Ahora `count` depende de la entrada del usuario.

Podemos reservar:

```c
int *numbers;

numbers = malloc(count * sizeof(*numbers));
```

La cantidad de memoria se decide **en runtime**.

```text
compile time
     │
     │
     ▼
program starts
     │
     ▼
user provides count
     │
     ▼
malloc()
     │
     ▼
dynamic storage
```

---

# 3. Heap vs Stack

Esta es una distinción fundamental.

## Stack

Se utiliza principalmente para:

- llamadas a funciones;
    
- parámetros;
    
- variables automáticas;
    
- información necesaria para mantener activaciones de funciones.
    

Ejemplo:

```c
void function(void)
{
    int value;

    value = 42;
}
```

`value` tiene almacenamiento automático.

---

## Heap

Se utiliza para almacenamiento dinámico:

```c
int *value;

value = malloc(sizeof(*value));
```

Ahora el objeto dinámico tiene un lifetime controlado mediante la gestión dinámica.

```text
STACK                       HEAP

value ────────────────────► object
```

---

# 4. El error conceptual más común

No pienses:

```text
STACK = variables
HEAP = pointers
```

Eso es incorrecto.

Un **puntero también es una variable** y puede estar en el stack.

Ejemplo:

```c
void function(void)
{
    int *p;

    p = malloc(sizeof(*p));
}
```

Conceptualmente:

```text
STACK
┌──────────────┐
│ p            │──────────────► HEAP
└──────────────┘                 │
                                 ▼
                            ┌──────────┐
                            │ int      │
                            └──────────┘
```

Tenemos:

```text
p       → stack
*p      → heap
```

---

# 5. ¿Dónde está `p`?

En:

```c
int *p;
```

`p` es un objeto automático si está declarado dentro de una función sin `static`.

Por tanto, normalmente tendrá almacenamiento asociado a la activación de la función.

```text
STACK

┌──────────────┐
│ p            │
│ address: ... │
└──────┬───────┘
       │
       │ points to
       ▼
HEAP
┌──────────────┐
│ dynamic int  │
│ value: 42    │
└──────────────┘
```

---

# 6. El heap no es el puntero

Esto:

```c
int *p;
```

crea:

```text
un puntero
```

No crea automáticamente:

```text
un objeto dinámico
```

El objeto dinámico aparece cuando haces:

```c
p = malloc(sizeof(*p));
```

Entonces:

```text
STACK                 HEAP

p ─────────────────► object
```

---

# 7. `malloc()` solicita almacenamiento

Cuando haces:

```c
p = malloc(sizeof(*p));
```

estás solicitando almacenamiento dinámico.

Si tiene éxito:

```text
p
│
▼
┌──────────────┐
│ dynamic obj  │
└──────────────┘
```

Si falla:

```c
p == NULL
```

Por eso:

```c
if (p == NULL)
    return 1;
```

---

# 8. ¿Quién gestiona el heap?

No eres tú directamente.

Tu programa solicita memoria mediante:

```c
malloc()
calloc()
realloc()
```

y la libera mediante:

```c
free()
```

La implementación de la biblioteca y el sistema operativo participan en la gestión real de la memoria.

Conceptualmente:

```text
PROGRAM
   │
   │ malloc()
   ▼
C LIBRARY / ALLOCATOR
   │
   │ requests memory
   ▼
OPERATING SYSTEM
   │
   ▼
PROCESS MEMORY
```

---

# 9. `malloc()` no significa necesariamente "mover el program break"

En explicaciones antiguas se presenta el heap como:

```text
          heap
           ↓
           ↑
program break
```

y `malloc()` como una operación que mueve el **program break**.

Esto es útil históricamente para entender `brk()`/`sbrk()`, pero es una simplificación.

Los allocators modernos pueden utilizar diferentes mecanismos, incluyendo:

- `brk`/`sbrk`;
    
- `mmap`;
    
- arenas;
    
- caches;
    
- diferentes estrategias de asignación.
    

Por tanto:

> No definas el heap moderno simplemente como "memoria entre el final del programa y el stack".

---

# 10. Fragmentación

El heap puede fragmentarse.

Supongamos:

```text
HEAP

┌──────┬──────┬──────┬──────┬──────┐
│ A    │ B    │ C    │ D    │ E    │
└──────┴──────┴──────┴──────┴──────┘
```

Liberamos:

```text
B
D
```

Resultado conceptual:

```text
┌──────┬──────┬──────┬──────┬──────┐
│ A    │ free │ C    │ free │ E    │
└──────┴──────┴──────┴──────┴──────┘
```

Tenemos memoria libre, pero dividida en bloques.

Esto es **fragmentation**.

---

# 11. Fragmentación externa

La **fragmentación externa** aparece cuando existen huecos libres separados.

Ejemplo:

```text
┌────┬────┬────┬────┬────┬────┐
│ A  │free│ B  │free│ C  │free│
└────┴────┴────┴────┴────┴────┘
```

Puede existir suficiente memoria libre total para una petición, pero no necesariamente un bloque contiguo adecuado según las restricciones del allocator.

---

# 12. Fragmentación interna

La **fragmentación interna** aparece cuando se asigna más espacio del estrictamente solicitado debido a alineamiento, metadata, tamaños de bins u otras decisiones del allocator.

Conceptualmente:

```text
requested
┌───────────────┐
│     data      │
└───────────────┘

allocated
┌────────────────────┐
│ data │ overhead    │
└────────────────────┘
```

No confundas esto con memoria que tú hayas perdido necesariamente por un bug.

Parte puede ser overhead normal del allocator.

---

# 13. Heap y lifetime

Aquí conectamos con el concepto anterior.

Cuando haces:

```c
p = malloc(sizeof(*p));
```

comienza el lifetime del objeto dinámico.

Cuando haces:

```c
free(p);
```

termina su lifetime.

```text
malloc()
   │
   ▼
┌──────────────┐
│    OBJECT    │
│              │
│    VALID     │
└──────────────┘
   │
   │ free()
   ▼
lifetime ends
```

---

# 14. Heap y ownership

Después de:

```c
p = malloc(sizeof(*p));
```

alguien debe ser responsable del bloque.

Ejemplo:

```c
int *createValue(void)
{
    int *p;

    p = malloc(sizeof(*p));

    if (p == NULL)
        return NULL;

    *p = 42;

    return p;
}
```

El caller recibe el puntero:

```c
int *value;

value = createValue();

if (value == NULL)
    return 1;

free(value);
value = NULL;
```

El ownership se ha transferido.

---

# 15. Heap y aliasing

Podemos tener múltiples punteros al mismo objeto:

```c
int *p;
int *q;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

q = p;
```

Conceptualmente:

```text
STACK

p ─────┐
       │
       ▼
     HEAP
   ┌────────┐
   │ object │
   └────────┘
       ▲
       │
q ─────┘
```

Hay:

```text
1 objeto
2 punteros
```

No:

```text
2 objetos
```

---

# 16. `free()` no "borra el heap"

Esto:

```c
free(p);
```

no significa:

```text
vaciar el heap
```

Significa:

> liberar el bloque dinámico asociado a esa reserva.

Ejemplo:

```text
HEAP

┌──────┬──────┬──────┬──────┐
│ A    │ B    │ C    │ D    │
└──────┴──────┴──────┴──────┘

free(B)

┌──────┬──────┬──────┬──────┐
│ A    │ free │ C    │ D    │
└──────┴──────┴──────┴──────┘
```

El allocator puede reutilizar posteriormente ese espacio.

---

# 17. `free()` no necesariamente devuelve inmediatamente memoria al sistema operativo

Otro error conceptual.

```c
free(p);
```

significa que el bloque queda disponible para el allocator.

No debes asumir:

```text
free()
  ↓
RAM inmediatamente devuelta al OS
```

Puede ocurrir:

```text
free()
  ↓
allocator recupera el bloque
  ↓
lo reutiliza posteriormente
```

El comportamiento exacto depende de la implementación del allocator y del sistema operativo.

---

# 18. Heap no significa "memoria lenta"

Es una simplificación decir:

```text
stack = rápida
heap = lenta
```

El acceso a un objeto del heap sigue siendo acceso a memoria RAM/cache como cualquier otro.

La diferencia principal está en:

- gestión del almacenamiento;
    
- lifetime;
    
- coste de asignación/liberación;
    
- localidad;
    
- metadata del allocator;
    
- fragmentación;
    
- comportamiento de caché.
    

El coste de `malloc()`/`free()` puede ser relevante, pero no porque "la RAM del heap sea más lenta".

---

# 19. Heap y caché

El CPU no distingue conceptualmente:

```text
"esto viene del heap"
```

La CPU trabaja con direcciones de memoria y sus jerarquías de caché.

Por tanto:

```text
heap
```

no significa:

```text
sin caché
```

La localidad de acceso sigue siendo importante.

Un array dinámico:

```c
int *numbers;

numbers = malloc(1000 * sizeof(*numbers));
```

puede tener una excelente localidad espacial.

---

# 20. Heap y `realloc()`

Ahora `realloc()` tiene sentido:

```c
tmp = realloc(numbers, newSize);
```

El allocator puede:

### Caso A — ampliar en el mismo lugar

```text
ANTES

numbers ──► [ A B C ]

DESPUÉS

numbers ──► [ A B C D E F ]
```

### Caso B — mover el bloque

```text
ANTES

numbers ──► [ A B C ]


DESPUÉS

numbers ──► [ A B C D E F ]
```

El bloque puede haber cambiado de dirección.

Por eso el puntero temporal:

```c
tmp = realloc(numbers, newSize);
```

es importante.

---

# 21. Heap y stack no son enemigos

No debes pensar:

```text
"Si uso heap, no uso stack."
```

Un programa C normal utiliza ambas formas de almacenamiento.

Ejemplo:

```c
int *createArray(size_t count)
{
    int *array;

    array = malloc(count * sizeof(*array));

    if (array == NULL)
        return NULL;

    return array;
}
```

Durante la ejecución:

```text
STACK
┌──────────────┐
│ count        │
│ array        │───────────────┐
└──────────────┘               │
                               ▼
                            HEAP
                         ┌──────────┐
                         │  array   │
                         └──────────┘
```

El stack participa en la ejecución aunque el objeto principal esté en el heap.

---

# 22. ¿Puedo utilizar solamente heap?

En un programa C normal, no debes plantearlo como:

```text
"Quiero eliminar completamente el stack."
```

Las funciones, llamadas, retornos y demás mecanismos de ejecución requieren almacenamiento asociado a las activaciones y al estado de ejecución.

Puedes diseñar determinadas estructuras para utilizar almacenamiento dinámico, pero eso no significa eliminar el stack del proceso.

---

# 23. Stack vs Heap

|Característica|Stack|Heap|
|---|---|---|
|Gestión principal|automática|explícita/dinámica|
|Asignación|ligada a ejecución de funciones|`malloc/calloc/realloc`|
|Liberación|automática al finalizar lifetime|`free()`|
|Lifetime|normalmente asociado al scope/activación|explícitamente gestionado|
|Tamaño|limitado|normalmente mayor/flexible|
|Fragmentación|no suele tratarse como allocator general|posible|
|Ownership explícito|normalmente no|fundamental|
|Riesgo típico|stack overflow|leaks, use-after-free|

> [!warning]  
> "Stack = scope" tampoco es una equivalencia exacta. Scope y storage duration son conceptos del lenguaje; stack y heap son términos de implementación.

---

# 24. Storage duration

Para razonar correctamente en C, conviene introducir **storage duration**.

C define categorías como:

- automatic;
    
- static;
    
- allocated;
    
- thread.
    

La memoria dinámica corresponde a **allocated storage duration**.

Ejemplo:

```c
int *p;

p = malloc(sizeof(*p));
```

El objeto obtenido tiene allocated storage duration.

Esto es más preciso que decir simplemente:

```text
"está en el heap"
```

---

# 25. Heap no es un concepto completamente definido por el estándar C

Esto es importante a nivel senior.

El estándar de C define:

```text
malloc()
calloc()
realloc()
free()
```

y las propiedades de los objetos y su almacenamiento.

Pero no exige que exista literalmente una región física llamada:

```text
HEAP
```

como aparece en los diagramas.

"Heap" es principalmente un concepto práctico de implementación.

Por eso:

> **C standard:** allocated storage duration.  
> **Implementación/OS:** heap, allocator, arenas, mappings, etc.

---

# 26. El modelo correcto

Evita pensar:

```text
C
│
├── stack
└── heap
```

como si fueran reglas absolutas del lenguaje.

Mejor:

```text
C abstract machine
       │
       ▼
storage duration
       │
       ├── automatic
       ├── static
       ├── allocated
       └── thread
                 │
                 ▼
        implementation
                 │
                 ▼
       stack / heap / mappings
```

Esta distinción es importante cuando avances hacia:

- sistemas operativos;
    
- ABI;
    
- linker;
    
- loaders;
    
- virtual memory;
    
- Linux internals.
    

---

# 27. Errores clásicos relacionados con heap

## Memory leak

```c
int *p;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

/* falta free(p) */
```

---

## Use-after-free

```c
free(p);

*p = 42;
```

---

## Double free

```c
free(p);
free(p);
```

---

## Dangling pointer

```c
free(p);

/* p conserva una dirección que ya no puede usarse para acceder al objeto */
```

---

## Buffer overflow

```c
int *numbers;

numbers = malloc(5 * sizeof(*numbers));

if (numbers == NULL)
    return 1;

numbers[5] = 42;
```

El último índice válido es:

```text
4
```

No:

```text
5
```

---

# 28. Heap y seguridad

Los errores de memoria dinámica son especialmente importantes en seguridad informática.

Errores como:

```text
heap buffer overflow
use-after-free
double free
heap corruption
```

pueden producir:

- crashes;
    
- corrupción de datos;
    
- comportamiento indefinido;
    
- vulnerabilidades de seguridad.
    

Por eso estudiar heap no es solo aprender memoria:

> Es una base de **systems programming** y de seguridad ofensiva/defensiva.

---

# 29. Herramientas

Para investigar errores relacionados con heap:

## Valgrind

```bash
valgrind --leak-check=full ./program
```

Puede detectar, entre otros:

```text
Invalid read
Invalid write
Use after free
Memory leak
```

## AddressSanitizer

Compilación:

```bash
gcc -Wall -Wextra -Werror -std=c17 \
    -fsanitize=address \
    main.c -o program
```

Ejecución:

```bash
./program
```

Es especialmente útil para encontrar errores de memoria durante desarrollo.

---

# 30. Modelo mental definitivo

```text
                    PROGRAM
                       │
          ┌────────────┴────────────┐
          │                         │
      AUTOMATIC                  ALLOCATED
       STORAGE                    STORAGE
          │                         │
       usually                   malloc()
       stack                     calloc()
                                 realloc()
                                    │
                                    ▼
                                   HEAP
                                    │
                           ┌────────┴────────┐
                           │                 │
                       ownership          lifetime
                           │                 │
                           └────────┬────────┘
                                    │
                                 aliasing
                                    │
                                    ▼
                              pointer validity
                                    │
                                    ▼
                                  free()
                                    │
                                    ▼
                            lifetime terminates
```

---

# 31. Reglas senior

> [!important] 1. Heap ≠ `malloc()`
> 
> `malloc()` es una API. El heap es una abstracción de implementación para gestionar almacenamiento dinámico.

> [!important] 2. Pointer ≠ object
> 
> ```c
> int *p;
> ```
> 
> crea un puntero.
> 
> ```c
> p = malloc(sizeof(*p));
> ```
> 
> proporciona almacenamiento dinámico para un objeto.

> [!important] 3. `free()` termina el lifetime
> 
> Después de:
> 
> ```c
> free(p);
> ```
> 
> no puedes acceder al objeto mediante `p`.

> [!important] 4. Heap ≠ memoria lenta
> 
> El rendimiento depende de allocation overhead, locality, cache behavior, fragmentation y otros factores.

> [!important] 5. Stack ≠ scope
> 
> Scope, storage duration y ubicación física son conceptos diferentes.

> [!important] 6. El heap pertenece al proceso
> 
> No es una región independiente de la RAM reservada exclusivamente para C.

---

# 32. Resumen de examen

> **El heap es una forma de describir el almacenamiento dinámico gestionado por la implementación de un programa. En C, las funciones `malloc()`, `calloc()`, `realloc()` y `free()` proporcionan la interfaz estándar para trabajar con allocated storage. El estándar de C no obliga a que exista literalmente una región denominada heap; esa organización depende de la implementación y del sistema operativo.**

---

# 33. Qué debes dominar antes de avanzar

```text
STACK
   │
   ├── automatic storage
   │
   └── function execution


HEAP
   │
   ├── allocated storage
   ├── malloc()
   ├── calloc()
   ├── realloc()
   └── free()
```

Y sobre todo:

```text
pointer
   ↓
object
   ↓
ownership
   ↓
lifetime
   ↓
aliasing
   ↓
validity
```

Si entiendes esta cadena, `malloc()` deja de ser simplemente:

```text
"una función que reserva memoria"
```

y pasa a ser parte de un **modelo de gestión de recursos**.

---

# 34. Próximo concepto

La siguiente pieza lógica es:

```text
HEAP
  ↓
MEMORY LAYOUT
  ↓
STACK
  ↓
STATIC / GLOBAL
  ↓
TEXT / CODE
  ↓
DATA / BSS
  ↓
VIRTUAL MEMORY
```

Después de eso, tiene sentido entrar en:

```text
pointer
    ↓
address
    ↓
indirection
    ↓
pointer arithmetic
    ↓
arrays
    ↓
buffer
```

Ese orden te dará una base mucho más sólida para **C, Linux internals y cybersecurity**.