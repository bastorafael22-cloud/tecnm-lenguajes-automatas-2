# Bitácora de Requerimientos y Auditoría IA — Sesión 08

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Inferencia y Promoción de Tipos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C  



## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Reglas de casteo implícito.

**Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:**

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C11 / GCC, utilizando bibliotecas estándar y respetando la compilación modular. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar la promoción de tipos entre `char`, `int` y `float`, identificar conversiones con pérdida de información y obtener el nombre de cada tipo. |
| **Output (O)** | Código modular en C que implemente las funciones solicitadas, sin incluir una función `main`, y con análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado en sistemas y compiladores.

Necesito implementar en lenguaje C las funciones de inferencia y promoción de tipos para un proyecto de Lenguajes y Autómatas II.

Contexto:
El programa trabaja con una jerarquía de tipos de datos:
CHAR < INT < FLOAT.

Requerimientos:
1. Implementar la función:
   Tipo promover(Tipo a, Tipo b);

   Esta función debe devolver el tipo de mayor jerarquía entre
   los dos tipos recibidos.
   Si alguno de los tipos es TIPO_ERROR, debe devolver TIPO_ERROR.

2. Implementar la función:
   int hay_perdida(Tipo destino, Tipo origen);

   Debe devolver 1 cuando el tipo destino sea de menor jerarquía
   que el tipo origen, porque puede existir pérdida de información.
   Debe devolver 0 cuando no exista pérdida.

3. Implementar la función:
   const char *nombre_tipo(Tipo t);

   Debe devolver el nombre correspondiente:
   TIPO_CHAR -> "char"
   TIPO_INT -> "int"
   TIPO_FLOAT -> "float"
   Cualquier otro valor -> "error".

Restricciones:
- Utilizar lenguaje C.
- Mantener el código modular.
- Incluir únicamente las funciones solicitadas.
- No agregar una función main.
- Respetar los nombres y tipos de retorno establecidos.
- Evitar bibliotecas no necesarias.
- El código debe compilar con las opciones -std=c11 -Wall -Wextra -Werror.

Entrega:
Proporciona el código completo y explica brevemente
la complejidad temporal y espacial de cada función.
```

---

## 3. Código Devuelto por el Modelo

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

**Análisis de complejidad:**

- `promover()`: complejidad temporal O(1), porque realiza comparaciones constantes.
- `hay_perdida()`: complejidad temporal O(1), porque solamente compara los tipos recibidos.
- `nombre_tipo()`: complejidad temporal O(1), porque utiliza una estructura `switch` con una cantidad fija de casos.
- **Complejidad espacial:** O(1) en las tres funciones, porque no utilizan memoria dinámica ni estructuras adicionales.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Combinación de tipos `char`, `int` y `float`, incluyendo casos con `TIPO_ERROR`.

**Verificación:**

- [x] Caso base y límites probados sin errores ni interrupciones (*Crash*).
- [x] No se identificaron accesos fuera de límites ni asignaciones de memoria dinámica que pudieran producir *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementación: O(1) en tiempo y O(1) en espacio.

### Reporte Crítico


> La IA no especificó claramente el comportamiento de la función `hay_perdida()` cuando recibe `TIPO_ERROR` como tipo de destino o de origen. En el código generado se agregó una condición que devuelve 1 cuando alguno de los argumentos es `TIPO_ERROR`. Sin embargo, este comportamiento no estaba definido explícitamente en los requerimientos ni fue comprobado por los seis casos del Test Harness. Por ello, se identificó como un caso borde que necesita una regla formal para evitar resultados ambiguos. Las pruebas realizadas confirmaron que las funciones cumplen con los seis casos establecidos.

---

**Firma digital:** [Tu usuario de GitHub]
