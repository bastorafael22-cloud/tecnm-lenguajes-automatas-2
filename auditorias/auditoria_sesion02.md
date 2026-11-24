# Bitácora de Requerimientos y Auditoría IA — Sesión 02

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Gestión en GitHub Web y Nodos Dinámicos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348 
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Lista enlazada de tokens en Heap.

Al realizar la implementación manual, identifiqué que el principal reto era reservar memoria para cada nodo utilizando `malloc()` y enlazar correctamente los nodos mediante el campo `siguiente`.

También fue necesario considerar que cada nodo debía almacenar el lexema en un arreglo de 16 caracteres, incluyendo el espacio para el carácter nulo `'\0'`. Por esta razón, se debe evitar copiar más caracteres de los que caben en el arreglo.

Otro problema importante fue liberar la memoria correctamente. Antes de utilizar `free()`, es necesario guardar la dirección del siguiente nodo para poder continuar recorriendo la lista sin acceder a memoria que ya fue liberada.

Para resolverlo, implementé las funciones `crear_nodo()`, `agregar_token()`, `contar_tokens()` y `liberar_lista()`. También agregué validaciones para los punteros nulos y para comprobar si la reserva de memoria fue exitosa.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99/C11 con GCC, utilizando la biblioteca estándar y la interfaz declarada en `api_sesion02.h`. |
| **Rol (R)** | Ingeniero de software especializado en programación de sistemas, estructuras de datos y compiladores. |
| **Especificación (E)** | Implementar una lista enlazada dinámica de tokens en el heap. Cada nodo debe reservarse con `malloc()`, almacenar su lexema y contener un puntero al siguiente nodo. Se deben implementar las funciones para crear, agregar, contar y liberar nodos. |
| **Output (O)** | Código modular en C, sin función `main()`, compatible con el harness y acompañado de un análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en la materia Lenguajes y Autómatas II.
Necesito implementar una lista enlazada dinámica de tokens en C,
utilizando memoria dinámica en el heap.

Rol:
Actúa como un ingeniero de software especializado en programación
de sistemas, estructuras de datos y desarrollo de compiladores.

Especificación:
Utiliza la interfaz declarada en api_sesion02.h, que contiene
la estructura NodoToken con un arreglo char lexema[16] y un
puntero struct NodoToken *siguiente.

Implementa exactamente estas funciones:

NodoToken *crear_nodo(const char *lexema);
void agregar_token(NodoToken **cabeza, const char *lexema);
int contar_tokens(NodoToken *cabeza);
void liberar_lista(NodoToken *cabeza);

Requisitos:
1. Reserva cada nodo mediante malloc().
2. Inicializa correctamente el lexema y el puntero siguiente.
3. Evita desbordamientos al copiar el lexema.
4. El último nodo debe tener siguiente == NULL.
5. Si crear_nodo recibe NULL, debe manejarlo sin provocar un fallo.
6. Si la reserva de memoria falla, no debe utilizar el puntero nulo.
7. agregar_token debe insertar el nuevo nodo al final de la lista.
8. contar_tokens(NULL) debe devolver 0.
9. liberar_lista(NULL) debe terminar sin errores.
10. Guarda el siguiente nodo antes de liberar el actual para
    evitar accesos a memoria liberada (Use-After-Free).
11. No utilices arreglos globales ni incluyas main().
12. Mantén las firmas originales y compila con:
    gcc -std=c11 -Wall -Wextra -Werror

Output:
Devuelve el código completo de las cuatro funciones y explica
su complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

Archivo: `ia_sesion02_c.c`

```c
#include "api_sesion02.h"
#include <stdlib.h>
#include <string.h>

NodoToken *crear_nodo(const char *lexema) {
    if (lexema == NULL) {
        return NULL;
    }

    NodoToken *nuevo = malloc(sizeof *nuevo);

    if (nuevo == NULL) {
        return NULL;
    }

    strncpy(nuevo->lexema, lexema, sizeof nuevo->lexema - 1);
    nuevo->lexema[sizeof nuevo->lexema - 1] = '\0';
    nuevo->siguiente = NULL;

    return nuevo;
}

void agregar_token(NodoToken **cabeza, const char *lexema) {
    if (cabeza == NULL || lexema == NULL) {
        return;
    }

    NodoToken *nuevo = crear_nodo(lexema);

    if (nuevo == NULL) {
        return;
    }

    if (*cabeza == NULL) {
        *cabeza = nuevo;
        return;
    }

    NodoToken *actual = *cabeza;

    while (actual->siguiente != NULL) {
        actual = actual->siguiente;
    }

    actual->siguiente = nuevo;
}

int contar_tokens(NodoToken *cabeza) {
    int contador = 0;
    NodoToken *actual = cabeza;

    while (actual != NULL) {
        contador++;
        actual = actual->siguiente;
    }

    return contador;
}

void liberar_lista(NodoToken *cabeza) {
    NodoToken *actual = cabeza;

    while (actual != NULL) {
        NodoToken *siguiente = actual->siguiente;

        free(actual);
        actual = siguiente;
    }
}
```

### Análisis de complejidad

- **`crear_nodo()`:** O(1), porque el espacio del lexema está limitado a 16 caracteres.
- **`agregar_token()`:** O(n), porque puede recorrer todos los nodos para encontrar el último.
- **`contar_tokens()`:** O(n), porque recorre cada nodo de la lista.
- **`liberar_lista()`:** O(n), porque libera los nodos uno por uno.
- **Complejidad espacial auxiliar:** O(1), porque las funciones utilizan una cantidad constante de variables auxiliares.
- **Memoria total de la lista:** O(n), porque cada token requiere un nodo reservado dinámicamente.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** creación de nodos, cantidad de tokens, orden de la lista, finalización del enlace, lista nula y liberación segura de una lista nula.

### Resultados de las pruebas

Tanto la implementación manual como la implementación de IA obtuvieron seis pruebas superadas de seis en las ejecuciones registradas en Google Colab.

```text
=== AUDITORIA SESION 02 - Lista enlazada de tokens en heap ===
Caso                               esperado:              obtenido:              resultado
[ 1] crear_nodo("if")->lexema           esperado: if           obtenido: if           OK
[ 2] contar_tokens(3 tokens)            esperado: 3            obtenido: 3            OK
[ 3] orden: segundo token               esperado: dos          obtenido: dos          OK
[ 4] ultimo->siguiente == NULL          esperado: 1            obtenido: 1            OK
[ 5] contar_tokens(NULL)                esperado: 0            obtenido: 0            OK
[ 6] liberar_lista(NULL)                esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion02.md
```

### Lista de verificación

- [x] El caso base y los límites probados se superaron sin errores.
- [x] Cada nodo se reserva dinámicamente utilizando `malloc()`.
- [x] El último nodo termina con `siguiente == NULL`.
- [x] `contar_tokens(NULL)` devuelve 0.
- [x] `liberar_lista(NULL)` no provoca errores.
- [x] La implementación guarda el siguiente nodo antes de liberar el actual.
- [x] La complejidad teórica coincide con las operaciones implementadas.

### Reporte Crítico

Las seis pruebas del harness pasaron correctamente; sin embargo, estas pruebas no verifican todos los casos posibles. Un caso límite adicional que debe revisarse es cuando se intenta crear un nodo con un lexema `NULL`, ya que no aparece entre los seis casos originales. Para evitar problemas, agregué una validación que devuelve `NULL` cuando el lexema no es válido y otra que comprueba si `malloc()` pudo reservar memoria.

También revisé la liberación de la lista para prevenir errores de tipo Use-After-Free. La solución consiste en guardar primero la dirección del siguiente nodo y después liberar el nodo actual, evitando utilizar su contenido una vez liberado.

La prueba del harness demuestra que los seis casos establecidos funcionan correctamente, aunque por sí sola no garantiza la ausencia de fugas de memoria en todas las situaciones. Por ello, también es importante revisar la lógica de liberación y, si se requiere una comprobación adicional, utilizar herramientas de análisis de memoria.

Esta actividad me permitió comprender cómo se almacenan estructuras de datos dinámicas en el heap, cómo se conectan mediante punteros y por qué es importante administrar correctamente la memoria en programas relacionados con compiladores.

---

**Firma digital:** [Usuario de GitHub]
