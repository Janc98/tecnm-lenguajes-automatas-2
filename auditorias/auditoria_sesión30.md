# Bitácora de Requerimientos y Auditoría IA — Sesión 30
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 3: Generador de TAC End-to-End  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Traductor completo de AST a TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Durante la implementación manual identifiqué que uno de los principales problemas fue realizar el recorrido recursivo del árbol de sintaxis abstracta, ya que era necesario reconocer correctamente los nodos numéricos, las variables y los operadores binarios. También tuve que cuidar el tamaño de los arreglos utilizados para almacenar las instrucciones TAC y el resultado final, evitando escribir información fuera de los límites de memoria. Otra dificultad fue controlar la generación de temporales consecutivos y evitar instrucciones innecesarias cuando una operación podía resolverse mediante el plegado de constantes.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato de Emisión Lineal Optimizada |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero experto en lenguaje C y desarrollo de
compiladores.

CONTEXTO:
Estoy desarrollando un generador de código intermedio para
la materia Lenguajes y Autómatas II. Necesito transformar un
árbol de sintaxis abstracta (AST) en código de tres direcciones
(TAC).

ROL:
Actúa como un programador especializado en compiladores,
estructuras de datos y optimización de código.

ESPECIFICACIÓN:
Implementa la función:

int generar_tac_opt(
    const Nodo *raiz,
    char lineas[][64],
    int max,
    char *resultado,
    int cap
);

Debes cumplir lo siguiente:

1. Reconocer nodos numéricos (N), variables (V) y operadores
   binarios (B).
2. Recorrer el AST recursivamente.
3. Aplicar Constant Folding cuando ambos operandos sean
   constantes.
4. Eliminar las identidades x + 0 y x * 1.
5. Resolver x * 0 sin generar una operación.
6. No plegar divisiones entre cero.
7. Generar temporales consecutivos desde _t0.
8. Guardar el resultado final en el parámetro resultado.
9. Controlar los límites de memoria de los arreglos.
10. Regresar la cantidad de instrucciones generadas.
11. Respetar exactamente la firma de la función.
12. No incluir main().

OUTPUT:
Entrega el código completo en C, modular, comentado y
compatible con GCC utilizando:

gcc -std=c11 -Wall -Wextra -Werror

Explica brevemente la complejidad temporal y espacial
de la implementación.]
```

---

## 3. Código Devuelto por el Modelo
```c

#include "api_sesion30.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <limits.h>
#include <errno.h>

typedef struct {
    int es_constante;
    long numero;
    char nombre[32];
} Resultado;

static int guardar(char *destino, int capacidad, const char *origen)
{
    int longitud;

    if (destino == NULL || origen == NULL || capacidad <= 0) {
        return 0;
    }

    longitud = snprintf(destino, (size_t)capacidad, "%s", origen);

    return longitud >= 0 && longitud < capacidad;
}

static int leer_numero(const char *texto, long *numero)
{
    char *fin;
    long valor;

    if (texto == NULL || numero == NULL || texto[0] == '\0') {
        return 0;
    }

    errno = 0;
    valor = strtol(texto, &fin, 10);

    if (errno == ERANGE || *fin != '\0') {
        return 0;
    }

    *numero = valor;
    return 1;
}

static int operar(char op, long a, long b, long *resultado)
{
    if (resultado == NULL) {
        return 0;
    }

    switch (op) {
        case '+':
            if ((b > 0 && a > LONG_MAX - b) ||
                (b < 0 && a < LONG_MIN - b)) {
                return 0;
            }
            *resultado = a + b;
            return 1;

        case '-':
            if ((b < 0 && a > LONG_MAX + b) ||
                (b > 0 && a < LONG_MIN + b)) {
                return 0;
            }
            *resultado = a - b;
            return 1;

        case '*':
            if (a != 0 && b != 0) {
                if (a == -1 && b == LONG_MIN) {
                    return 0;
                }
                if (b == -1 && a == LONG_MIN) {
                    return 0;
                }
                if (a > 0 && b > 0 && a > LONG_MAX / b) {
                    return 0;
                }
                if (a > 0 && b < 0 && b < LONG_MIN / a) {
                    return 0;
                }
                if (a < 0 && b > 0 && a < LONG_MIN / b) {
                    return 0;
                }
                if (a < 0 && b < 0 && a < LONG_MAX / b) {
                    return 0;
                }
            }
            *resultado = a * b;
            return 1;

        case '/':
            if (b == 0 || (a == LONG_MIN && b == -1)) {
                return 0;
            }
            *resultado = a / b;
            return 1;

        default:
            return 0;
    }
}

static int generar(
    const Nodo *nodo,
    char lineas[][64],
    int max,
    int *cantidad,
    int *temporales,
    Resultado *salida)
{
    Resultado izquierda;
    Resultado derecha;
    long valor;
    char temporal[32];
    char instruccion[64];

    if (nodo == NULL || cantidad == NULL ||
        temporales == NULL || salida == NULL) {
        return 0;
    }

    if (nodo->tipo == 'n') {
        if (!leer_numero(nodo->valor, &valor)) {
            return 0;
        }

        salida->es_constante = 1;
        salida->numero = valor;
        salida->nombre[0] = '\0';
        return 1;
    }

    if (nodo->tipo == 'v') {
        salida->es_constante = 0;
        salida->numero = 0;

        return guardar(
            salida->nombre,
            (int)sizeof(salida->nombre),
            nodo->valor
        );
    }

    if (nodo->tipo != 'o' ||
        !generar(nodo->izq, lineas, max, cantidad,
                 temporales, &izquierda) ||
        !generar(nodo->der, lineas, max, cantidad,
                 temporales, &derecha)) {
        return 0;
    }

    if (izquierda.es_constante && derecha.es_constante &&
        operar(nodo->op, izquierda.numero,
               derecha.numero, &valor)) {
        salida->es_constante = 1;
        salida->numero = valor;
        salida->nombre[0] = '\0';
        return 1;
    }

    if (nodo->op == '+' && derecha.es_constante &&
        derecha.numero == 0) {
        *salida = izquierda;
        return 1;
    }

    if (nodo->op == '+' && izquierda.es_constante &&
        izquierda.numero == 0) {
        *salida = derecha;
        return 1;
    }

    if (nodo->op == '*' && derecha.es_constante &&
        derecha.numero == 1) {
        *salida = izquierda;
        return 1;
    }

    if (nodo->op == '*' && izquierda.es_constante &&
        izquierda.numero == 1) {
        *salida = derecha;
        return 1;
    }

    if (nodo->op == '*' &&
        ((izquierda.es_constante && izquierda.numero == 0) ||
         (derecha.es_constante && derecha.numero == 0))) {
        salida->es_constante = 1;
        salida->numero = 0;
        salida->nombre[0] = '\0';
        return 1;
    }

    if (*cantidad >= max || lineas == NULL) {
        return 0;
    }

    if (izquierda.es_constante) {
        if (snprintf(temporal, sizeof(temporal),
                     "%ld", izquierda.numero) >=
            (int)sizeof(temporal)) {
            return 0;
        }
    } else {
        if (!guardar(temporal, (int)sizeof(temporal),
                     izquierda.nombre)) {
            return 0;
        }
    }

    if (derecha.es_constante) {
        if (snprintf(instruccion, sizeof(instruccion),
                     "%ld", derecha.numero) >=
            (int)sizeof(instruccion)) {
            return 0;
        }
    } else {
        if (!guardar(instruccion, (int)sizeof(instruccion),
                     derecha.nombre)) {
            return 0;
        }
    }

    if (snprintf(
            temporal,
            sizeof(temporal),
            "_t%d",
            (*temporales)++) >= (int)sizeof(temporal)) {
        return 0;
    }

    {
        char operando_izq[32];
        char operando_der[32];

        if (!guardar(operando_izq, (int)sizeof(operando_izq),
                     izquierda.es_constante ? "" : izquierda.nombre)) {
            return 0;
        }

        if (!guardar(operando_der, (int)sizeof(operando_der),
                     derecha.es_constante ? "" : derecha.nombre)) {
            return 0;
        }

        if (izquierda.es_constante) {
            (void)snprintf(operando_izq, sizeof(operando_izq),
                           "%ld", izquierda.numero);
        }

        if (derecha.es_constante) {
            (void)snprintf(operando_der, sizeof(operando_der),
                           "%ld", derecha.numero);
        }

        if (snprintf(
                instruccion,
                sizeof(instruccion),
                "%s = %s %c %s",
                temporal,
                operando_izq,
                nodo->op,
                operando_der) >= (int)sizeof(instruccion)) {
            return 0;
        }
    }

    if (!guardar(lineas[*cantidad], 64, instruccion)) {
        return 0;
    }

    (*cantidad)++;

    salida->es_constante = 0;
    salida->numero = 0;

    return guardar(
        salida->nombre,
        (int)sizeof(salida->nombre),
        temporal
    );
}

int generar_tac_opt(
    const Nodo *raiz,
    char lineas[][64],
    int max,
    char *resultado,
    int cap)
{
    int cantidad = 0;
    int temporales = 0;
    Resultado final;

    if (raiz == NULL || resultado == NULL || cap <= 0 || max < 0) {
        return -1;
    }

    if (!generar(raiz, lineas, max, &cantidad,
                 &temporales, &final)) {
        return -1;
    }

    if (final.es_constante) {
        if (snprintf(resultado, (size_t)cap,
                     "%ld", final.numero) >= cap) {
            return -1;
        }
    } else {
        if (snprintf(resultado, (size_t)cap,
                     "%s", final.nombre) >= cap) {
            return -1;
        }
    }

    return cantidad;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Suite de 5 programas complejos

### Salida del harness
```text
=== AUDITORIA SESION 30 - AST a TAC optimizado de punta a punta ===
Caso                               esperado:              obtenido:              resultado
[ 1] x + 2 * 3 (linea)                  esperado: _t0 = x + 6  obtenido: _t0 = x + 6  OK
[ 2] x + 2 * 3 (resultado)              esperado: _t0          obtenido: _t0          OK
[ 3] x * y + 2 * 3
     esperado: _t1 = _t0 + 6
     obtenido: _t1 = _t0 + 6
     -> OK
[ 4] todo constante: sin lineas         esperado: 0            obtenido: 0            OK
[ 5] todo constante: resultado          esperado: 45           obtenido: 45           OK
[ 6] x + 0 (identidad)                  esperado: x            obtenido: x            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Durante la revisión del código identifiqué que un caso importante que debía comprobarse era la división entre cero. Si los dos operandos son constantes, el optimizador normalmente intenta calcular el resultado durante la compilación, pero no debe realizar esta operación cuando el divisor es cero.
Para solucionar este problema se agregó una validación antes de realizar la división. Cuando el divisor es cero, la operación no se pliega y se conserva como una instrucción TAC para que llegue al backend.
También se revisó el manejo de los límites de memoria para evitar que el programa escribiera fuera de los arreglos. Esto permitió mejorar la seguridad de la implementación y comprender que un optimizador no solamente debe generar menos instrucciones, sino también conservar el comportamiento correcto del programa original.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragoza 
**Firma digital:** [Usuario de GitHub]
