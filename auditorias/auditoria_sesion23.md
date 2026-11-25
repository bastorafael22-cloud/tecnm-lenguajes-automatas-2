# Bitácora de Requerimientos y Auditoría IA — Sesión 23

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Etiquetas y Saltos Incondicionales  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Generador de etiquetas e instrucciones `goto`.

Al realizar el código manualmente, identifiqué que la principal dificultad era generar etiquetas consecutivas sin repetirlas, utilizando un contador que debía incrementarse después de cada etiqueta creada. También tuve que cuidar que el formato de salida fuera correcto, por ejemplo, `L0`, `L1`, `L2`, sin confundir una etiqueta con una instrucción de salto.

Otro problema fue escribir las cadenas dentro del buffer de destino sin exceder su capacidad, ya que esto podría provocar errores de memoria. Además, tuve que diferenciar entre un salto incondicional, que debe escribir `GOTO L2`, y una etiqueta de destino, que debe escribirse como `L2:`.

Para resolverlo, utilicé `snprintf()` para generar las cadenas de forma segura, comprobé que los parámetros fueran válidos y agregué el incremento del contador para que las etiquetas fueran consecutivas. También consideré el caso de recibir una etiqueta `NULL`, para que la función generara una cadena vacía sin provocar errores.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | Lenguaje C11 con GCC, utilizando bibliotecas estándar y respetando la interfaz declarada en `api_sesion23.h`. |
| **Rol (R)** | Ingeniero de software especializado en programación de sistemas, compiladores y generación de código intermedio. |
| **Especificación (E)** | Implementar `nueva_etiqueta()` y `emitir_flujo()`. Generar etiquetas consecutivas con formato `L0`, `L1`, `L2`, escribir instrucciones `GOTO` y marcas de destino, controlar la capacidad del buffer y validar los parámetros recibidos. |
| **Output (O)** | Código modular en C, compatible con el contrato proporcionado, sin función `main()`, más un análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en Lenguajes y Autómatas II. Debo implementar funciones
en C que permitan generar etiquetas únicas y escribir instrucciones
de salto incondicional para un flujo de código.

Rol:
Actúa como un ingeniero de software especializado en programación
de sistemas, compiladores y manejo seguro de cadenas.

Especificación:
Implementa exactamente las siguientes funciones:

void nueva_etiqueta(char *dst, int cap, int *contador);
void emitir_flujo(char *dst, int cap, const char *etiqueta, int es_salto);

Requisitos:
1. La función nueva_etiqueta() debe generar etiquetas con formato L0,
   L1, L2, etc., utilizando el contador recibido.
2. El contador debe incrementarse después de generar cada etiqueta.
3. La función emitir_flujo() debe escribir "GOTO Lx" cuando
   es_salto sea distinto de cero.
4. Si es_salto es cero, debe escribir la marca de destino "Lx:".
5. Si la etiqueta es NULL, debe escribir una cadena vacía.
6. Comprueba que dst no sea NULL y que cap sea mayor que cero.
7. No escribas fuera de la capacidad del buffer.
8. Utiliza snprintf() para generar las cadenas de manera segura.
9. Respeta las firmas declaradas en api_sesion23.h.
10. No incluyas main(), porque el programa de pruebas ya lo contiene.
11. El código debe compilar con GCC utilizando las opciones
    -Wall -Wextra -Werror.

Output:
Devuelve el código completo de la implementación y un análisis
breve de su complejidad temporal y espacial. No cambies
la interfaz original.
```

---

## 3. Código Devuelto por el Modelo

Archivo: `ia_sesion23_c.c`

```c
#include "api_sesion23.h"
#include <stdio.h>
#include <string.h>

void nueva_etiqueta(char *dst, int cap, int *contador) {
    if (dst != NULL && cap > 0 && contador != NULL) {
        snprintf(dst, cap, "L%d", *contador);
        (*contador)++;
    }
}

void emitir_flujo(char *dst, int cap, const char *etiqueta, int es_salto) {
    if (dst != NULL && cap > 0) {
        if (etiqueta == NULL) {
            dst[0] = '\0';
        } else if (es_salto) {
            snprintf(dst, cap, "GOTO %s", etiqueta);
        } else {
            snprintf(dst, cap, "%s:", etiqueta);
        }
    }
}
```

### Análisis de complejidad

- **Complejidad temporal:** O(1), porque ambas funciones realizan operaciones de formato de cadena y no recorren estructuras de datos.
- **Complejidad espacial:** O(1), porque solamente utilizan el buffer proporcionado y no reservan memoria dinámica adicional.
- **Memoria dinámica:** No se utiliza `malloc()` ni se crean arreglos auxiliares.
- **Modificación:** Las funciones escriben directamente en el buffer de destino que recibe cada una.
- **Seguridad:** Se utiliza `snprintf()` para limitar la cantidad de caracteres escritos según la capacidad del buffer.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** generación de etiquetas con diferentes contadores, emisión de saltos `GOTO`, marcas de destino y recepción de una etiqueta `NULL`.

### Resultados de la ejecución

```text
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
```

### Lista de verificación

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

El principal aspecto que tuve que revisar en el código generado por la IA fue el manejo de la capacidad del buffer y los parámetros recibidos por las funciones. Aunque `snprintf()` permite evitar escribir fuera del espacio disponible, si el buffer es demasiado pequeño la etiqueta o la instrucción podría quedar incompleta.

Para evitar errores, la implementación incluye comprobaciones de que el buffer exista, que su capacidad sea mayor que cero y que el contador sea válido antes de generar una nueva etiqueta. También se contempla el caso en el que la etiqueta recibida sea `NULL`, devolviendo una cadena vacía en lugar de intentar utilizarla.

Al ejecutar el harness, comprobé que las etiquetas se generaban correctamente, que las instrucciones `GOTO` y las marcas de destino tenían el formato esperado y que los seis casos de prueba fueron superados. Esta práctica me ayudó a comprender la importancia de utilizar etiquetas para controlar el flujo de un programa y de validar los buffers al generar instrucciones mediante código.

---

**Firma digital:** [Usuario de GitHub]  
**Fecha:** 25 de noviembre de 2026
