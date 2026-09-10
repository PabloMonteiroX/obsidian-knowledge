# Stack — Memoria automática en C

> [!abstract] Idea clave  
> El **stack** es una estructura de almacenamiento utilizada habitualmente por las implementaciones para gestionar las **activaciones de funciones** y objetos con **automatic storage duration**.
> 
> En C:
> 
> ```c
> void function(void)
> {
>     int value;
> }
> ```
> 
> `value` tiene almacenamiento automático.
> 
> La implementación normalmente utiliza el **stack** para gestionarlo.

---

# 1. ¿Qué es el stack?

El **stack** es una estructura de memoria gestionada principalmente de forma automática durante la ejecución de las funciones.

Cuando una función es llamada:

```c
function();
```

la ejecución necesita almacenar información asociada a esa llamada.

Conceptualmente:

```text
main()
  │
  ├── local data
  ├── return information
  └── function state
         │
         ▼
      function()
         │
         ├── local data
         └── function state
```

Cada llamada genera una nueva **stack frame**.

---

# 2. Stack frame

Una **stack frame** es el espacio asociado a una llamada concreta a una función.

Ejemplo:

```c
void function(void)
{
    int value;

    value = 42;
}
```

Cuando se ejecuta:

```text
main()
   │
   ▼
function()
```

conceptualmente:

```text
STACK

┌─────────────────────┐
│ frame: function()   │
│                     │
│ value = 42          │
└─────────────────────┘
│ frame: main()       │
└─────────────────────┘
```

Cuando `function()` termina, su frame deja de ser necesario.

```text
function()
    │
    ▼
return
    │
    ▼
frame removed
```

---

# 3. ¿Qué puede contener una stack frame?

La organización exacta depende de la arquitectura y ABI, pero conceptualmente puede contener información como:

- variables automáticas;
    
- información necesaria para la llamada;
    
- dirección de retorno;
    
- registros salvados;
    
- otros datos temporales.
    

No debes asumir que todos estos elementos aparecen siempre de la misma manera.

El compilador puede optimizar considerablemente la función.

---

# 4. Ejemplo básico

```c
#include <stdio.h>

void function(void)
{
    int value;

    value = 42;

    printf("%d\n", value);
}

int main(void)
{
    function();

    return 0;
}
```

Durante `function()`:

```text
STACK

┌──────────────────┐
│ function frame   │
│                  │
│ value = 42       │
└──────────────────┘
│ main frame       │
└──────────────────┘
```

Cuando vuelve a `main()`:

```text
STACK

┌──────────────────┐
│ main frame       │
└──────────────────┘
```

---

# 5. Stack ≠ variables

No todas las variables de un programa están necesariamente en el stack.

Ejemplo:

```c
int globalValue;
```

Esta variable tiene **static storage duration**.

No es una variable automática.

Otro ejemplo:

```c
static int counter;
```

tampoco tiene automatic storage duration.

Por tanto:

```text
variable
   ≠
stack
```

La ubicación concreta depende de la clase de almacenamiento, compilador, ABI y optimizaciones.

---

# 6. Automatic storage duration

Este es el concepto importante del lenguaje C.

Una variable local normal:

```c
void function(void)
{
    int value;
}
```

tiene **automatic storage duration**.

Su almacenamiento existe durante la ejecución de la función/block correspondiente.

Conceptualmente:

```text
enter function
      │
      ▼
object exists
      │
      │ use
      ▼
leave function
      │
      ▼
object lifetime ends
```

---

# 7. Scope ≠ lifetime

No confundas:

```text
scope
```

con:

```text
lifetime
```

**Scope** determina dónde puede utilizarse un identificador.

**Lifetime** determina durante cuánto tiempo existe el objeto.

Ejemplo:

```c
void function(void)
{
    int value;

    value = 42;
}
```

`value` tiene un scope determinado por el bloque.

Su almacenamiento automático tiene un lifetime asociado a la ejecución del bloque.

Son conceptos diferentes.

---

# 8. Stack ≠ scope

Tampoco es correcto decir:

```text
scope = stack
```

Son conceptos de niveles diferentes:

```text
C language
    │
    ├── scope
    ├── lifetime
    ├── storage duration
    │
    ▼
implementation
    │
    └── stack
```

El estándar de C no dice simplemente:

> "todas las variables locales están físicamente en el stack".

La implementación decide cómo materializar el programa.

---

# 9. El compilador puede eliminar una variable del stack

Observa:

```c
void function(void)
{
    int value;

    value = 42;
}
```

Un compilador con optimización puede determinar que `value` no necesita existir como una ubicación concreta en memoria.

Puede utilizar un registro o incluso eliminar completamente la operación si no tiene efectos observables.

Por tanto:

> Una variable automática no implica necesariamente una celda física permanente en el stack.

---

# 10. Stack y punteros

Volvemos a un concepto fundamental.

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
┌───────────────┐
│ p             │──────────────┐
└───────────────┘              │
                               ▼
                              HEAP
                         ┌────────────┐
                         │ dynamic    │
                         │ object     │
                         └────────────┘
```

Aquí tenemos:

```text
p   → almacenamiento automático
*p  → almacenamiento dinámico
```

Son dos objetos diferentes.

---

# 11. Stack y heap trabajan juntos

No son dos mundos independientes.

Ejemplo:

```c
int *createValue(void)
{
    int *value;

    value = malloc(sizeof(*value));

    if (value == NULL)
        return NULL;

    *value = 42;

    return value;
}
```

Durante la función:

```text
STACK
┌────────────────┐
│ value          │──────────────┐
└────────────────┘              │
                                ▼
HEAP                       ┌──────────┐
                           │    42    │
                           └──────────┘
```

Cuando `createValue()` termina:

```text
STACK
value desaparece
```

pero:

```text
HEAP
┌──────────┐
│    42    │
└──────────┘
```

continúa existiendo.

Esto demuestra nuevamente:

> **Lifetime del puntero ≠ lifetime del objeto apuntado.**

---

# 12. ¿Por qué el objeto del heap sobrevive?

Porque el objeto tiene:

```text
allocated storage duration
```

y no automatic storage duration.

La función termina:

```text
stack frame → desaparece
```

pero:

```text
heap object → continúa existiendo
```

hasta que se libera:

```c
free(value);
```

---

# 13. Stack unwinding

Cuando una función termina, su frame deja de estar activo.

Ejemplo:

```text
main()
  │
  ▼
A()
  │
  ▼
B()
```

Durante `B()`:

```text
STACK

┌────────────┐
│ B frame    │
├────────────┤
│ A frame    │
├────────────┤
│ main frame │
└────────────┘
```

Cuando `B()` retorna:

```text
STACK

┌────────────┐
│ A frame    │
├────────────┤
│ main frame │
└────────────┘
```

Cuando `A()` retorna:

```text
STACK

┌────────────┐
│ main frame │
└────────────┘
```

---

# 14. Recursividad

La naturaleza del stack se entiende muy bien con recursion.

```c
void countdown(int n)
{
    if (n == 0)
        return;

    countdown(n - 1);
}
```

Si:

```c
countdown(3);
```

conceptualmente:

```text
STACK

┌────────────────┐
│ countdown(0)   │
├────────────────┤
│ countdown(1)   │
├────────────────┤
│ countdown(2)   │
├────────────────┤
│ countdown(3)   │
├────────────────┤
│ main()         │
└────────────────┘
```

Cada llamada necesita su propio estado.

---

# 15. Stack overflow

Si la profundidad de las llamadas crece demasiado:

```text
STACK

┌──────────────┐
│ function     │
├──────────────┤
│ function     │
├──────────────┤
│ function     │
├──────────────┤
│ function     │
├──────────────┤
│ ...          │
├──────────────┤
│ ...          │
└──────────────┘
        │
        ▼
   stack limit
        │
        ▼
  stack overflow
```

Ejemplo clásico:

```c
void function(void)
{
    function();
}
```

No existe una condición de terminación.

Las llamadas continúan creciendo hasta superar los recursos disponibles.

---

# 16. Stack overflow ≠ heap overflow

Son problemas diferentes.

### Stack overflow

Exceso de uso del almacenamiento asociado al stack.

Ejemplo:

```c
void function(void)
{
    function();
}
```

### Heap overflow

Acceso fuera de los límites de una reserva dinámica.

Ejemplo:

```c
int *numbers;

numbers = malloc(5 * sizeof(*numbers));

if (numbers == NULL)
    return 1;

numbers[5] = 42;
```

Aquí:

```text
índices válidos:
0 1 2 3 4
```

`numbers[5]` está fuera del bloque.

---

# 17. Stack buffer overflow

También puedes tener un buffer en almacenamiento automático:

```c
void function(void)
{
    char buffer[10];

    buffer[10] = 'A';
}
```

Aquí el array tiene:

```text
buffer[0] ... buffer[9]
```

`buffer[10]` está fuera de sus límites.

Esto es un:

```text
stack buffer overflow
```

---

# 18. Stack y seguridad

Los errores de memoria en el stack son relevantes en seguridad.

Por ejemplo:

```text
stack buffer overflow
```

históricamente ha sido una clase importante de vulnerabilidad.

Puede afectar a:

- datos locales;
    
- información de control;
    
- integridad de la ejecución.
    

Los sistemas modernos incorporan mitigaciones como:

```text
stack canaries
ASLR
NX / DEP
control-flow protections
```

Pero estos mecanismos pertenecen al nivel de sistema/compilador/OS, no al lenguaje C en sí.

---

# 19. Dirección del stack

En muchas arquitecturas tradicionales, el stack crece hacia direcciones inferiores:

```text
direcciones altas
       │
       ▼
┌──────────────┐
│              │
│    STACK     │
│      ↓       │
└──────────────┘
       │
       ▼
direcciones bajas
```

Pero:

> **No debes convertir esto en una regla del lenguaje C.**

Es una característica habitual de determinadas arquitecturas/ABI.

La dirección de crecimiento no forma parte del modelo abstracto de C.

---

# 20. Stack pointer

A nivel de arquitectura existe normalmente un registro utilizado para gestionar el stack.

En x86-64:

```text
RSP
```

significa:

```text
Stack Pointer
```

El registro apunta a una posición relevante del stack actual.

También existe:

```text
RBP
```

que puede utilizarse como frame pointer.

Pero los compiladores modernos pueden omitir el frame pointer cuando optimizan:

```text
-fomit-frame-pointer
```

Por tanto:

> No asumas que cada función tiene necesariamente un frame pointer tradicional.

---

# 21. Stack frame y ABI

La organización de una llamada depende de la **ABI**.

La ABI determina convenciones como:

- cómo se pasan argumentos;
    
- dónde se devuelve un resultado;
    
- qué registros debe preservar una función;
    
- cómo se organiza el stack;
    
- alineamiento;
    
- calling convention.
    

Ejemplo conceptual:

```text
caller
  │
  │ call
  ▼
callee
  │
  ├── arguments
  ├── saved state
  ├── local storage
  └── return information
```

La implementación concreta depende de la plataforma.

---

# 22. Variables grandes en el stack

Ejemplo:

```c
void function(void)
{
    char buffer[1000000];
}
```

Estás solicitando un objeto automático muy grande.

Esto puede consumir una cantidad significativa de stack.

Si el límite del stack es insuficiente:

```text
stack overflow
```

Por eso estructuras grandes o cuyo tamaño se decide dinámicamente suelen gestionarse mediante almacenamiento dinámico.

---

# 23. VLA y stack

C permite arrays de longitud variable en determinadas versiones/modos:

```c
void function(size_t count)
{
    int numbers[count];
}
```

El tamaño se conoce durante la ejecución.

Eso no significa que sea heap.

Es almacenamiento automático:

```text
count
  ↓
VLA
  ↓
automatic storage duration
```

La implementación puede utilizar el stack para materializarlo.

---

# 24. Retornar la dirección de una variable local

Error clásico:

```c
int *function(void)
{
    int value;

    value = 42;

    return &value;
}
```

Problema:

```text
function()
    │
    ▼
value existe
    │
    ▼
return
    │
    ▼
stack frame deja de estar activo
    │
    ▼
value ya no existe
```

El puntero devuelto no proporciona acceso válido a ese objeto.

Es un caso de referencia a un objeto cuyo lifetime terminó.

---

# 25. Correcto: devolver almacenamiento dinámico

```c
int *function(void)
{
    int *value;

    value = malloc(sizeof(*value));

    if (value == NULL)
        return NULL;

    *value = 42;

    return value;
}
```

Ahora:

```text
STACK
value ───────────► HEAP
                    │
                    ▼
                  42
```

Cuando termina la función:

```text
STACK
value desaparece
```

pero:

```text
HEAP
42
```

continúa existiendo.

El caller debe liberar el recurso.

---

# 26. Stack y lifetime

Modelo conceptual:

```text
function starts
      │
      ▼
stack frame active
      │
      ├── automatic objects exist
      │
      ▼
function returns
      │
      ▼
frame no longer active
      │
      ▼
automatic object lifetime ends
```

No significa necesariamente que los bytes se borren físicamente.

Significa que el objeto ya no tiene un lifetime activo y no puede utilizarse legítimamente.

---

# 27. `free()` no se utiliza para el stack

Esto es incorrecto:

```c
void function(void)
{
    int value;

    free(&value);
}
```

`value` no fue obtenido mediante una asignación dinámica compatible con `free()`.

Su almacenamiento es automático.

Su lifetime se gestiona automáticamente.

Correcto:

```c
void function(void)
{
    int value;

    value = 42;
}
```

No necesitas:

```c
free(&value);
```

---

# 28. Stack vs Heap

|Característica|Stack|Heap|
|---|---|---|
|Concepto|estructura de ejecución/almacenamiento automático habitual|almacenamiento dinámico gestionado por allocator|
|Gestión|principalmente automática|explícita mediante API|
|C típico|automatic storage duration|allocated storage duration|
|Reserva|asociada a ejecución|`malloc/calloc/realloc`|
|Liberación|automática al terminar lifetime|`free()`|
|Flexibilidad|menor|mayor|
|Recursión|genera frames adicionales|no directamente|
|Fragmentación del allocator|no aplica como heap allocator|posible|
|Error típico|stack overflow|leak/use-after-free/heap overflow|

---

# 29. Stack no significa "rápido"

Es común escuchar:

```text
stack = rápido
heap = lento
```

Es una simplificación excesiva.

El stack suele tener ventajas prácticas:

- gestión muy simple;
    
- estructura LIFO;
    
- poca metadata de allocator;
    
- buena localidad;
    
- coste bajo para muchas operaciones.
    

Pero el rendimiento real depende de:

- CPU;
    
- cachés;
    
- compilador;
    
- optimizaciones;
    
- ABI;
    
- allocator;
    
- patrón de acceso.
    

No es correcto pensar que:

```text
heap memory = slower RAM
```

---

# 30. Modelo mental correcto

No pienses:

```text
STACK = memoria temporal
HEAP = memoria permanente
```

Eso es incorrecto.

El heap tampoco es necesariamente "permanente".

Un objeto del heap puede vivir:

```text
1 microsegundo
```

o:

```text
toda la ejecución del proceso
```

dependiendo de cuándo se libere.

La diferencia fundamental es **cómo se gestiona el almacenamiento y su lifetime**.

---

# 31. Modelo completo

```text
                    PROCESS
                       │
          ┌────────────┴────────────┐
          │                         │
      AUTOMATIC                  ALLOCATED
       STORAGE                    STORAGE
          │                         │
          ▼                         ▼
        STACK                      HEAP
          │                         │
          │                    malloc()
          │                    calloc()
          │                    realloc()
          │                         │
          ▼                         ▼
    function frame              object
          │                         │
          ▼                         ▼
     return                    free()
          │                         │
          ▼                         ▼
   lifetime ends              lifetime ends
```

---

# 32. Stack + Heap + Pointer

Esta imagen mental es especialmente importante:

```text
STACK                         HEAP

┌───────────────┐
│ p             │───────────►┌───────────────┐
└───────────────┘            │ dynamic obj   │
                             │               │
                             └───────────────┘
```

`p`:

```text
pointer object
automatic storage
```

`*p`:

```text
dynamic object
allocated storage
```

Son objetos diferentes con lifetimes diferentes.

---

# 33. Las cuatro preguntas

Cuando encuentres una variable, pregunta:

### 1. ¿Qué es?

```text
pointer?
int?
array?
struct?
```

### 2. ¿Dónde tiene almacenamiento?

```text
automatic?
static?
allocated?
```

### 3. ¿Cuánto dura?

```text
scope/lifetime/storage duration
```

### 4. ¿Quién gestiona su lifetime?

```text
automatic?
free()?
```

Estas preguntas evitan gran parte de la confusión entre stack y heap.

---

# 34. Resumen senior

> [!important]  
> **Stack** no es simplemente "la memoria donde están las variables locales".
> 
> Es una forma habitual de implementar el almacenamiento asociado a las llamadas de funciones y objetos con **automatic storage duration**.
> 
> En C, debes separar:
> 
> ```text
> scope
> lifetime
> storage duration
> stack
> ```
> 
> porque no son sinónimos.

La relación práctica:

```text
automatic storage duration
            │
            ▼
      normalmente stack
            │
            ▼
       stack frame
            │
            ▼
      function call
```

Mientras que:

```text
allocated storage duration
            │
            ▼
      malloc/calloc
            │
            ▼
           heap
            │
            ▼
          free()
```

> [!tip] Regla para recordar
> 
> **Stack:** el runtime/implementación gestiona normalmente el almacenamiento asociado a la ejecución.
> 
> **Heap:** tú gestionas explícitamente el lifetime del objeto mediante la interfaz de memoria dinámica.
> 
> Y la distinción realmente importante en C es:
> 
> ```text
> automatic storage duration
>             vs
> allocated storage duration
> ```

---

# 35. Siguiente paso lógico

Después de **heap → stack**, el siguiente concepto debería ser:

```text
MEMORY LAYOUT
│
├── text / code
├── read-only data
├── data
├── bss
├── heap
└── stack
```

Y después:

```text
virtual memory
     ↓
process address space
     ↓
virtual address
     ↓
physical memory
     ↓
MMU
     ↓
pages
     ↓
page tables
```

Ahí es donde la explicación de **stack/heap deja de ser una simplificación** y empieza a conectar directamente con Linux, arquitectura de computadores y cybersecurity.