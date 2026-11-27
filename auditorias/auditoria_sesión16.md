# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Firmas de Función y Parámetros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24531399  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Registro de prototipos y aridad  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual, la principal dificultad fue comprender cómo recorrer el catálogo de funciones y comparar correctamente el nombre y la cantidad de parámetros. También tuve que considerar las validaciones para evitar errores cuando el catálogo o el nombre fueran NULL, así como cuando la cantidad de argumentos fuera negativa. Otro detalle importante fue utilizar la función strcmp() para comparar los nombres y recordar incluir la biblioteca string.h. Finalmente, comprendí que debía devolver diferentes valores dependiendo de si la función existía y si su cantidad de argumentos era correcta.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Coincidencia estricta de parámetros |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy desarrollando la práctica de la sesión 16 de la materia
Lenguajes y Autómatas II. El objetivo es crear un verificador
de firmas de funciones en lenguaje C.

ROL:
Actúa como un ingeniero de software especializado en C y
desarrollo de compiladores.

ESPECIFICACIÓN:
Implementa la función:

int verificar_llamada(const Funcion *catalogo, int total,
                      const char *nombre, int nargs);

Debes respetar exactamente la estructura Funcion y la firma
declarada en api_sesion16.h.

La función debe cumplir las siguientes condiciones:

1. Recorrer el catálogo completo de funciones.
2. Buscar el nombre de manera exacta utilizando strcmp().
3. Devolver 1 si el nombre existe y la aridad coincide.
4. Devolver 0 si el nombre existe, pero la cantidad de
   argumentos es diferente.
5. Devolver -1 si el nombre no existe.
6. Devolver -1 si el catálogo es NULL.
7. Devolver -1 si el nombre es NULL.
8. Devolver -1 si la cantidad de argumentos es negativa.
9. Aceptar funciones con cero parámetros únicamente cuando
   se reciben cero argumentos.
10. Evitar accesos fuera de los límites del catálogo.

OUTPUT:
Entrega el código completo para ia_sesion16_c.c.
No incluyas la función main().
Incluye las bibliotecas necesarias y comentarios.
El código debe compilar con:

gcc -std=c11 -Wall -Wextra -Werror

Utiliza una estructura diferente a una implementación
manual sencilla, manteniendo el mismo resultado.]
```

---

## 3. Código Devuelto por el Modelo
```c
/*
 * ia_sesion16_c.c
 * Fase 2: El Cientifico
 * Verificacion de firmas de funciones.
 */

#include "api_sesion16.h"
#include <string.h>

int verificar_llamada(const Funcion *catalogo, int total,
                      const char *nombre, int nargs) {

    int posicion = 0;

    /* Revisar los datos de entrada */
    if (catalogo == NULL || nombre == NULL || nargs < 0) {
        return -1;
    }

    /* Recorrer el catalogo */
    while (posicion < total) {

        /* Comprobar si encontramos la funcion */
        if (catalogo[posicion].nombre != NULL) {

            if (strcmp(catalogo[posicion].nombre, nombre) == 0) {

                /* Verificar la cantidad de argumentos */
                return (catalogo[posicion].aridad == nargs) ? 1 : 0;
            }
        }

        posicion++;
    }

    /* No se encontro la funcion */
    return -1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas con número incorrecto de args

### Salida del harness
```text
=== AUDITORIA SESION 16 - Registro de prototipos y aridad ===
[ 1] suma con 3 argumentos         esperado: 1            obtenido: 1            OK
[ 2] suma con 2 argumentos         esperado: 0            obtenido: 0            OK
[ 3] suma con 4 argumentos         esperado: 0            obtenido: 0            OK
[ 4] funcion no registrada        esperado: -1           obtenido: -1           OK
[ 5] main sin argumentos          esperado: 1            obtenido: 1            OK
[ 6] nombre NULL                  esperado: -1           obtenido: -1            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Durante la revisión de la implementación generada por la inteligencia artificial identifiqué que faltaban pruebas para algunas condiciones especiales, como recibir un catálogo NULL, una cantidad de argumentos negativa o un catálogo vacío. Aunque el código de la IA contempla las validaciones de catálogo NULL y aridad negativa, estas situaciones no aparecen en el harness proporcionado. Para comprobar mejor el funcionamiento, revisé las condiciones del contrato y verifiqué que la función devolviera -1 ante datos inválidos. También comprobé que se ignoraran los nombres NULL dentro del catálogo. Con esto comprendí que una implementación puede superar todas las pruebas del harness y aun así necesitar pruebas adicionales para revisar otros casos borde.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
