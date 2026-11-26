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
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*Al realizar el código manualmente, tuve dificultad para manejar correctamente los archivos en C utilizando las funciones fopen, fgetc y fclose. También tuve que considerar cómo contar las líneas cuando el archivo no termina con un salto de línea, cómo identificar un archivo vacío y cómo controlar los errores cuando una ruta no existe. Otra dificultad fue asegurarme de cerrar los archivos correctamente para evitar problemas durante la ejecución.
>

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
[Actúa como un ingeniero de software especializado en C y desarrollo de sistemas.

CONTEXTO:
Estoy trabajando en la materia Lenguajes y Autómatas II.
Necesito implementar funciones para manipular archivos de texto
y controlar los errores mediante códigos de retorno.

ROL:
Actúa como un programador experto en C, pero explica y desarrolla
el código de manera sencilla.

ESPECIFICACIONES:
1. Implementa la función:
   int escribir_texto(const char *ruta, const char *texto);

2. Implementa la función:
   int contar_lineas(const char *ruta);

3. Utiliza las bibliotecas estándar de C.
4. No agregues una función main().
5. Respeta exactamente las firmas de las funciones.
6. La función escribir_texto debe regresar 0 si escribe
   correctamente y -1 si ocurre algún error.
7. La función contar_lineas debe regresar -1 si el archivo
   no existe o presenta errores de lectura.
8. Debe contar correctamente las líneas aunque la última
   no termine con un salto de línea.
9. Un archivo vacío debe regresar 0.
10. Todos los archivos deben cerrarse correctamente.
11. El código debe compilar con:
    gcc -std=c11 -Wall -Wextra -Werror

OUTPUT:
Entrega únicamente el código completo en C, modular,
ordenado y compatible con el contrato api_sesion03.h.
Incluye un análisis breve de la complejidad.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion03.h"
#include <stdio.h>

int escribir_texto(const char *ruta, const char *texto) {
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

int contar_lineas(const char *ruta) {
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
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Detección de errores de compilación GCC

### Salida del harness
```text
=== AUDITORIA SESION 03 - Archivos: escribir y contar lineas ===
Caso                               esperado:              obtenido:              resultado

[ 1] escribir_texto(...)==0             esperado: 0            obtenido: 0            OK
[ 2] contar_lineas(3 lineas)             esperado: 3            obtenido: 3            OK
[ 3] contar_lineas(sin salto final)      esperado: 2            obtenido: 2            OK
[ 4] contar_lineas(archivo vacio)        esperado: 0            obtenido: 0            OK
[ 5] contar_lineas(inexistente)          esperado: -1           obtenido: -1           OK
[ 6] escribir_texto(ruta invalida)       esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la IA, identifiqué que era importante comprobar correctamente los errores durante la lectura del archivo y verificar que se cerrara después de utilizarlo. También revisé el caso de los archivos vacíos y aquellos que no terminan con un salto de línea. Para asegurar su correcto funcionamiento, se utilizó la variable hay_datos, que permite identificar si existe contenido pendiente de contar como última línea. De esta manera, se evita contar incorrectamente las líneas y se cumple con los casos establecidos en el contrato.

## Caso borde omitido por la IA
La IA no consideró inicialmente todos los posibles errores durante la manipulación de archivos, especialmente cuando ocurre un problema durante la lectura o el cierre del archivo. También era necesario comprobar el caso de un archivo vacío y otro que no termina con salto de línea. Para solucionarlo, se agregó la variable hay_datos, que permite identificar si quedó contenido pendiente de contar como última línea. Así se logró que el programa contara correctamente las líneas y superara las seis pruebas del harness.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
