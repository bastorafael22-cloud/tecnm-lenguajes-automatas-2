# Bitácora de Requerimientos y Auditoría IA — Sesión 25

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Bucles While y For  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Estructura de ciclos en instrucciones TAC.

Al realizar el código manualmente, tuve dificultad para acomodar correctamente las etiquetas de inicio y fin de los ciclos `while` y `for`. También tuve que identificar en qué momento se debía colocar el salto condicional, el salto de regreso y la actualización del iterador, ya que si se colocan en un orden incorrecto el ciclo puede no funcionar como se espera.

Otro problema fue controlar el contador de líneas y el contador de etiquetas, porque cada ciclo necesita dos etiquetas diferentes y no deben repetirse al generar varios ciclos. Para resolverlo, utilicé variables para guardar las etiquetas de inicio y fin, incrementé el contador de etiquetas y comprobé que todavía hubiera espacio disponible antes de escribir cada instrucción.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | Lenguaje C11 con GCC y uso de la interfaz declarada en `api_sesion25.h`. |
| **Rol (R)** | Ingeniero de software especializado en programación de sistemas, compiladores y control de flujo. |
| **Especificación (E)** | Implementar `emitir_while()` y `emitir_for()` para generar el esqueleto TAC de ciclos. Utilizar etiquetas de inicio y fin únicas, generar saltos condicionales, regresar al inicio del ciclo y colocar la actualización del iterador antes del salto de regreso. Si `inicio` o `post` son `NULL`, se debe omitir la instrucción correspondiente. |
| **Output (O)** | Código modular en C, sin `main()`, que respete las firmas originales, controle el espacio disponible en el arreglo y tenga un análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en la materia Lenguajes y Autómatas II.
Necesito implementar dos funciones en C que generen el esqueleto
de código de ciclos while y for utilizando instrucciones TAC,
etiquetas y saltos condicionales.

Rol:
Actúa como un ingeniero de software especializado en programación
de sistemas, compiladores y generación de código intermedio.

Especificación:
Implementa exactamente las siguientes funciones:

void emitir_while(char lineas[][64], int max, int *n,
                  const char *cond, int *etiquetas);

void emitir_for(char lineas[][64], int max, int *n,
               const char *inicio, const char *cond,
               const char *post, int *etiquetas);

Requisitos:
1. La función emitir_while() debe generar cuatro instrucciones:
   marca de inicio, salto condicional, GOTO de regreso
   y marca de fin.
2. El salto condicional debe utilizar IFFALSE hacia la etiqueta fin.
3. El GOTO de regreso debe apuntar a la etiqueta inicio.
4. La función emitir_for() debe generar la inicialización,
   marca de inicio, salto condicional, actualización, GOTO
   de regreso y marca de fin.
5. Si inicio es NULL, se omite la inicialización.
6. Si post es NULL, se omite la actualización.
7. Cada ciclo debe utilizar dos etiquetas diferentes.
8. El contador de etiquetas debe incrementarse en dos por ciclo.
9. Las instrucciones deben agregarse en la posición indicada por *n.
10. El contador *n debe incrementarse por cada instrucción escrita.
11. No escribas más allá del límite max.
12. Utiliza snprintf() para escribir las instrucciones
    en buffers de 64 caracteres.
13. Respeta las firmas de api_sesion25.h.
14. No incluyas main(), porque el harness ya lo contiene.
15. El código debe compilar con GCC usando:
    -std=c11 -Wall -Wextra -Werror.

Output:
Devuelve el código completo de las dos funciones y un análisis
breve de complejidad temporal y espacial. No cambies la
interfaz original.
```

---

## 3. Código Devuelto por el Modelo


```c
#include "api_sesion25.h"
#include <stdio.h>
#include <string.h>

void emitir_while(char lineas[][64], int max, int *n, const char *cond, int *etiquetas) {
    int inicio = *etiquetas;
    int fin = *etiquetas + 1;
    *etiquetas += 2;

    if (*n < max) {
        snprintf(lineas[*n], 64, "L%d:", inicio);
        (*n)++;
    }
    if (*n < max) {
        snprintf(lineas[*n], 64, "IFFALSE %s GOTO L%d", cond, fin);
        (*n)++;
    }
    if (*n < max) {
        snprintf(lineas[*n], 64, "GOTO L%d", inicio);
        (*n)++;
    }
    if (*n < max) {
        snprintf(lineas[*n], 64, "L%d:", fin);
        (*n)++;
    }
}

void emitir_for(char lineas[][64], int max, int *n, const char *inicio, const char *cond, const char *post, int *etiquetas) {
    int lbl_inicio = *etiquetas;
    int lbl_fin = *etiquetas + 1;
    *etiquetas += 2;

    if (inicio != NULL && *n < max) {
        snprintf(lineas[*n], 64, "%s", inicio);
        (*n)++;
    }
    if (*n < max) {
        snprintf(lineas[*n], 64, "L%d:", lbl_inicio);
        (*n)++;
    }
    if (*n < max) {
        snprintf(lineas[*n], 64, "IFFALSE %s GOTO L%d", cond, lbl_fin);
        (*n)++;
    }
    if (post != NULL && *n < max) {
        snprintf(lineas[*n], 64, "%s", post);
        (*n)++;
    }
    if (*n < max) {
        snprintf(lineas[*n], 64, "GOTO L%d", lbl_inicio);
        (*n)++;
    }
    if (*n < max) {
        snprintf(lineas[*n], 64, "L%d:", lbl_fin);
        (*n)++;
    }
}
```

### Análisis de complejidad

- **Complejidad temporal:** O(1), porque cada función genera una cantidad fija de instrucciones y no realiza recorridos de estructuras de datos.
- **Complejidad espacial:** O(1), porque se utilizan los buffers recibidos y algunas variables auxiliares.
- **Memoria dinámica:** No se utiliza `malloc()` ni memoria dinámica adicional.
- **Etiquetas:** Cada ciclo utiliza dos etiquetas y aumenta el contador para evitar colisiones.
- **Buffers:** Las instrucciones se escriben utilizando `snprintf()` con una capacidad de 64 caracteres.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Generación de ciclos `while` y `for`, etiquetas de inicio y fin, inicialización, actualización del iterador y ciclo sin actualización.

### Salida del harness

```text
=== AUDITORIA SESION 25 - Esqueleto TAC de while y for ===
Caso                               esperado:              obtenido:              resultado
[ 1] while: marca de inicio             esperado: L0:          obtenido: L0:          OK
[ 2] while: condicion
     esperado: IFFALSE a < b GOTO L1
     obtenido: IFFALSE a < b GOTO L1
     -> OK
[ 3] dos ciclos sin colision            esperado: L2:          obtenido: L2:          OK
[ 4] for: inicializacion                esperado: i = 0        obtenido: i = 0        OK
[ 5] for: actualizacion                 esperado: i = i + 1    obtenido: i = i + 1    OK
[ 6] for sin post (NULL)                esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion25.md
```

### Lista de verificación

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.


### Reporte Crítico

Al revisar el código generado por la IA, identifiqué que un caso borde que no está contemplado completamente es cuando el arreglo de líneas no tiene suficiente espacio para guardar todas las instrucciones de un ciclo. En esa situación, las funciones pueden escribir solamente una parte del ciclo, pero aun así consumir las dos etiquetas, dejando el código incompleto. Esto lo detecté al revisar las condiciones que comprueban `*n < max` antes de cada escritura.

También comprobé que el `for` coloca correctamente la actualización del iterador antes del salto de regreso y que cuando `post` es `NULL` no se agrega esa instrucción. Las pruebas del harness confirmaron que los casos solicitados funcionan correctamente y que las seis pruebas fueron superadas. Sin embargo, el caso de espacio insuficiente no fue probado por el harness, por lo que queda como una limitación que se debe considerar en una implementación más completa.

Esta práctica me ayudó a comprender cómo se representan los ciclos mediante etiquetas y saltos, cómo se controla la ejecución de una condición y cómo la actualización del iterador permite que un ciclo `for` avance correctamente.

---

**Firma digital:** [Usuario de GitHub]  
**Fecha:** 25 de noviembre de 2026
