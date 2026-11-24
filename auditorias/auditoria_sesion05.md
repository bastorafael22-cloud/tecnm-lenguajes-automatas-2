# Bitácora de Requerimientos y Auditoría IA — Sesión 05

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Recorridos del AST y Notación Polaca Inversa (RPN)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348

**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Implementar el recorrido postorden de un árbol sintáctico abstracto (AST) y guardar su representación RPN en un buffer.

La parte más importante fue respetar el orden del recorrido: primero el subárbol izquierdo, después el subárbol derecho y al final el nodo actual. También fue necesario controlar la memoria del buffer: debía reservarse un espacio para el terminador `\0`, separar los tokens con un solo espacio y evitar que quedara un espacio al final. Otro caso que debía contemplarse era recibir un árbol `NULL`, que debe producir una cadena vacía.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariantes de recorrido recursivo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt estructurado

```text
Actúa como ingeniero de software especializado en C y compiladores.
Implementa la función:
void postorden(ASTNode *n, char *buf, int cap);

Usa la estructura ASTNode y el contrato de api_sesion05.h. Recorre el
árbol en postorden (izquierdo, derecho, raíz) y genera notación polaca
inversa. Escribe los números y operadores en el buffer, separados por
un solo espacio y sin espacio final. No uses printf ni agregues main.

Respeta cap: nunca escribas más de cap-1 caracteres y conserva el
terminador nulo. Si el árbol es NULL, el resultado debe ser una cadena
vacía. El código debe compilar con:
gcc -std=c11 -Wall -Wextra -Werror

Evita escribir tokens parcialmente cuando el buffer no tenga espacio.
Explica los casos límite y la complejidad temporal y espacial.
```

---

## 3. Código devuelto por el modelo


```c
#include "api_sesion05.h"
#include <stdio.h>
#include <string.h>

static void postorden_recursivo(
    ASTNode *n,
    char *buf,
    int cap,
    int *usados,
    int *truncado
) {
    if (n == NULL || *truncado) {
        return;
    }

    // Recorrer el subarbol izquierdo
    postorden_recursivo(
        n->izq, buf, cap, usados, truncado
    );

    // Recorrer el subarbol derecho
    postorden_recursivo(
        n->der, buf, cap, usados, truncado
    );

    if (*truncado) {
        return;
    }

    // Procesar la raiz
    char temp[32];

    if (n->tipo == NODO_NUM) {
        snprintf(temp, sizeof(temp), "%d", n->valor);
    } else {
        snprintf(temp, sizeof(temp), "%c", n->op);
    }

    int longitud = (int)strlen(temp);
    int espacio = (*usados > 0) ? 1 : 0;

    // Comprobar capacidad del buffer
    if (*usados + espacio + longitud >= cap) {
        *truncado = 1;
        return;
    }

    // Agregar espacio entre tokens
    if (espacio) {
        buf[(*usados)++] = ' ';
    }

    // Copiar el token al buffer
    memcpy(buf + *usados, temp, (size_t)longitud);
    *usados += longitud;

    // Mantener terminador de cadena
    buf[*usados] = '\0';
}

void postorden(ASTNode *n, char *buf, int cap) {
    if (buf == NULL || cap <= 0) {
        return;
    }

    buf[0] = '\0';

    int usados = 0;
    int truncado = 0;

    postorden_recursivo(
        n, buf, cap, &usados, &truncado
    );
}
```

### Complejidad del algoritmo

- **Temporal:** O(n), porque cada nodo se visita una vez cuando el buffer permite completar el recorrido.
- **Espacial:** O(h), debido a las llamadas recursivas, donde h representa la altura del árbol.
- No utiliza memoria dinámica ni necesita malloc() o free().

**Nota:** Este código corresponde al archivo `artesano_sesion05_c.c` y debe compilarse junto con `api_sesion05.h` y `harness_sesion05_c.c`.

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Validación contra desbordamiento de pila

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementada.


**Resultado de la ejecución local de los archivos incluidos:** 5/5 pruebas superadas. La compilación se realizó con `-Wall -Wextra -Werror`.

### Reporte crítico

Al revisar el manejo del buffer se detectó un caso límite que no estaba cubierto por los cinco ejemplos principales: que quedara espacio suficiente para escribir el separador, pero no para escribir el token siguiente y el terminador. Una implementación que agregue primero el espacio podría dejar un espacio final cuando el token no cabe. Para resolverlo, la función calcula primero el espacio requerido y solo escribe el separador y el token cuando ambos caben completos. Si no caben, detiene las escrituras posteriores.

También se comprobó que el árbol `NULL` deja el buffer como cadena vacía y que la función no necesita memoria dinámica, por lo que no realiza `malloc` ni `free`. El harness de cinco casos comprueba las salidas RPN indicadas, pero por sí solo no constituye una prueba exhaustiva de desbordamiento de pila ni de todos los tamaños de buffer; esas propiedades se revisan en el código y pueden complementarse con pruebas específicas de límites.


---

**Firma digital / usuario de GitHub:** [Escribe tu usuario de GitHub]
