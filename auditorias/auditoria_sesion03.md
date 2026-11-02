# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Google Colab, Mágicos y Flujo de Entrega  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Manipulación de archivos y llamadas OS  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> 
Al realizar el código manualmente, tuve dificultad para controlar correctamente la lectura y escritura de archivos. También tuve que considerar que un archivo puede estar vacío o que su última línea no termine con un salto de línea. Otro problema fue comprobar que los archivos se abrieran y cerraran correctamente para evitar errores durante la ejecución. Para solucionarlo utilicé las funciones fopen(), fputs(), fgetc() y fclose() de la biblioteca estándar de C.
---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Salida estándar y códigos de retorno ABI |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Pega aquí el prompt formal que redactaste para la IA]
```
Contexto:
Estoy trabajando en la Sesión 03 de Lenguajes y Autómatas II. Necesito desarrollar funciones en lenguaje C para escribir texto en archivos y contar sus líneas, utilizando la biblioteca estándar y respetando el contrato api_sesion03.h.

Rol:
Actúa como un ingeniero de software especializado en lenguaje C, programación de sistemas y compiladores.

Especificación:
Implementa las siguientes funciones respetando exactamente sus firmas:

int escribir_texto(const char *ruta, const char *texto);
int contar_lineas(const char *ruta);

La función escribir_texto debe abrir un archivo, escribir el texto recibido y cerrarlo correctamente. Debe devolver 0 si la operación es exitosa y -1 si ocurre un error.

La función contar_lineas debe leer un archivo y contar sus líneas. Debe considerar correctamente los archivos vacíos, los archivos con salto de línea final y los archivos cuya última línea no termina con salto de línea. Si el archivo no existe o ocurre un error de lectura, debe devolver -1.

Utiliza fopen(), fputs(), fgetc() y fclose(). Comprueba los errores de apertura, escritura, lectura y cierre. No utilices main(), no modifiques las firmas y procura que el código sea diferente de una solución básica de principiante, pero fácil de comprender.

Output:
Entrega el código completo para ia_sesion03_c.c, incluyendo api_sesion03.h. Debe compilar con GCC utilizando -std=c11 -Wall -Wextra -Werror y superar los seis casos del harness de la Sesión 03.
---

## 3. Código Devuelto por el Modelo
```c
// Código devuelto por el LLM
```
#include "api_sesion03.h"
#include <stdio.h>

int escribir_texto(const char *ruta, const char *texto)
{
    FILE *archivo;
    int resultado;

    if (ruta == NULL || texto == NULL) {
        return -1;
    }

    archivo = fopen(ruta, "w");

    if (archivo == NULL) {
        return -1;
    }

    resultado = fputs(texto, archivo);

    if (fclose(archivo) != 0) {
        return -1;
    }

    if (resultado == EOF) {
        return -1;
    }

    return 0;
}

int contar_lineas(const char *ruta)
{
    FILE *archivo;
    int caracter;
    int total = 0;
    int hay_datos = 0;

    if (ruta == NULL) {
        return -1;
    }

    archivo = fopen(ruta, "r");

    if (archivo == NULL) {
        return -1;
    }

    while ((caracter = fgetc(archivo)) != EOF) {
        if (caracter == '\n') {
            total++;
            hay_datos = 0;
        } else {
            hay_datos = 1;
        }
    }

    if (ferror(archivo)) {
        fclose(archivo);
        return -1;
    }

    if (hay_datos) {
        total++;
    }

    if (fclose(archivo) != 0) {
        return -1;
    }

    return total;
}
---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Detección de errores de compilación GCC
=== AUDITORIA SESION 03 - Archivos: escribir y contar lineas ===
Caso                               esperado:              obtenido:              resultado
[ 1] escribir_texto(...)==0             esperado: 0            obtenido: 0            OK
[ 2] contar_lineas(3 lineas)            esperado: 3            obtenido: 3            OK
[ 3] contar_lineas(sin salto final)     esperado: 2            obtenido: 2            OK
[ 4] contar_lineas(archivo vacio)       esperado: 0            obtenido: 0            OK
[ 5] contar_lineas(inexistente)         esperado: -1           obtenido: -1           OK
[ 6] escribir_texto(ruta invalida)      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion03.md
### Salida del harness
```text
=== AUDITORIA SESION 03 - Archivos: escribir y contar lineas ===
Caso                               esperado:              obtenido:              resultado
[ 1] escribir_texto(...)==0             esperado: 0            obtenido: 0            OK
[ 2] contar_lineas(3 lineas)            esperado: 3            obtenido: 3            OK
[ 3] contar_lineas(sin salto final)     esperado: 2            obtenido: 2            OK
[ 4] contar_lineas(archivo vacio)       esperado: 0            obtenido: 0            OK
[ 5] contar_lineas(inexistente)         esperado: -1           obtenido: -1           OK
[ 6] escribir_texto(ruta invalida)      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion03.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la IA, identifiqué que era importante comprobar correctamente los errores durante la lectura del archivo y verificar que se cerrara después de utilizarlo. También revisé el caso de los archivos vacíos y aquellos que no terminan con un salto de línea. Para asegurar su correcto funcionamiento, se utilizó la variable hay_datos, que permite identificar si existe contenido pendiente de contar como última línea. De esta manera, se evita contar incorrectamente las líneas y se cumple con los casos establecidos en el contrato.
--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
