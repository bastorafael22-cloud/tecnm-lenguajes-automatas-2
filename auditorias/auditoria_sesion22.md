# Bitácora de Requerimientos y Auditoría IA — Sesión 22

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Generación de TAC para Expresiones Aritmético-Lógicas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Traductor de nodos AST a TAC.

**Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:**

> La principal dificultad fue comprender cómo recorrer el árbol de expresiones para generar las instrucciones en el orden correcto. Fue necesario utilizar un recorrido postorden, procesando primero el hijo izquierdo y después el derecho, antes de generar la instrucción del operador. También se debe controlar el uso de temporales para evitar que se repitan y respetar el límite de líneas y caracteres de cada buffer. En esta fase no se logró completar la implementación manual, por lo que no se pudo compilar el archivo `artesano_sesion22_c.c`.

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Traducir nodos de un AST a instrucciones TAC mediante recorrido postorden, conservando la precedencia de operadores y generando temporales únicos. |
| **Output (O)** | Código modular en C, sin `main()`, que respete la firma establecida y los límites de los buffers. |

### 2.2 Prompt Estructurado

```text
**Contexto:**

Estoy trabajando en la Sesión 22 de Lenguajes y Autómatas II.
Necesito hacer un programa en C que pueda convertir un árbol AST en instrucciones de código de tres direcciones (TAC), respetando el orden de las operaciones.

**Rol:**

Actúa como un programador con conocimientos de C y compiladores que me ayude a realizar esta actividad.

**Especificación:**

Quiero que implementes la función:

```c
int generar_tac(const Nodo *raiz, char lineas[][64], int max);
```

La función debe recorrer el árbol y generar las instrucciones correspondientes a cada operación.

Necesito que primero se procesen los hijos y después el operador, utilizando el recorrido postorden. También quiero que cada operación tenga un temporal diferente, como `_t0`, `_t1`, `_t2`, para evitar que se repitan.

Las hojas del árbol solamente deben devolver su valor y no deben generar instrucciones. Además, el programa debe respetar el máximo de líneas permitido y no escribir más de 63 caracteres en cada línea.

**Output:**

Entrégame el código completo de la función, bien organizado y fácil de entender, compatible con `api_sesion22.h`. No incluyas `main()` y asegúrate de que compile correctamente en C con `-Wall -Wextra -Werror`.
```

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion22.h"
#include <stdio.h>
#include <string.h>

typedef struct {
    char nombre[64][16];
    int usados;
} Pool;

static const char *nuevo_temporal(Pool *p, int *contador) {
    if (p->usados >= 64) return "_t?";
    snprintf(p->nombre[p->usados],
             sizeof p->nombre[0], "_t%d", (*contador)++);
    return p->nombre[p->usados++];
}

static const char *generar_tac_recursivo(
    const Nodo *raiz,
    char lineas[][64],
    int max,
    int *linea_actual,
    int *temp_counter,
    Pool *p
) {
    if (raiz == NULL) {
        return "";
    }

    if (raiz->tipo == 'H') {
        return raiz->valor;
    }

    const char *izq_val = generar_tac_recursivo(
        raiz->izq, lineas, max, linea_actual, temp_counter, p
    );

    const char *der_val = generar_tac_recursivo(
        raiz->der, lineas, max, linea_actual, temp_counter, p
    );

    const char *temp = nuevo_temporal(p, temp_counter);

    if (*linea_actual < max) {
        snprintf(lineas[*linea_actual], 64, "%s = %s %c %s",
                 temp, izq_val, raiz->op, der_val);
        (*linea_actual)++;
    }

    return temp;
}

int generar_tac(const Nodo *raiz, char lineas[][64], int max) {
    Pool pool;
    pool.usados = 0;

    int temp_counter = 0;
    int lineas_emitidas = 0;

    if (raiz == NULL || raiz->tipo == 'H') {
        return 0;
    }

    generar_tac_recursivo(
        raiz, lineas, max, &lineas_emitidas, &temp_counter, &pool
    );

    return lineas_emitidas;
}
```

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Expresiones anidadas con paréntesis y operaciones aritméticas.

### Salida del harness

```text
=== AUDITORIA SESION 22 - Traductor de AST a TAC con precedencia ===
Caso                               esperado:              obtenido:              resultado
[ 1] a + b * c                          esperado: _t0 = b * c  obtenido: _t0 = b * c  OK
[ 2] a + b * c (2a linea)
     esperado: _t1 = a + _t0
     obtenido: _t1 = a + _t0
     -> OK
[ 3] (a + b) * c
     esperado: _t1 = _t0 * c
     obtenido: _t1 = _t0 * c
     -> OK
[ 4] x * (y - z) + w
     esperado: _t2 = _t1 + w
     obtenido: _t2 = _t1 + w
     -> OK
[ 5] hoja sin operaciones               esperado: 0            obtenido: 0            OK
[ 6] (a + b) * (c - d)
     esperado: _t2 = _t0 * _t1
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion22.md
```

**Checklist de auditoría:**

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> La implementación de IA superó los seis casos del harness. Sin embargo, se identificó que no se validan explícitamente algunos casos adicionales, como recibir un puntero `lineas` NULL, un valor negativo en `max` o un árbol con más de 64 operaciones, ya que el pool de temporales tiene una capacidad fija de 64 elementos. Además, cuando se alcanza el límite del pool, la función devuelve `_t?`, lo que podría generar una instrucción incorrecta. Estos casos no fueron evaluados por el harness proporcionado y sería necesario agregar validaciones y pruebas adicionales para garantizar un funcionamiento seguro.

---

**Firma digital:** Rafal Antonio Castillo Basto
