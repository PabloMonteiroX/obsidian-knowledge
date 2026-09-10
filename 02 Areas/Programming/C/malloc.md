---
type: concept
domain: programming
language: C
status: learning
---
# `malloc()` — Memoria dinámica en C

> [!abstract] Idea clave  
> `malloc()` solicita un bloque de memoria dinámica durante la ejecución del programa.
> 
> **Reserva → devuelve dirección → usamos memoria → `free()` libera.**

---

## 1. ¿Qué es `malloc()`?

`malloc` significa **memory allocation**.

Es una función de la biblioteca estándar de C que permite solicitar memoria dinámica durante el **runtime**.

```c
#include <stdlib.h>

void *malloc(size_t size);
```

La memoria obtenida mediante `malloc()` se utiliza normalmente como **heap storage**.

---

## 2. ¿Para qué sirve?

Principalmente para reservar memoria cuyo:

- tamaño se conoce en runtime;
    
- tiempo de vida debe superar el de una función;
    
- tamaño no resulta práctico determinar de antemano.
    

Ejemplo:

```c
int n;

scanf("%d", &n);

int *array = malloc(n * sizeof(int));
```

Si:

```text
n = 5
```

se reserva espacio para:

```text
┌─────┬─────┬─────┬─────┬─────┐
│ int │ int │ int │ int │ int │
└─────┴─────┴─────┴─────┴─────┘
```

---

## 3. ¿Qué devuelve?

```c
void *malloc(size_t size);
```

Devuelve un **puntero al comienzo del bloque reservado**.

Ejemplo:

```c
int *p = malloc(sizeof(int));
```

Conceptualmente:

```text
STACK                         HEAP

┌──────────┐                 ┌────────────┐
│    p     │ ──────────────► │  int       │
└──────────┘                 └────────────┘
   pointer                    allocated
```

### Importante

En C no es necesario hacer cast:

```c
int *p = malloc(sizeof(int));
```

Correcto.

No es necesario:

```c
int *p = (int *)malloc(sizeof(int));
```

El resultado de `malloc()` es `void *`, que puede convertirse implícitamente a otro tipo de puntero en C.

---

## 4. ¿Qué contiene la memoria?

`malloc()` **no inicializa** la memoria.

```c
int *p = malloc(sizeof(int));
```

La memoria obtenida contiene valores indeterminados.

Por tanto, no debemos hacer:

```c
printf("%d\n", *p);
```

antes de haber almacenado un valor válido.

Correcto:

```c
int *p = malloc(sizeof(int));

if (p == NULL)
    return 1;

*p = 42;

printf("%d\n", *p);
```

---

# 5. Comprobar `NULL`

Una reserva puede fallar.

```c
int *p = malloc(sizeof(int));

if (p == NULL)
{
    return 1;
}
```

Si `malloc()` no puede satisfacer la petición, devuelve:

```c
NULL
```

### Regla

```text
malloc()
   │
   ├── éxito ──► dirección válida
   │
   └── fallo ──► NULL
```

**Nunca des por hecho que `malloc()` tuvo éxito.**

---

# 6. Liberar memoria

La memoria reservada dinámicamente debe liberarse mediante:

```c
free()
```

Ejemplo:

```c
int *p = malloc(sizeof(int));

if (p == NULL)
    return 1;

*p = 42;

free(p);
```

Relación fundamental:

```text
malloc() ─────────────► free()
  reserve                release
```

---

# 7. Después de `free()`

Después de:

```c
free(p);
```

el bloque de memoria ya no puede utilizarse.

Pero la variable `p` sigue existiendo.

```text
p ─────────► memoria liberada
             ❌
```

Por seguridad:

```c
free(p);
p = NULL;
```

Ahora:

```text
p ─────────► NULL
```

Esto evita reutilizar accidentalmente el puntero.

---

# 8. Errores graves

## 8.1 Memory leak

Reservar memoria y perder su dirección sin liberarla:

```c
int *p = malloc(sizeof(int));

p = NULL;
```

El bloque continúa reservado, pero ya no tenemos su dirección.

```text
HEAP

┌────────────┐
│   bloque   │
└────────────┘
       X
       │
   sin pointer
```

Esto es un **memory leak**.

---

## 8.2 Use-after-free

Utilizar memoria después de liberarla:

```c
int *p = malloc(sizeof(int));

*p = 42;

free(p);

printf("%d\n", *p);  // ERROR
```

La memoria ya no pertenece al programa.

---

## 8.3 Double free

Liberar dos veces el mismo bloque:

```c
int *p = malloc(sizeof(int));

free(p);
free(p);  // ERROR
```

Esto produce **undefined behavior**.

---

## 8.4 Invalid free

No puedes hacer:

```c
int x = 42;

free(&x);  // ERROR
```

`x` no fue obtenido mediante una función de asignación dinámica compatible con `free()`.

---

# 9. `free(NULL)`

Esto es válido:

```c
free(NULL);
```

No hace nada.

Por eso este patrón es seguro:

```c
free(p);
p = NULL;

free(p);
```

El segundo `free()` recibe `NULL`.

---

# 10. Stack vs Heap

## Stack

```c
void function(void)
{
    int x = 42;
}
```

`x` tiene almacenamiento automático.

Su vida está asociada a la ejecución de la función.

```text
function()
    │
    ├── x
    │
    └── return
         ↓
       x deja de existir
```

## Heap

```c
int *createNumber(void)
{
    int *p = malloc(sizeof(int));

    if (p == NULL)
        return NULL;

    *p = 42;

    return p;
}
```

El bloque reservado no desaparece simplemente porque termine `createNumber()`.

```c
int *number = createNumber();

printf("%d\n", *number);

free(number);
```

Aquí:

```text
createNumber()

STACK                      HEAP
┌───────┐                 ┌───────┐
│   p   │ ──────────────► │  42   │
└───────┘                 └───────┘
     │
     └── desaparece al retornar

number ─────────────────► bloque sigue vivo
```

---

# 11. `malloc()` no significa "pedir RAM al kernel"

Esta es una simplificación que conviene evitar.

Conceptualmente:

```text
Programa
   │
   ▼
malloc()
   │
   ▼
memory allocator
   │
   ▼
memoria dinámica
```

El allocator administra bloques de memoria y puede solicitar más memoria al sistema operativo cuando sea necesario.

Por tanto:

> `malloc()` es una API de asignación de memoria, no una llamada directa equivalente a "dame RAM".

---

# 12. Patrón profesional

Un patrón básico:

```c
#include <stdlib.h>

int main(void)
{
    int *numbers;

    numbers = malloc(5 * sizeof(*numbers));

    if (numbers == NULL)
        return 1;

    numbers[0] = 10;
    numbers[1] = 20;
    numbers[2] = 30;
    numbers[3] = 40;
    numbers[4] = 50;

    free(numbers);
    numbers = NULL;

    return 0;
}
```

### ¿Por qué `sizeof(*numbers)`?

Esto:

```c
malloc(5 * sizeof(*numbers));
```

es preferible a:

```c
malloc(5 * sizeof(int));
```

porque el tipo está relacionado directamente con el puntero.

Si posteriormente cambia:

```c
int *numbers;
```

por:

```c
long *numbers;
```

la reserva se adapta automáticamente.

---

# 13. Modelo mental

Cuando veas:

```c
int *p = malloc(sizeof(int));
```

piensa:

```text
1. Necesito memoria
       ↓
2. Solicito N bytes
       ↓
3. malloc() intenta reservarlos
       ↓
4. Devuelve una dirección
       ↓
5. p guarda esa dirección
       ↓
6. Compruebo p != NULL
       ↓
7. Utilizo la memoria
       ↓
8. free(p)
       ↓
9. p = NULL
```

---

# 14. Regla de ownership

La pregunta profesional no es solamente:

> "¿Dónde está el `malloc()`?"

La pregunta importante es:

> **¿Quién es responsable de liberar esta memoria?**

Cada reserva debe tener un **owner** claro.

```text
malloc()
   │
   ▼
ownership
   │
   ├── usar
   ├── pasar
   └── liberar
         │
         ▼
       free()
```

Si no puedes determinar quién debe ejecutar `free()`, probablemente tienes un problema de diseño.

---

# 15. Checklist

Antes de dar por terminado un código con `malloc()`:

-  `#include <stdlib.h>`
    
-  ¿El tamaño calculado es correcto?
    
-  ¿Compruebo `malloc() == NULL`?
    
-  ¿Inicializo la memoria antes de leerla?
    
-  ¿Quién es el owner del bloque?
    
-  ¿Existe un `free()`?
    
-  ¿El `free()` ocurre exactamente una vez?
    
-  ¿No uso la memoria después de `free()`?
    
-  ¿No hago `free()` sobre memoria que no corresponde?
    
-  ¿Puedo evitar perder el puntero?
    
-  ¿Hay posibles leaks en las rutas de error?
    

---

# 16. Las 4 funciones que debes relacionar

|Función|Propósito|
|---|---|
|`malloc()`|Reserva memoria|
|`calloc()`|Reserva e inicializa a cero|
|`realloc()`|Cambia el tamaño de una reserva|
|`free()`|Libera una reserva|

```text
             malloc()
                │
                ▼
          ┌───────────┐
          │  memoria  │
          └───────────┘
             ▲     │
             │     │
         realloc   │
             │     │
             └─────┘
                │
                ▼
              free()
```

---

# 17. Resumen de examen

> **`malloc()` reserva un bloque de memoria dinámica de `size` bytes y devuelve un puntero al comienzo del bloque. Si no puede realizar la reserva, devuelve `NULL`. La memoria obtenida no está inicializada y debe liberarse mediante `free()` cuando deja de ser necesaria.**

### Sintaxis

```c
void *malloc(size_t size);
```

### Uso mínimo correcto

```c
int *p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

*p = 42;

free(p);
p = NULL;
```

---

## Conceptos que debes dominar después

```text
malloc
  │
  ├── NULL
  ├── sizeof
  ├── pointer
  ├── heap
  ├── ownership
  ├── free
  │
  ├── memory leak
  ├── use-after-free
  ├── double free
  └── invalid free

        ↓

realloc()
        ↓
dynamic data structures
        ↓
linked lists
        ↓
trees
        ↓
hash tables
```

> [!important] Regla senior  
> **Memory management in C = allocation + ownership + lifetime + release.**
> 
> `malloc()` es solo el primer paso. El verdadero dominio está en controlar correctamente el **lifetime** de cada bloque de memoria.


Relacionado:

- [[Punteros]]
- [[Memoria]]
- [[free]]
- [[sizeof]]