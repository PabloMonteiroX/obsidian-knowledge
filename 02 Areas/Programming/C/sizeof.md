# `sizeof` — Tamaño de tipos y objetos en C

> [!abstract] Idea clave  
> `sizeof` permite conocer **cuántos bytes ocupa un tipo u objeto en memoria**.
> 
> No calcula el valor de una variable.
> 
> Calcula su **tamaño en bytes**.

---

## 1. ¿Qué es `sizeof`?

`sizeof` es un **operador del lenguaje C**.

No es una función.

```c
sizeof(int)
```

Devuelve el número de bytes necesarios para representar un objeto del tipo indicado.

Ejemplo:

```c
printf("%zu\n", sizeof(int));
```

En una plataforma concreta podría mostrar:

```text
4
```

Pero **no debes asumir que `int` siempre ocupa 4 bytes**.

El tamaño depende de la implementación de C y de la plataforma.

---

# 2. ¿Qué devuelve?

El resultado de `sizeof` tiene tipo:

```c
size_t
```

Por eso el especificador habitual de `printf` es:

```c
%zu
```

Ejemplo:

```c
#include <stdio.h>

int main(void)
{
    printf("%zu\n", sizeof(int));

    return 0;
}
```

### Regla

```text
sizeof(...) → size_t
```

No:

```text
sizeof(...) → int
```

---

# 3. `sizeof` mide bytes

La unidad de medida de `sizeof` es el **byte de C**.

```c
sizeof(char)
```

siempre devuelve:

```text
1
```

Esto es una propiedad fundamental del lenguaje:

> `sizeof(char) == 1`

Pero un byte de C no tiene necesariamente 8 bits.

El número de bits de un byte viene determinado por:

```c
CHAR_BIT
```

definido en:

```c
#include <limits.h>
```

En sistemas modernos normalmente:

```text
CHAR_BIT = 8
```

---

# 4. `sizeof(type)`

Puedes aplicar `sizeof` directamente a un tipo:

```c
sizeof(char)
sizeof(short)
sizeof(int)
sizeof(long)
sizeof(long long)
sizeof(float)
sizeof(double)
```

Ejemplo:

```c
printf("%zu\n", sizeof(char));
printf("%zu\n", sizeof(int));
printf("%zu\n", sizeof(double));
```

Los valores exactos **no deben memorizarse como universales**.

Lo que debes conocer son las relaciones y garantías del estándar.

---

# 5. `sizeof(variable)`

También puedes aplicarlo a una expresión u objeto:

```c
int number;

sizeof(number);
```

Es equivalente, en tamaño, a:

```c
sizeof(int);
```

Ejemplo:

```c
int number = 42;

printf("%zu\n", sizeof(number));
```

---

# 6. El uso profesional: `sizeof *pointer`

Este patrón es especialmente importante:

```c
int *p;

p = malloc(sizeof(*p));
```

En lugar de:

```c
p = malloc(sizeof(int));
```

¿Por qué?

Porque `sizeof(*p)` expresa directamente:

> "Reserva exactamente el tamaño del objeto al que apunta `p`."

Esto reduce la posibilidad de inconsistencias cuando cambia el tipo.

---

# 7. `sizeof` + `malloc`

Aquí aparece una de las aplicaciones más importantes.

Incorrecto:

```c
int *numbers;

numbers = malloc(10 * 4);
```

¿Por qué?

Porque estás suponiendo que:

```text
sizeof(int) == 4
```

No deberías codificar ese supuesto.

Correcto:

```c
int *numbers;

numbers = malloc(10 * sizeof(*numbers));
```

Conceptualmente:

```text
10 elementos
     ×
tamaño de un elemento
     =
bytes necesarios
```

```c
malloc(10 * sizeof(*numbers));
```

---

# 8. Arrays

Supongamos:

```c
int numbers[10];
```

Entonces:

```c
sizeof(numbers)
```

devuelve el tamaño total del array.

Conceptualmente:

```text
┌─────┬─────┬─────┬─────┬─────┐
│ int │ int │ int │ int │ int │ ...
└─────┴─────┴─────┴─────┴─────┘
             10 elementos
```

El tamaño total es:

```text
10 × sizeof(int)
```

Por tanto:

```c
sizeof(numbers) == 10 * sizeof(numbers[0])
```

---

# 9. Obtener el número de elementos de un array

Uno de los patrones clásicos de C:

```c
int numbers[10];

size_t count = sizeof(numbers) / sizeof(numbers[0]);
```

Resultado:

```text
10
```

También:

```c
size_t count = sizeof(numbers) / sizeof(*numbers);
```

Esto funciona porque:

```text
tamaño total del array
──────────────────────
tamaño de un elemento
=
número de elementos
```

---

# 10. La trampa más importante: arrays vs pointers

Esto es fundamental.

```c
int numbers[10];
int *p = numbers;
```

Tenemos:

```text
numbers
   │
   ▼
┌─────┬─────┬─────┬─────┐
│ int │ int │ int │ ... │
└─────┴─────┴─────┴─────┘
```

Pero:

```text
p
│
└──────────────► numbers[0]
```

Ahora:

```c
sizeof(numbers)
```

mide **todo el array**.

Mientras que:

```c
sizeof(p)
```

mide **el puntero**.

No la memoria apuntada.

---

# 11. Ejemplo crítico

```c
int numbers[10];
int *p = numbers;

printf("%zu\n", sizeof(numbers));
printf("%zu\n", sizeof(p));
```

Conceptualmente:

```text
sizeof(numbers)
        ↓
tamaño de 10 int

sizeof(p)
        ↓
tamaño de una dirección/puntero
```

Por tanto:

> `sizeof(pointer)` NO devuelve el tamaño de la memoria a la que apunta.

Este error aparece constantemente en C.

---

# 12. `sizeof` y arrays dinámicos

Con:

```c
int *numbers = malloc(10 * sizeof(*numbers));
```

esto:

```c
sizeof(numbers)
```

**no** devuelve el tamaño de los 10 `int`.

Devuelve el tamaño del puntero.

```text
numbers
   │
   ▼
HEAP
┌─────┬─────┬─────┬─────┐
│ int │ int │ int │ ... │
└─────┴─────┴─────┴─────┘

sizeof(numbers)
       ↓
 tamaño del puntero
```

`sizeof` no puede descubrir automáticamente cuántos elementos has reservado con `malloc`.

Por eso debes conservar esa información:

```c
size_t count = 10;

int *numbers = malloc(count * sizeof(*numbers));
```

---

# 13. `sizeof` no ejecuta normalmente la expresión

Considera:

```c
int x = 10;

sizeof(x++);
```

El `x++` no se evalúa porque el tamaño de `x` puede determinarse sin ejecutar la expresión.

Por tanto:

```c
x
```

continúa siendo:

```text
10
```

### Excepción importante

Cuando el operando tiene **tipo VLA (Variable Length Array)**, la evaluación puede ser necesaria.

Por eso no conviene reducir la regla a:

> "`sizeof` nunca evalúa su operando."

La regla correcta es:

> Para operandos cuyo tamaño puede determinarse en compile time, `sizeof` normalmente no evalúa la expresión. Los VLA son una excepción relevante.

---

# 14. `sizeof` ocurre en compile time... normalmente

Ejemplo:

```c
sizeof(int)
```

El compilador conoce el tamaño de `int`.

Por eso puede determinarlo durante la compilación.

Esto es diferente de:

```c
malloc()
```

que realiza una operación de asignación durante runtime.

```text
sizeof
  │
  └── información de tamaño

malloc
  │
  └── asignación de memoria en runtime
```

Esta distinción es importante.

---

# 15. `sizeof(struct)`

Supongamos:

```c
struct Person
{
    int age;
    char initial;
};
```

Podrías esperar:

```text
sizeof(int) + sizeof(char)
```

pero no necesariamente será así.

El compilador puede introducir **padding** para satisfacer requisitos de alineación.

Por eso:

```c
sizeof(struct Person)
```

puede ser mayor que la suma de los tamaños de sus miembros.

---

# 16. Padding y alignment

Ejemplo conceptual:

```text
struct Person

┌───────────────┐
│     int       │
├───────────────┤
│     char      │
├───────────────┤
│    padding    │
├───────────────┤
│    padding    │
└───────────────┘
```

El padding permite que los objetos estén correctamente alineados.

Por eso nunca debes asumir:

```c
sizeof(struct Person) ==
sizeof(int) + sizeof(char)
```

La forma correcta de conocer el tamaño es:

```c
sizeof(struct Person)
```

---

# 17. `sizeof` y strings

Este es otro punto crítico.

```c
char text[] = "hello";
```

El array contiene:

```text
'h' 'e' 'l' 'l' 'o' '\0'
```

Por tanto:

```c
sizeof(text)
```

incluye el `'\0'`.

Resultado:

```text
6
```

Pero:

```c
strlen(text)
```

devuelve:

```text
5
```

### Diferencia

```text
sizeof → tamaño del objeto en bytes
strlen → número de caracteres antes de '\0'
```

No son equivalentes.

---

# 18. `sizeof` vs `strlen`

||`sizeof`|`strlen`|
|---|---|---|
|Tipo|operador|función|
|Mide|tamaño del objeto|longitud del string|
|Unidad|bytes|caracteres|
|Necesita recorrer string|no|sí|
|Incluye `'\0'`|si forma parte del objeto|no|
|Funciona con arrays|sí|solo strings válidos|
|Funciona con punteros|mide el puntero|puede recorrer el string|

Ejemplo:

```c
char text[] = "hello";

sizeof(text);  // 6
strlen(text);  // 5
```

---

# 19. El error clásico con funciones

Esto:

```c
void printSize(int numbers[])
{
    printf("%zu\n", sizeof(numbers));
}
```

**NO** obtiene el tamaño del array original.

En un parámetro de función:

```c
int numbers[]
```

se ajusta a:

```c
int *numbers
```

Por tanto:

```c
sizeof(numbers)
```

mide el tamaño del puntero.

### Solución

Pasar el tamaño explícitamente:

```c
void printSize(int numbers[], size_t count)
{
    printf("%zu\n", count);
}
```

Llamada:

```c
int numbers[10];

printSize(numbers, 10);
```

---

# 20. `sizeof` + tipos definidos por el usuario

También funciona con:

```c
struct
union
enum
typedef
```

Ejemplo:

```c
typedef struct
{
    int id;
    char name[32];
} Person;
```

Puedes hacer:

```c
Person person;

printf("%zu\n", sizeof(person));
printf("%zu\n", sizeof(Person));
```

---

# 21. `sizeof` + `char`

Regla absoluta:

```c
sizeof(char) == 1
```

Pero:

```c
sizeof(char) == 1 byte
```

no significa necesariamente:

```text
1 byte = 8 bits
```

Para conocer los bits:

```c
#include <limits.h>

printf("%d\n", CHAR_BIT);
```

En la mayoría de sistemas actuales:

```text
CHAR_BIT = 8
```

---

# 22. Patrón profesional para asignaciones

### Array de `int`

```c
size_t count = 10;

int *numbers = malloc(count * sizeof(*numbers));
```

### Array de `struct`

```c
size_t count = 20;

Person *people = malloc(count * sizeof(*people));
```

### Array de `char`

```c
size_t size = 256;

char *buffer = malloc(size * sizeof(*buffer));
```

Aunque para `char`:

```c
malloc(size);
```

también expresa correctamente la cantidad de bytes, `sizeof(*buffer)` mantiene el patrón general.

---

# 23. Overflow en el cálculo del tamaño

Aquí aparece un problema más avanzado.

Esto:

```c
size_t count = ...;

int *p = malloc(count * sizeof(*p));
```

puede tener un problema si:

```text
count × sizeof(*p)
```

desborda el rango de `size_t`.

Entonces puedes terminar solicitando menos memoria de la necesaria.

Posteriormente:

```c
p[i]
```

puede escribir fuera del bloque reservado.

### Idea senior

Antes de una asignación dinámica basada en cantidades externas, hay que considerar:

```text
cantidad
   ×
tamaño del elemento
   ↓
overflow
```

Para código robusto, el cálculo del tamaño también forma parte de la seguridad de memoria.

---

# 24. `sizeof` y `size_t`

`size_t` es el tipo destinado a representar tamaños de objetos y cantidades relacionadas con memoria.

Ejemplo:

```c
size_t size;

size = sizeof(int);
```

Y:

```c
size_t count = 10;

int *numbers = malloc(count * sizeof(*numbers));
```

Este patrón es natural en C.

---

# 25. Modelo mental

Cuando veas:

```c
sizeof(expression)
```

pregúntate:

```text
¿Qué objeto/tipo estoy midiendo?
             │
             ▼
       ¿Es un array?
        /       \
      sí         no
      │           │
      ▼           ▼
  tamaño       tamaño del
   total         objeto
                  │
                  ▼
            ¿es un puntero?
                  │
                  ▼
          mide el puntero,
          NO lo que apunta
```

---

# 26. Las tres preguntas que debes hacerte

Ante cualquier `sizeof`, piensa:

### 1. ¿Qué estoy midiendo?

```c
sizeof(numbers)
```

¿`numbers` es un array o un puntero?

### 2. ¿Cuál es el tipo?

```c
sizeof(*p)
```

¿Qué tipo tiene `*p`?

### 3. ¿Necesito el tamaño o la cantidad de elementos?

No son lo mismo:

```text
sizeof(array)
       ↓
bytes

sizeof(array) / sizeof(array[0])
       ↓
elementos
```

---

# 27. Patrón fundamental

```c
int numbers[10];

size_t bytes = sizeof(numbers);
size_t count = sizeof(numbers) / sizeof(numbers[0]);
```

Resultado conceptual:

```text
bytes
  ↓
tamaño total del array

count
  ↓
10 elementos
```

---

# 28. Checklist

Antes de utilizar `sizeof`:

-  ¿Estoy midiendo un tipo o un objeto?
    
-  ¿Estoy confundiendo bytes con elementos?
    
-  ¿Estoy confundiendo un array con un puntero?
    
-  ¿Estoy dentro de una función?
    
-  ¿El parámetro realmente es un puntero?
    
-  ¿Estoy usando `sizeof(*pointer)` para una asignación dinámica?
    
-  ¿Estoy considerando padding/alignment en `struct`?
    
-  ¿Estoy confundiendo `sizeof` con `strlen`?
    
-  ¿El resultado debe almacenarse en `size_t`?
    
-  ¿Existe posibilidad de overflow en un cálculo de tamaño?
    

---

# 29. Resumen de examen

> **`sizeof` es un operador de C que determina el tamaño, en bytes, de un tipo u objeto. Su resultado tiene tipo `size_t`. En arrays, devuelve el tamaño total del array; aplicado a un puntero, devuelve el tamaño del puntero y no de la memoria apuntada.**

### Ejemplos esenciales

```c
sizeof(int);
```

```c
sizeof(variable);
```

```c
sizeof(array);
```

```c
sizeof(*pointer);
```

```c
sizeof(array) / sizeof(array[0]);
```

### Patrón profesional

```c
int *numbers = malloc(count * sizeof(*numbers));
```

---

# 30. Conexiones

```text
                    sizeof
                      │
        ┌─────────────┼─────────────┐
        │             │             │
      types         arrays       pointers
        │             │             │
        │             │             └── sizeof(pointer)
        │             │
        │             └── sizeof(array)
        │                    │
        │                    └── / sizeof(element)
        │
        └── sizeof(struct)
                  │
                  └── padding / alignment

                      │
                      ▼

                    malloc
                      │
                      ▼
             count * sizeof(*ptr)
                      │
                      ▼
                dynamic memory
                      │
                      ▼
                     free
```

> [!important] Regla senior  
> **`sizeof` te dice cuánto ocupa el objeto, no cuánto "parece" ocupar y no cuánto espacio has reservado indirectamente.**
> 
> Especialmente:
> 
> ```c
> sizeof(array)   // tamaño del array
> sizeof(pointer) // tamaño del puntero
> ```
> 
> No confundas **object size**, **pointer size** y **allocation size**.