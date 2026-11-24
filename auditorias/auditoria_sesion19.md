# Bitácora de Requerimientos y Auditoría IA — Sesión 19

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Estructuras (struct) y Offsets de Miembros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Cálculo de offsets de miembros en estructuras y tamaño total de memoria.

**Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:**

> Durante la implementación manual identifiqué que el principal problema era calcular correctamente la posición de cada miembro dentro de una estructura, tomando en cuenta la alineación de memoria y el padding.
>
> Comprendí que cada campo necesita una posición que sea múltiplo de su alineación. Por ejemplo, un campo de tipo char ocupa 1 byte, mientras que un int necesita una alineación de 4 bytes.
>
> Para resolverlo implementé una función que calcula el espacio de relleno necesario antes de colocar cada campo. También fue necesario calcular el tamaño total de la estructura y agregar padding final para que coincidiera con el múltiplo de la alineación más grande.
>
> Otro problema que consideré fue validar las listas vacías, los tipos desconocidos y los nombres de miembros que no existen, devolviendo -1 cuando la operación no es válida.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas, compiladores y administración de memoria. |
| **Especificación (E)** | Implementar funciones para calcular los offsets de los miembros de una estructura y su tamaño total, aplicando alineación individual y padding final. Rechazar listas NULL, tipos desconocidos y miembros inexistentes. |
| **Output (O)** | Código modular en C, compatible con las firmas del archivo api_sesion19.h, sin función main() y compilable con -Wall -Wextra -Werror. |

### 2.2 Prompt Estructurado

```text
=== PROMPT DE SOLICITUD (C-R-E-O) ===

[CONTEXTO]
Estoy trabajando en la Sesión 19 de la materia Lenguajes y Autómatas II. El objetivo es calcular manualmente y mediante código cómo el compilador de C dispone los campos de una estructura (struct) en memoria física aplicando las reglas de alineación (alignment) y relleno (padding) de la arquitectura estándar.

[ROL]
Actúa como un Ingeniero de Software experto en Sistemas y Compiladores de C de bajo nivel, con experiencia en gestión de memoria y alineación de datos.

[ESPECIFICACIÓN]
Necesito que implementes en lenguaje C las siguientes funciones basándote en la interfaz declarada en 'api_sesion19.h':

1. `int offset_campo(const Campo *campos, int total, const char *nombre);` 
   - Debe calcular el offset acumulado de cada miembro, alineando el offset actual al múltiplo del tamaño del tipo del miembro antes de ubicarlo en memoria.
   - Debe retornar el offset del miembro cuyo nombre coincida con el parámetro `nombre`.
   - Si el miembro no existe, la lista es NULL, el total es menor a 0 o se encuentra un tipo desconocido, debe retornar -1.

2. `int tamano_struct(const Campo *campos, int total);`
   - Debe calcular el tamaño final de la estructura acumulando los offsets de forma consecutiva respetando las alineaciones individuales de cada campo.
   - Al finalizar, debe añadir un padding final para redondear el tamaño total al múltiplo del tamaño del miembro con la alineación más grande dentro del struct.
   - Si el total es 0, retorna 0. Si hay errores, retorna -1.

Tipos de datos soportados y su alineación básica (en bytes):
- "char": 1 byte
- "short": 2 bytes
- "int": 4 bytes
- "float": 4 bytes
- "double": 8 bytes

[OUTPUT]
Devuelve únicamente el código de implementación en C que cumpla de forma exacta con la cabecera 'api_sesion19.h', sin incluir la función main() ni explicaciones adicionales redundantes, garantizando que compile limpiamente bajo las banderas de advertencia estricta (-Wall -Wextra -Werror).
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion19.h"
#include <string.h>

static int obtener_alineacion(const char *tipo) {
    if (strcmp(tipo, "char") == 0) return 1;
    if (strcmp(tipo, "short") == 0) return 2;
    if (strcmp(tipo, "int") == 0) return 4;
    if (strcmp(tipo, "float") == 0) return 4;
    if (strcmp(tipo, "double") == 0) return 8;
    return -1;
}

int offset_campo(const Campo *campos, int total,
                 const char *nombre) {
    if (campos == NULL || total < 0 || nombre == NULL)
        return -1;

    int offset_actual = 0;

    for (int i = 0; i < total; i++) {
        int alin = obtener_alineacion(campos[i].tipo);
        if (alin == -1) return -1;

        if (offset_actual % alin != 0) {
            offset_actual += alin - (offset_actual % alin);
        }

        if (strcmp(campos[i].nombre, nombre) == 0) {
            return offset_actual;
        }

        offset_actual += alin;
    }

    return -1;
}

int tamano_struct(const Campo *campos, int total) {
    if (campos == NULL || total < 0) return -1;
    if (total == 0) return 0;

    int offset_actual = 0;
    int max_alin = 1;

    for (int i = 0; i < total; i++) {
        int alin = obtener_alineacion(campos[i].tipo);
        if (alin == -1) return -1;

        if (alin > max_alin) {
            max_alin = alin;
        }

        if (offset_actual % alin != 0) {
            offset_actual += alin - (offset_actual % alin);
        }

        offset_actual += alin;
    }

    if (offset_actual % max_alin != 0) {
        offset_actual += max_alin -
                         (offset_actual % max_alin);
    }

    return offset_actual;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Alineación de miembros, padding, cálculo de tamaño total y búsqueda de miembros inexistentes.

### Salida del harness

```text
=== AUDITORIA SESION 19 - Offsets de miembros y tamano de struct ===
Caso                               esperado:              obtenido:              resultado
[ 1] struct char e int: b               esperado: 4            obtenido: 4            OK
[ 2] struct de tres campos: c           esperado: 8            obtenido: 8            OK
[ 3] tamano total con padding           esperado: 12           obtenido: 12           OK
[ 4] double seguido de char             esperado: 16           obtenido: 16           OK
[ 5] miembro inexistente                esperado: -1           obtenido: -1           OK
[ 6] struct sin campos                  esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion19.md
```

### Verificación de criterios

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

> Al revisar la implementación de la inteligencia artificial, observé que no contemplaba explícitamente la validación de nombres de campos NULL dentro de la estructura ni la validación de tipos NULL.
>
> Aunque las pruebas proporcionadas por el harness fueron superadas, identifiqué que estos casos adicionales podrían provocar errores al utilizar strcmp() con un puntero NULL.
>
> Para mejorar la seguridad de la implementación, es recomendable validar que los nombres y tipos de los campos no sean NULL antes de realizar las comparaciones.
>
> También comprobé que la IA aplicaba correctamente el padding individual y el padding final, comparando sus resultados con mi implementación manual.
>
> Finalmente, ejecuté las dos versiones con el harness y ambas obtuvieron 6 de 6 pruebas superadas, sin advertencias de compilación.

---

---

**Firma digital:** [Usuario de GitHub]
