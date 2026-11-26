# Bitácora de Requerimientos y Auditoría IA — Sesión 07

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Evaluación de Expresiones Aritméticas y Casos Límite  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Desarrollar un evaluador recursivo de operaciones aritméticas mediante un árbol sintáctico abstracto (AST).

Una de las principales dificultades durante el desarrollo fue comprender cómo recorrer correctamente los nodos del árbol utilizando recursividad. Primero se debe evaluar el subárbol izquierdo, después el derecho y finalmente realizar la operación correspondiente.

También fue necesario controlar los casos en los que se intenta realizar una división o un módulo entre cero, ya que estas operaciones pueden provocar errores durante la ejecución del programa.

Para identificar los problemas se utilizó una variable llamada `error`, con los siguientes valores:

- `0`: No existe ningún error.
- `1`: División o módulo entre cero.
- `2`: Nodo inválido o árbol NULL.

Otro aspecto importante fue detener la evaluación cuando alguno de los nodos hijos presenta un error. De esta manera se evita continuar realizando operaciones cuando la expresión ya contiene un problema.

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C estándar C11, compilador GCC y bibliotecas estándar. |
| **Rol (R)** | Ingeniero de software especializado en programación, árboles sintácticos y compiladores. |
| **Especificación (E)** | Evaluar expresiones aritméticas mediante recursividad. Detectar divisiones y módulos entre cero, propagar errores y rechazar nodos inválidos. |
| **Output (O)** | Función modular `int evaluar(ASTNode *n, int *error)`, sin `main()`, con análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado

```text
Actúa como ingeniero de software especializado en C
y desarrollo de compiladores.

Implementa la función:

int evaluar(ASTNode *n, int *error);

Utiliza la estructura ASTNode declarada en
api_sesion07.h. Respeta exactamente la firma.

La función debe evaluar árboles sintácticos
de operaciones aritméticas utilizando recursividad.

Reglas:

1. Si el nodo es NODO_NUM, devuelve su valor.
2. Si el nodo es NODO_BINOP, evalúa primero
   el subárbol izquierdo y después el derecho.
3. Implementa las operaciones +, -, *, / y %.
4. Antes de realizar una división, comprueba
   que el divisor no sea cero.
5. Antes de realizar un módulo, comprueba
   que el divisor no sea cero.
6. Si existe división o módulo entre cero,
   establece error = 1.
7. Si el nodo es NULL o inválido,
   establece error = 2.
8. Si un nodo hijo presenta un error,
   propaga el error y no evalúes el padre.
9. Si el puntero error es NULL, evita
   acceder a memoria inválida.

No agregues main().
No modifiques la estructura ASTNode.
No cambies la firma de la función.

El código debe compilar con:

gcc -std=c11 -Wall -Wextra -Werror

Entrega código modular, comentarios claros
y explicación de complejidad temporal y espacial.
```

## 3. Fase 2: Código Devuelto por el Modelo

**Código completo en C:**

```c
#include "api_sesion07.h"

static int evaluar_recursivo(ASTNode *n, int *error) {

    // Detener la evaluacion si ya existe un error
    if (*error != 0) {
        return 0;
    }

    // Comprobar si el nodo es NULL
    if (n == NULL) {
        *error = 2;
        return 0;
    }

    // Si es un nodo numerico, devolver su valor
    if (n->tipo == NODO_NUM) {
        return n->valor;
    }

    // Comprobar que sea un nodo de operacion
    if (n->tipo != NODO_BINOP) {
        *error = 2;
        return 0;
    }

    // Evaluar primero el subarbol izquierdo
    int izq_val = evaluar_recursivo(n->izq, error);

    // Propagar el error sin evaluar el padre
    if (*error != 0) {
        return 0;
    }

    // Evaluar el subarbol derecho
    int der_val = evaluar_recursivo(n->der, error);

    // Propagar el error sin continuar
    if (*error != 0) {
        return 0;
    }

    // Ejecutar la operacion correspondiente
    switch (n->op) {

        case '+':
            return izq_val + der_val;

        case '-':
            return izq_val - der_val;

        case '*':
            return izq_val * der_val;

        case '/':
            if (der_val == 0) {
                *error = 1;
                return 0;
            }
            return izq_val / der_val;

        case '%':
            if (der_val == 0) {
                *error = 1;
                return 0;
            }
            return izq_val % der_val;

        default:
            *error = 2;
            return 0;
    }
}

// Funcion principal del evaluador
int evaluar(ASTNode *n, int *error) {

    // Validar el puntero de error
    if (error == NULL) {
        return 0;
    }

    // Inicializar el estado de error
    *error = 0;

    // Iniciar la evaluacion recursiva
    return evaluar_recursivo(n, error);
}
```

### 3.1 Complejidad del algoritmo

- **Complejidad temporal: O(n).** Cada nodo del árbol se visita como máximo una vez durante la evaluación.
- **Complejidad espacial: O(h).** Depende de la altura del árbol y de las llamadas recursivas que se mantienen en la pila.
- **Memoria dinámica:** No se utiliza `malloc()` ni `free()`, ya que la función trabaja con los nodos recibidos.

## 4. Fase 3: El Auditor (Resultados del Test Harness)

Se ejecutó la auditoría del evaluador recursivo utilizando seis casos de prueba. Los resultados obtenidos coinciden con los valores esperados.

### 4.1 Resultados de las pruebas

| Caso | Expresión o situación | Resultado esperado | Resultado obtenido | Estado |
| :---: | :--- | :---: | :---: | :---: |
| 1 | `(2+3)*4` | 20 | 20 | OK |
| 2 | `7/0` | Error 1 | Error 1 | OK |
| 3 | `7%%0` (módulo entre cero) | Error 1 | Error 1 | OK |
| 4 | Árbol `NULL` | Error 2 | Error 2 | OK |
| 5 | `10-2-3` | 5 | 5 | OK |
| 6 | `(8/2)+1` | 5 | 5 | OK |

### 4.2 Resultado de la auditoría

```text
\n=== PRUEBAS CON IA ===
=== AUDITORIA SESION 07 - Evaluador recursivo con deteccion de errores ===
Caso                               esperado:              obtenido:              resultado
[ 1] (2+3)*4                            esperado: 20           obtenido: 20           OK
[ 2] 7/0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 3] 7%%0 -> error 1                    esperado: 1            obtenido: 1            OK
[ 4] arbol NULL -> error 2              esperado: 2            obtenido: 2            OK
[ 5] 10-2-3                             esperado: 5            obtenido: 5            OK
[ 6] (8/2)+1                            esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion07.md
```

<badge color="success">6 de 6 pruebas superadas</badge>

### 4.3 Lista de verificación

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

## 5. Reporte Crítico

Durante la revisión de la implementación se verificó que el evaluador detuviera el recorrido cuando alguno de los nodos hijos presenta un error. Para conseguirlo, se agregaron comprobaciones después de evaluar el subárbol izquierdo y el derecho. Así se evita ejecutar operaciones innecesarias cuando ya se detectó un problema.

También se comprobó que la división y el módulo revisaran el valor del divisor antes de realizar sus respectivas operaciones. Cuando el divisor es cero, se establece el código de error 1 y se detiene la evaluación.

Otro caso importante fue la recepción de un árbol NULL o de un tipo de nodo desconocido. En estas situaciones, la función establece el código de error 2 para indicar que la estructura recibida no es válida.

Finalmente, se ejecutó el Test Harness y se obtuvieron seis pruebas superadas de seis posibles. Esto demuestra que la implementación cumple con los casos evaluados de operaciones aritméticas, evaluación recursiva y detección de errores.

Como limitación, las pruebas realizadas no cubren todos los posibles desbordamientos de enteros ni los límites de profundidad de la recursividad. Estos aspectos podrían revisarse en una auditoría más amplia.

---

**Firma digital:** [Usuario de GitHub]
