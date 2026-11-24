# Bitácora de Requerimientos y Auditoría IA — Sesión 06

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Sistema de Tipos Primitivos y Verificación Semántica  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Chequeo de compatibilidad de tipos.

Al desarrollar la función de compatibilidad de tipos, la principal dificultad fue comprender cómo establecer una jerarquía entre los tipos primitivos y determinar cuáles asignaciones podían realizarse de manera segura.

Se identificó que los tipos tienen diferentes rangos:

- `CHAR`: tipo de menor rango.
- `INT`: tipo de rango intermedio.
- `FLOAT`: tipo de mayor rango.

La regla principal consiste en permitir asignaciones hacia un tipo del mismo rango o de mayor rango. Por ejemplo, asignar un `INT` a un `FLOAT` es compatible, pero asignar un `FLOAT` a un `INT` debe rechazarse porque puede existir pérdida de información.

También fue necesario considerar el tipo `TIPO_ERROR`, que nunca debe ser compatible con ningún otro tipo.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Matriz de compatibilidad estática |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado

```text
Actúa como ingeniero de software especializado en C y compiladores.

Implementa las siguientes funciones:

int compatible(Tipo destino, Tipo origen);
const char *nombre_tipo(Tipo t);

Utiliza la estructura Tipo declarada en api_sesion06.h.

La jerarquía de tipos es:
CHAR < INT < FLOAT.

Reglas:
1. Permitir asignaciones cuando el destino tenga
   el mismo rango o un rango superior al origen.
2. Rechazar asignaciones que conviertan un tipo
   de mayor rango a uno de menor rango.
3. TIPO_ERROR nunca debe ser compatible.
4. nombre_tipo debe devolver "char", "int",
   "float" o "error", según corresponda.
5. Validar también los valores de enumeración
   que no pertenezcan a los tipos definidos.

No modificar las firmas de las funciones.
No agregar main().
No utilizar bibliotecas no estándar.

El código debe compilar con:

gcc -std=c11 -Wall -Wextra -Werror

Entrega el código modular y explica su
complejidad temporal, espacial y casos límite.
```

---

## 3. Código Devuelto por el Modelo


```c
#include "api_sesion06.h"

/* Comprobar que el tipo sea valido */
static int tipo_valido(Tipo t) {
    return t >= TIPO_CHAR && t <= TIPO_FLOAT;
}

/* Verificar compatibilidad entre tipos */
int compatible(Tipo destino, Tipo origen) {

    /* TIPO_ERROR nunca es compatible */
    if (destino == TIPO_ERROR || origen == TIPO_ERROR) {
        return 0;
    }

    /* Rechazar valores de tipo no definidos */
    if (!tipo_valido(destino) || !tipo_valido(origen)) {
        return 0;
    }

    /* Jerarquia: CHAR < INT < FLOAT */
    return (int)destino >= (int)origen;
}

/* Obtener el nombre del tipo */
const char *nombre_tipo(Tipo t) {

    switch (t) {
        case TIPO_CHAR:
            return "char";

        case TIPO_INT:
            return "int";

        case TIPO_FLOAT:
            return "float";

        case TIPO_ERROR:
        default:
            return "error";
    }
}
```

### Complejidad del algoritmo

- **Temporal:** O(1), porque se realizan comparaciones y operaciones constantes.
- **Espacial:** O(1), porque no se utilizan arreglos dinámicos ni estructuras adicionales.
- No necesita `malloc()` ni `free()`.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Rechazo de asignaciones incompatibles y comprobación de nombres de tipos.

- [x] Los casos del harness se ejecutaron correctamente, sin errores.
- [x] Se revisaron las condiciones de compatibilidad y los valores de tipos inválidos.
- [x] La complejidad teórica coincide con la implementación.

### Resultados de la auditoría

```text
=== AUDITORIA SESION 06 - Tabla de compatibilidad de tipos ===

[ 1] compatible(INT,INT)                esperado: 1   obtenido: 1   OK
[ 2] compatible(FLOAT,INT)               esperado: 1   obtenido: 1   OK
[ 3] compatible(INT,FLOAT)               esperado: 0   obtenido: 0   OK
[ 4] compatible(INT,ERROR)               esperado: 0   obtenido: 0   OK
[ 5] nombre_tipo(INT)                    esperado: int obtenido: int OK
[ 6] nombre_tipo(ERROR)                  esperado: error obtenido: error OK

RESULTADO: 6/6 pruebas superadas

TODO EN VERDE.
```

### Reporte Crítico

Al revisar el código generado por la IA se identificó un caso límite que no estaba contemplado directamente en los seis ejemplos del harness: recibir un valor de enumeración que no corresponda a ninguno de los tipos definidos.

La implementación original comparaba directamente los valores de los tipos. Esto podía provocar que un valor no definido fuera considerado compatible dependiendo de su valor numérico.

Para solucionar este problema se agregó la función auxiliar `tipo_valido()`, que verifica que los valores pertenezcan al rango de tipos permitidos: CHAR, INT y FLOAT. Si se recibe un valor inválido, la función devuelve 0 y rechaza la asignación.

También se comprobó que `TIPO_ERROR` siempre sea rechazado y que la función `nombre_tipo()` devuelva correctamente el texto correspondiente a cada tipo.

Finalmente, el programa fue validado con el test harness proporcionado en la práctica, obteniendo **6 de 6 pruebas superadas**.

El resultado confirma que se cumplen los seis casos solicitados. La validación de enumeraciones fuera de rango se añadió mediante revisión del código y constituye una comprobación adicional que no forma parte de los seis casos originales.

---

**Firma digital:** [Usuario de GitHub]
