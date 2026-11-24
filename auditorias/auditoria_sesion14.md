# Bitácora de Requerimientos y Auditoría IA — Sesión 14

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Declaración vs Uso de Variables  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Detección de variables no declaradas.

Cuando hice el programa por mi cuenta, se me complicó llevar el control de las variables y saber cuándo podía utilizarlas. También tuve que revisar que no registrara el mismo nombre dos veces y que el programa distinguiera nombres parecidos, como `x` y `xy`. Además, debía asegurarme de no pasarme del espacio disponible en la tabla para evitar errores.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar la declaración previa, evitar redeclaraciones y verificar que una variable exista antes de utilizarla. |
| **Output (O)** | Código modular en C y análisis de complejidad. |

### 2.2 Prompt Estructurado

Actúa como un ingeniero de software especializado en sistemas y compiladores. Implementa las funciones `declarar()` y `usar()` en C99, respetando exactamente las firmas del archivo `api_sesion14.h`.

La función `declarar(Tabla *tabla, const char *nombre)` debe insertar un identificador solamente si no existe previamente en la tabla. Debe devolver 1 si lo inserta, 0 si ya estaba declarado y -1 si recibe una tabla NULL, un nombre NULL o un nombre vacío. También debe respetar el límite `TABLA_MAX`.

La función `usar(const Tabla *tabla, const char *nombre)` debe devolver 1 solamente si el identificador ya está declarado y 0 en cualquier otro caso. Las comparaciones deben ser exactas, por lo que nombres como `x` y `xy` no deben confundirse.

Utiliza C99, bibliotecas estándar, evita incluir `main()` y respeta la estructura y las firmas del contrato. Entrega el código modular y una explicación breve de su complejidad.

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion14.h"
#include <string.h>
#include <stddef.h>

int declarar(Tabla *tabla, const char *nombre) {
    if (!tabla || !nombre || *nombre == '\0') {
        return -1;
    }

    /* Buscar si ya existe para evitar redeclaración */
    for (int i = 0; i < tabla->total; i++) {
        if (strcmp(tabla->nombres[i], nombre) == 0) {
            return 0;
        }
    }

    if (tabla->total >= TABLA_MAX) {
        return 0;
    }

    tabla->nombres[tabla->total] = nombre;
    tabla->total++;
    return 1;
}

int usar(const Tabla *tabla, const char *nombre) {
    if (!tabla || !nombre) {
        return 0;
    }

    for (int i = 0; i < tabla->total; i++) {
        if (strcmp(tabla->nombres[i], nombre) == 0) {
            return 1;
        }
    }

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Uso antes de declarar y redeclaración.

- ☒ Caso base y límites evaluados sin Crash.
- ☒ No se detectaron accesos fuera de límites ni Memory Leaks en los casos ejecutados.
- ☒ La complejidad teórica coincide con la implementada.

### Resultados de las pruebas

| Caso | Resultado esperado | Resultado obtenido |
|---|---|---|
| Usar sin declarar | 0 | 0 — OK |
| Usar después de declarar | 1 | 1 — OK |
| Redeclarar el mismo nombre | 0 | 0 — OK |
| Total tras redeclarar | 1 | 1 — OK |
| Prefijo no es el mismo nombre | 0 | 0 — OK |
| Declarar nombre vacío | -1 | -1 — OK |

**Resultado final: 6/6 pruebas superadas.**

### Reporte Crítico

La IA omitió comprobar el caso en el que la tabla alcanza su capacidad máxima de 16 elementos y también cuando el contador `total` tiene un valor inválido. Lo detecté al revisar las condiciones de inserción y compararlas con los casos del harness, ya que las pruebas verifican el uso de variables, la redeclaración, los prefijos y los nombres vacíos, pero no prueban una tabla completamente llena. Como mejora, agregaría pruebas para llenar las 16 posiciones y verificar que una nueva declaración no altere el total ni provoque accesos fuera de límites.

---

**Firma digital:** [Usuario de GitHub]
