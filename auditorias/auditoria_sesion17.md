# Bitácora de Requerimientos y Auditoría IA — Sesión 17

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Comprobación Semántica de Retornos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Validación de tipo de sentencia return.

Al realizar el ejercicio manualmente, identifiqué que la principal dificultad era comprobar correctamente que el tipo de dato que devuelve una expresión fuera compatible con el tipo declarado por la función. También fue necesario considerar las rutas de ejecución, porque una función puede tener diferentes caminos y alguno podría terminar sin ejecutar un `return`.

Otro problema fue establecer el orden correcto de las validaciones. Primero se debe comprobar si existe incompatibilidad de tipos y después revisar si todas las rutas tienen retorno. Además, tuve que considerar casos especiales como las funciones `void`, los apuntadores nulos y los conteos de rutas incorrectos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Verificar la compatibilidad entre el tipo de retorno de una función y el tipo de la expresión devuelta. También comprobar que todas las rutas de ejecución tengan un retorno cuando sea necesario. |
| **Output (O)** | Código modular en C, respetando la firma establecida, sin función `main()` y con análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Necesito que implementes una función en lenguaje C para mi laboratorio
de Lenguajes y Autómatas II, correspondiente a la Sesión 17:
Comprobación Semántica de Retornos y Rutas de Control.

CONTEXTO:
Estoy trabajando con C99 y GCC. La implementación debe ser compatible
con el archivo api_sesion17.h y compilar utilizando las opciones
-Wall -Wextra -Werror.

ROL:
Actúa como un ingeniero de software especializado en sistemas,
compiladores y análisis semántico.

ESPECIFICACIÓN:
Implementa exactamente la siguiente función:

int diagnosticar_retorno(
    const char *tipo_funcion,
    const char *tipo_expresion,
    int rutas_totales,
    int rutas_con_return
);

La función debe cumplir estas reglas:

1. Regresar 0 cuando el tipo de retorno sea compatible y todas las
   rutas de ejecución tengan return.

2. Regresar 1 cuando exista incompatibilidad entre el tipo declarado
   por la función y el tipo de la expresión devuelta.

3. Regresar 2 cuando exista al menos una ruta de ejecución que no
   tenga return.

4. Regresar -1 cuando alguno de los tipos sea NULL o cuando el
   conteo de rutas sea incoherente, incluyendo cero rutas o más
   retornos que rutas totales.

5. La conversión solamente se acepta cuando el tipo destino sea
   igual o más ancho que el tipo de origen.

6. Una función void no puede devolver un valor de otro tipo.
   Una función void solamente es válida cuando la expresión devuelta
   también es void.

7. La comprobación de tipos debe realizarse antes de revisar las
   rutas de ejecución. Si el tipo es incompatible, debe regresar 1
   aunque también exista una ruta sin retorno.

8. Debe detectar correctamente los casos donde una función tiene
   diferentes caminos de ejecución y alguno no termina en return.

OUTPUT:
Devuelve únicamente la implementación completa de la función,
incluyendo las bibliotecas necesarias.

No agregues main(), porque el archivo harness_sesion17_c.c ya
contiene esa función.

Respeta exactamente la firma original y asegúrate de que el código
compile sin advertencias con:

gcc -std=c11 -Wall -Wextra -Werror

También explica brevemente la complejidad temporal y espacial
de la implementación.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion17.h"
#include <string.h>
#include <stddef.h>

int diagnosticar_retorno(const char *tipo_funcion,
                         const char *tipo_expresion,
                         int rutas_totales,
                         int rutas_con_return) {

    /* 1. Validación de punteros nulos y coherencia de rutas */
    if (tipo_funcion == NULL || tipo_expresion == NULL) {
        return -1;
    }

    if (rutas_totales <= 0 ||
        rutas_con_return < 0 ||
        rutas_con_return > rutas_totales) {
        return -1;
    }

    /* 2. Validación de reglas de tipos */
    int es_void_funcion =
        (strcmp(tipo_funcion, "void") == 0);

    int es_void_expresion =
        (strcmp(tipo_expresion, "void") == 0);

    if (es_void_funcion) {
        if (!es_void_expresion) {
            return 1;
        }
    } else {
        if (es_void_expresion) {
            return 1;
        }

        /* Comparación de tipos */
        if (strcmp(tipo_funcion, tipo_expresion) != 0) {

            /* Permitir ensanchamiento de tipo: int -> double */
            if (strcmp(tipo_funcion, "double") == 0 &&
                strcmp(tipo_expresion, "int") == 0) {

                /* Conversión válida */

            } else {
                return 1;
            }
        }
    }

    /* 3. Validación de rutas de control */
    if (rutas_con_return < rutas_totales) {
        return 2;
    }

    /* 4. Todo correcto */
    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Funciones sin retorno o retornos mixtos.

**Comando utilizado:**

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion17_c.c ia_sesion17_c.c -o test_ia && ./test_ia
```

**Resultado obtenido:**

```text
=== AUDITORIA SESION 17 - Validacion de retornos y rutas ===
Caso                               esperado:              obtenido:              resultado
[ 1] int con int y dos rutas       esperado: 0            obtenido: 0            OK
[ 2] void devuelve valor           esperado: 1            obtenido: 1            OK
[ 3] ruta sin retorno              esperado: 2            obtenido: 2            OK
[ 4] double acepta int             esperado: 0            obtenido: 0            OK
[ 5] int no acepta double          esperado: 1            obtenido: 1            OK
[ 6] cero rutas de ejecucion       esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

**Lista de comprobación:**

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni asignaciones dinámicas que puedan provocar *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementación para los casos evaluados.

### Reporte Crítico

Al revisar el código de la IA, identifiqué que la conversión de tipos estaba limitada a un caso específico: `int` hacia `double`. Aunque el laboratorio establece que se deben aceptar conversiones cuando el tipo destino sea igual o más ancho que el tipo de origen, la implementación no contempla una clasificación general de los tipos de datos.

Para mejorar esta parte, la solución sería implementar una comparación de anchura mediante una tabla o función auxiliar que asigne un nivel a cada tipo y permita aceptar cualquier conversión válida de acuerdo con ese orden.

También comprobé que el programa respeta la prioridad de las validaciones: primero revisa la compatibilidad de tipos y después verifica las rutas de ejecución.

El código entregado por la IA superó los seis casos del harness, obteniendo **6/6 pruebas superadas**. Sin embargo, el resultado verde no significa que se hayan comprobado todas las conversiones posibles, ya que el harness solamente evalúa los casos especificados en el laboratorio.

---

**Firma digital:** [Usuario de GitHub]
