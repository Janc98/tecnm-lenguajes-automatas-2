# Bitácora de Requerimientos y Auditoría IA — Sesión 28
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Optimización: Constant Folding  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Plegado y simplificación de constantes  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la función manualmente, identifiqué que debía comprobar que los operandos fueran números enteros completos antes de realizar las operaciones. También tuve que considerar que las variables no debían modificarse cuando no fuera posible calcular el resultado. Una dificultad fue utilizar funciones como strtol para convertir las cadenas y snprintf para guardar el resultado sin exceder la capacidad del arreglo. Además, tuve que controlar la división entre cero para evitar errores durante la ejecución.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Reducción estática en compilación |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[**Contexto:** Estoy trabajando en la materia Lenguajes y Autómatas II y necesito implementar una optimización local llamada Constant Folding en lenguaje C.

**Rol:** Actúa como un ingeniero de software especializado en compiladores y programación en C.

**Especificación:**
Desarrolla una función llamada `plegar()` con la siguiente firma exacta:

```c
void plegar(char *dst, int cap, const char *res,
            const char *op, const char *a, const char *b);
```

La función debe cumplir los siguientes requisitos:

1. Identificar si los dos operandos son números enteros completos.
2. Calcular operaciones de suma, resta, multiplicación y división cuando ambos operandos sean constantes.
3. Aplicar las identidades algebraicas:
   - x + 0 = x
   - x * 1 = x
   - x * 0 = 0
4. Si los operandos contienen variables, conservar la operación original cuando no exista una identidad aplicable.
5. No realizar divisiones entre cero.
6. Evitar desbordamientos de enteros.
7. Utilizar `snprintf` para controlar la escritura en el destino.
8. Validar los parámetros de entrada.
9. Incluir las bibliotecas estándar necesarias.
10. No incluir `main()`, porque el programa será probado mediante un harness externo.

**Output:** Entrega únicamente el código modular en C, compatible con `-std=c11 -Wall -Wextra -Werror`, y explica brevemente su complejidad temporal y espacial.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion28.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <limits.h>

/* Comprueba si una cadena contiene un entero completo */
static int convertir_entero(const char *texto, int *valor) {
    char *final;
    long numero;

    if (texto == NULL || *texto == '\0') {
        return 0;
    }

    errno = 0;
    numero = strtol(texto, &final, 10);

    if (errno != 0 || *final != '\0' ||
        numero < INT_MIN || numero > INT_MAX) {
        return 0;
    }

    *valor = (int)numero;
    return 1;
}

/* Conserva la instruccion TAC original */
static void conservar(char *dst, int cap, const char *res,
                      const char *op, const char *a, const char *b) {
    snprintf(dst, (size_t)cap, "%s = %s %s %s",
             res, a, op, b);
}

void plegar(char *dst, int cap, const char *res,
            const char *op, const char *a, const char *b) {
    int x = 0, y = 0;
    int constante_x, constante_y;
    long long valor = 0;

    if (dst == NULL || cap <= 0 || res == NULL ||
        op == NULL || a == NULL || b == NULL) {
        return;
    }

    constante_x = convertir_entero(a, &x);
    constante_y = convertir_entero(b, &y);

    /* Aplicar identidades algebraicas */
    if (strcmp(op, "+") == 0) {
        if (constante_y && y == 0) {
            snprintf(dst, (size_t)cap, "%s = %s", res, a);
            return;
        }

        if (constante_x && x == 0) {
            snprintf(dst, (size_t)cap, "%s = %s", res, b);
            return;
        }
    }

    if (strcmp(op, "*") == 0) {
        if ((constante_x && x == 0) ||
            (constante_y && y == 0)) {
            snprintf(dst, (size_t)cap, "%s = 0", res);
            return;
        }

        if (constante_y && y == 1) {
            snprintf(dst, (size_t)cap, "%s = %s", res, a);
            return;
        }

        if (constante_x && x == 1) {
            snprintf(dst, (size_t)cap, "%s = %s", res, b);
            return;
        }
    }

    /* Sin dos constantes, se conserva la operacion */
    if (!constante_x || !constante_y || op[1] != '\0') {
        conservar(dst, cap, res, op, a, b);
        return;
    }

    /* Evaluar la operacion durante la compilacion */
    switch (op[0]) {
        case '+':
            valor = (long long)x + y;
            break;

        case '-':
            valor = (long long)x - y;
            break;

        case '*':
            valor = (long long)x * y;
            break;

        case '/':
            if (y == 0 || (x == INT_MIN && y == -1)) {
                conservar(dst, cap, res, op, a, b);
                return;
            }
            valor = x / y;
            break;

        default:
            conservar(dst, cap, res, op, a, b);
            return;
    }

    /* Evitar resultados fuera del rango de int */
    if (valor < INT_MIN || valor > INT_MAX) {
        conservar(dst, cap, res, op, a, b);
        return;
    }

    snprintf(dst, (size_t)cap, "%s = %lld", res, valor);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Simplificación de expresiones constantes

### Salida del harness
```text
=== AUDITORIA SESION 28 - Plegado de constantes e identidades ===
Caso                               esperado:              obtenido:              resultado
[ 1] 3 + 5                              esperado: _t0 = 8      obtenido: _t0 = 8      OK
[ 2] 2 * 3                              esperado: _t0 = 6      obtenido: _t0 = 6      OK
[ 3] 7 - 10                             esperado: _t0 = -3     obtenido: _t0 = -3     OK
[ 4] x + 0                              esperado: _t0 = x      obtenido: _t0 = x      OK
[ 5] x + y (sin plegar)                 esperado: _t0 = x + y  obtenido: _t0 = x + y  OK
[ 6] 10 / 0 (no plegar)                 esperado: _t0 = 10 / 0 obtenido: _t0 = 10 / 0 OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la IA, observé que el harness solamente comprueba seis casos principales y no evalúa todas las situaciones posibles. Por ejemplo, no prueba directamente las identidades de multiplicación por uno y por cero, ni los casos de desbordamiento de enteros.
También identifiqué que la función strtol permite espacios en blanco al inicio de una cadena, por lo que sería necesario agregar una validación adicional para aceptar únicamente números enteros completos sin espacios ni caracteres adicionales.
Para prevenir errores importantes, el código contempla la división entre cero, el desbordamiento de operaciones y conserva la instrucción original cuando no puede realizar el plegado. Las seis pruebas del harness se ejecutaron correctamente en ambas versiones.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
