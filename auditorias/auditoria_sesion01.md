# Bitácora de Requerimientos y Auditoría IA — Sesión 01

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** [C]

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Inversión de buffer in-place (`char *`).

Al realizar la función manualmente, identifiqué que debía recorrer la cadena mediante índices o punteros sin utilizar memoria adicional. La dificultad fue determinar cómo encontrar el último carácter sin contar el carácter nulo `'\0'` y cómo intercambiar los caracteres del inicio y del final.

En mi primer intento, la función no realizaba los intercambios necesarios. Por eso, las pruebas de las cadenas `"HOLA"`, `"abc"` y `"AB"` fallaron, mientras que la cadena vacía, la cadena de un carácter y el caso `NULL` pasaron.

Para resolverlo, utilicé dos índices: uno al inicio y otro al final de la cadena. Intercambié los caracteres mientras el índice inicial fuera menor que el final. También agregué una validación para que la función terminara si recibía un puntero `NULL`.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99/C11 con GCC, utilizando la biblioteca estándar y respetando la interfaz declarada en `api_sesion01.h`. |
| **Rol (R)** | Ingeniero de software especializado en programación de sistemas y manejo de memoria. |
| **Especificación (E)** | Implementar `void invertir(char *s);`. Invertir la cadena en el mismo buffer, sin `malloc` ni arreglos auxiliares. Tiempo O(n) y memoria adicional O(1). La función debe aceptar `NULL`, cadenas vacías y cadenas de cualquier longitud. |
| **Output (O)** | Código modular en C, compatible con el contrato proporcionado, sin función `main()` y con análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en Lenguajes y Autómatas II. Debo implementar una función
en C que invierta una cadena mutable en el mismo espacio de memoria.

Rol:
Actúa como un ingeniero de software especializado en programación de
sistemas, punteros y manejo seguro de memoria.

Especificación:
Implementa exactamente la siguiente función:

void invertir(char *s);

Requisitos:
1. Si s es NULL, la función debe terminar sin provocar un error.
2. La cadena debe invertirse en el mismo buffer.
3. No utilices malloc, memoria dinámica ni arreglos auxiliares.
4. Utiliza dos índices o punteros y una variable temporal de tipo char.
5. La cadena vacía y las cadenas de un solo carácter deben funcionar.
6. No incluyas main(), porque el programa de pruebas ya lo contiene.
7. El código debe compilar con GCC y las opciones
   -Wall -Wextra -Werror.
8. Mantén la firma declarada en api_sesion01.h.
9. Explica la complejidad temporal y espacial.

Output:
Devuelve el código completo de la implementación y un análisis breve
de complejidad. No cambies la interfaz original.
```

---

## 3. Código Devuelto por el Modelo

Archivo: `ia_sesion01_c.c`

```c
#include "api_sesion01.h"
#include <stddef.h>
#include <string.h>

void invertir(char *s)
{
    size_t inicio;
    size_t fin;
    char temporal;

    if (s == NULL) {
        return;
    }

    fin = strlen(s);

    if (fin == 0) {
        return;
    }

    fin--;

    inicio = 0;

    while (inicio < fin) {
        temporal = s[inicio];
        s[inicio] = s[fin];
        s[fin] = temporal;

        inicio++;
        fin--;
    }
}
```

### Análisis de complejidad

- **Complejidad temporal:** O(n), porque se recorre la cadena para calcular su longitud y después se intercambian los caracteres desde ambos extremos.
- **Complejidad espacial:** O(1), porque únicamente se utilizan dos índices y una variable temporal.
- **Memoria dinámica:** No se utiliza `malloc` ni se crea una copia de la cadena.
- **Modificación:** La cadena original queda invertida en la misma dirección de memoria.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** cadenas normales, cadenas vacías, longitudes pares e impares, una cadena de un carácter y un puntero `NULL`.

### Resultados del primer intento

En la primera ejecución de la implementación manual, se obtuvieron 3 de 6 pruebas superadas. Fallaron los casos `"HOLA"`, `"abc"` y `"AB"` debido a que la función todavía no realizaba la inversión.

En la primera ejecución del archivo de IA, el compilador indicó un error de referencia indefinida a `invertir`, debido a que el archivo contenía únicamente el comentario `TODO` y todavía no incluía la implementación.

### Resultados después de corregir la función

```text
=== AUDITORIA SESION 01 - Inversion de buffer in-place ===
Caso                               esperado:              obtenido:              resultado
[ 1] invertir("HOLA")                   esperado: ALOH         obtenido: ALOH         OK
[ 2] invertir("abc")                    esperado: cba          obtenido: cba          OK
[ 3] invertir("")                       esperado:              obtenido:              OK
[ 4] invertir("AB")                     esperado: BA           obtenido: BA           OK
[ 5] invertir("a")                      esperado: a            obtenido: a            OK
[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md
```

### Lista de verificación

- [x] Caso base y límites superados sin errores.
- [x] No se utilizan arreglos auxiliares ni memoria dinámica.
- [x] La función acepta `NULL` y cadenas vacías.
- [x] La complejidad teórica coincide con la implementación.
- [x] Las seis pruebas del harness pasan después de la corrección.

### Reporte Crítico

El primer problema detectado fue que la implementación inicial no intercambiaba los caracteres de la cadena, por lo que tres pruebas fallaron. Posteriormente, la implementación de IA tampoco podía compilar porque todavía no contenía una definición de la función `invertir()`.

Para solucionarlo, se implementó la función utilizando dos índices y una variable temporal. También se agregó una validación para el puntero `NULL` y se comprobó que la cadena vacía no provocara accesos inválidos. Al ejecutar nuevamente el harness, los seis casos pasaron correctamente.

Esta auditoría demuestra que no basta con generar código mediante inteligencia artificial: es necesario compilarlo, probar los casos límite y verificar que cumpla con los requisitos establecidos.

---

**Firma digital:** [Usuario de GitHub]
