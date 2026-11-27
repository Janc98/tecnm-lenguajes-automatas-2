# Bitácora de Requerimientos y Auditoría IA — Sesión 22
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Generación de TAC para Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Traductor de nodos AST a TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al desarrollar el programa manualmente, identifiqué que debía tener cuidado con el manejo de los arreglos de caracteres y los límites de memoria. También tuve que organizar correctamente el recorrido recursivo del árbol para procesar primero el hijo izquierdo, después el derecho y finalmente el operador. Una dificultad fue generar los nombres de los temporales y construir las instrucciones sin superar los 63 caracteres permitidos por línea. Para evitar errores utilicé funciones como snprintf y strcpy, además de verificar que no se excediera el máximo de instrucciones.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Conservación de precedencia de operadores |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en C y compiladores.

Necesito implementar una función que convierta un árbol AST en instrucciones TAC.

La función debe tener exactamente esta firma:

int generar_tac(const Nodo *raiz, char lineas[][64], int max);

Requisitos:
1. Recorrer el árbol en postorden.
2. Procesar primero el hijo izquierdo y después el derecho.
3. Generar una instrucción por cada operador binario.
4. Utilizar temporales consecutivos _t0, _t1, _t2.
5. Las hojas no deben generar instrucciones.
6. No superar el máximo de líneas permitido.
7. No escribir más de 63 caracteres por línea.
8. Utilizar snprintf para construir las instrucciones.
9. Respetar api_sesion22.h.
10. No incluir main().
11. Compilar con -std=c11 -Wall -Wextra -Werror.

Entrega el código completo para ia_sesion22_c.c y explica brevemente su complejidad temporal y espacial.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion22.h"
#include <stdio.h>
#include <string.h>

static int generar_nodo(
    const Nodo *nodo,
    char lineas[][64],
    int max,
    int *total,
    char salida[64]
) {
    char op_izq[64];
    char op_der[64];
    int n;

    if (nodo == NULL) {
        return 0;
    }

    if (nodo->izq == NULL && nodo->der == NULL) {
        n = snprintf(salida, 64, "%s", nodo->valor);
        return n >= 0 && n < 64;
    }

    if (nodo->izq == NULL || nodo->der == NULL) {
        return 0;
    }

    if (!generar_nodo(nodo->izq, lineas, max, total, op_izq)) {
        return 0;
    }

    if (!generar_nodo(nodo->der, lineas, max, total, op_der)) {
        return 0;
    }

    if (*total >= max) {
        return 0;
    }

    n = snprintf(
        lineas[*total],
        64,
        "_t%d = %s %c %s",
        *total,
        op_izq,
        nodo->op,
        op_der
    );

    if (n < 0 || n > 31) {
        return 0;
    }

    n = snprintf(salida, 64, "_t%d", *total);

    if (n < 0 || n >= 64) {
        return 0;
    }

    (*total)++;

    return 1;
}

int generar_tac(const Nodo *raiz, char lineas[][64], int max) {
    int total = 0;
    char resultado[64];

    if (raiz == NULL || lineas == NULL || max <= 0) {
        return 0;
    }

    if (!generar_nodo(raiz, lineas, max, &total, resultado)) {
        return 0;
    }

    return total;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones anidadas con paréntesis

### Salida del harness
```text
=== AUDITORIA SESION 22 - Traductor de AST a TAC con precedencia ===

[ 1] a + b * c
     obtenido: _t0 = b * c  OK

[ 2] a + b * c (2a linea)
     obtenido: _t1 = a + _t0
     -> OK

[ 3] (a + b) * c
     obtenido: _t1 = _t0 * c
     -> OK

[ 4] x * (y - z) + w
     obtenido: _t2 = _t1 + w
     -> OK

[ 5] hoja sin operaciones
     obtenido: 0  OK

[ 6] (a + b) * (c - d)
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la IA, identifiqué que no valida explícitamente el campo tipo de cada nodo. La función distingue las hojas y los operadores principalmente por la existencia de sus hijos, por lo que un nodo con un tipo incorrecto podría ser procesado como si fuera válido.
Por ejemplo, un nodo con tipo = 'X' y dos hijos podría generar una instrucción TAC aunque no corresponda a una hoja ni a un operador binario válido.
Las seis pruebas del harness no detectaron este problema porque utilizan árboles correctamente construidos. Como mejora, propongo validar que las hojas tengan tipo 'H' y los operadores tengan tipo 'B', además de agregar pruebas con nodos inválidos para comprobar el comportamiento del traductor.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
