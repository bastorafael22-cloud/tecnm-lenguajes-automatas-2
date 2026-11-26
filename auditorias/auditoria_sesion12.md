# Bitácora de Requerimientos y Auditoría IA — Sesión 12

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Atributos de Símbolos y Memoria  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348 
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Estructura de registro de símbolos.

Al realizar el ejercicio manualmente, tuve dificultad para comprender cómo calcular el desplazamiento de una variable dentro de la memoria, ya que no basta con sumar el tamaño de los tipos de datos. Es necesario respetar la alineación, lo que puede generar espacios adicionales entre variables. También tuve que identificar el tamaño de cada tipo de dato y considerar cómo validar los tipos desconocidos y los valores negativos del cursor para evitar asignaciones incorrectas.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Calcular el tamaño de los tipos de datos y asignar offsets respetando la alineación natural, validando registros y cursores. |
| **Output (O)** | Código modular en C y análisis de complejidad. |

### 2.2 Prompt Estructurado

Actúa como un ingeniero de software especializado en C y compiladores. Desarrolla un programa modular para calcular el ancho de los tipos de datos y asignar offsets alineados dentro de un registro de símbolo.

Utiliza C99 compatible con GCC y bibliotecas estándar. Implementa las funciones:

- `int ancho_de_tipo(const char *tipo);`
- `int asignar_offset(Simbolo *simbolo, int cursor);`

Los tamaños deben ser: char = 1, short = 2, int = 4, float = 4, double = 8 y puntero = 8 bytes. La función de ancho debe devolver -1 si el tipo es desconocido o nulo.

La función asignar_offset debe validar que el registro no sea NULL, que el cursor no sea negativo y que el tipo sea válido. Debe calcular el offset respetando la alineación del tipo, guardar el offset en el símbolo y devolver el cursor correspondiente al siguiente espacio disponible. No debe permitir que el cursor retroceda ni provocar desbordamientos.

Respeta las firmas originales, no agregues main(), utiliza comentarios claros y entrega únicamente el código modular junto con un análisis breve de complejidad.

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion12.h"
#include <string.h>
#include <limits.h>

int ancho_de_tipo(const char *tipo) {
    if (tipo == NULL)
        return -1;

    if (strcmp(tipo, "char") == 0) return 1;
    if (strcmp(tipo, "short") == 0) return 2;
    if (strcmp(tipo, "int") == 0) return 4;
    if (strcmp(tipo, "float") == 0) return 4;
    if (strcmp(tipo, "double") == 0) return 8;
    if (strcmp(tipo, "puntero") == 0) return 8;

    return -1;
}

int asignar_offset(Simbolo *simbolo, int cursor) {
    if (simbolo == NULL || cursor < 0)
        return -1;

    int ancho = ancho_de_tipo(simbolo->tipo);
    if (ancho == -1)
        return -1;

    /* Calcular alineación */
    int offset = cursor;
    int resto = offset % ancho;

    if (resto != 0) {
        offset += ancho - resto;
    }

    /* Evitar desbordamiento */
    if (offset > INT_MAX - ancho)
        return -1;

    simbolo->offset = offset;

    return offset + ancho;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

```text
=== AUDITORIA SESION 12 - Registro de simbolo y offsets alineados ===
Caso                               esperado:              obtenido:              resultado
[ 1] ancho_de_tipo("int")               esperado: 4            obtenido: 4            OK
[ 2] ancho_de_tipo("double")            esperado: 8            obtenido: 8            OK
[ 3] int en cursor 0                    esperado: 4            obtenido: 4            OK
[ 4] char e int: offset de b            esperado: 4            obtenido: 4            OK
[ 5] double desde cursor 4              esperado: 8            obtenido: 8            OK
[ 6] registro NULL                      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion12.md
```

**Pruebas ejecutadas:** Alineación de tipos y tamaños en bytes.

- ☒ Caso base y límites evaluados sin Crash.
- ☒ No existen accesos fuera de límites ni Memory Leaks detectados en las pruebas realizadas.
- ☒ La complejidad teórica coincide con la implementada.

### Resultados de las pruebas

| Caso | Esperado | Obtenido |
|---|---:|---:|
| `ancho_de_tipo("int")` | 4 | 4 — OK |
| `ancho_de_tipo("double")` | 8 | 8 — OK |
| `int en cursor 0` | 4 | 4 — OK |
| `char e int: offset de b` | 4 | 4 — OK |
| `double desde cursor 4` | 8 | 8 — OK |
| `registro NULL` | -1 | -1 — OK |

**Resultado final:** 6/6 pruebas superadas.

### Reporte Crítico

La IA omitió comprobar directamente algunos casos límite, como recibir un cursor negativo o un tipo desconocido, además de verificar posibles desbordamientos al calcular offsets. El harness comprobó los casos principales, pero no todas las condiciones inválidas. Lo detecté al revisar los requisitos y observar que las pruebas solamente verificaban tamaños de tipos conocidos, alineación básica y un registro NULL. Como mejora, agregué validaciones para cursor negativo, tipo nulo o desconocido y desbordamiento de enteros, evitando asignaciones incorrectas y haciendo el código más seguro.

---

**Firma digital:** [Usuario de GitHub]
