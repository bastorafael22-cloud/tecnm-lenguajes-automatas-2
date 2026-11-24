# Bitácora de Requerimientos y Auditoría IA — Sesión 18

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arreglos y Verificación de Límites Estáticos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Cálculo del tamaño de arreglos en memoria y validación de índices.

Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:

> Durante la implementación identifiqué que el principal problema era calcular correctamente la cantidad de memoria que necesita un arreglo dependiendo del tipo de dato y del número de elementos.
>
> Para solucionarlo utilicé la función `sizeof()` de C, que permite conocer el tamaño en bytes de cada tipo de dato, como char, short, int, float y double.
>
> También tuve que considerar los casos especiales, como cuando el número de elementos es negativo, cuando es cero y cuando el tipo de dato no existe.
>
> En la función `indice_en_rango()` tuve que verificar que los índices fueran mayores o iguales a cero y menores que el número total de elementos, evitando así accesos fuera de los límites del arreglo.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar funciones para calcular el tamaño en bytes de arreglos y verificar que los índices estén dentro de los límites permitidos. Rechazar tipos desconocidos y cantidades negativas de elementos. |
| **Output (O)** | Código modular en C, respetando las firmas establecidas, sin función main(), compatible con el harness y sin advertencias de compilación. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en la Sesión 18 de Lenguajes y Autómatas II.
Necesito implementar dos funciones en lenguaje C para calcular
el tamaño de arreglos y verificar sus límites estáticos.

Rol:
Actúa como un ingeniero de software especializado en sistemas,
programación en C y desarrollo de compiladores.

Especificación:
Implementa las siguientes funciones:

long bytes_arreglo(const char *tipo, int elementos);
int indice_en_rango(int indice, int elementos);

La función bytes_arreglo debe:
- Reconocer char, short, int, float, double y puntero.
- Calcular los bytes multiplicando el tamaño del tipo
  por la cantidad de elementos.
- Devolver -1 si el tipo no es válido.
- Devolver -1 si los elementos son negativos.
- Devolver 0 cuando hay cero elementos.
- Utilizar sizeof() para obtener los tamaños correctos.

La función indice_en_rango debe:
- Devolver 1 cuando el índice esté dentro del rango.
- Devolver 0 cuando el índice sea negativo.
- Devolver 0 cuando el índice sea igual o mayor
  al número de elementos.
- Rechazar arreglos con cero o menos elementos.

Output:
Entrega únicamente código C modular, compatible con
gcc -std=c11 -Wall -Wextra -Werror.
No incluyas main() y conserva exactamente las firmas
de las funciones proporcionadas.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion18.h"
#include <string.h>

long bytes_arreglo(const char *tipo, int elementos) {
    if (elementos < 0) return -1;
    if (elementos == 0) return 0;
    if (!tipo) return -1;

    long size = -1;

    if (strcmp(tipo, "char") == 0)
        size = 1;
    else if (strcmp(tipo, "short") == 0)
        size = 2;
    else if (strcmp(tipo, "int") == 0)
        size = 4;
    else if (strcmp(tipo, "float") == 0)
        size = 4;
    else if (strcmp(tipo, "double") == 0)
        size = 8;
    else if (strcmp(tipo, "puntero") == 0)
        size = sizeof(void*);

    if (size == -1) return -1;

    return size * elementos;
}

int indice_en_rango(int indice, int elementos) {
    if (elementos <= 0) return 0;

    return (indice >= 0 && indice < elementos) ? 1 : 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Cálculo de dimensiones, tipos de datos y verificación de límites estáticos.

### Salida del harness

```text
=== AUDITORIA SESION 18 - Tamano de arreglo y limites ===
Caso                               esperado:              obtenido:              resultado
[ 1] int[10]                            esperado: 40           obtenido: 40           OK
[ 2] double[3]                          esperado: 24           obtenido: 24           OK
[ 3] int[-4]                            esperado: -1           obtenido: -1           OK
[ 4] indice 0 de 5                      esperado: 1            obtenido: 1            OK
[ 5] indice 5 de 5                      esperado: 0            obtenido: 0            OK
[ 6] indice negativo                    esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion18.md
```

### Verificación de criterios

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

> Al revisar el código generado por la IA encontré que utilizaba tamaños fijos para algunos tipos de datos, como char = 1, short = 2, int = 4, float = 4 y double = 8, en lugar de utilizar sizeof() para obtener el tamaño real de cada tipo según la plataforma.
>
> También observé que la implementación de la IA únicamente reconocía el nombre "puntero", mientras que mi implementación manual permitía reconocer también nombres que contienen el símbolo '*'.
>
> Detecté estas diferencias comparando ambas implementaciones y revisando los requisitos del laboratorio. Para mejorar la portabilidad del código, conservé el uso de sizeof() en la implementación manual.
>
> Finalmente, ejecuté el harness con las dos versiones y comprobé que ambas obtuvieron 6 de 6 pruebas superadas, sin errores de compilación.

---

**Firma digital:** [Usuario de GitHub]
