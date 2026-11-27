# Bitácora de Requerimientos y Auditoría IA — Sesión 27
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Indexación de Arreglos en TAC  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24351399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de desplazamientos base + offset  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar el código manualmente, identifiqué que era necesario calcular correctamente el desplazamiento de memoria antes de acceder a un elemento del arreglo. En los arreglos de una dimensión se multiplica el índice por el ancho del elemento. En los arreglos de dos dimensiones primero se calcula el índice lineal mediante la operación i * filas + j y después se multiplica por el ancho. También tuve que cuidar que los temporales no se repitieran y que las instrucciones no excedieran el tamaño de los arreglos de caracteres.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Aplanado de direcciones unidimensionales |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy trabajando en la materia Lenguajes y Autómatas II.
Necesito implementar funciones en lenguaje C para generar
código intermedio de tres direcciones (TAC) para arreglos
de una y dos dimensiones.

ROL:
Actúa como un ingeniero de software especializado en
compiladores y generación de código intermedio.

ESPECIFICACIÓN:
Implementa las funciones emitir_acceso() y
emitir_acceso_2d() respetando exactamente las firmas
del archivo api_sesion27.h.

Para arreglos 1D:
1. Multiplicar el índice por el ancho del elemento.
2. Generar un temporal para el desplazamiento.
3. Generar otro temporal para leer base[offset].

Para arreglos 2D:
1. Calcular i * filas.
2. Sumar j al resultado.
3. Multiplicar el índice lineal por el ancho.
4. Leer el elemento del arreglo utilizando el offset.

Cada temporal debe ser único y consecutivo.
Si base, los índices, lineas o temporales son NULL,
no se deben generar instrucciones.
Respetar el límite max de instrucciones.
Utilizar snprintf para evitar desbordamientos.

OUTPUT:
Entrega únicamente código C compatible con GCC,
utilizando las bibliotecas estándar necesarias.
No incluir main(), porque el programa será probado
mediante un harness externo.
El código debe compilar con:
gcc -std=c11 -Wall -Wextra -Werror]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion27.h"
#include <stdio.h>
#include <string.h>

int emitir_acceso(
    char lineas[][64],
    int max,
    const char *base,
    const char *indice,
    int ancho,
    int *temporales
) {
    int actual;
    char desplazamiento[64];
    char lectura[64];

    if (base == NULL || indice == NULL ||
        lineas == NULL || temporales == NULL || max < 2) {
        return 0;
    }

    actual = *temporales;

    if (snprintf(desplazamiento, sizeof(desplazamiento),
                 "_t%d = %s * %d",
                 actual, indice, ancho) >= (int)sizeof(desplazamiento)) {
        return 0;
    }

    if (snprintf(lectura, sizeof(lectura),
                 "_t%d = %s[_t%d]",
                 actual + 1, base, actual) >= (int)sizeof(lectura)) {
        return 0;
    }

    strcpy(lineas[0], desplazamiento);
    strcpy(lineas[1], lectura);

    *temporales += 2;

    return 2;
}

int emitir_acceso_2d(
    char lineas[][64],
    int max,
    const char *base,
    const char *i,
    const char *j,
    int filas,
    int ancho,
    int *temporales
) {
    int actual;
    char instrucciones[4][64];

    if (base == NULL || i == NULL || j == NULL ||
        lineas == NULL || temporales == NULL || max < 4) {
        return 0;
    }

    actual = *temporales;

    if (snprintf(instrucciones[0], 64,
                 "_t%d = %s * %d",
                 actual, i, filas) >= 64) {
        return 0;
    }

    if (snprintf(instrucciones[1], 64,
                 "_t%d = _t%d + %s",
                 actual + 1, actual, j) >= 64) {
        return 0;
    }

    if (snprintf(instrucciones[2], 64,
                 "_t%d = _t%d * %d",
                 actual + 2, actual + 1, ancho) >= 64) {
        return 0;
    }

    if (snprintf(instrucciones[3], 64,
                 "_t%d = %s[_t%d]",
                 actual + 3, base, actual + 2) >= 64) {
        return 0;
    }

    memcpy(lineas, instrucciones, sizeof(instrucciones));

    *temporales = actual + 4;

    return 4;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Arreglos multidimensionales en TAC

### Salida del harness
```text
=== AUDITORIA SESION 27 - Desplazamientos para arreglos 1D y 2D ===
Caso                               esperado:              obtenido:              resultado
[ 1] 1D: i * 4                          esperado: _t0 = i * 4  obtenido: _t0 = i * 4  OK
[ 2] 1D: A[_t0]                         esperado: _t1 = A[_t0] obtenido: _t1 = A[_t0] OK
[ 3] 2D: i * 3                          esperado: _t0 = i * 3  obtenido: _t0 = i * 3  OK
[ 4] 2D: offset por ancho
     esperado: _t2 = _t1 * 4
     obtenido: _t2 = _t1 * 4
     -> OK
[ 5] 2D: M[_t2]                         esperado: _t3 = M[_t2] obtenido: _t3 = M[_t2] OK
[ 6] base NULL (sin acceso)             esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la IA, observé que no se contemplaban todos los posibles casos de error relacionados con los parámetros recibidos. Por ejemplo, el código comprueba que la base, los índices y los temporales no sean NULL, además de verificar que exista espacio suficiente para las instrucciones. Sin embargo, no valida valores negativos en el ancho del elemento, las filas o el contador de temporales.
También observé que el código depende de que los nombres de los temporales y las cadenas recibidas tengan un tamaño adecuado. Aunque el harness proporcionado no presentó problemas, estos casos podrían provocar resultados incorrectos en otras situaciones.
Como resultado, identifiqué que las seis pruebas del harness comprueban el funcionamiento principal, pero no todos los posibles casos límite. Para esta actividad conservé las firmas y la implementación solicitada, comprobando que ambas versiones compilaran correctamente y generaran las instrucciones TAC esperadas.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
