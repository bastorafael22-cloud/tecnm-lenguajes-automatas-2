# Bitácora de Requerimientos y Auditoría IA — Sesión 07

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Evaluación de Expresiones Aritméticas y Casos Límite  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** [ C | Rust ] 

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Evaluador recursivo de operaciones.

La principal dificultad al desarrollar el evaluador fue implementar correctamente las operaciones aritméticas utilizando un árbol sintáctico abstracto (AST).

Fue necesario comprender cómo recorrer los nodos de manera recursiva, primero evaluando el subárbol izquierdo y después el derecho para finalmente ejecutar la operación correspondiente.

Uno de los problemas más importantes fue controlar la división entre cero y el módulo entre cero, ya que estas operaciones pueden provocar errores durante la ejecución del programa.

También fue necesario implementar un mecanismo para identificar errores mediante una variable llamada `error`, utilizando los siguientes valores:

- `0`: No existe ningún error.
- `1`: División o módulo entre cero.
- `2`: Nodo inválido o árbol NULL.

Otro aspecto importante fue evitar que los errores de los nodos hijos se propagaran incorrectamente o que se continuara evaluando una expresión después de detectar un problema.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C estándar C11, compilador GCC y bibliotecas estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas, árboles sintácticos y compiladores. |
| **Especificación (E)** | Evaluar expresiones aritméticas mediante recursividad. Detectar división y módulo entre cero antes de ejecutar las operaciones. Propagar errores y rechazar nodos inválidos. |
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

---

## 3. Código Devuelto por el Modelo



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

### Complejidad del algoritmo

- **Temporal:** O(n), porque cada nodo del árbol se visita una sola vez.
- **Espacial:** O(h), debido a las llamadas recursivas, donde h representa la altura del árbol.
- No utiliza memoria dinámica ni necesita `malloc()` o `free()`.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Evaluación de expresiones complejas y detección de división y módulo entre cero.

- [x] Se implementaron comprobaciones para evitar divisiones y módulos entre cero.
- [x] Se implementó la propagación de errores y la validación de nodos NULL.
- [x] La complejidad teórica coincide con la implementación recursiva.



### Reporte Crítico

Al revisar la implementación inicial se identificó un problema importante: si una operación dentro de un nodo hijo generaba un error, el programa podía continuar evaluando los nodos superiores.

Esto podía provocar operaciones innecesarias y resultados incorrectos.

Para solucionar este problema se agregaron comprobaciones después de evaluar cada subárbol. Si la variable `error` contiene un valor diferente de cero, la función detiene inmediatamente el recorrido y devuelve el control sin ejecutar la operación del nodo padre.

También se verificó que la división y el módulo comprueben el divisor antes de realizar sus respectivas operaciones. De esta manera se evita ejecutar una operación inválida cuando el divisor es cero.

Otro caso límite contemplado fue recibir un árbol NULL o un tipo de nodo desconocido. En ambos casos se establece el error 2.

El harness contempla seis casos principales que permiten comprobar operaciones aritméticas, división entre cero, módulo entre cero, árboles nulos y evaluación recursiva de expresiones.

La salida de 6/6 pruebas debe confirmarse ejecutando el harness después de crear correctamente los archivos necesarios. Las pruebas del harness no comprueban todos los posibles desbordamientos de enteros ni todos los límites de profundidad recursiva.

---

**Firma digital:** [Usuario de GitHub]
