# Bitácora de Requerimientos y Auditoría IA — Sesión 13

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Manejo de Ámbitos (Scope Global/Local)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Pila de tablas de símbolos (Stack Scopes).

Al realizar el ejercicio manualmente, tuve dificultad para comprender cómo realizar la búsqueda de variables en diferentes ámbitos y cómo recorrer los ámbitos padre hasta llegar al global. También identifiqué que se debe permitir declarar el mismo nombre en diferentes bloques, pero evitar que se repita dentro del mismo ámbito. Otra dificultad fue controlar la cantidad máxima de nombres permitidos y evitar acceder a posiciones fuera del arreglo.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar declaraciones locales y búsqueda jerárquica de identificadores, permitiendo ocultamiento de variables (shadowing). |
| **Output (O)** | Código modular en C y análisis de complejidad. |

### 2.2 Prompt Estructurado

Actúa como un ingeniero de software especializado en C y desarrollo de compiladores. Implementa un sistema de manejo de ámbitos globales y locales mediante estructuras enlazadas por un puntero al ámbito padre.

Utiliza C99 compatible con GCC y bibliotecas estándar. Respeta las siguientes funciones:

- `int declarar(Ambito *ambito, const char *nombre);`
- `const char *buscar(const Ambito *ambito, const char *nombre);`

La función declarar() debe validar que el ámbito y el nombre sean válidos, evitar nombres duplicados dentro del mismo ámbito y respetar el límite AMBITO_MAX. Debe devolver 1 si declara correctamente, 0 si el nombre ya existe y -1 si hay errores o el ámbito está lleno.

La función buscar() debe recorrer el ámbito actual y después sus padres, hasta encontrar el identificador. Debe devolver el nombre del ámbito donde se encontró o NULL si no existe.

Permite que el mismo identificador sea declarado en diferentes ámbitos para representar shadowing. No agregues main(), no cambies las firmas y entrega código modular con comentarios y análisis de complejidad.

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion13.h"
#include <string.h>

int declarar(Ambito *ambito, const char *nombre) {
    if (ambito == NULL || nombre == NULL || nombre[0] == '\0') {
        return -1;
    }

    if (ambito->total >= AMBITO_MAX) {
        return -1;
    }

    for (int i = 0; i < ambito->total; i++) {
        if (strcmp(ambito->nombres[i], nombre) == 0) {
            return 0;
        }
    }

    ambito->nombres[ambito->total] = nombre;
    ambito->total++;

    return 1;
}

const char *buscar(const Ambito *ambito, const char *nombre) {
    if (ambito == NULL || nombre == NULL) {
        return NULL;
    }

    const Ambito *actual = ambito;

    while (actual != NULL) {
        for (int i = 0; i < actual->total; i++) {
            if (strcmp(actual->nombres[i], nombre) == 0) {
                return actual->nombre;
            }
        }

        actual = actual->padre;
    }

    return NULL;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Búsqueda jerárquica de identificadores.

- ☒ Caso base y límites evaluados sin Crash.
- ☒ No se detectaron accesos fuera de límites ni Memory Leaks en las pruebas realizadas.
- ☒ La complejidad teórica coincide con la implementada.

### Resultados de las pruebas

| Caso | Resultado esperado | Resultado obtenido |
|---|---|---|
| `declarar(&g, "x")` | 1 | 1 — OK |
| Buscar en el mismo ámbito | global | global — OK |
| Buscar subiendo al padre | global | global — OK |
| Shadowing en el hijo | bloque | bloque — OK |
| Tres niveles hasta la raíz | raiz | raiz — OK |
| Nombre no declarado | NULL | NULL — OK |

**Resultado final:** 6/6 pruebas superadas.

### Reporte Crítico

La IA omitió validar algunos casos límite, como comprobar si el contador `total` del ámbito tiene un valor negativo o superior al máximo permitido, y si los nombres almacenados son válidos. Esto podría ocasionar problemas al recorrer el arreglo de nombres. Lo detecté revisando las condiciones de entrada de las funciones y observando que el harness prueba principalmente búsquedas normales, shadowing y nombres inexistentes. Como mejora, agregaría validaciones del contador y pruebas con ámbitos llenos, nombres vacíos y entradas inválidas, para evitar accesos incorrectos a memoria y mejorar la seguridad del programa.

---

**Firma digital:** [Usuario de GitHub]
