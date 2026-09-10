# `calloc()` — Reserva e inicialización de memoria en C

> [!abstract] Idea clave  
> `calloc()` reserva memoria dinámica para un número determinado de elementos y **establece todos los bits del bloque a cero**.
> 
> ```c
> calloc(count, size)
> ```
> 
> Conceptualmente:
> 
> ```text
> número de elementos × tamaño de cada elemento
> ```
> 
> A diferencia de `malloc()`, `calloc()` inicializa la memoria.

---

# 1. ¿Qué es `calloc()`?

`calloc()` significa **contiguous allocation**.

Es una función de la biblioteca estándar de C utilizada para reservar memoria dinámica para un conjunto de elementos.

Está declarada en:

```c
#include <stdlib.h>
```

Prototipo:

```c
void *calloc(size_t nmemb, size_t size);
```

Ejemplo:

```c
int *numbers;

numbers = calloc(10, sizeof(*numbers));
```

Se solicita memoria para:

```text
10 elementos
     ×
sizeof(int)
```

---

# 2. ¿Qué devuelve?

Devuelve:

```c
void *
```

igual que `malloc()`.

En caso de éxito:

```text
dirección del primer byte del bloque
```

En caso de fallo:

```c
NULL
```

Por tanto:

```c
int *numbers;

numbers = calloc(10, sizeof(*numbers));

if (numbers == NULL)
    return 1;
```

---

# 3. Diferencia fundamental con `malloc()`

## `malloc()`

```c
int *numbers;

numbers = malloc(10 * sizeof(*numbers));
```

La memoria **no está inicializada**.

## `calloc()`

```c
int *numbers;

numbers = calloc(10, sizeof(*numbers));
```

La memoria se inicializa a cero.

Conceptualmente:

```text
malloc()

┌────┬────┬────┬────┐
│ ?? │ ?? │ ?? │ ?? │
└────┴────┴────┴────┘
```

```text
calloc()

┌────┬────┬────┬────┐
│  0 │  0 │  0 │  0 │
└────┴────┴────┴────┘
```

> [!important]  
> No confundas "memoria inicializada a cero" con una regla universal de representación de cualquier tipo como si fuera una conversión semántica.
> 
> Para objetos de tipos enteros, por ejemplo, el resultado esperado de una representación de bits cero es cero. Para tipos más complejos, hay que distinguir representación de bits y valor del tipo.

---

# 4. Parámetros

`calloc()` recibe dos argumentos:

```c
calloc(nmemb, size);
```

### `nmemb`

Número de elementos.

### `size`

Tamaño de cada elemento en bytes.

Ejemplo:

```c
calloc(10, sizeof(int));
```

Significa:

```text
10 elementos
     ×
tamaño de int
```

---

# 5. `calloc()` frente a `malloc()`

||`malloc()`|`calloc()`|
|---|---|---|
|Argumentos|1|2|
|Elementos|implícito|explícito|
|Tamaño|bytes totales|elementos × bytes|
|Inicialización|no|cero|
|Devuelve|`void *`|`void *`|
|Error|`NULL`|`NULL`|
|Liberación|`free()`|`free()`|

---

# 6. Ejemplo básico

```c
#include <stdlib.h>

int main(void)
{
    int *numbers;
    size_t count;
    size_t i;

    count = 5;

    numbers = calloc(count, sizeof(*numbers));

    if (numbers == NULL)
        return 1;

    for (i = 0; i < count; i++)
        numbers[i] = i;

    free(numbers);
    numbers = NULL;

    return 0;
}
```

Ciclo de vida:

```text
calloc()
   ↓
allocation
   ↓
memoria inicializada
   ↓
uso
   ↓
free()
```

---

# 7. ¿Por qué `calloc()` recibe dos tamaños?

Compara:

```c
malloc(count * sizeof(*numbers));
```

con:

```c
calloc(count, sizeof(*numbers));
```

La segunda forma expresa directamente:

```text
count elementos
×
tamaño de cada elemento
```

Esto hace que la intención sea especialmente clara cuando estás trabajando con arrays dinámicos.

---

# 8. `calloc()` + `sizeof`

Patrón recomendado:

```c
int *numbers;

numbers = calloc(count, sizeof(*numbers));
```

Evita:

```c
numbers = calloc(count, sizeof(int));
```

cuando el objetivo es mantener el tipo ligado directamente al puntero.

El patrón:

```c
sizeof(*numbers)
```

se adapta automáticamente si cambia el tipo de `numbers`.

---

# 9. Comprobar `NULL`

Exactamente igual que con `malloc()`:

```c
numbers = calloc(count, sizeof(*numbers));

if (numbers == NULL)
{
    return 1;
}
```

Nunca asumas que una asignación dinámica siempre tiene éxito.

---

# 10. Liberación

La memoria obtenida mediante `calloc()` se libera con:

```c
free(numbers);
```

No existe:

```c
freecalloc(numbers);
```

La pareja es:

```text
calloc() ─────────► free()
```

Ejemplo:

```c
int *numbers;

numbers = calloc(10, sizeof(*numbers));

if (numbers == NULL)
    return 1;

/* use numbers */

free(numbers);
numbers = NULL;
```

---

# 11. `calloc()` y `free()` tienen una relación de ownership

Cuando haces:

```c
numbers = calloc(10, sizeof(*numbers));
```

algún componente del programa adquiere la responsabilidad sobre ese bloque.

Debes poder responder:

```text
¿Quién lo posee?
      ↓
¿Quién lo utiliza?
      ↓
¿Quién lo libera?
```

La ausencia de una respuesta clara puede producir:

- memory leaks;
    
- double free;
    
- use-after-free.
    

---

# 12. Error: olvidar `free()`

```c
int *numbers;

numbers = calloc(100, sizeof(*numbers));

if (numbers == NULL)
    return 1;

/* uso */

return 0;
```

Falta:

```c
free(numbers);
```

Resultado:

```text
allocation
    ↓
uso
    ↓
return
    ↓
❌ bloque no liberado
```

---

# 13. Error: use-after-free

```c
int *numbers;

numbers = calloc(10, sizeof(*numbers));

if (numbers == NULL)
    return 1;

free(numbers);

numbers[0] = 42;    // ERROR
```

El bloque ya fue liberado.

Esto es:

> **use-after-free**

y produce **undefined behavior**.

---

# 14. Error: double free

```c
int *numbers;

numbers = calloc(10, sizeof(*numbers));

if (numbers == NULL)
    return 1;

free(numbers);
free(numbers);      // ERROR
```

Solución defensiva:

```c
free(numbers);
numbers = NULL;
```

Después:

```c
free(numbers);
```

es seguro porque:

```c
free(NULL);
```

no realiza ninguna acción.

---

# 15. `calloc()` no es simplemente "malloc + memset"

Conceptualmente puedes pensar:

```text
calloc()
   ↓
reserva memoria
   ↓
inicializa a cero
```

Pero a nivel de API y semántica, `calloc()` expresa una operación específica de asignación de memoria.

No conviene definirla simplemente como:

```c
malloc() + memset()
```

aunque el efecto práctico de inicializar memoria puede parecerse.

---

# 16. `calloc()` y arrays dinámicos

Este es uno de sus usos naturales:

```c
size_t count;

count = 100;

int *numbers = calloc(count, sizeof(*numbers));
```

Resultado conceptual:

```text
┌─────┬─────┬─────┬─────┬─────┐
│  0  │  0  │  0  │  0  │ ... │
└─────┴─────┴─────┴─────┴─────┘
             100 elementos
```

---

# 17. Diferencia con un array automático

Esto:

```c
int numbers[10];
```

no implica que los elementos sean cero.

Por ejemplo:

```c
int numbers[10];
```

en un bloque automático deja sus elementos sin inicializar.

En cambio:

```c
int *numbers = calloc(10, sizeof(*numbers));
```

obtiene un bloque dinámico inicializado a cero.

---

# 18. `calloc()` no significa "cualquier objeto se convierte mágicamente en cero"

Hay que mantener precisión.

"Todos los bits a cero" y "valor numérico cero" no son conceptos idénticos para todos los tipos de C.

Por eso, al estudiar `calloc()`, recuerda:

```text
calloc()
   ↓
storage with zeroed bytes
```

y no una regla simplificada del tipo:

```text
calloc()
   ↓
todos los tipos tienen necesariamente su valor semántico cero
```

Esta distinción es importante cuando avances hacia:

- representación de objetos;
    
- tipos de puntero;
    
- floating point;
    
- padding;
    
- object representation.
    

---

# 19. Overflow en `calloc()`

Existe una ventaja conceptual importante frente a escribir manualmente:

```c
malloc(count * size);
```

porque aquí el cálculo de:

```text
count × size
```

puede desbordar `size_t`.

Por ejemplo:

```c
size_t count;
size_t size;

count = ...;
size = ...;

malloc(count * size);
```

Si existe overflow, puedes terminar solicitando menos memoria de la necesaria.

Con:

```c
calloc(count, size);
```

la interfaz recibe ambos valores por separado y la implementación puede detectar un producto que no puede representarse adecuadamente.

> [!important]  
> No significa que `calloc()` haga seguros todos los cálculos relacionados con memoria. Significa que la interfaz permite al allocator comprobar el producto `nmemb × size` antes de realizar la reserva.

---

# 20. Patrón profesional

```c
#include <stdlib.h>

int main(void)
{
    int *numbers;
    size_t count;
    size_t i;

    count = 10;

    numbers = calloc(count, sizeof(*numbers));

    if (numbers == NULL)
        return 1;

    for (i = 0; i < count; i++)
        numbers[i] = 0;

    free(numbers);
    numbers = NULL;

    return 0;
}
```

Observa que el `for` vuelve a escribir cero.

Eso es deliberado solo para mostrar el contenido inicial y el acceso a los elementos. En código real sería redundante si únicamente necesitas que el array esté a cero.

---

# 21. Comparación mental

```text
malloc()
   │
   ├── reserva
   │
   └── contenido inicial indeterminado


calloc()
   │
   ├── reserva
   │
   └── almacenamiento inicializado a cero


realloc()
   │
   └── cambia el tamaño de una reserva existente


free()
   │
   └── libera la reserva
```

---

# 22. Las cuatro operaciones

```text
              MEMORIA DINÁMICA
                     │
       ┌─────────────┼─────────────┐
       │             │             │
    malloc()       calloc()      realloc()
       │             │             │
    reserve       reserve +      resize
                  zero
       │             │             │
       └─────────────┼─────────────┘
                     │
                     ▼
                   free()
                     │
                     ▼
                  release
```

---

# 23. Checklist

Antes de utilizar `calloc()`:

-  ¿Necesito memoria dinámica?
    
-  ¿Necesito que el almacenamiento empiece a cero?
    
-  ¿El número de elementos está en `size_t`?
    
-  ¿Estoy usando `sizeof(*pointer)`?
    
-  ¿Compruebo `NULL`?
    
-  ¿Tengo claro quién es el owner?
    
-  ¿Existe un `free()`?
    
-  ¿Evito usar el bloque después de `free()`?
    
-  ¿Evito hacer `free()` dos veces?
    
-  ¿He considerado posibles overflow en tamaños?
    

---

# 24. Resumen de examen

> **`calloc()` reserva memoria dinámica para `nmemb` elementos de `size` bytes cada uno y devuelve un puntero al bloque reservado. Si la reserva falla, devuelve `NULL`. El almacenamiento obtenido se inicializa con todos sus bits a cero. La memoria debe liberarse mediante `free()`.**

### Sintaxis

```c
void *calloc(size_t nmemb, size_t size);
```

### Uso

```c
int *numbers;

numbers = calloc(count, sizeof(*numbers));

if (numbers == NULL)
    return 1;

free(numbers);
numbers = NULL;
```

---

# 25. Conexiones

```text
                    sizeof
                      │
                      ▼
                  object size
                      │
                      ▼
              ┌───────────────┐
              │               │
           malloc()        calloc()
              │               │
           allocate       allocate + zero
              │               │
              └───────┬───────┘
                      │
                      ▼
                  ownership
                      │
                      ▼
                   lifetime
                      │
                      ▼
                    free()
                      │
                      ▼
                deallocation
```

> [!important] Regla senior  
> **`calloc()` no solo reserva memoria: expresa que quieres una colección de objetos cuyo almacenamiento comienza con todos sus bits a cero.**
> 
> La secuencia que debes dominar es:
> 
> ```text
> sizeof → malloc/calloc → ownership → use → free
> ```
> 
> El siguiente concepto lógico es **`realloc()`**, porque completa el ciclo de gestión dinámica: **allocate → resize → release**.