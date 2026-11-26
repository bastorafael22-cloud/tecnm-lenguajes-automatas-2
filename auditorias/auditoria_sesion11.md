# Bitácora de Requerimientos y Auditoría IA — Sesión 11

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arquitectura de la Tabla de Símbolos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula: 24531348 
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Tabla Hash con encadenamiento.

Al desarrollar la tabla hash de manera manual, tuve dificultades para comprender cómo calcular el índice de cada nombre mediante la función hash y cómo manejar las colisiones cuando dos nombres llegaban a la misma cubeta. También tuve que entender cómo utilizar las estructuras enlazadas para agregar nuevos nodos sin perder los símbolos que ya estaban almacenados. Otra dificultad fue administrar correctamente la memoria al reservar espacio para cada símbolo y cuidar que las cadenas de caracteres terminaran correctamente.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar la función hash djb2 y la inserción de símbolos con manejo de colisiones mediante listas enlazadas, evitando nombres duplicados. |
| **Output (O)** | Código modular en C, respetando las funciones declaradas en el archivo de cabecera y con análisis de complejidad. |

### 2.2 Prompt Estructurado

Actúa como un ingeniero de software especializado en C y desarrollo de compiladores. Implementa una tabla de símbolos utilizando una tabla hash con la función de dispersión djb2 y manejo de colisiones mediante encadenamiento.

Utiliza C99 compatible con GCC y únicamente bibliotecas estándar. Respeta exactamente las siguientes funciones:

- `unsigned int hash_djb2(const char *clave);`
- `int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas, const char *nombre, int tipo, int offset);`

La función hash debe comenzar con 5381 y multiplicar el valor por 33, sumando cada carácter de la clave.

La función de inserción debe validar los argumentos, calcular la cubeta, revisar si el nombre ya existe, evitar duplicados, crear un nodo dinámicamente, copiar el nombre de forma segura y conservar las colisiones mediante una lista enlazada. Debe devolver 1 si inserta correctamente, 0 si el símbolo ya existe y -1 si los argumentos son inválidos o falla la reserva de memoria.

No agregues una función `main()`, no cambies las firmas y entrega únicamente el código modular necesario, con comentarios y explicación breve de su complejidad.

---

## 3. Código Devuelto por el Modelo

La IA generó una implementación que utiliza la función hash djb2 y listas enlazadas para manejar las colisiones. También valida los argumentos, reserva memoria dinámicamente y evita insertar nombres repetidos.

El código generado fue el siguiente:

```c
#include "api_sesion11.h"
#include <stdlib.h>
#include <string.h>

unsigned int hash_djb2(const char *clave) {
    if (!clave) return 0;
    unsigned long hash = 5381;
    int c;

    while ((c = (unsigned char)*clave++)) {
        hash = ((hash << 5) + hash) + c;
    }

    return (unsigned int)hash;
}

int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas,
                   const char *nombre, int tipo, int offset) {

    if (tabla == NULL || nombre == NULL || cubetas == 0) {
        return -1;
    }

    unsigned int hash_val = hash_djb2(nombre);
    unsigned int index = hash_val % cubetas;

    Simbolo *actual = tabla[index];

    while (actual != NULL) {
        if (strcmp(actual->nombre, nombre) == 0) {
            return 0;
        }
        actual = actual->sig;
    }

    Simbolo *nuevo = (Simbolo *)malloc(sizeof(Simbolo);

    if (nuevo == NULL) {
        return -1;
    }

    strncpy(nuevo->nombre, nombre, sizeof(nuevo->nombre) - 1);
    nuevo->nombre[sizeof(nuevo->nombre) - 1] = '\0';

    nuevo->tipo = tipo;
    nuevo->offset = offset;

    nuevo->sig = tabla[index];
    tabla[index] = nuevo;

    return 1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                               esperado:              obtenido:              resultado
[ 1] hash_djb2("")                      esperado: 5381         obtenido: 5381         OK
[ 2] hash_djb2("abc")                   esperado: 193485963    obtenido: 193485963    OK
[ 3] insertar x dos veces               esperado: 0            obtenido: 0            OK
[ 4] dos claves, una cubeta             esperado: 2            obtenido: 2            OK
[ 5] offset de beta en cadena           esperado: 4            obtenido: 4            OK
[ 6] insertar en tabla NULL             esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion11.md
Los números entre corchetes son el contador de casos. Mientras algo falle verás FALLA en esa línea, el total será menor y el programa terminará con código de salida 1.
```

**Pruebas ejecutadas:** Manejo de claves duplicadas y colisiones.

- [x] Caso base y límites evaluados sin Crash.
- [x] No se comprobaron todos los casos de memoria ni los nombres largos.
- [x] La complejidad teórica corresponde a una tabla hash con encadenamiento.
### Resultados de las pruebas
---

| Caso | Resultado esperado | Resultado obtenido |
|---|---:|---:|
| `hash_djb2("")` | 5381 | 5381 — OK |
| `hash_djb2("abc")` | 193485963 | 193485963 — OK |
| Insertar `x` dos veces | 0 | 0 — OK |
| Dos claves, una cubeta | 2 | 2 — OK |
| Offset de `beta` en cadena | 4 | 4 — OK |
| Insertar en tabla NULL | -1 | -1 — OK |

**Resultado final:** 6/6 pruebas superadas.

### Reporte Crítico

El caso borde que la IA omitió fue el manejo de nombres de variables mayores a 15 caracteres, porque el arreglo `nombre` de la estructura `Simbolo` solo permite almacenar 15 caracteres más el terminador nulo. Aunque la IA utilizó `strncpy()` para evitar escribir fuera del arreglo, el nombre se puede truncar y esto puede provocar problemas al buscar nombres repetidos o distinguir símbolos diferentes con prefijos iguales.

Lo detecté al revisar la estructura de datos y comparar el tamaño del arreglo con la manera en que se copia el nombre. El harness no incluye pruebas para nombres largos, por lo que las seis pruebas en verde no garantizan que todos los casos estén resueltos. Como mejora, agregaría pruebas con nombres de más de 15 caracteres, nombres con el mismo prefijo y una revisión de la memoria reservada para comprobar que todos los nodos se liberan correctamente.

---

**Firma digital:** [Usuario de GitHub]
