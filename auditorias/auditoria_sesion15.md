# Bitácora de Requerimientos y Auditoría IA — Sesión 15

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Bloques Anidados y Ciclo de Vida de Identificadores  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Operaciones Push y Pop de ámbito.

Al realizar la implementación manual encontré dificultades para manejar correctamente la memoria dinámica y los enlaces entre ámbitos. Tuve que asegurarme de que cada nuevo ámbito se enlazara con su ámbito padre mediante un puntero. También fue necesario controlar que al cerrar un bloque se liberara únicamente la memoria del ámbito actual, sin eliminar el ámbito padre.

Otra dificultad fue evitar errores cuando se intentaba crear un ámbito con un puntero NULL o cerrar el ámbito raíz.

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar push_ambito() y pop_ambito() para crear y liberar ámbitos anidados, conservando los enlaces con el padre. |
| **Output (O)** | Código modular en C y análisis de complejidad. |

### 2.2 Prompt Estructurado

```text

Necesito que me ayudes a desarrollar las funciones `push_ambito()` y `pop_ambito()` para mi práctica de Lenguajes y Autómatas II, correspondiente
al tema de Bloques Anidados y Ciclo de Vida de Identificadores.


Estoy trabajando en lenguaje C99 utilizando GCC y las bibliotecas estándar. La estructura `Ambito` ya está definida en el archivo `api_sesion15.h`,
por lo que necesito que respetes su estructura y las funciones establecidas.


Quiero que actúes como un ingeniero de software especializado en programación de sistemas y desarrollo de compiladores,
que pueda ayudarme a implementar correctamente el manejo de ámbitos y memoria dinámica.

**Especificaciones:**

Necesito que desarrolles las siguientes funciones:

1. `push_ambito()` debe crear un nuevo ámbito utilizando memoria dinámica.
2. El nuevo ámbito debe guardar el nombre recibido.
3. Debe enlazar correctamente el nuevo ámbito con su ámbito padre.
4. Si el ámbito actual o el nombre son NULL, debe devolver NULL.
5. Si ocurre un error al reservar memoria con `malloc()`, debe devolver NULL.
6. `pop_ambito()` debe devolver el ámbito padre del ámbito actual.
7. Debe liberar únicamente la memoria del ámbito que se está cerrando.
8. Si el ámbito actual es NULL o corresponde a la raíz, debe devolver NULL sin liberar memoria.
9. El código no debe incluir `main()`, porque el programa de pruebas ya cuenta con uno.
10. Debes respetar las firmas originales declaradas en `api_sesion15.h`.

```

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion15.h"
#include <stdlib.h>

Ambito *push_ambito(Ambito *actual, const char *nombre) {
    if (actual == NULL || nombre == NULL) {
        return NULL;
    }

    Ambito *nuevo = (Ambito *)malloc(sizeof(Ambito));

    if (nuevo == NULL) {
        return NULL;
    }

    nuevo->nombre = nombre;
    nuevo->padre = actual;

    return nuevo;
}

Ambito *pop_ambito(Ambito *actual) {
    if (actual == NULL || actual->padre == NULL) {
        return NULL;
    }

    Ambito *padre = actual->padre;

    free(actual);

    return padre;
}
```

**Análisis de complejidad:**

- push_ambito(): O(1), porque realiza una reserva de memoria y asignaciones de punteros.
- pop_ambito(): O(1), porque obtiene el padre y libera únicamente el ámbito actual.
- Espacio adicional: O(1) por operación.

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Aislamiento de variables fuera de bloque y operaciones Push y Pop de ámbitos.

- [x] Caso base y límites superados sin Crash.
- [x] No se detectaron accesos fuera de límites.
- [x] La complejidad teórica coincide con la implementada.


**Resultado obtenido: 6/6 pruebas superadas.**

### Salida del Test Harness

```text


[ 1] nombre del ambito nuevo       esperado: if obtenido: if OK
[ 2] profundidad con dos push      esperado: 3 obtenido: 3 OK
[ 3] pop devuelve el padre         esperado: 1 obtenido: 1 OK
[ 4] pop sobre la raiz             esperado: 1 obtenido: 1 OK
[ 5] push con padre NULL           esperado: 1 obtenido: 1 OK
[ 6] profundidad tras pop          esperado: 2 obtenido: 2 OK

RESULTADO: 6/6 pruebas superadas

```

### Reporte Crítico

El principal caso borde que debía revisarse era intentar ejecutar pop_ambito() sobre el ámbito raíz. Si se liberara la raíz, se podría provocar un error porque esta memoria puede pertenecer al programa y no haber sido reservada mediante malloc().

La solución fue comprobar si el ámbito actual es NULL o si su puntero padre es NULL antes de liberar memoria. De esta manera, la función devuelve NULL cuando se intenta eliminar la raíz.

También se verificó que push_ambito() rechazara un ámbito padre NULL y que comprobara si malloc() había fallado antes de utilizar la memoria reservada.

Al ejecutar el Test Harness, tanto la implementación manual como la generada por IA obtuvieron 6 de 6 pruebas superadas.

---

**Firma digital:** [Usuario de GitHub]
