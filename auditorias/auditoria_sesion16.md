# Bitácora de Requerimientos y Auditoría IA — Sesión 16

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Firmas de Función y Parámetros Formales  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafal Antonio Castillo Basto  
**Matrícula:** 24531348  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Registro de prototipos y aridad.

Al realizar la implementación manual tuve algunas dificultades para comparar correctamente el nombre de las funciones y la cantidad de argumentos que recibe cada una. Tuve que utilizar la función `strcmp()` para comprobar que los nombres coincidieran exactamente y recorrer el catálogo para encontrar la función solicitada.

También tuve que considerar los casos en los que una función no estuviera registrada, el nombre fuera NULL o la cantidad de argumentos fuera incorrecta. Lo más importante fue distinguir entre una función inexistente y una llamada con una cantidad de parámetros diferente.

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC utilizando bibliotecas estándar. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Verificar que el nombre de la función exista en el catálogo y que la cantidad de argumentos coincida exactamente con su aridad registrada. |
| **Output (O)** | Código modular en C y análisis de complejidad temporal y espacial. |

### 2.2 Prompt Estructurado

**Contexto:**

Estoy trabajando en una práctica de Lenguajes y Autómatas II utilizando C99 y GCC. Necesito implementar un verificador de llamadas a funciones que utilice un catálogo de firmas previamente registradas en el archivo `api_sesion16.h`.

**Rol:**

Quiero que actúes como un ingeniero de software especializado en programación de sistemas y desarrollo de compiladores, con conocimientos sobre firmas de funciones, parámetros formales y análisis semántico.

**Especificación:**

Necesito que desarrolles la función `verificar_llamada()` tomando en cuenta los siguientes requisitos:

1. Recorrer el catálogo de funciones para buscar el nombre solicitado.
2. Comparar los nombres de manera exacta utilizando `strcmp()`.
3. Verificar que la cantidad de argumentos de la llamada coincida con la aridad registrada.
4. Devolver 1 cuando el nombre exista y la cantidad de argumentos sea correcta.
5. Devolver 0 cuando la función exista, pero la cantidad de argumentos sea diferente.
6. Devolver -1 cuando el nombre de la función no esté registrado.
7. Si el catálogo es NULL, el nombre es NULL o la cantidad de argumentos es negativa, devolver -1.
8. Una función registrada con cero parámetros solamente debe aceptar llamadas con cero argumentos.
9. Respetar la firma original declarada en `api_sesion16.h`.
10. No incluir `main()`, porque el programa de pruebas ya cuenta con uno.

**Output:**

Quiero que entregues el código completo y modular en C, acompañado de un análisis de complejidad temporal y espacial. El código debe compilar correctamente con `-Wall -Wextra -Werror`, sin errores ni advertencias.

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion16.h"
#include <string.h>
#include <stddef.h>

int verificar_llamada(const Funcion *catalogo, int total,
                      const char *nombre, int nargs) {
    if (catalogo == NULL || nombre == NULL || nargs < 0) {
        return -1;
    }

    for (int i = 0; i < total; i++) {
        if (catalogo[i].nombre != NULL &&
            strcmp(catalogo[i].nombre, nombre) == 0) {
            return (catalogo[i].aridad == nargs) ? 1 : 0;
        }
    }

    return -1;
}
```

**Análisis de complejidad:**

- Complejidad temporal: O(n), porque en el peor caso se necesita recorrer todo el catálogo de funciones.
- Complejidad espacial: O(1), porque solamente se utilizan variables auxiliares y no se reserva memoria adicional.

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Llamadas con número incorrecto de argumentos.

- [x] Caso base y límites superados sin Crash.
- [x] No existen accesos fuera de límites ni *Memory Leaks*.
- [x] La complejidad teórica coincide con la implementada.


**Resultado obtenido: 6/6 pruebas superadas.**

### Salida del Test Harness

```text
=== AUDITORIA SESION 16 - Registro de prototipos y aridad ===

[ 1] suma con 3 argumentos       esperado: 1  obtenido: 1  OK
[ 2] suma con 2 argumentos       esperado: 0  obtenido: 0  OK
[ 3] suma con 4 argumentos       esperado: 0  obtenido: 0  OK
[ 4] funcion no registrada      esperado: -1 obtenido: -1 OK
[ 5] main sin argumentos        esperado: 1  obtenido: 1  OK
[ 6] nombre NULL                esperado: -1 obtenido: -1 OK

RESULTADO: 6/6 pruebas superadas

```

### Reporte Crítico

El principal caso borde que debía revisarse era cuando se intentaba verificar una función que no estaba registrada en el catálogo. También era necesario comprobar que una función con cero parámetros solamente aceptara cero argumentos.

Para solucionar estos casos, se agregó una validación que comprueba si el catálogo o el nombre son NULL y si la cantidad de argumentos es negativa. Después, se recorre el catálogo comparando los nombres de las funciones y verificando su aridad.

Al ejecutar el Test Harness, todas las pruebas fueron superadas. Tanto la implementación manual como la generada por IA obtuvieron un resultado de 6/6 pruebas superadas.

---

**Firma digital:** [Usuario de GitHub]
