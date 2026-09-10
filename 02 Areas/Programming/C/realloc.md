# `realloc()` — Redimensionar memoria dinámica en C

> [!abstract] Idea clave  
> `realloc()` cambia el tamaño de un bloque de memoria dinámica previamente asignado.
> 
> ```c
> realloc(pointer, newSize);
> ```
> 
> Puede:
> 
> - ampliar el bloque;
>     
> - reducirlo;
>     
> - moverlo a otra dirección;
>     
> - mantener la misma dirección.
>     
> 
> La regla crítica:
> 
> > **Nunca sobrescribas directamente el único puntero al bloque con `realloc()` sin considerar el caso de fallo.**

---

# 1. ¿Qué es `realloc()`?

`realloc()` significa **reallocate**.

Es una función de la biblioteca estándar de C utilizada para cambiar el tamaño de un bloque de memoria obtenido mediante una asignación dinámica.

Está declarada en:

```c
#include <stdlib.h>
```

Prototipo:

```c
void *realloc(void *ptr, size_t size);
```

Ejemplo:

```c
int *numbers;

numbers = malloc(5 * sizeof(*numbers));

if (numbers == NULL)
    return 1;

/* ... */

int *tmp;

tmp = realloc(numbers, 10 * sizeof(*numbers));
```

---

# 2. ¿Para qué sirve?

Supongamos que inicialmente necesitas:

```text
5 elementos
```

y posteriormente necesitas:

```text
10 elementos
```

Con `malloc()` tendrías que gestionar una nueva reserva y copiar los datos.

`realloc()` proporciona una operación específica para redimensionar el bloque:

```c
numbers = realloc(numbers, 10 * sizeof(*numbers));
```

Conceptualmente:

```text
ANTES

┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │
└────┴────┴────┴────┴────┘
          5 elementos


DESPUÉS

┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │ ?? │ ?? │ ?? │ ?? │ ?? │
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
                         10 elementos
```

Los bytes de la parte antigua se conservan hasta el límite correspondiente al nuevo tamaño.

---

# 3. ¿Qué devuelve?

`realloc()` devuelve:

```c
void *
```

Si tiene éxito:

```text
dirección del bloque redimensionado
```

Puede ser:

```text
la misma dirección
```

o:

```text
una dirección diferente
```

Esto es fundamental.

```text
                 realloc()
                    │
          ┌─────────┴─────────┐
          │                   │
       misma                 otra
     dirección             dirección
          │                   │
          └─────────┬─────────┘
                    ▼
              nuevo bloque
```

Por eso **no debes asumir que el puntero conserva su dirección**.

---

# 4. Puede mover el bloque

Antes:

```text
numbers ─────► HEAP A
```

Después de `realloc()`:

```text
numbers ─────► HEAP B
```

Conceptualmente:

```text
ANTES

numbers
   │
   ▼
┌─────────────┐
│   bloque A  │
└─────────────┘


DESPUÉS

numbers
   │
   ▼
┌─────────────────────┐
│      bloque B       │
└─────────────────────┘

bloque A ya no es el bloque utilizado
```

Por eso cualquier puntero que apunte a una posición dentro del bloque puede quedar invalidado si `realloc()` mueve la memoria.

---

# 5. El error más peligroso

Este patrón:

```c
numbers = realloc(numbers, newSize);
```

puede ser problemático.

¿Por qué?

Porque si `realloc()` falla y devuelve:

```c
NULL
```

puedes perder la referencia al bloque original.

Conceptualmente:

```text
numbers ─────► bloque original
                 │
                 ▼
              realloc()
                 │
               fallo
                 │
                 ▼
                NULL

numbers = NULL
```

El bloque original puede seguir existiendo, pero has perdido su dirección.

Eso produce un **memory leak**.

---

# 6. Patrón correcto

Utiliza un puntero temporal:

```c
int *tmp;

tmp = realloc(numbers, newSize);

if (tmp == NULL)
{
    free(numbers);
    return 1;
}

numbers = tmp;
```

Pero observa algo importante:

> No siempre debes hacer `free(numbers)` inmediatamente cuando `realloc()` falla.

Si quieres conservar el bloque original y seguir trabajando con él, simplemente puedes dejarlo intacto:

```c
int *tmp;

tmp = realloc(numbers, newSize);

if (tmp == NULL)
{
    /* numbers sigue siendo válido */
    return 1;
}

numbers = tmp;
```

Este patrón es especialmente importante.

---

# 7. Regla de ownership

Antes:

```text
numbers
   │
   ▼
bloque A
```

Después de un `realloc()` exitoso:

```text
numbers
   │
   ▼
bloque redimensionado
```

Si falla:

```text
numbers
   │
   ▼
bloque original
```

La reserva original **sigue siendo responsabilidad del owner**.

Por eso:

```c
tmp = realloc(numbers, newSize);

if (tmp == NULL)
{
    /* numbers sigue siendo válido */
    return 1;
}

numbers = tmp;
```

mantiene correctamente el ownership.

---

# 8. Redimensionar hacia arriba

Ejemplo:

```c
size_t oldCount;
size_t newCount;
int *numbers;
int *tmp;

oldCount = 5;
newCount = 10;

numbers = malloc(oldCount * sizeof(*numbers));

if (numbers == NULL)
    return 1;

tmp = realloc(numbers, newCount * sizeof(*numbers));

if (tmp == NULL)
{
    free(numbers);
    return 1;
}

numbers = tmp;
```

Ahora `numbers` tiene espacio para:

```text
10 int
```

Los primeros 5 valores conservan su contenido.

La memoria adicional no debe asumirse inicializada.

---

# 9. Redimensionar hacia abajo

También puedes reducir:

```c
tmp = realloc(numbers, 5 * sizeof(*numbers));
```

Si originalmente tenías:

```text
10 elementos
```

ahora el bloque tiene espacio para:

```text
5 elementos
```

Los datos que quedan dentro del nuevo tamaño se conservan.

Los elementos que quedan fuera del nuevo límite dejan de formar parte del bloque utilizable.

```text
ANTES

┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │ 60 │ 70 │ 80 │
└────┴────┴────┴────┴────┴────┴────┴────┘


DESPUÉS

┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │
└────┴────┴────┴────┴────┘
```

---

# 10. ¿Qué ocurre con los datos?

Cuando `realloc()` tiene éxito, el contenido anterior se conserva hasta:

```text
min(oldSize, newSize)
```

Es decir:

```text
tamaño antiguo
        │
        ▼
┌────────────────────┐
│ datos conservados  │
└────────────────────┘
        │
        ▼
tamaño nuevo
```

Si amplías:

```text
datos antiguos → conservados
espacio nuevo  → no inicializado
```

Si reduces:

```text
parte conservada → permanece
parte eliminada  → deja de estar disponible
```

---

# 11. `realloc(ptr, 0)`

Este caso requiere especial cuidado.

No debes utilizar:

```c
realloc(ptr, 0);
```

como patrón normal de liberación.

Si quieres liberar memoria:

```c
free(ptr);
```

es mucho más claro y expresa directamente la intención.

> [!important]  
> Para código moderno y portable, utiliza `free()` cuando tu intención sea liberar el bloque y no redimensionarlo.

---

# 12. `realloc(NULL, size)`

Existe una propiedad muy útil:

```c
realloc(NULL, size);
```

se comporta como una asignación equivalente a:

```c
malloc(size);
```

Esto permite diseñar algunas estructuras de gestión de memoria de forma uniforme.

Ejemplo conceptual:

```c
int *numbers = NULL;

numbers = realloc(numbers, 10 * sizeof(*numbers));
```

Pero para código introductorio, sigue siendo más claro utilizar `malloc()` cuando estás realizando una primera asignación.

---

# 13. `realloc()` no inicializa memoria nueva

Supongamos:

```c
int *numbers;

numbers = malloc(5 * sizeof(*numbers));

if (numbers == NULL)
    return 1;

numbers[0] = 10;
numbers[1] = 20;
numbers[2] = 30;
numbers[3] = 40;
numbers[4] = 50;
```

Ahora:

```c
int *tmp;

tmp = realloc(numbers, 10 * sizeof(*numbers));
```

Los elementos:

```text
0 → 10
1 → 20
2 → 30
3 → 40
4 → 50
```

se conservan.

Pero no debes asumir que:

```text
5 → 0
6 → 0
7 → 0
...
```

La memoria adicional no está automáticamente inicializada a cero.

---

# 14. `realloc()` y `sizeof`

Patrón correcto:

```c
tmp = realloc(numbers, newCount * sizeof(*numbers));
```

No:

```c
tmp = realloc(numbers, newCount * 4);
```

No asumas:

```text
sizeof(int) == 4
```

Usa:

```c
sizeof(*numbers)
```

para mantener el tamaño ligado al tipo real.

---

# 15. `realloc()` y arrays dinámicos

Un uso clásico:

```c
size_t capacity;
int *numbers;
int *tmp;

capacity = 10;

numbers = malloc(capacity * sizeof(*numbers));

if (numbers == NULL)
    return 1;
```

Más adelante:

```c
capacity = 20;

tmp = realloc(numbers, capacity * sizeof(*numbers));

if (tmp == NULL)
{
    free(numbers);
    return 1;
}

numbers = tmp;
```

Esto permite construir estructuras dinámicas.

Por ejemplo:

```text
capacity = 10
      ↓
┌──────────────────────┐
│      10 elementos    │
└──────────────────────┘

          realloc()

capacity = 20
      ↓
┌────────────────────────────────────────┐
│             20 elementos               │
└────────────────────────────────────────┘
```

---

# 16. `size` vs `capacity`

Este concepto será muy importante cuando estudies estructuras dinámicas.

No confundas:

```text
size
```

con:

```text
capacity
```

Ejemplo:

```text
capacity = 10
size = 4
```

Significa:

```text
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │    │    │    │    │
└────┴────┴────┴────┴────┴────┴────┴────┘
  ←────── size ──────→
  ←──────── capacity ────────────────→
```

`realloc()` modifica normalmente la **capacity**.

Tu estructura debe controlar por separado cuántos elementos estás utilizando.

---

# 17. Punteros interiores

Este caso es peligroso:

```c
int *numbers;
int *element;

numbers = malloc(10 * sizeof(*numbers));

if (numbers == NULL)
    return 1;

element = &numbers[5];

int *tmp;

tmp = realloc(numbers, 20 * sizeof(*numbers));
```

Si `realloc()` mueve el bloque, `element` puede dejar de apuntar a una ubicación válida del nuevo bloque.

Por tanto:

> Los punteros derivados de un bloque pueden quedar invalidados por `realloc()`.

No mantengas referencias internas al bloque a través de un `realloc()` sin revisar su validez.

---

# 18. `realloc()` puede invalidar punteros

Antes:

```text
numbers ───────► bloque
element ───────► numbers[5]
```

Después de mover:

```text
numbers ───────► nuevo bloque

element ───────► dirección antigua
                  ❌
```

Por eso `realloc()` afecta no solo al puntero principal, sino también a los punteros que dependen de ese bloque.

---

# 19. Patrón robusto completo

```c
#include <stdlib.h>

int main(void)
{
    int *numbers;
    int *tmp;
    size_t count;
    size_t newCount;

    count = 5;
    newCount = 10;

    numbers = malloc(count * sizeof(*numbers));

    if (numbers == NULL)
        return 1;

    tmp = realloc(numbers, newCount * sizeof(*numbers));

    if (tmp == NULL)
    {
        free(numbers);
        return 1;
    }

    numbers = tmp;

    free(numbers);
    numbers = NULL;

    return 0;
}
```

El patrón importante es:

```text
malloc
  ↓
ownership
  ↓
tmp = realloc(...)
  ↓
¿NULL?
 ┌───────┴───────┐
 sí              no
 │                │
original          numbers = tmp
sigue válido      │
 │                ▼
 │              use
 │                │
 └───────┐        │
         │        │
         ▼        ▼
             free()
```

---

# 20. El error de sobrescribir el puntero

Evita como patrón general:

```c
numbers = realloc(numbers, newSize);
```

No porque sea siempre incorrecto, sino porque dificulta gestionar correctamente el fallo.

Prefiere:

```c
tmp = realloc(numbers, newSize);

if (tmp == NULL)
{
    /* numbers sigue siendo válido */
    return 1;
}

numbers = tmp;
```

Esto preserva la referencia original si la operación falla.

---

# 21. `realloc()` no es `malloc()` + `free()`

Conceptualmente puede implicar una nueva reserva y copia interna si necesita mover el bloque, pero `realloc()` proporciona una operación semánticamente específica:

```text
redimensionar una reserva existente
```

No debes implementar manualmente esa lógica salvo que exista una razón concreta.

---

# 22. Ciclo completo de memoria dinámica

Ahora tienes las cuatro piezas:

```text
                    MEMORIA DINÁMICA
                           │
          ┌────────────────┼────────────────┐
          │                │                │
       malloc()          calloc()        realloc()
          │                │                │
       reserve        reserve + zero      resize
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                         use
                           │
                           ▼
                         free()
                           │
                           ▼
                       release
```

---

# 23. Las cuatro funciones

|Función|Operación|
|---|---|
|`malloc()`|Reserva memoria|
|`calloc()`|Reserva + inicializa a cero|
|`realloc()`|Redimensiona una reserva|
|`free()`|Libera una reserva|

### Relación

```text
malloc()
   │
   ▼
allocated block
   │
   ├── realloc() → resized block
   │
   └── free()    → released
```

---

# 24. Errores que debes reconocer inmediatamente

### Perder el puntero original

```c
numbers = realloc(numbers, newSize);
```

sin gestionar adecuadamente el fallo.

---

### Usar punteros antiguos

```c
element = &numbers[5];

realloc(numbers, newSize);

*element = 42;       // potencialmente inválido
```

---

### Utilizar el bloque antiguo después de un `realloc()` exitoso

```c
tmp = realloc(numbers, newSize);

if (tmp != NULL)
{
    /* numbers puede haber quedado invalidado */
}
```

Después del éxito, trabaja con:

```c
numbers = tmp;
```

---

### Asumir que la memoria nueva está a cero

```c
tmp = realloc(numbers, largerSize);

/* ❌ no asumas que la zona nueva contiene 0 */
```

---

### Confundir `size` con `capacity`

```text
capacity = memoria disponible
size     = elementos utilizados
```

Son conceptos diferentes.

---

# 25. Checklist

Antes de utilizar `realloc()`:

-  ¿`ptr` procede de una asignación dinámica válida?
    
-  ¿Estoy pasando el tamaño correcto?
    
-  ¿Estoy usando `sizeof(*ptr)`?
    
-  ¿Estoy gestionando el posible `NULL`?
    
-  ¿Conservo el puntero original mientras compruebo el resultado?
    
-  ¿Existen otros punteros que apunten al bloque?
    
-  ¿Podrían quedar invalidados?
    
-  ¿Estoy asumiendo que la memoria nueva está inicializada?
    
-  ¿Distingo `size` de `capacity`?
    
-  ¿Tengo claro quién mantiene el ownership?
    
-  ¿Existe finalmente un `free()`?
    

---

# 26. Resumen de examen

> **`realloc()` cambia el tamaño de un bloque de memoria dinámica previamente asignado. Puede mantener el bloque en la misma dirección o moverlo a otra. Si tiene éxito, devuelve un puntero al bloque redimensionado. Si falla, devuelve `NULL` y la reserva original permanece sin modificar.**

### Sintaxis

```c
void *realloc(void *ptr, size_t size);
```

### Patrón recomendado

```c
int *tmp;

tmp = realloc(numbers, newCount * sizeof(*numbers));

if (tmp == NULL)
{
    /* numbers sigue siendo válido */
    return 1;
}

numbers = tmp;
```

---

# 27. Modelo mental definitivo

```text
                    realloc()
                       │
                       ▼
              ¿redimensionar?
                       │
          ┌────────────┴────────────┐
          │                         │
       éxito                      fallo
          │                         │
          ▼                         ▼
   mismo bloque              bloque original
       o nuevo                    intacto
          │                         │
          ▼                         ▼
    nuevo puntero              conservar
          │
          ▼
         use
          │
          ▼
        free()
```

> [!important] Regla senior  
> **`realloc()` puede cambiar la dirección del bloque.**
> 
> Por eso el patrón:
> 
> ```c
> tmp = realloc(ptr, newSize);
> ```
> 
> es fundamental.
> 
> Primero comprueba el resultado. **Después** actualiza el puntero principal.
> 
> ```text
> old pointer
>      │
>      ▼
>   realloc()
>      │
>      ▼
>    tmp
>      │
>   ┌──┴──┐
>   │     │
> NULL  valid
>   │     │
>   │     ▼
>   │   ptr = tmp
>   │
>   ▼
> original remains valid
> ```
> 
> Dominar `realloc()` significa entender no solo el tamaño, sino **ownership, lifetime, aliasing y validez de los punteros**.