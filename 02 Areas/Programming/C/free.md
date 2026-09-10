# `free()` — Liberación de memoria dinámica en C

> [!abstract] Idea clave  
> `free()` libera un bloque de memoria obtenido mediante una función de asignación dinámica.
> 
> ```c
> malloc() → usar → free()
> ```
> 
> `free()` **no libera una variable**. Libera el **bloque de memoria dinámicamente asignado** al que apunta el puntero.

---

# 1. ¿Qué es `free()`?

`free()` es una función de la biblioteca estándar de C utilizada para liberar memoria dinámica.

Está declarada en:

```c
#include <stdlib.h>
```

Su prototipo es:

```c
void free(void *ptr);
```

Ejemplo:

```c
int *p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

*p = 42;

free(p);
```

La secuencia conceptual es:

```text
malloc()
   │
   ▼
┌──────────────┐
│ memoria      │
│ reservada    │
└──────────────┘
       │
       │ usar
       ▼
     free()
       │
       ▼
 memoria liberada
```

---

# 2. ¿Qué recibe?

Recibe un puntero:

```c
void free(void *ptr);
```

Por ejemplo:

```c
int *p = malloc(sizeof(*p));

free(p);
```

No necesitas hacer cast:

```c
free((void *)p);    // innecesario
```

Simplemente:

```c
free(p);
```

---

# 3. ¿Qué hace realmente?

Supongamos:

```c
int *p = malloc(5 * sizeof(*p));
```

Tenemos:

```text
STACK                         HEAP

┌──────────┐                 ┌─────┬─────┬─────┬─────┬─────┐
│    p     │ ──────────────► │ int │ int │ int │ int │ int │
└──────────┘                 └─────┴─────┴─────┴─────┴─────┘
```

Cuando hacemos:

```c
free(p);
```

el bloque dinámico deja de estar disponible para el programa.

```text
STACK                         HEAP

┌──────────┐                 ┌───────────────────────────────┐
│    p     │ ──────────────► │     bloque liberado           │
└──────────┘                 └───────────────────────────────┘
```

Pero aquí aparece un detalle crítico:

> `free(p)` **no cambia el valor de `p`**.

`p` continúa conteniendo la antigua dirección.

---

# 4. `free()` no pone el puntero a `NULL`

Esto:

```c
free(p);
```

NO equivale a:

```c
p = NULL;
```

Después de:

```c
free(p);
```

tenemos conceptualmente:

```text
p ─────────► dirección antigua
             ❌ bloque liberado
```

Por eso es habitual:

```c
free(p);
p = NULL;
```

Ahora:

```text
p ─────────► NULL
```

### Importante

Asignar `NULL` **después** de `free()` no libera memoria.

La liberación la realiza:

```c
free(p);
```

Esto:

```c
p = NULL;
```

solo cambia el valor del puntero.

---

# 5. `free(NULL)` es seguro

Esto es válido:

```c
free(NULL);
```

No realiza ninguna acción.

Esto permite patrones como:

```c
int *p = NULL;

/* ... */

free(p);
p = NULL;
```

También facilita algunas rutas de error:

```c
int *a = malloc(...);
int *b = malloc(...);

if (a == NULL || b == NULL)
{
    free(a);
    free(b);
    return 1;
}
```

Aunque `a` o `b` puedan ser `NULL`, `free()` puede recibirlos de forma segura.

---

# 6. ¿Qué memoria puedo liberar?

Debes liberar memoria obtenida mediante una función de asignación dinámica compatible.

Por ejemplo:

```c
int *p = malloc(sizeof(*p));

free(p);
```

También:

```c
int *p = calloc(10, sizeof(*p));

free(p);
```

Y memoria gestionada mediante `realloc()` también termina liberándose con:

```c
free(p);
```

---

# 7. ¿Qué NO puedo liberar?

No puedes hacer:

```c
int x = 42;

free(&x);
```

`x` tiene almacenamiento automático.

No fue obtenido mediante `malloc()`.

---

Tampoco:

```c
int array[10];

free(array);
```

`array` es un array con almacenamiento automático.

---

Y tampoco:

```c
char *text = "hello";

free(text);
```

Ese puntero apunta a un string literal; no es un bloque que hayas obtenido mediante una asignación dinámica.

---

# 8. `use-after-free`

Uno de los errores más importantes de C:

```c
int *p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

*p = 42;

free(p);

printf("%d\n", *p);   // ERROR
```

Después de:

```c
free(p);
```

no puedes acceder al objeto liberado mediante `p`.

Esto se denomina:

> **Use-after-free**

Es **undefined behavior**.

---

# 9. ¿Por qué `p = NULL` ayuda?

Compara:

```c
free(p);

printf("%d\n", *p);
```

con:

```c
free(p);
p = NULL;

printf("%d\n", *p);
```

En el segundo caso, el error es mucho más fácil de detectar conceptualmente porque:

```text
p → NULL
```

Intentar:

```c
*p
```

sobre `NULL` sigue siendo un error, pero ya no estás conservando una dirección aparentemente válida que apunta a memoria liberada.

### Patrón

```c
free(p);
p = NULL;
```

Es una buena práctica cuando el puntero continúa existiendo y podría reutilizarse.

---

# 10. Double free

Otro error grave:

```c
int *p = malloc(sizeof(*p));

free(p);
free(p);       // ERROR
```

Estás intentando liberar dos veces el mismo bloque.

Esto produce **undefined behavior**.

Una forma de evitarlo:

```c
free(p);
p = NULL;

free(p);       // seguro
```

Porque:

```c
free(NULL);
```

es válido.

---

# 11. Memory leak

El problema contrario:

```c
int *p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

/* usamos p */

return 0;
```

Si llegamos al `return` sin:

```c
free(p);
```

hemos terminado el programa sin liberar explícitamente el bloque.

Durante la ejecución, perder una referencia sin liberar el bloque produce un:

> **memory leak**

Ejemplo más grave:

```c
void function(void)
{
    int *p = malloc(sizeof(*p));

    if (p == NULL)
        return;

    /* ... */

    return;
}
```

Cuando termina `function()`, `p` desaparece.

Pero el bloque dinámico que apuntaba `p` puede quedar sin referencia.

```text
STACK
┌───────┐
│   p   │ ─────────► HEAP
└───────┘             ┌─────────┐
                      │ bloque  │
                      └─────────┘

return
  ↓

p desaparece

                      ┌─────────┐
                      │ bloque  │ ← nadie puede acceder
                      └─────────┘
```

---

# 12. Ownership

Aquí está el concepto senior.

Cuando haces:

```c
int *p = malloc(sizeof(*p));
```

alguien debe ser responsable de ese bloque.

Ese concepto se denomina:

> **ownership**

Debes poder responder:

> ¿Quién es responsable de hacer `free()`?

Por ejemplo:

```c
int *createNumber(void)
{
    int *p = malloc(sizeof(*p));

    if (p == NULL)
        return NULL;

    *p = 42;

    return p;
}
```

Aquí la función devuelve la propiedad del bloque al caller.

```c
int *number = createNumber();

if (number == NULL)
    return 1;

/* ownership de number */

printf("%d\n", *number);

free(number);
```

Conceptualmente:

```text
createNumber()
      │
      │ malloc
      ▼
    bloque
      │
      │ return
      ▼
   caller
      │
      │ ownership
      ▼
    free()
```

---

# 13. Ownership transfer

Una función puede transferir ownership.

Ejemplo:

```c
int *createNumber(void)
{
    int *p = malloc(sizeof(*p));

    if (p == NULL)
        return NULL;

    *p = 42;

    return p;
}
```

El caller recibe:

```c
int *number = createNumber();
```

Ahora el caller es responsable de:

```c
free(number);
```

Esto permite diseñar APIs claras.

### Regla

```text
Who allocates?
Who owns?
Who frees?
```

Estas tres preguntas son fundamentales en código C con memoria dinámica.

---

# 14. `free()` no significa necesariamente "devuelve inmediatamente RAM al sistema"

Esta simplificación:

> "`free()` devuelve la RAM al sistema operativo."

no es necesariamente correcta.

El allocator puede mantener memoria para reutilizarla posteriormente dentro del proceso.

Desde el punto de vista del programa:

```c
free(p);
```

significa que el bloque deja de estar disponible para ese objeto y puede ser reutilizado por futuras operaciones de asignación.

Por tanto:

```text
free()
  ↓
bloque disponible para el allocator
```

No necesariamente:

```text
free()
  ↓
RAM inmediatamente devuelta al kernel
```

---

# 15. `free()` y el tamaño

Observa:

```c
int *p = malloc(100 * sizeof(*p));

free(p);
```

No necesitas hacer:

```c
free(p, 100);
```

`free()` solo recibe:

```c
free(p);
```

El allocator conoce internamente la información necesaria para gestionar el bloque.

Por eso:

```c
free(p);
```

es suficiente.

---

# 16. No hagas aritmética antes de `free()`

Supongamos:

```c
int *p = malloc(10 * sizeof(*p));

int *q = p + 5;

free(q);    // ERROR
```

El puntero que entregas a `free()` debe corresponder al puntero adecuado al bloque asignado.

No debes liberar una dirección interior del bloque:

```text
p
│
▼
┌────┬────┬────┬────┬────┬────┐
│ 0  │ 1  │ 2  │ 3  │ 4  │ 5  │
└────┴────┴────┴────┴────┴────┘
                          ▲
                          q

free(q)  ← ERROR
```

La dirección correcta es:

```c
free(p);
```

---

# 17. Un `free()` por cada allocation

Regla práctica:

```text
1 malloc → 1 free
```

Por ejemplo:

```c
int *a = malloc(...);
int *b = malloc(...);
int *c = malloc(...);
```

Necesitas gestionar:

```c
free(a);
free(b);
free(c);
```

Pero en código real esto se complica por las rutas de error.

---

# 18. Gestión de errores

Ejemplo:

```c
int *a = malloc(10 * sizeof(*a));

if (a == NULL)
    return 1;

int *b = malloc(20 * sizeof(*b));

if (b == NULL)
{
    free(a);
    return 1;
}
```

Aquí:

```text
malloc(a)
   │
   ├── fallo → return
   │
   └── éxito
        │
        ▼
    malloc(b)
        │
        ├── fallo → free(a)
        │
        └── éxito
```

Esto es **resource management**.

No basta con gestionar el camino feliz (_happy path_).

---

# 19. Patrón completo

```c
#include <stdlib.h>

int main(void)
{
    int *numbers;
    size_t count;

    count = 10;

    numbers = malloc(count * sizeof(*numbers));

    if (numbers == NULL)
        return 1;

    for (size_t i = 0; i < count; i++)
        numbers[i] = 0;

    /* use numbers */

    free(numbers);
    numbers = NULL;

    return 0;
}
```

El ciclo de vida es:

```text
              ALLOCATION
                   │
                   ▼
                malloc
                   │
                   ▼
              ownership
                   │
                   ▼
                 use
                   │
                   ▼
                 free
                   │
                   ▼
              pointer = NULL
```

---

# 20. `malloc` + `free` como pareja conceptual

No estudies `free()` aislado.

Piensa siempre:

```text
┌───────────────┐
│ malloc/calloc │
└───────┬───────┘
        │
        ▼
    allocation
        │
        ▼
     ownership
        │
        ▼
       use
        │
        ▼
       free
        │
        ▼
    deallocation
```

El verdadero concepto es:

> **allocation lifetime**

---

# 21. Errores que debes reconocer inmediatamente

### Memory leak

```c
p = malloc(...);
/* no free */
```

### Use-after-free

```c
free(p);
*p = 42;
```

### Double free

```c
free(p);
free(p);
```

### Invalid free

```c
int x;

free(&x);
```

### Interior pointer

```c
p = malloc(...);
free(p + 1);
```

### Puntero perdido

```c
p = malloc(...);
p = NULL;
```

---

# 22. `free()` + Valgrind

Una herramienta fundamental para detectar errores de memoria:

```bash
valgrind --leak-check=full ./program
```

Puede ayudarte a detectar:

- memory leaks;
    
- invalid reads;
    
- invalid writes;
    
- use-after-free;
    
- problemas relacionados con liberación de memoria.
    

Ejemplo:

```c
int *p = malloc(sizeof(*p));

*p = 42;
```

sin:

```c
free(p);
```

Valgrind puede reportar memoria todavía accesible o perdida dependiendo del caso y del estado de las referencias.

---

# 23. AddressSanitizer

En GCC/Clang también puedes utilizar:

```bash
-fsanitize=address
```

Por ejemplo:

```bash
gcc -Wall -Wextra -Werror -std=c17 \
    -fsanitize=address \
    main.c -o program
```

Y ejecutar:

```bash
./program
```

Es especialmente útil para detectar:

```text
use-after-free
buffer overflow
heap overflow
stack overflow
double free
```

---

# 24. Checklist profesional

Antes de hacer `free()`:

-  ¿El bloque fue obtenido dinámicamente?
    
-  ¿Tengo el puntero correcto al inicio del bloque?
    
-  ¿Este bloque sigue siendo mío?
    
-  ¿Ya se liberó anteriormente?
    
-  ¿Existe otro owner?
    
-  ¿Estoy dentro de una ruta de error?
    
-  ¿Después del `free()` existe código que pueda reutilizar el puntero?
    

Después de `free()`:

-  ¿Necesito hacer `p = NULL`?
    
-  ¿No vuelvo a acceder al bloque?
    
-  ¿No vuelvo a hacer `free()` sobre el mismo bloque?
    

---

# 25. Resumen de examen

> **`free()` libera un bloque de memoria dinámica previamente obtenido mediante una función de asignación compatible. Recibe un puntero al bloque y no devuelve ningún valor. `free(NULL)` no realiza ninguna acción. Acceder al bloque después de liberarlo, liberarlo dos veces o liberar una dirección que no corresponde a una asignación válida produce comportamiento indefinido.**

### Sintaxis

```c
void free(void *ptr);
```

### Uso

```c
int *p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

*p = 42;

free(p);
p = NULL;
```

---

# 26. Modelo mental definitivo

```text
                 malloc()
                    │
                    ▼
              ┌───────────┐
              │  BLOCK    │
              │ allocated │
              └─────┬─────┘
                    │
                    │ ownership
                    ▼
                   use
                    │
          ┌─────────┴─────────┐
          │                   │
       success              error
          │                   │
          └─────────┬─────────┘
                    ▼
                  free()
                    │
                    ▼
              BLOCK RELEASED
                    │
                    ▼
                p = NULL
```

---

# 27. Conexiones

```text
                 MEMORY MANAGEMENT
                        │
          ┌─────────────┼─────────────┐
          │             │             │
       malloc()       sizeof()      free()
          │             │             │
       allocate       calculate      release
          │             │             │
          └─────────────┼─────────────┘
                        │
                        ▼
                    ownership
                        │
                        ▼
                     lifetime
                        │
          ┌─────────────┼─────────────┐
          │             │             │
      memory leak   use-after-free  double free
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                    Valgrind
                        │
                        ▼
                 AddressSanitizer
```

> [!important] Regla senior  
> **`free()` no es simplemente "borrar memoria". Es el final del lifetime de un bloque dinámicamente asignado.**
> 
> En código C serio debes poder identificar:
> 
> **allocation → ownership → lifetime → deallocation**
> 
> Si no puedes explicar quién posee un bloque y quién debe liberarlo, tienes un problema de diseño de memoria.


### Relacionado
[[Memory Management]]
[[Stack]]
[[Heap]]
[[malloc]]
[[calloc]]
[[realloc]]