# Bitácora de Requerimientos y Auditoría IA — Sesión 21

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Fundamentos de Código de 3 Direcciones (TAC)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Representación en Cuádruplos `(op, a1, a2, res)` y generación de temporales únicos.

**Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:**

> Al realizar la implementación manual identifiqué que era necesario controlar correctamente el contador para generar temporales únicos, comenzando desde `_t0` y aumentando después de cada llamada. También fue importante utilizar `snprintf()` para escribir los resultados en el buffer sin exceder su capacidad. En los cuádruplos tuve que contemplar los argumentos NULL, sustituyéndolos por un guion bajo (`_`) para conservar el formato solicitado.

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C11 / GCC sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Generar temporales únicos mediante un contador y construir cuádruplos con el formato `(op, a1, a2, res)`. Sustituir argumentos NULL por `_` y respetar la capacidad del buffer. |
| **Output (O)** | Código modular en C, sin `main()`, compatible con `api_sesion21.h` y compilación con `-Wall -Wextra -Werror`. |

### 2.2 Prompt Estructurado

```text

**Rol:** Actúa como un programador experto en sistemas y desarrollo de compiladores utilizando el lenguaje C (estándar C11).

**Especificación:** 
Implementa dos funciones en C siguiendo estrictamente las firmas definidas en el archivo de cabecera `api_sesion21.h`:
1. `void nuevo_temporal(char *dst, int cap, int *contador);` 
   - Debe escribir en el búfer `dst` (respetando el límite de capacidad `cap`) un identificador con el formato `_tK`, donde `K` es el valor entero apuntado por `contador`.
   - Debe incrementar en 1 el valor del contador apuntado por `contador` inmediatamente después de usarlo.
   - Debe validar que los punteros no sean nulos y `cap` sea mayor a 0.

2. `void emitir_cuadruplo(char *dst, int cap, const char *op, const char *a1, const char *a2, const char *res);`
   - Debe dar formato a un cuádruplo en el búfer `dst` con el formato exacto `(op, a1, a2, res)` respetando la capacidad `cap`.
   - Si cualquiera de los parámetros (`op`, `a1`, `a2`, `res`) es nulo (`NULL`), se debe formatear y representar en la cadena resultante con un guion bajo `_`.
   - Asegurar que no ocurran desbordamientos de búfer mediante el uso seguro de `snprintf`.

**Output:** Devuelve únicamente el código fuente de las dos funciones en C sin incluir la función `main`, listo para ser incluido y compilado junto con el archivo de cabecera correspondiente.
```

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion21.h"
#include <stdio.h>
#include <string.h>

void nuevo_temporal(char *dst, int cap, int *contador) {
    if (dst != NULL && cap > 0 && contador != NULL) {
        snprintf(dst, cap, "_t%d", *contador);
        (*contador)++;
    }
}

void emitir_cuadruplo(char *dst, int cap, const char *op,
                      const char *a1, const char *a2,
                      const char *res) {
    if (dst != NULL && cap > 0) {
        const char *final_op = op ? op : "_";
        const char *final_a1 = a1 ? a1 : "_";
        const char *final_a2 = a2 ? a2 : "_";
        const char *final_res = res ? res : "_";

        snprintf(dst, cap, "(%s, %s, %s, %s)",
                 final_op, final_a1, final_a2, final_res);
    }
}
```

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Generación de temporales y representación de cuádruplos.

### Salida del harness

```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nuevo_temporal(_t0)                esperado: _t0          obtenido: _t0          OK
[ 2] nuevo_temporal(_t7)                esperado: _t7          obtenido: _t7          OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple                  esperado: (=, a, _, x) obtenido: (=, a, _, x) OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _)
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion21.md
```

**Checklist de auditoría:**

- [x] Caso base y límites superados sin *Crash*.
- [x] No existen accesos fuera de límites ni *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> La implementación de IA resolvió correctamente los seis casos del harness. Sin embargo, no se probaron explícitamente algunas condiciones adicionales, como un buffer de capacidad cero, un contador NULL, un destino NULL o un buffer demasiado pequeño para almacenar el resultado completo. La implementación incluye validaciones para los punteros y utiliza `snprintf()` para limitar la escritura, pero sería conveniente agregar pruebas específicas para comprobar estos casos. No se registraron fallas en los seis casos oficiales.

---
**Firma digital:** [Usuario de GitHub]
