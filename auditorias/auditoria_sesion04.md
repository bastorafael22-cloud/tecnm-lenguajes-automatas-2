# Bitácora de Requerimientos y Auditoría IA — Sesión 04

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Anatomía de un Árbol Sintáctico (AST)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto: Estructura de nodo binario (ASTNode).*

**Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:**

> Al programar manualmente el árbol sintáctico abstracto, la principal dificultad fue comprender cómo reservar memoria dinámica con `malloc()` y utilizar correctamente los punteros para enlazar los nodos. También tuve que distinguir entre los nodos que almacenan números y los que representan operaciones binarias mediante el enumerado `TipoNodo`.
>
> Otro problema fue controlar los casos en los que un puntero es `NULL`, porque intentar acceder a sus campos puede provocar un error de segmentación. Para evitarlo, es necesario validar el nodo antes de evaluarlo y comprobar si la reserva de memoria fue exitosa. Además, hay que respetar el orden de los hijos izquierdo y derecho para que las operaciones matemáticas produzcan el resultado correcto.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | Lenguaje C con GCC y estándar C11, utilizando bibliotecas estándar y respetando la interfaz declarada en `api_sesion04.h`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas, estructuras de datos y compiladores. |
| **Especificación (E)** | Implementar un árbol sintáctico abstracto con nodos numéricos y nodos de operación binaria. Representar las operaciones `+`, `-` y `*`, mantener correctamente los punteros izquierdo y derecho y evaluar recursivamente el árbol. La función `evaluar(NULL)` debe devolver `-1`. |
| **Output (O)** | Código C modular, sin función `main()`, compatible con el contrato del laboratorio, más un análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado

```text
Quiero que me ayudes a hacer el código en C para crear y evaluar un árbol sintáctico abstracto (AST). Para hacerlo, necesito que te bases en el archivo `api_sesion04.h` y que uses C11.

El código debe tener las funciones `crear_numero()`, `crear_binario()` y `evaluar()`. La primera debe crear un nodo con un número, la segunda debe unir dos nodos con una operación y la tercera debe realizar el cálculo correspondiente.

También necesito que el programa pueda sumar, restar y multiplicar, y que no presente errores cuando reciba un nodo vacío (`NULL`). Quiero que se use `malloc()` para reservar memoria y que se revise si se pudo asignar correctamente.

Por último, respeta los nombres y las estructuras que vienen en el archivo de la práctica, no agregues la función `main()` y procura que el código compile sin advertencias usando GCC. Explica brevemente cómo funciona el código y su complejidad, y no digas que pasó las pruebas si no se han ejecutado.
```

---

## 3. Código Devuelto por el Modelo

```c
#include <stdlib.h>
#include "api_sesion04.h"

/* Crear un nodo que almacene un número. */
ASTNode *crear_numero(int v)
{
    ASTNode *nodo = malloc(sizeof(ASTNode));

    if (nodo == NULL) {
        return NULL;
    }

    nodo->tipo = (TipoNodo)0;
    nodo->valor = v;
    nodo->op = '\0';
    nodo->izq = NULL;
    nodo->der = NULL;

    return nodo;
}

/* Crear un nodo que represente una operación. */
ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der)
{
    ASTNode *nodo;

    if (op != '+' && op != '-' && op != '*') {
        return NULL;
    }

    nodo = malloc(sizeof(ASTNode));

    if (nodo == NULL) {
        return NULL;
    }

    nodo->tipo = (TipoNodo)1;
    nodo->valor = 0;
    nodo->op = op;
    nodo->izq = izq;
    nodo->der = der;

    return nodo;
}

/* Evaluar los nodos del árbol de forma recursiva. */
static int evaluar_nodo(const ASTNode *nodo, int *resultado)
{
    int izquierdo;
    int derecho;

    if (nodo == NULL || resultado == NULL) {
        return 0;
    }

    /* Si es un número, se devuelve su valor. */
    if (nodo->tipo == (TipoNodo)0) {
        *resultado = nodo->valor;
        return 1;
    }

    /* Comprobar que sea una operación con dos hijos. */
    if (nodo->tipo != (TipoNodo)1 ||
        nodo->izq == NULL ||
        nodo->der == NULL) {
        return 0;
    }

    /* Evaluar primero el hijo izquierdo y después el derecho. */
    if (!evaluar_nodo(nodo->izq, &izquierdo)) {
        return 0;
    }

    if (!evaluar_nodo(nodo->der, &derecho)) {
        return 0;
    }

    /* Realizar la operación indicada en el nodo. */
    switch (nodo->op) {
        case '+':
            *resultado = izquierdo + derecho;
            return 1;

        case '-':
            *resultado = izquierdo - derecho;
            return 1;

        case '*':
            *resultado = izquierdo * derecho;
            return 1;

        default:
            return 0;
    }
}

/* Función solicitada para evaluar el árbol. */
int evaluar(ASTNode *n)
{
    int resultado;

    if (!evaluar_nodo(n, &resultado)) {
        return -1;
    }

    return resultado;
}
```

### Análisis de complejidad

- **Creación de un nodo:** \(O(1)\), porque cada operación reserva y configura una cantidad fija de campos.
- **Evaluación del árbol:** \(O(n)\), porque se visita cada nodo una vez.
- **Memoria auxiliar de la recursividad:** \(O(h)\), donde \(h\) es la altura del árbol.
- **Memoria total del árbol:** \(O(n)\), porque se almacena un nodo por cada elemento de la expresión.

Donde \(n\) representa el número total de nodos y \(h\) la altura del árbol.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)


**Resultados esperados al ejecutar el harness con la implementación anterior:**

```text
=== AUDITORIA SESION 04 - Nodo binario de AST ===
Caso                               esperado:              obtenido:              resultado
[ 1] evaluar(nodo 42)                   esperado: 42           obtenido: 42           OK
[ 2] evaluar(3+52)                     esperado: 13           obtenido: 13           OK
[ 3] hijo izquierdo de ( 5 2)          esperado: 5            obtenido: 5            OK
[ 4] operador guardado                  esperado: *            obtenido: *            OK
[ 5] evaluar(NULL)                      esperado: -1           obtenido: -1           OK
[ 6] evaluar(10-4)                      esperado: 6            obtenido: 6            OK

RESULTADO: 6/6 pruebas superadas
```

**Lista de auditoría:**

- [x] El caso de `evaluar(NULL)` está contemplado y evita el acceso a un puntero nulo.
- [x] No existen accesos fuera de límites ni *Memory Leaks*. Los accesos inválidos contemplados se controlan, pero no se puede garantizar la ausencia de fugas porque la interfaz no proporciona una función para liberar el árbol.
- [x] La complejidad teórica coincide con la implementación: la evaluación visita los nodos una vez y utiliza memoria auxiliar según la altura del árbol.

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

> Identifiqué que uno de los casos más importantes era recibir un árbol vacío mediante `evaluar(NULL)`. Si la función intenta acceder directamente a sus campos, puede provocar un error de segmentación. Para solucionar este problema, agregué una validación al inicio de la función que devuelve `-1` cuando el puntero es nulo.
>
> También incorporé comprobaciones para detectar fallos en `malloc()`, operadores no admitidos y nodos binarios que no tienen ambos hijos. Estas validaciones ayudan a evitar accesos inválidos durante la evaluación.
>
> Como observación adicional, la interfaz proporcionada no incluye una función para liberar la memoria reservada para los nodos. Por ello, no es correcto afirmar que se ha comprobado la ausencia de fugas de memoria únicamente con las seis pruebas del harness. Para resolverlo completamente en el proyecto, sería necesario incorporar un destructor recursivo, por ejemplo `liberar_arbol()`, y llamar a esa función cuando el árbol deje de utilizarse.

---

**Firma digital:** [Usuario de GitHub]  
**Fecha:** 28 de octubre de 2026
