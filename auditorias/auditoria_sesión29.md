# Bitácora de Requerimientos y Auditoría IA — Sesión 29
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Optimización: Dead Code Elimination  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Detección de código inalcanzable  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar manualmente la función, tuve dificultad para controlar las instrucciones que debían eliminarse después de encontrar un GOTO o RETURN. También fue necesario tener cuidado al recorrer el arreglo de cadenas, porque cada instrucción tiene un espacio limitado de 64 caracteres. Otro problema fue compactar las instrucciones sin modificar el orden original ni sobrescribir información que todavía necesitaba procesar. Para resolverlo utilicé una variable de control que indica cuándo se encuentra código inalcanzable y que se desactiva al encontrar una etiqueta.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Análisis de flujo tras retornos y gotos |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy desarrollando la sesión 29 de la materia Lenguajes y Autómatas II.
El objetivo es implementar la optimización de código muerto (Dead Code
Elimination) en lenguaje C.

ROL:
Actúa como un ingeniero de software especializado en compiladores,
lenguaje C y optimización de código intermedio.

ESPECIFICACIÓN:
Desarrolla una función llamada eliminar_muerto() con la siguiente firma:

int eliminar_muerto(char lineas[][64], int n);

La función debe:
1. Recibir un arreglo de instrucciones TAC y su cantidad.
2. Detectar instrucciones GOTO y RETURN.
3. Eliminar las instrucciones posteriores a estos comandos hasta
   encontrar una etiqueta que termine en dos puntos.
4. Conservar las etiquetas y las instrucciones alcanzables.
5. Compactar el arreglo original sin utilizar memoria dinámica.
6. Mantener el orden original de las instrucciones.
7. Devolver la nueva cantidad de instrucciones.
8. Manejar correctamente una lista vacía.
9. Evitar accesos fuera de los límites del arreglo.
10. No incluir la función main().

OUTPUT:
Entrega únicamente el código completo en C, con las bibliotecas
necesarias, compatible con GCC y las opciones -Wall -Wextra -Werror.
Incluye funciones auxiliares si son necesarias.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion29.h"
#include <string.h>
#include <ctype.h>

static int es_etiqueta(const char *linea) {
    int longitud = (int)strlen(linea);

    while (longitud > 0 &&
           isspace((unsigned char)linea[longitud - 1])) {
        longitud--;
    }

    return longitud > 0 && linea[longitud - 1] == ':';
}

static int es_salto(const char *linea) {
    while (isspace((unsigned char)*linea)) {
        linea++;
    }

    size_t longitud = strlen(linea);

    if (longitud >= 4 &&
        strncmp(linea, "GOTO", 4) == 0 &&
        (linea[4] == '\0' ||
         isspace((unsigned char)linea[4]))) {
        return 1;
    }

    if (longitud >= 6 &&
        strncmp(linea, "RETURN", 6) == 0 &&
        (linea[6] == '\0' ||
         isspace((unsigned char)linea[6]))) {
        return 1;
    }

    return 0;
}

int eliminar_muerto(char lineas[][64], int n) {
    int lectura;
    int escritura = 0;
    int descartar = 0;

    for (lectura = 0; lectura < n; lectura++) {

        if (es_etiqueta(lineas[lectura])) {
            descartar = 0;
        }

        if (!descartar) {
            if (lectura != escritura) {
                strcpy(lineas[escritura], lineas[lectura]);
            }

            escritura++;

            if (es_salto(lineas[lectura])) {
                descartar = 1;
            }
        }
    }

    return escritura;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Eliminación de bloques inalcanzables

### Salida del harness
```text
=== AUDITORIA SESION 29 - Eliminacion de codigo inalcanzable ===
Caso                               esperado:              obtenido:              resultado
[ 1] tras GOTO: nuevo tamano            esperado: 4            obtenido: 4            OK
[ 2] tras GOTO: etiqueta viva           esperado: L1:          obtenido: L1:          OK
[ 3] tras RETURN: nuevo tamano          esperado: 4            obtenido: 4            OK
[ 4] tras RETURN: etiqueta viva         esperado: L5:          obtenido: L5:          OK
[ 5] sin codigo muerto                  esperado: 3            obtenido: 3            OK
[ 6] lista vacia                        esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Uno de los problemas que identifiqué fue que la implementación debía comprobar correctamente los límites de las cadenas al detectar las instrucciones GOTO y RETURN. Si no se verifica la longitud de la instrucción, se pueden realizar consultas innecesarias después del final de la cadena. Para solucionar este problema agregué una comprobación con strlen() antes de comparar las palabras GOTO y RETURN.
También consideré el caso de una lista vacía y la situación en la que aparecen varias instrucciones inalcanzables antes de encontrar una etiqueta. Esto permite que la función elimine correctamente todo el bloque muerto y conserve la etiqueta junto con las instrucciones posteriores.
Con estas consideraciones, la implementación resulta más segura y cumple con el propósito de la optimización.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
