# Bitácora de Requerimientos y Auditoría IA — Sesión 08

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Inferencia y Promoción de Tipos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Implementar las reglas de promoción de tipos y detectar cuándo una conversión puede ocasionar pérdida de información.

Al desarrollar las funciones manualmente, una de las principales dificultades fue comprender cómo comparar los tipos de datos utilizando su jerarquía. Fue necesario identificar que `char` tiene menor rango que `int`, y que `int` tiene menor rango que `float`.

También tuve que considerar el tipo especial `TIPO_ERROR`, porque no debe tratarse como un tipo de dato normal. Por eso, antes de comparar los tipos, agregué una condición para comprobar si alguno de los valores recibidos representa un error.

En la función `hay_perdida()` tuve que establecer una comparación para identificar cuándo el tipo de destino es más pequeño que el tipo de origen. Esto permite reconocer cuándo una conversión puede perder información.

Finalmente, en `nombre_tipo()` utilicé una estructura `switch` para devolver el nombre correspondiente a cada tipo y contemplar los valores no reconocidos.

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C11, compilador GCC y bibliotecas estándar, respetando la compilación modular. |
| **Rol (R)** | Ingeniero de software especializado en sistemas, programación en C y desarrollo de compiladores. |
| **Especificación (E)** | Implementar la promoción de tipos entre `char`, `int` y `float`; identificar conversiones que puedan ocasionar pérdida de información y obtener el nombre de cada tipo. |
| **Output (O)** | Código modular en C que implemente `promover()`, `hay_perdida()` y `nombre_tipo()`, sin incluir una función `main()` y con análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado en sistemas y compiladores.

Necesito implementar en lenguaje C las funciones de
inferencia y promoción de tipos para un proyecto
de Lenguajes y Autómatas II.

Contexto:
El programa trabaja con la siguiente jerarquía:

CHAR < INT < FLOAT

Requerimientos:

1. Implementar la función:
   Tipo promover(Tipo a, Tipo b);

   Debe devolver el tipo de mayor jerarquía
   entre los dos tipos recibidos.

   Si alguno de los tipos es TIPO_ERROR,
   debe devolver TIPO_ERROR.

2. Implementar la función:
   int hay_perdida(Tipo destino, Tipo origen);

   Debe devolver 1 cuando el tipo destino tenga
   menor jerarquía que el tipo origen.

   Debe devolver 0 cuando no exista pérdida.

3. Implementar la función:
   const char *nombre_tipo(Tipo t);

   Debe devolver el nombre correspondiente:

   TIPO_CHAR -> "char"
   TIPO_INT -> "int"
   TIPO_FLOAT -> "float"

   Para cualquier otro valor debe devolver "error".

Restricciones:

- Utilizar lenguaje C.
- Incluir api_sesion08.h.
- Mantener el código modular.
- No agregar una función main().
- Respetar exactamente las firmas establecidas.
- Evitar bibliotecas innecesarias.
- El código debe compilar con:

  gcc -std=c11 -Wall -Wextra -Werror

Entrega:

Proporciona el código completo y explica
la complejidad temporal y espacial de cada función.
```

---

## 3. Código Devuelto por el Modelo

**Archivo:** `ia_sesion08_c.c`

```c
#include "api_sesion08.h"

Tipo promover(Tipo a, Tipo b) {
    if (a == TIPO_ERROR || b == TIPO_ERROR) {
        return TIPO_ERROR;
    }

    return (a > b) ? a : b;
}

int hay_perdida(Tipo destino, Tipo origen) {
    if (destino == TIPO_ERROR || origen == TIPO_ERROR) {
        return 1;
    }

    return (destino < origen) ? 1 : 0;
}

const char *nombre_tipo(Tipo t) {
    switch (t) {
        case TIPO_CHAR:
            return "char";

        case TIPO_INT:
            return "int";

        case TIPO_FLOAT:
            return "float";

        default:
            return "error";
    }
}
```

### 3.1 Análisis de complejidad

- **Función promover():** Complejidad temporal O(1), porque realiza comparaciones constantes para determinar cuál tipo tiene mayor jerarquía.
- **Función hay_perdida():** Complejidad temporal O(1), porque solamente compara los valores de los tipos recibidos.
- **Función nombre_tipo():** Complejidad temporal O(1), porque utiliza una estructura `switch` con una cantidad fija de opciones.
- **Complejidad espacial:** O(1) en las tres funciones, porque no utilizan memoria dinámica ni estructuras adicionales.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

Se ejecutó el programa de pruebas para comprobar que las funciones implementadas cumplieran con las reglas de promoción de tipos y detección de pérdida de información.

### 4.1 Resultados obtenidos

| Caso | Prueba | Esperado | Obtenido | Resultado |
| :---: | :--- | :---: | :---: | :---: |
| 1 | `promover(CHAR,INT)` | int | int | OK |
| 2 | `promover(INT,FLOAT)` | float | float | OK |
| 3 | `promover(FLOAT,FLOAT)` | float | float | OK |
| 4 | `promover(ERROR,INT)` | error | error | OK |
| 5 | `hay_perdida(CHAR,INT)` | 1 | 1 | OK |
| 6 | `hay_perdida(INT,CHAR)` | 0 | 0 | OK |

### 4.2 Evidencia de ejecución

```text
=== AUDITORIA SESION 08 - Reglas de promocion y perdida ===
Caso                               esperado:              obtenido:              resultado
[ 1] promover(CHAR,INT)                 esperado: int          obtenido: int          OK
[ 2] promover(INT,FLOAT)                esperado: float        obtenido: float        OK
[ 3] promover(FLOAT,FLOAT)              esperado: float        obtenido: float        OK
[ 4] promover(ERROR,INT)                esperado: error        obtenido: error        OK
[ 5] hay_perdida(CHAR,INT)              esperado: 1            obtenido: 1            OK
[ 6] hay_perdida(INT,CHAR)              esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion08.md
```

<badge color="success">6/6 pruebas superadas</badge>

### 4.3 Lista de verificación

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

## 5. Reporte Crítico

Al revisar el código generado por la IA, identifiqué que no se contempló completamente el caso de recibir valores de tipo que no pertenecen a los enumeradores definidos en `api_sesion08.h`.

Por ejemplo, la función `promover()` comprueba si alguno de los argumentos es `TIPO_ERROR`, pero no valida otros valores desconocidos. Si recibe un valor inválido, como `(Tipo)99`, podría devolverlo como si fuera un tipo válido.

Detecté esta situación al revisar las condiciones de la función `promover()` y compararlas con la estructura `switch` de `nombre_tipo()`, donde sí existe una opción `default` que devuelve `"error"`.

También observé que la función `hay_perdida()` devuelve 1 cuando recibe `TIPO_ERROR`, aunque el contrato original no especifica expresamente qué comportamiento debe tener en ese caso. Por lo tanto, es una decisión que convendría definir y comprobar mediante pruebas adicionales.

Es importante mencionar que estos casos adicionales no forman parte de los seis escenarios evaluados por el harness. Por esta razón, aunque las seis pruebas fueron superadas, todavía se pueden realizar más verificaciones para mejorar la validación de los tipos recibidos.

---

**Firma digital:** [Usuario de GitHub]

