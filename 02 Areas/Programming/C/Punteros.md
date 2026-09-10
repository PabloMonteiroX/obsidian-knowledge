---
type: concept
domain: programming
language: C
status: learning
---
# Pointers — Punteros en C

> [!abstract] Idea central  
> Un **puntero** es un objeto cuyo valor representa una dirección de memoria o, más precisamente, una referencia a un objeto o función.
> 
> La idea fundamental:
> 
> ```text
> variable
>    │
>    │ &
>    ▼
> address
>    │
>    │ *
>    ▼
> object
> ```
> 
> En C:
> 
> ```c
> int value;
> int *p;
> 
> p = &value;
> ```
> 
> `p` contiene la dirección de `value`.
> 
> `*p` permite acceder al objeto al que apunta.

---

# 1. ¿Qué problema resuelve un puntero?

Sin punteros:

```c
int value;

value = 42;
```

Tenemos directamente:

```text
value → 42
```

Con un puntero:

```c
int value;
int *p;

value = 42;
p = &value;
```

Tenemos:

```text
p ─────────► value
              │
              ▼
             42
```

Ahora podemos acceder al mismo objeto indirectamente mediante `p`.

---

# 2. Un puntero es una variable

Esto:

```c
int *p;
```

declara un objeto llamado `p` cuyo tipo es:

```text
pointer to int
```

No significa:

```text
"p es un int"
```

Significa:

```text
"p puede almacenar una referencia/dirección apropiada para un int"
```

Conceptualmente:

```text
p
│
▼
address
```

---

# 3. `&` — operador address-of

El operador:

```c
&
```

obtiene la dirección de un objeto.

Ejemplo:

```c
int value;

value = 42;

int *p;

p = &value;
```

Conceptualmente:

```text
value
┌────────────┐
│     42     │
└────────────┘
      ▲
      │
     &value
      │
      ▼
p ─── address
```

Por tanto:

```c
p == &value
```

es verdadero mientras `p` conserve esa dirección.

---

# 4. `*` — operador de indirección

El operador:

```c
*
```

también tiene otro significado dependiendo del contexto.

En:

```c
int *p;
```

forma parte del declarador:

```text
p → pointer to int
```

En:

```c
*p
```

es el operador de **indirección**.

Significa acceder al objeto al que `p` apunta.

Ejemplo:

```c
int value;
int *p;

value = 42;
p = &value;

printf("%d\n", *p);
```

Resultado:

```text
42
```

---

# 5. `&` y `*` como operaciones conceptualmente inversas

Tenemos:

```c
p = &value;
```

y:

```c
*p
```

Conceptualmente:

```text
&value
   ↓
 address
   ↓
   p
   ↓
  *p
   ↓
 value
```

Por eso:

```c
*p == value
```

cuando `p` apunta correctamente a `value`.

---

# 6. Leer mediante un puntero

```c
int value;
int *p;

value = 42;
p = &value;

printf("%d\n", *p);
```

El flujo es:

```text
p
│
│ contains address
▼
value
│
▼
42
```

`*p` significa:

> Accede al objeto localizado mediante la referencia almacenada en `p`.

---

# 7. Modificar mediante un puntero

El puntero no sirve únicamente para leer.

También podemos modificar:

```c
int value;
int *p;

value = 42;
p = &value;

*p = 100;
```

Ahora:

```c
printf("%d\n", value);
```

produce:

```text
100
```

Porque:

```text
p ─────► value
         │
         ▼
        100
```

No hemos creado otro `int`.

Hemos modificado el mismo objeto.

---

# 8. Pointer ≠ pointee

Hay que distinguir:

```text
pointer
```

de:

```text
pointee
```

Ejemplo:

```c
int value;
int *p;

p = &value;
```

Tenemos:

```text
p
│
│ pointer
▼
value
│
│ pointee
▼
int object
```

`p` y `value` son objetos diferentes.

---

# 9. Tipos de puntero

El tipo importa.

```c
int *p;
char *c;
double *d;
```

Tenemos:

```text
p → pointer to int
c → pointer to char
d → pointer to double
```

El tipo indica al compilador cómo interpretar el objeto apuntado y afecta a operaciones como la aritmética de punteros.

---

# 10. Puntero a `int`

```c
int value;
int *p;

p = &value;
```

Entonces:

```c
*p
```

es un `int`.

---

# 11. Puntero a `char`

```c
char letter;
char *p;

letter = 'A';
p = &letter;
```

Entonces:

```c
*p
```

es un `char`.

---

# 12. Puntero a `double`

```c
double value;
double *p;

value = 3.14;
p = &value;
```

Entonces:

```c
*p
```

es un `double`.

---

# 13. El tamaño de un puntero

No confundas:

```c
sizeof(int)
```

con:

```c
sizeof(int *)
```

Ejemplo:

```c
int value;
int *p;
```

Puede ocurrir:

```text
sizeof(value) → 4
sizeof(p)     → 8
```

en una plataforma típica de 64 bits.

El tamaño exacto depende de la implementación.

> [!important]  
> El tamaño del puntero **no tiene por qué coincidir** con el tamaño del objeto apuntado.

---

# 14. Un puntero también ocupa memoria

Esto:

```c
int *p;
```

crea un objeto `p`.

Por tanto:

```text
STACK
┌──────────────┐
│ p            │
│ pointer      │
└──────────────┘
```

si tiene almacenamiento automático.

Y ese objeto contiene un valor que permite referirse a otro objeto.

---

# 15. Punteros y stack

Ejemplo:

```c
void function(void)
{
    int value;
    int *p;

    value = 42;
    p = &value;
}
```

Conceptualmente:

```text
STACK

┌───────────────┐
│ p             │──────┐
├───────────────┤      │
│ value = 42    │◄─────┘
└───────────────┘
```

Aquí:

```text
p
```

apunta a:

```text
value
```

---

# 16. Punteros y heap

Ahora:

```c
int *p;

p = malloc(sizeof(*p));
```

Conceptualmente:

```text
STACK
┌───────────────┐
│ p             │──────────────┐
└───────────────┘              │
                               ▼
HEAP                     ┌──────────────┐
                         │ dynamic int  │
                         └──────────────┘
```

Tenemos:

```text
p    → stack
*p   → heap
```

Esta distinción es fundamental.

---

# 17. Punteros y `malloc()`

`malloc()` devuelve:

```c
void *
```

Ejemplo:

```c
int *p;

p = malloc(sizeof(*p));
```

El almacenamiento devuelto está adecuadamente alineado para los tipos soportados por `malloc`.

Después:

```c
*p = 42;
```

utilizamos el almacenamiento para un `int`.

---

# 18. `NULL`

Un puntero puede contener un valor nulo:

```c
int *p;

p = NULL;
```

Conceptualmente:

```text
p ─────► NULL
```

`NULL` significa que el puntero no representa una referencia válida a un objeto.

Podemos comprobarlo:

```c
if (p == NULL)
{
    /* no object */
}
```

---

# 19. `NULL` no es una dirección de un objeto

No debes pensar:

```text
NULL = dirección de memoria 0
```

como regla general del lenguaje.

`NULL` es una **null pointer constant** utilizada para representar un puntero nulo.

La representación concreta depende de la implementación.

---

# 20. Puntero no inicializado

Esto es peligroso:

```c
int *p;

*p = 42;
```

`p` no ha sido inicializado.

Su valor es indeterminado.

No sabemos dónde apunta.

Por tanto:

```c
*p = 42;
```

produce comportamiento indefinido.

---

# 21. Puntero nulo vs puntero no inicializado

No son lo mismo.

### Puntero nulo

```c
int *p;

p = NULL;
```

Estado conocido:

```text
p → no object
```

### Puntero no inicializado

```c
int *p;
```

Estado no inicializado.

```text
p → indeterminate value
```

Nunca hagas:

```c
*p = 42;
```

sin establecer primero un destino válido.

---

# 22. Dangling pointer

Después de:

```c
int *p;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

free(p);
```

`p` puede conservar el antiguo valor.

Pero el objeto ya no existe.

```text
p ─────► ❌ freed object
```

Esto es un **dangling pointer**.

---

# 23. Solución habitual

Después de liberar:

```c
free(p);
p = NULL;
```

Ahora:

```text
p ─────► NULL
```

Esto ayuda a evitar reutilizar accidentalmente el puntero.

Pero recuerda:

```text
p = NULL
```

no libera memoria.

La liberación es:

```c
free(p);
```

---

# 24. Pointer arithmetic

Los punteros pueden utilizarse en operaciones aritméticas específicas.

Supongamos:

```c
int numbers[4];

int *p;

p = numbers;
```

Entonces:

```c
p + 1
```

no significa necesariamente:

```text
address + 1 byte
```

Significa avanzar un elemento de tipo `int`.

Conceptualmente:

```text
p
│
▼
numbers[0]

p + 1
│
▼
numbers[1]

p + 2
│
▼
numbers[2]
```

---

# 25. ¿Cuántos bytes avanza `p + 1`?

Depende de:

```c
sizeof(*p)
```

Si:

```text
sizeof(int) = 4
```

entonces conceptualmente:

```text
p + 1 → +4 bytes
```

Si:

```text
sizeof(int) = 8
```

entonces:

```text
p + 1 → +8 bytes
```

Por eso el tipo del puntero importa.

---

# 26. Arrays y punteros

Este concepto es esencial:

```c
int numbers[4];
```

Podemos escribir:

```c
int *p;

p = numbers;
```

En la mayoría de expresiones, `numbers` se convierte en un puntero al primer elemento.

Conceptualmente:

```text
numbers
   │
   ▼
┌────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │
└────┴────┴────┴────┘
  ▲
  │
  p
```

Entonces:

```c
*p
```

accede a:

```c
numbers[0]
```

---

# 27. Array indexing es pointer arithmetic

Una de las equivalencias fundamentales de C:

```c
numbers[i]
```

es equivalente a:

```c
*(numbers + i)
```

Por ejemplo:

```c
numbers[2]
```

equivale a:

```c
*(numbers + 2)
```

Conceptualmente:

```text
numbers
   │
   ▼
[0] [1] [2] [3]
          ▲
          │
      numbers + 2
```

---

# 28. `p[i]`

También podemos escribir:

```c
p[i]
```

si `p` apunta a un elemento de una secuencia válida.

Conceptualmente:

```c
p[i]
```

equivale a:

```c
*(p + i)
```

Esto explica gran parte del funcionamiento interno de arrays en C.

---

# 29. Punteros y strings

Los strings en C son arrays de `char` terminados por:

```c
'\0'
```

Ejemplo:

```c
char text[] = "hello";
```

Conceptualmente:

```text
┌────┬────┬────┬────┬────┬────┐
│ h  │ e  │ l  │ l  │ o  │ \0 │
└────┴────┴────┴────┴────┴────┘
  ▲
  │
 text
```

También podemos recorrerlo mediante un puntero:

```c
char *p;

p = text;

while (*p != '\0')
{
    p++;
}
```

---

# 30. Punteros y funciones

C también permite punteros a funciones.

Ejemplo conceptual:

```c
int (*operation)(int, int);
```

Esto significa:

```text
operation
    ↓
pointer to function
    ↓
returns int
    ↓
takes two int arguments
```

Los function pointers son importantes para:

- callbacks;
    
- tablas de funciones;
    
- interfaces;
    
- sistemas embebidos;
    
- programación genérica en C.
    

---

# 31. Punteros a punteros

Un puntero puede apuntar a otro puntero.

```c
int value;
int *p;
int **pp;

p = &value;
pp = &p;
```

Conceptualmente:

```text
pp
│
▼
p
│
▼
value
│
▼
42
```

Entonces:

```c
*pp
```

produce:

```text
p
```

y:

```c
**pp
```

produce:

```text
value
```

---

# 32. ¿Por qué necesitamos `**`?

Aparece mucho en C.

Por ejemplo:

```c
void allocate(int **p)
{
    *p = malloc(sizeof(**p));
}
```

La función necesita modificar el puntero del caller.

Conceptualmente:

```text
caller
  │
  │ p
  ▼
NULL

      │
      │ &p
      ▼

function
  │
  ▼
int **p
```

La función puede cambiar:

```text
p
```

del caller.

---

# 33. Punteros y paso por referencia

C utiliza **pass-by-value**.

Esto es importante.

Cuando haces:

```c
void function(int *p);
```

el puntero se pasa por valor.

La función recibe una copia del puntero.

Pero ambas copias pueden apuntar al mismo objeto:

```text
CALLER

p ─────┐
       │
       ▼
     object
       ▲
       │
FUNCTION
q ─────┘
```

Por eso la función puede modificar:

```c
*q
```

y el caller verá el cambio en el objeto.

---

# 34. Pero no puede cambiar directamente el puntero del caller

Ejemplo:

```c
void function(int *p)
{
    p = NULL;
}
```

Esto modifica solamente la copia local:

```text
caller:
p ─────► object

function:
p ─────► NULL
```

El `p` del caller sigue apuntando al objeto.

Para modificar el puntero del caller necesitas:

```c
void function(int **p)
```

---

# 35. `const` y punteros

Esta parte es fundamental.

```c
const int *p;
```

Significa:

> puntero a `const int`

A través de `p` no puedes modificar el `int`:

```c
*p = 42; /* error */
```

Pero puedes cambiar dónde apunta:

```c
p = &other;
```

---

# 36. `int *const`

Ahora:

```c
int *const p = &value;
```

Significa:

> puntero constante a `int`

No puedes cambiar el puntero:

```c
p = &other; /* error */
```

Pero puedes modificar el objeto:

```c
*p = 42;
```

---

# 37. `const int *const`

```c
const int *const p = &value;
```

Tenemos:

```text
const pointer
     +
pointer to const int
```

No puedes:

```c
p = &other;
```

ni:

```c
*p = 42;
```

---

# 38. Tabla de `const`

|Declaración|Puedes cambiar `p`|Puedes modificar `*p`|
|---|--:|--:|
|`int *p`|Sí|Sí|
|`const int *p`|Sí|No|
|`int *const p`|No|Sí|
|`const int *const p`|No|No|

---

# 39. Punteros y ownership

Un puntero puede participar en ownership.

Ejemplo:

```c
int *p;

p = malloc(sizeof(*p));
```

Podemos establecer:

```text
p = owner
```

Pero después:

```c
int *q;

q = p;
```

tenemos:

```text
p ─────┐
       ▼
     object
       ▲
       │
q ─────┘
```

Ahora existe aliasing.

Debemos saber quién es responsable de:

```c
free()
```

---

# 40. Punteros y lifetime

Un puntero no prolonga automáticamente el lifetime del objeto.

Ejemplo:

```c
int *getValue(void)
{
    int value;

    value = 42;

    return &value;
}
```

El puntero apunta a un objeto automático cuyo lifetime termina al salir de la función.

Por tanto, el puntero devuelto no proporciona acceso válido al objeto.

---

# 41. Puntero a objeto vs dirección numérica

No debes reducir un puntero simplemente a:

```text
"un número"
```

En muchos sistemas modernos una dirección puede representarse mediante bits, pero semánticamente en C un puntero tiene un **tipo** y reglas específicas de uso.

Por ejemplo:

```c
int *p;
char *q;
```

aunque sus representaciones puedan tener el mismo tamaño:

```text
sizeof(p) == sizeof(q)
```

no significa que sean intercambiables sin considerar las reglas del lenguaje.

---

# 42. Comparar punteros

Hay diferentes operaciones.

Podemos comprobar:

```c
p == q
```

para saber si representan el mismo puntero según las reglas del lenguaje.

También:

```c
p != q
```

Pero las comparaciones relacionales:

```c
p < q
```

tienen restricciones y no deben tratarse simplemente como comparar dos números arbitrarios.

En arrays, las comparaciones entre punteros a elementos de la misma matriz tienen un significado definido.

---

# 43. Punteros fuera de un array

Existe un detalle importante.

Para un array:

```c
int numbers[4];
```

es válido conceptualmente formar:

```c
numbers + 4
```

que apunta **one past the end**.

Pero no puedes hacer:

```c
*(numbers + 4)
```

porque no existe un elemento en esa posición.

```text
[0] [1] [2] [3] [one-past]
 ↑               ↑
start            válido como puntero
                 pero no para dereference
```

---

# 44. Dereference

**Dereference** significa utilizar un puntero para acceder al objeto al que apunta.

```c
int value;
int *p;

p = &value;

*p = 42;
```

Aquí:

```c
*p
```

es una operación de dereference.

---

# 45. Dereference de `NULL`

Esto es incorrecto:

```c
int *p;

p = NULL;

*p = 42;
```

No existe un objeto válido al que acceder.

El comportamiento es indefinido.

---

# 46. Dereference de un dangling pointer

También es incorrecto:

```c
int *p;

p = malloc(sizeof(*p));

if (p == NULL)
    return 1;

free(p);

*p = 42;
```

El objeto ya no existe.

Tenemos:

```text
p
│
▼
freed storage
```

No podemos hacer dereference.

---

# 47. Puntero válido

Un puntero utilizable para dereference debe referirse a un objeto o región de memoria apropiada para esa operación y respetar las reglas del lenguaje.

Por ejemplo:

```c
int value;
int *p;

p = &value;

*p = 42;
```

Aquí:

```text
p ─────► value
```

es una relación válida.

---

# 48. Los tres estados que debes distinguir

```text
INITIALIZED
    │
    ▼
p ─────► valid object
```

o:

```text
NULL
    │
    ▼
p ─────► no object
```

o:

```text
DANGLING
    │
    ▼
p ─────► object lifetime ended
```

Y existe además:

```text
UNINITIALIZED
    │
    ▼
p ─────► indeterminate
```

No los mezcles.

---

# 49. Modelo completo

```text
                         POINTER
                            │
                            ▼
                    ┌───────────────┐
                    │      p        │
                    │ pointer value │
                    └───────┬───────┘
                            │
                            │ refers to
                            ▼
                       ┌──────────┐
                       │  OBJECT  │
                       └──────────┘
                            │
                ┌───────────┼───────────┐
                │           │           │
             lifetime    ownership    type
                │
                ▼
             valid?
                │
        ┌───────┴────────┐
        │                │
       YES               NO
        │                │
      *p             dangling /
                     invalid use
```

---

# 50. Punteros + memoria dinámica

Esta es la conexión que debes dominar:

```c
int *p;

p = malloc(10 * sizeof(*p));

if (p == NULL)
    return 1;
```

Tenemos:

```text
STACK                         HEAP

┌──────────────┐             ┌────┬────┬────┬────┐
│ p            │────────────►│    │    │    │... │
└──────────────┘             └────┴────┴────┴────┘
                              10 × int
```

Después:

```c
free(p);
p = NULL;
```

```text
STACK

┌──────────────┐
│ p = NULL     │
└──────────────┘

HEAP

bloque liberado
```

---

# 51. Checklist de punteros

Cuando veas un puntero, pregunta:

### Tipo

```text
¿A qué tipo apunta?
```

### Valor

```text
¿Qué contiene actualmente?
```

### Destino

```text
¿A qué objeto apunta?
```

### Lifetime

```text
¿Ese objeto todavía existe?
```

### Ownership

```text
¿Quién es responsable de liberarlo?
```

### Aliasing

```text
¿Hay otros punteros al mismo objeto?
```

### Dereference

```text
¿Es válido hacer *p?
```

### Arithmetic

```text
¿Estoy dentro de una secuencia válida?
```

---

# 52. Las cuatro operaciones fundamentales

Memoriza conceptualmente:

```text
&x
 │
 ▼
address/reference to x
```

```text
p
 │
 ▼
pointer value
```

```text
*p
 │
 ▼
object referenced by p
```

```text
p + i
 │
 ▼
i elements forward
```

Estas cuatro ideas explican una enorme parte de C.

---

# 53. Ejemplo final

```c
#include <stdio.h>

int main(void)
{
    int value;
    int *p;

    value = 42;
    p = &value;

    printf("value: %d\n", value);
    printf("*p: %d\n", *p);

    *p = 100;

    printf("value: %d\n", value);
    printf("*p: %d\n", *p);

    return 0;
}
```

Flujo:

```text
value = 42

p = &value

       p
       │
       ▼
   ┌───────┐
   │  42   │
   └───────┘
       ▲
       │
      *p

*p = 100

       p
       │
       ▼
   ┌───────┐
   │  100  │
   └───────┘
```

---

# 54. Modelo mental definitivo

> [!important] No pienses:
> 
> ```text
> pointer = address
> ```
> 
> como definición completa.
> 
> Piensa:
> 
> ```text
> pointer
>    │
>    ▼
> typed reference to an object/function
>    │
>    ├── address/value
>    ├── type
>    ├── lifetime
>    ├── validity
>    └── aliasing
> ```
> 
> Y recuerda:
> 
> ```text
> &x  → obtiene una referencia/dirección apropiada para x
> p   → contiene el valor del puntero
> *p  → accede al objeto apuntado
> ```
> 
> El salto conceptual importante es:
> 
> ```text
> pointer ≠ object
> ```
> 
> Un puntero **se refiere a** un objeto; no es el objeto.

---

# 55. Mapa de conceptos

```text
POINTERS
│
├── declaration
│   └── int *p
│
├── address-of
│   └── &x
│
├── dereference
│   └── *p
│
├── NULL
│
├── initialization
│
├── pointer types
│
├── pointer arithmetic
│   ├── p + i
│   ├── p - i
│   └── one-past-the-end
│
├── arrays
│   └── p[i] == *(p + i)
│
├── strings
│
├── pointers to pointers
│   └── **
│
├── const pointers
│
├── function pointers
│
├── ownership
│
├── lifetime
│
├── aliasing
│
└── validity
    ├── valid
    ├── NULL
    ├── uninitialized
    └── dangling
```

> [!tip] Siguiente paso lógico
> 
> Ahora que tienes:
> 
> ```text
> heap
> stack
> pointers
> ```
> 
> el siguiente concepto debería ser **memory layout / address space**:
> 
> ```text
> Process Address Space
> │
> ├── Text
> ├── Read-only data
> ├── Data
> ├── BSS
> ├── Heap
> └── Stack
> ```
> 
> Después de eso, el salto natural es:
> 
> ```text
> pointers
>      ↓
> arrays
>      ↓
> pointer arithmetic
>      ↓
> strings
>      ↓
> buffers
>      ↓
> memory safety
> ```
> 
> Esa secuencia te prepara directamente para `libft`, C de 42, Linux y posteriormente **memory corruption / exploitation**.


### Relacionado
[[Memory Management]]
[[Stack]]
[[Heap]]
[[malloc]]
[[calloc]]
[[realloc]]
[[free]]