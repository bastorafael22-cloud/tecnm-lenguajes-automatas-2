# Bitácora de Requerimientos y Auditoría IA — Sesión 09

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Reporte Formal de Errores Semánticos y Diagnóstico  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Diagnóstico con línea y columna.

**Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:**

> Al realizar la implementación manual, la principal dificultad fue construir una cadena de texto con el formato exacto que utiliza GCC para mostrar los errores. Se tuvo que respetar el orden del nombre del archivo, número de línea, columna, nivel del diagnóstico y mensaje. También fue importante controlar la capacidad del buffer para evitar escribir más caracteres de los permitidos. Para resolverlo utilicé la función `snprintf()`, que permite limitar la cantidad de caracteres escritos y agregar el carácter nulo `\0` al final de la cadena. Además, agregué una validación para evitar trabajar con un buffer nulo o con una capacidad menor o igual a cero.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C11 / GCC, utilizando bibliotecas estándar como `stdio.h` y respetando el contrato de `api_sesion09.h`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar una función que genere diagnósticos con el formato `archivo:línea:columna: error: mensaje`, controle la capacidad del buffer y garantice la terminación con `\0`. |
| **Output (O)** | Código modular en C, sin función `main`, respetando exactamente la firma solicitada y con análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado en sistemas y compiladores.

Necesito implementar en lenguaje C una función para generar reportes
formales de errores semánticos con el formato utilizado por GCC.

Contexto:
El proyecto utiliza el archivo api_sesion09.h, que declara la siguiente
función:

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap);

Especificaciones:
1. La función debe construir un mensaje con el siguiente formato exacto:
   archivo:línea:columna: error: mensaje

2. Debe respetar el orden de los elementos y no agregar espacios
   adicionales ni puntos finales.

3. Debe utilizar el buffer de salida proporcionado por el llamador.

4. Nunca debe escribir más de cap-1 caracteres y el resultado debe
   terminar con el carácter nulo '\0'.

5. Debe validar que salida no sea NULL y que cap sea mayor que cero
   antes de escribir.

Restricciones:
- Utilizar lenguaje C.
- Incluir la cabecera api_sesion09.h.
- Mantener exactamente la firma de la función.
- No incluir una función main().
- Utilizar bibliotecas estándar.
- El código debe compilar con:
  gcc -std=c11 -Wall -Wextra -Werror

Entrega:
Proporciona únicamente la implementación modular de la función
reportar() y explica brevemente su complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion09.h"
#include <stdio.h>

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap) {
    if (salida != NULL && cap > 0) {
        snprintf(salida, (size_t)cap, "%s:%d:%d: error: %s",
                 archivo, linea, columna, mensaje);
    }
}
```

**Análisis de complejidad:**

- **Complejidad temporal:** O(n), donde n representa la longitud del texto que se debe construir, porque `snprintf()` procesa los caracteres del archivo, el mensaje y los valores numéricos.
- **Complejidad espacial auxiliar:** O(1), porque la función no reserva memoria dinámica y utiliza únicamente el buffer proporcionado por el llamador.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Generación de diagnósticos con archivo, línea, columna, mensaje y capacidad limitada del buffer.

**Resultados obtenidos:**

| Caso | Prueba | Resultado esperado | Resultado obtenido |
| :--- | :--- | :--- | :--- |
| 1 | Diagnóstico completo | `main.c:14:5: error: tipo incompatible` | Correcto |
| 2 | Mensaje corto | `a.c:1:1: error: x` | Correcto |
| 3 | Línea y columna en cero | `x.c:0:0: error: y` | Correcto |
| 4 | Buffer con capacidad 10 | `main.c:14` | Correcto |

**Salida de la auditoría:**

```text
=== AUDITORIA SESION 09 - Diagnostico con linea y columna ===
Caso                               esperado:              obtenido:              resultado
[ 1] main.c:14:5
     esperado: main.c:14:5: error: tipo incompatible
     obtenido: main.c:14:5: error: tipo incompatible
     -> OK
[ 2] a.c:1:1
     esperado: a.c:1:1: error: x
     obtenido: a.c:1:1: error: x
     -> OK
[ 3] linea 0
     esperado: x.c:0:0: error: y
     obtenido: x.c:0:0: error: y
     -> OK
[ 4] cap=10 trunca                      esperado: main.c:14    obtenido: main.c:14    OK

RESULTADO: 4/4 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion09.md
```

**Verificación:**

- [x] Caso base y límites superados sin *Crash*.
- [x] No se identificaron accesos fuera de límites ni *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementación.
- [x] Los cuatro casos del Test Harness fueron aprobados.
- [x] El programa compiló con `-std=c11 -Wall -Wextra -Werror`.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> La IA no contempló explícitamente qué debía suceder cuando el buffer de salida fuera `NULL` o cuando su capacidad fuera cero o negativa. Aunque la implementación generada ya incluía una condición para validar `salida != NULL` y `cap > 0`, fue importante revisar que la función no intentara escribir en esas condiciones. También se verificó el caso de un buffer pequeño con capacidad de 10 caracteres, donde la salida debía truncarse exactamente a `main.c:14`, conservando el carácter nulo al final. El Test Harness permitió comprobar este comportamiento y confirmó que los cuatro casos establecidos fueron superados. Como observación adicional, el harness no prueba directamente los argumentos `NULL` ni las capacidades no positivas.

---

**Firma digital:** [Tu usuario de GitHub]
