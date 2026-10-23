# Bitácora de Requerimientos y Auditoría IA — Sesión 03

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Google Colab, Mágicos y Flujo de Entrega  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Rafael Antonio Castillo Basto  
**Matrícula:** 24531348 
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Manipulación de archivos y llamadas al sistema operativo.

Al realizar la implementación manual, identifiqué que uno de los principales retos era trabajar correctamente con los archivos en C, abrirlos en el modo adecuado y cerrarlos después de realizar las operaciones. También fue necesario controlar los errores que pueden presentarse cuando un archivo no existe o no se puede abrir.

Otra dificultad fue contar las líneas de un archivo, especialmente cuando el texto no termina con un salto de línea. Para resolverlo, utilicé `fgetc()` para leer el archivo carácter por carácter y comprobar si el último carácter leído era `'\n'`.

También identifiqué que la función `system()` no devuelve directamente el código de salida del comando, sino un estado que debe interpretarse mediante las macros correspondientes. Por eso, es importante utilizar `WIFEXITED()` y `WEXITSTATUS()` para obtener correctamente el resultado de la ejecución.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99/C11 con GCC, utilizando la biblioteca estándar de C y las funciones POSIX disponibles en el entorno de Linux. |
| **Rol (R)** | Ingeniero de software especializado en programación de sistemas, compiladores y manejo de archivos. |
| **Especificación (E)** | Implementar funciones para escribir texto en archivos, contar líneas y ejecutar comandos externos. Manejar errores mediante valores de retorno y obtener correctamente los códigos de salida de los procesos. |
| **Output (O)** | Código modular en C, sin función `main()`, compatible con el contrato de `api_sesion03.h`, acompañado de un análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en la materia Lenguajes y Autómatas II y necesito
implementar funciones en C para manipular archivos de texto y
ejecutar comandos externos en un entorno Linux con GCC.

Rol:
Actúa como un ingeniero de software especializado en programación
de sistemas operativos, manejo de archivos y códigos de salida.

Especificación:
Implementa exactamente las siguientes funciones:

int escribir_texto(const char *ruta, const char *texto);
int contar_lineas(const char *ruta);
int ejecutar(const char *comando);

Requisitos:
1. escribir_texto debe abrir el archivo, escribir el contenido y
   cerrar el archivo correctamente.
2. Si la ruta o el texto son NULL, debe devolver -1.
3. Si el archivo no puede abrirse o ocurre un error de escritura,
   debe devolver -1.
4. contar_lineas debe leer el archivo y devolver el número correcto
   de líneas, incluso cuando la última línea no termina en '\n'.
5. Si el archivo no existe o no puede leerse, debe devolver -1.
6. ejecutar debe utilizar system() para ejecutar el comando.
7. Debe devolver el código de salida real del comando mediante
   WIFEXITED() y WEXITSTATUS().
8. Si el comando es NULL, system() falla o no se obtiene un estado
   de terminación normal, debe manejarse el error.
9. Mantén exactamente las firmas declaradas en api_sesion03.h.
10. No incluyas main(), porque el harness contiene esa función.
11. El código debe compilar sin advertencias con:
    gcc -std=c11 -Wall -Wextra -Werror

Output:
Devuelve la implementación completa de las tres funciones y
explica su complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo

Archivo: `ia_sesion03_c.c`

```c
#include "api_sesion03.h"
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>

int escribir_texto(const char *ruta, const char *texto) {
    if (ruta == NULL || texto == NULL) {
        return -1;
    }

    FILE *archivo = fopen(ruta, "w");
    if (archivo == NULL) {
        return -1;
    }

    int resultado = fputs(texto, archivo);
    int error_cierre = fclose(archivo);

    if (resultado == EOF || error_cierre == EOF) {
        return -1;
    }

    return 0;
}

int contar_lineas(const char *ruta) {
    if (ruta == NULL) {
        return -1;
    }

    FILE *archivo = fopen(ruta, "r");
    if (archivo == NULL) {
        return -1;
    }

    int lineas = 0;
    int caracter;
    int ultimo = '\n';

    while ((caracter = fgetc(archivo)) != EOF) {
        if (caracter == '\n') {
            lineas++;
        }

        ultimo = caracter;
    }

    if (ferror(archivo)) {
        fclose(archivo);
        return -1;
    }

    if (ultimo != '\n') {
        lineas++;
    }

    if (fclose(archivo) == EOF) {
        return -1;
    }

    return lineas;
}

int ejecutar(const char *comando) {
    if (comando == NULL) {
        return -1;
    }

    int estado = system(comando);

    if (estado == -1) {
        return -1;
    }

    if (!WIFEXITED(estado)) {
        return -1;
    }

    return WEXITSTATUS(estado);
}
```

### Análisis de complejidad

- **`escribir_texto()`:** O(n), donde n es la cantidad de caracteres que se escriben en el archivo.
- **`contar_lineas()`:** O(n), porque necesita leer todos los caracteres del archivo para contar sus líneas.
- **`ejecutar()`:** Su costo depende del comando externo que se ejecute y del tiempo que tarde en terminar.
- **Complejidad espacial auxiliar:** O(1), porque las funciones utilizan una cantidad constante de variables auxiliares.
- **Manejo de recursos:** Los archivos abiertos se cierran antes de que la función termine, incluso cuando se detecta un error de lectura.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** escritura de archivos, conteo de líneas con y sin salto final, archivo inexistente y ejecución de comandos exitosos y fallidos.

### Resultados de las pruebas

De acuerdo con las ejecuciones registradas en Google Colab, tanto la implementación manual como la implementación de IA superaron los seis casos establecidos por el harness.

```text
=== AUDITORIA SESION 03 - Archivos y codigos de salida ===
Caso                               esperado:              obtenido:              resultado
[ 1] escribir_texto(...)==0             esperado: 0            obtenido: 0            OK
[ 2] contar_lineas(3 lineas)            esperado: 3            obtenido: 3            OK
[ 3] contar_lineas(sin salto final)     esperado: 2            obtenido: 2            OK
[ 4] contar_lineas(inexistente)         esperado: -1           obtenido: -1           OK
[ 5] ejecutar("true")                   esperado: 0            obtenido: 0            OK
[ 6] ejecutar("false")                  esperado: 1            obtenido: 1            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion03.md
```

### Lista de verificación

- [x] Los seis casos del harness se superaron sin errores.
- [x] Los archivos se cierran después de utilizarlos.
- [x] Un archivo inexistente se reporta mediante el valor -1.
- [x] Se distingue una última línea que no termina con salto de línea.
- [x] El comando `true` devuelve 0 y el comando `false` devuelve 1.
- [x] La función `ejecutar()` comprueba si el proceso terminó normalmente antes de extraer su código de salida.

### Reporte Crítico

Las seis pruebas del harness se superaron correctamente. Sin embargo, al revisar la implementación inicial de IA, identifiqué que utilizaba `WEXITSTATUS()` sin comprobar primero si el proceso había terminado normalmente. Esto podía provocar un resultado incorrecto cuando el proceso terminaba por una señal, en lugar de finalizar con un código de salida normal.

Para solucionar este problema, agregué la condición `WIFEXITED(estado)` antes de utilizar `WEXITSTATUS(estado)`. De esta manera, la función devuelve el código de salida cuando la terminación es normal y devuelve -1 cuando no puede obtenerse un código de salida válido.

También revisé las funciones de archivos para comprobar los errores de escritura, lectura y cierre. Estas verificaciones son importantes porque una operación puede fallar incluso después de que el archivo se haya abierto correctamente.

Esta actividad me permitió comprender que un programa no solo debe realizar correctamente sus operaciones, sino también informar si cada paso terminó de manera exitosa. Esto es importante en los compiladores y en los sistemas automatizados, donde una etapa debe poder detectar los errores de la etapa anterior y decidir si puede continuar.

---

**Firma digital:** [Usuario de GitHub]
