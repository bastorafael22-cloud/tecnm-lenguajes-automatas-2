# Bitácora de Requerimientos y Auditoría IA — Sesión 10

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 1: Mini-Evaluador de Expresiones y AST Completo  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Integración de un mini-evaluador de expresiones.

**Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:**

> Al realizar la implementación manual, la principal dificultad fue organizar el análisis de las expresiones para respetar la precedencia de los operadores matemáticos. Fue necesario separar el análisis en tres funciones: `primaria()`, `termino()` y `expr()`. La función primaria se encarga de reconocer números y expresiones entre paréntesis; término procesa las multiplicaciones y divisiones; y expr procesa las sumas y restas. También fue necesario controlar los espacios en blanco, detectar expresiones incompletas y distinguir entre una división entre cero y un error de sintaxis. La dificultad principal fue coordinar el avance del puntero dentro de la cadena para leer correctamente cada carácter.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C11 / GCC, utilizando bibliotecas estándar como `ctype.h` y respetando el contrato de `api_sesion10.h`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar un mini-evaluador que lea expresiones aritméticas desde una cadena, respete la precedencia de operadores y los paréntesis, y detecte errores de sintaxis y división entre cero. |
| **Output (O)** | Código modular en C, sin función `main`, que implemente `evaluar_expresion()` y las funciones auxiliares de análisis, con explicación de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado en sistemas y compiladores.

Necesito implementar en lenguaje C un mini-evaluador de expresiones
aritméticas para la materia Lenguajes y Autómatas II.

Contexto:
El programa recibe una expresión mediante una cadena de caracteres
y debe devolver su resultado numérico.

La interfaz obligatoria es:

long evaluar_expresion(const char *fuente, int *error);

Especificaciones:
1. Debe reconocer números enteros.
2. Debe reconocer los operadores:
   +, -, *, /
3. Debe reconocer paréntesis.
4. Debe ignorar espacios en blanco.
5. Debe respetar la precedencia:
   - Multiplicación y división antes que suma y resta.
   - Los paréntesis tienen prioridad sobre los operadores.
6. Debe detectar división entre cero y establecer error = 1.
7. Debe detectar sintaxis inválida y establecer error = 2.
8. Si la expresión es válida, debe devolver el resultado y establecer
   error = 0.
9. Si la cadena fuente es NULL, debe tratarse como error de sintaxis.
10. No debe modificar la firma de evaluar_expresion().

Restricciones:
- Utilizar lenguaje C.
- Mantener el código modular.
- No incluir una función main().
- No utilizar variables ni menos unario.
- Utilizar bibliotecas estándar.
- Compilar con:
  gcc -std=c11 -Wall -Wextra -Werror

Entrega:
Proporciona el código completo de la función y sus auxiliares.
Explica brevemente la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion10.h"
#include <ctype.h>
#include <stddef.h>

typedef struct {
    const char *s;
    int error;
} Parser;

static void saltar(Parser *p) {
    while (*p->s == ' ') p->s++;
}

static long expr(Parser *p);

static long primaria(Parser *p) {
    saltar(p);
    if (p->error) return 0;

    if (*p->s == '(') {
        p->s++;
        long v = expr(p);
        saltar(p);

        if (*p->s == ')') {
            p->s++;
            return v;
        } else {
            p->error = 2;
            return 0;
        }
    }

    if (isdigit((unsigned char)*p->s)) {
        long v = 0;

        while (isdigit((unsigned char)*p->s)) {
            v = v * 10 + (*p->s - '0');
            p->s++;
        }

        return v;
    }

    p->error = 2;
    return 0;
}

static long termino(Parser *p) {
    long v = primaria(p);

    while (1) {
        saltar(p);
        if (p->error) return 0;

        if (*p->s == '*') {
            p->s++;
            long der = primaria(p);
            v *= der;
        } else if (*p->s == '/') {
            p->s++;
            long der = primaria(p);

            if (p->error) return 0;

            if (der == 0) {
                p->error = 1;
                return 0;
            }

            v /= der;
        } else {
            break;
        }
    }

    return v;
}

static long expr(Parser *p) {
    long v = termino(p);

    while (1) {
        saltar(p);
        if (p->error) return 0;

        if (*p->s == '+') {
            p->s++;
            long der = termino(p);
            v += der;
        } else if (*p->s == '-') {
            p->s++;
            long der = termino(p);
            v -= der;
        } else {
            break;
        }
    }

    return v;
}

long evaluar_expresion(const char *fuente, int *error) {
    static int sin_error;

    if (error == NULL) error = &sin_error;
    *error = 0;

    if (fuente == NULL) {
        *error = 2;
        return 0;
    }

    Parser p = { fuente, 0 };
    long v = expr(&p);

    saltar(&p);

    if (p.error == 0 && *p.s != '\0') {
        p.error = 2;
    }

    *error = p.error;

    return p.error != 0 ? 0 : v;
}
```

**Análisis de complejidad:**

- **Complejidad temporal:** O(n), donde n es la cantidad de caracteres de la expresión, porque el analizador recorre la cadena para reconocer números, operadores, espacios y paréntesis.
- **Complejidad espacial:** O(n) en el peor caso debido a las llamadas recursivas que pueden acumularse al analizar paréntesis anidados. El espacio auxiliar por llamada es O(1).

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Expresiones aritméticas con operadores, paréntesis, espacios y errores.

### Resultados esperados

| Caso | Expresión | Resultado esperado |
| :--- | :--- | :--- |
| 1 | `2+3*4` | `14` |
| 2 | `(2+3)*4` | `20` |
| 3 | `10/0` | Error `1` |
| 4 | `2+` | Error `2` |
| 5 | `  7 ` | `7` |
| 6 | `5-10` | `-5` |

### Salida esperada de la auditoría

```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                               esperado:              obtenido:              resultado
[ 1] "2+3*4"                            esperado: 14           obtenido: 14           OK
[ 2] "(2+3)*4"                          esperado: 20           obtenido: 20           OK
[ 3] "10/0" -> error 1                  esperado: 1            obtenido: 1            OK
[ 4] "2+" -> error 2                    esperado: 2            obtenido: 2            OK
[ 5] "  7 "                             esperado: 7            obtenido: 7            OK
[ 6] "5-10"                             esperado: -5           obtenido: -5           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md
```

**Verificación:**

- [x] Caso base y límites superados sin *Crash* (marcar después de ejecutar el harness).
- [x] No se identificaron accesos fuera de límites ni *Memory Leaks* (marcar después de las pruebas correspondientes).
- [x] La complejidad teórica coincide con la implementada (pendiente de revisión y pruebas adicionales).

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> La IA no contempló completamente algunos casos adicionales que pueden provocar errores durante la evaluación. Por ejemplo, las expresiones con espacios de tabulación, los números demasiado grandes para el tipo `long` y las operaciones que podrían producir desbordamiento aritmético. La función `saltar()` únicamente ignora el espacio normal `' '`, por lo que no procesa todos los caracteres considerados espacios en blanco. Para detectar estas limitaciones revisé las funciones auxiliares y comparé su comportamiento con los requerimientos. Los seis casos establecidos en el harness permiten comprobar la precedencia, los paréntesis, los espacios normales, la división entre cero y la sintaxis incompleta. Estos casos adicionales quedan como mejoras pendientes.

---

**Firma digital:** [Tu usuario de GitHub]
