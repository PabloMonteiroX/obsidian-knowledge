# Memory Management — Ownership, Lifetime, Aliasing y Validez de Punteros

> [!abstract] Idea central  
> La memoria dinámica en C no se domina memorizando `malloc()` y `free()`.
> 
> Se domina entendiendo:
> 
> ```text
> ownership
>     ↓
> lifetime
>     ↓
> aliasing
>     ↓
> pointer validity
> ```
> 
> Estos conceptos explican la mayoría de los errores graves relacionados con memoria en C.

---

# 1. Ownership

## ¿Qué significa ownership?

**Ownership** significa **responsabilidad sobre un recurso**.

En memoria dinámica:

> El owner de un bloque es quien tiene la responsabilidad de determinar cuándo deja de utilizarlo y liberarlo correctamente.

Ejemplo:

```c
int *numbers;

numbers = malloc(10 * sizeof(*numbers));

if (numbers == NULL)
    return 1;
```

Después de `malloc()` alguien debe ser responsable del bloque.

```text
numbers
   │
   ▼
┌────────────────────┐
│ dynamic memory     │
└────────────────────┘
        ▲
        │
      owner
```

---

# 2. ¿Por qué importa el ownership?

Porque C no tiene **garbage collector**.

El lenguaje no sabe automáticamente:

```text
"Este bloque ya no se utiliza."
```

Por tanto, debes diseñar quién es responsable.

Ejemplo:

```c
int *createNumber(void)
{
    int *number;

    number = malloc(sizeof(*number));

    if (number == NULL)
        return NULL;

    *number = 42;

    return number;
}
```

El bloque creado dentro de `createNumber()` continúa existiendo después de que termine la función.

El ownership pasa al caller:

```c
int *number;

number = createNumber();

if (number == NULL)
    return 1;

/* caller owns number */

free(number);
number = NULL;
```

Conceptualmente:

```text
createNumber()
      │
      │ malloc()
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

# 3. Ownership transfer

El ownership puede **transferirse**.

Antes:

```text
function A
    │
    ▼
  bloque
```

Después de devolverlo:

```text
function A
    │
    │ transfer
    ▼
  caller
    │
    ▼
  bloque
```

Ejemplo:

```c
int *createBuffer(size_t count)
{
    int *buffer;

    buffer = malloc(count * sizeof(*buffer));

    if (buffer == NULL)
        return NULL;

    return buffer;
}
```

La función crea el recurso, pero el caller recibe la responsabilidad.

---

# 4. Regla mental del ownership

Para cada bloque dinámico debes poder responder:

```text
WHO OWNS IT?
```

Después:

```text
WHO USES IT?
```

Y finalmente:

```text
WHO FREES IT?
```

Si la respuesta no está clara, el diseño es propenso a errores.

---

# 5. Lifetime

**Lifetime** es el período durante el cual un objeto existe y puede utilizarse legítimamente.

Para memoria dinámica:

```text
allocation
     │
     ▼
┌─────────────┐
│   lifetime  │
│             │
│    valid    │
│             │
└─────────────┘
     │
     ▼
deallocation
```

Ejemplo:

```c
int *p;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

*p = 42;

free(p);
```

El objeto dinámico existe desde la asignación hasta la liberación.

---

# 6. Lifetime ≠ scope

Este es un concepto importante.

**Scope** y **lifetime** no son lo mismo.

### Scope

Determina dónde puede utilizarse un identificador.

### Lifetime

Determina cuánto tiempo existe el objeto.

Ejemplo:

```c
int *createNumber(void)
{
    int *p;

    p = malloc(sizeof(*p));

    if (p == NULL)
        return NULL;

    *p = 42;

    return p;
}
```

La variable:

```c
p
```

deja de existir cuando termina `createNumber()`.

Pero el objeto dinámico:

```text
*p
```

puede continuar existiendo.

```text
STACK
┌─────────┐
│    p    │ ───────────► HEAP
└─────────┘              │
                         ▼
                    ┌────────┐
                    │   42   │
                    └────────┘

return
  │
  ▼

p desaparece

HEAP
┌────────┐
│   42   │
└────────┘
   ▲
   │
   │ continúa existiendo
```

Por eso:

> El scope de un puntero puede terminar mientras el lifetime del objeto apuntado continúa.

---

# 7. Lifetime de memoria dinámica

El patrón fundamental:

```text
malloc()
   │
   ▼
object lifetime starts
   │
   │
   │       valid use
   │
   ▼
free()
   │
   ▼
object lifetime ends
```

Después de `free()`:

```text
❌ no puedes utilizar el objeto
```

---

# 8. Use-after-free

Este error consiste en utilizar un objeto después de terminar su lifetime.

```c
int *p;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

*p = 42;

free(p);

*p = 100;      /* ERROR */
```

Después de:

```c
free(p);
```

el objeto ya no tiene lifetime activo.

El acceso posterior produce **undefined behavior**.

---

# 9. Dangling pointer

Un **dangling pointer** es un puntero que conserva una dirección asociada a un objeto cuyo lifetime ya terminó.

Ejemplo:

```c
int *p;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

free(p);
```

Ahora:

```text
p ─────────► memoria cuyo objeto ya no existe
```

`p` sigue siendo una variable válida.

Pero el objeto al que anteriormente apuntaba ya no lo es.

Por eso:

```c
*p
```

es inválido.

---

# 10. `p = NULL`

Después de:

```c
free(p);
```

puedes hacer:

```c
p = NULL;
```

Esto elimina el dangling pointer en `p`.

```text
ANTES

p ───────► objeto liberado


DESPUÉS

p ───────► NULL
```

Pero atención:

```c
p = NULL;
```

**no libera memoria**.

La liberación fue:

```c
free(p);
```

---

# 11. Aliasing

**Aliasing** significa que existen varios punteros que permiten acceder al mismo objeto.

Ejemplo:

```c
int *p;
int *q;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

q = p;
```

Ahora:

```text
p ─────┐
       │
       ▼
   ┌────────┐
   │ objeto │
   └────────┘
       ▲
       │
q ─────┘
```

`p` y `q` son dos punteros diferentes.

Pero ambos apuntan al mismo objeto.

---

# 12. ¿Por qué el aliasing importa?

Porque modificar mediante uno afecta al mismo objeto visto mediante el otro.

```c
*p = 42;

printf("%d\n", *q);
```

Resultado:

```text
42
```

Porque:

```text
p ─────► objeto ◄───── q
```

No existen dos objetos.

Existe:

```text
1 objeto
2 punteros
```

---

# 13. Aliasing y ownership

Aquí aparece una distinción fundamental:

> **Aliasing no implica ownership.**

Podemos tener:

```text
owner ───────┐
             │
             ▼
          objeto
             ▲
             │
observer ────┘
```

Por ejemplo:

```c
int *p;
int *q;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

q = p;
```

Puedes decidir conceptualmente:

```text
p = owner
q = observer
```

Entonces:

```c
free(p);
```

termina el lifetime del objeto.

Y ahora `q` queda dangling.

```text
q ─────► ❌
```

---

# 14. Double free mediante aliasing

Este error es especialmente importante.

```c
int *p;
int *q;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

q = p;

free(p);
free(q);       /* ERROR */
```

¿Por qué?

Porque:

```text
p ─────┐
       ▼
     objeto
       ▲
       │
q ─────┘
```

`p` y `q` apuntaban al mismo bloque.

El primer:

```c
free(p);
```

ya liberó el objeto.

El segundo:

```c
free(q);
```

intenta liberarlo otra vez.

Esto es **double free** y produce undefined behavior.

---

# 15. Pointer validity

La **validez de un puntero** depende del estado del objeto o región de memoria a la que apunta y del contexto en el que se utiliza.

Un puntero puede existir como variable pero dejar de ser válido para acceder al objeto.

Ejemplo:

```c
int *p;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

free(p);
```

La variable:

```text
p
```

sigue existiendo.

Pero ya no puedes utilizarla para acceder al objeto liberado.

---

# 16. Puntero válido ≠ puntero no NULL

Este error conceptual es muy común.

No es suficiente comprobar:

```c
if (p != NULL)
```

para concluir:

```text
"p es válido."
```

Ejemplo:

```c
free(p);

if (p != NULL)
{
    /* ❌ p puede ser dangling */
}
```

Después de `free()` el valor de `p` normalmente no cambia.

Por tanto puede seguir siendo diferente de `NULL`.

```text
p != NULL
    ≠
p apunta a un objeto válido
```

---

# 17. `realloc()` y pointer validity

`realloc()` hace que estos conceptos sean todavía más importantes.

Supongamos:

```c
int *p;
int *q;

p = malloc(10 * sizeof(*p));

if (p == NULL)
    return 1;

q = p;
```

Ahora:

```text
p ─────┐
       ▼
     bloque
       ▲
       │
q ─────┘
```

Después:

```c
p = realloc(p, 20 * sizeof(*p));
```

Si `realloc()` mueve el bloque:

```text
p ─────────► nuevo bloque

q ─────────► dirección antigua
               ❌
```

`q` puede quedar invalidado.

---

# 18. Regla crítica de `realloc()`

Después de un `realloc()` exitoso:

```text
los punteros derivados del bloque anterior
pueden dejar de ser válidos
```

Por eso:

```c
int *element;

element = &numbers[5];

tmp = realloc(numbers, newSize);

if (tmp == NULL)
    return 1;

numbers = tmp;
```

No debes asumir automáticamente que:

```c
element
```

sigue apuntando al elemento correcto.

---

# 19. Ownership + Lifetime + Aliasing

Estos tres conceptos están conectados.

Ejemplo:

```c
int *owner;
int *alias;

owner = malloc(sizeof(*owner));

if (owner == NULL)
    return 1;

alias = owner;
```

Tenemos:

```text
owner ─────┐
           │
           ▼
         objeto
           ▲
           │
alias ─────┘
```

### Ownership

`owner` es responsable.

### Aliasing

`alias` también apunta al objeto.

### Lifetime

El objeto existe hasta:

```c
free(owner);
```

Después:

```text
owner ─────► memoria liberada
alias ─────► memoria liberada
```

Ambos valores pueden seguir existiendo como variables, pero ya no pueden utilizarse para acceder al objeto.

---

# 20. Modelo completo

```text
                 malloc()
                    │
                    ▼
               allocation
                    │
                    ▼
                ownership
                    │
                    ▼
                 lifetime
                    │
             ┌──────┴──────┐
             │             │
          pointer       pointer
             │             │
             └──────┬──────┘
                    │
                 aliasing
                    │
                    ▼
                   use
                    │
                    ▼
                  free()
                    │
                    ▼
             lifetime ends
                    │
             ┌──────┴──────┐
             │             │
       dangling ptr   invalid access
             │             │
             └──────┬──────┘
                    ▼
             undefined behavior
```

---

# 21. Los cuatro conceptos en una frase

### Ownership

> ¿Quién es responsable del recurso?

### Lifetime

> ¿Durante cuánto tiempo existe el objeto?

### Aliasing

> ¿Cuántos punteros pueden referirse al mismo objeto?

### Pointer validity

> ¿Puedo utilizar este puntero para acceder legítimamente al objeto en este momento?

---

# 22. Ejemplo completo

```c
#include <stdlib.h>

int main(void)
{
    int *owner;
    int *alias;

    owner = malloc(sizeof(*owner));

    if (owner == NULL)
        return 1;

    *owner = 42;

    alias = owner;

    *alias = 100;

    free(owner);
    owner = NULL;

    /*
     * alias es ahora un dangling pointer.
     *
     * No utilizar:
     *
     * printf("%d\n", *alias);
     */

    return 0;
}
```

Analízalo conceptualmente:

```text
malloc()
   ↓
object created
   ↓
owner owns object
   ↓
alias created
   ↓
two pointers / one object
   ↓
free(owner)
   ↓
object lifetime ends
   ↓
alias becomes dangling
```

---

# 23. Error de diseño: ownership ambiguo

Imagina una API:

```c
int *getData(void);
```

Y nadie sabe quién debe hacer:

```c
free()
```

Entonces aparecen preguntas:

```text
¿La función devuelve ownership?
¿El caller debe liberar?
¿La función mantiene ownership?
¿Puede almacenarse el puntero?
¿Puede modificarse?
¿Sigue siendo válido después de otra operación?
```

Una API C robusta debe dejar estas reglas claras.

---

# 24. Contrato de memoria

Cuando diseñes una función que trabaja con memoria dinámica, piensa en un **memory contract**:

```text
INPUT
  │
  ▼
¿Quién posee el objeto?
  │
  ▼
¿Quién puede modificarlo?
  │
  ▼
¿Cuánto dura?
  │
  ▼
¿Quién lo libera?
```

Ejemplo:

```c
int *createBuffer(size_t count);
```

Contrato conceptual:

```text
createBuffer()
    │
    ├── allocates
    │
    ├── returns ownership
    │
    └── caller must free()
```

---

# 25. Herramientas para detectar estos errores

## Valgrind

```bash
valgrind --leak-check=full ./program
```

Especialmente útil para:

```text
memory leaks
invalid reads
invalid writes
use-after-free
```

## AddressSanitizer

```bash
-fsanitize=address
```

Por ejemplo:

```bash
gcc -Wall -Wextra -Werror -std=c17 \
    -fsanitize=address \
    main.c -o program
```

Puede detectar muchos errores relacionados con:

```text
heap overflow
use-after-free
double free
invalid memory access
```

---

# 26. Checklist senior

Cuando veas un puntero dinámico, pregunta:

### Ownership

-  ¿Quién posee el bloque?
    
-  ¿Quién debe liberarlo?
    
-  ¿Puede transferirse el ownership?
    

### Lifetime

-  ¿Cuándo comienza?
    
-  ¿Cuándo termina?
    
-  ¿Estoy accediendo después de `free()`?
    

### Aliasing

-  ¿Hay otros punteros al mismo objeto?
    
-  ¿Quién los controla?
    
-  ¿Pueden quedar dangling?
    

### Pointer validity

-  ¿El objeto sigue existiendo?
    
-  ¿El puntero apunta al bloque correcto?
    
-  ¿Un `realloc()` pudo invalidarlo?
    
-  ¿Estoy comprobando simplemente `NULL` cuando debería comprobar algo más?
    

---

# 27. Conexión con `malloc`, `calloc`, `realloc` y `free`

```text
                    MEMORY
                      │
        ┌─────────────┼─────────────┐
        │             │             │
     malloc()       calloc()      realloc()
        │             │             │
        └─────────────┼─────────────┘
                      │
                      ▼
                  allocation
                      │
                      ▼
                  ownership
                      │
                      ▼
                   lifetime
                      │
                      ▼
                   aliasing
                      │
                      ▼
                pointer validity
                      │
                      ▼
                    free()
                      │
                      ▼
               lifetime ends
```

---

# 28. Modelo mental definitivo

> [!important] Senior mental model
> 
> No pienses:
> 
> ```text
> pointer → memory
> ```
> 
> Piensa:
> 
> ```text
> pointer
>    │
>    ▼
> reference to an object
>    │
>    ├── ownership
>    ├── lifetime
>    ├── aliasing
>    └── validity
> ```
> 
> Y cuando aparezca:
> 
> ```c
> malloc()
> realloc()
> free()
> ```
> 
> pregúntate siempre:
> 
> ```text
> WHO OWNS IT?
> HOW LONG DOES IT LIVE?
> WHO ELSE POINTS TO IT?
> IS THIS POINTER STILL VALID?
> ```
> 
> Si puedes responder esas cuatro preguntas, estás empezando a razonar sobre memoria **como programador C**, no simplemente a utilizar `malloc()` y `free()`.