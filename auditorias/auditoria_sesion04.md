# Bitácora de Requerimientos y Auditoría IA — Sesión 04
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Anatomía de un Árbol Sintáctico (AST)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul  
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de nodo binario (ASTNode)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*La principal dificultad fue comprender cómo reservar memoria dinámica para cada nodo utilizando malloc() y cómo conectar correctamente los apuntadores izquierdo y derecho. También tuve que tener cuidado al utilizar las estructuras y comprobar que la memoria se reservara correctamente antes de guardar los valores. Otra dificultad fue implementar la evaluación recursiva para recorrer el árbol sin perder el orden de las operaciones.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gramática E -> E + T | T con tipos de nodo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy trabajando en la Sesión 04 de Lenguajes y Autómatas II.
Necesito implementar un Árbol Sintáctico Abstracto (AST) en C,
utilizando la estructura ASTNode definida en api_sesion04.h.

ROL:
Actúa como un ingeniero de software especializado en lenguaje C
y desarrollo de compiladores. Explica el código de manera sencilla.

ESPECIFICACIÓN:
Implementa las siguientes funciones respetando exactamente
las firmas del archivo api_sesion04.h:

ASTNode *crear_numero(int v);
ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der);
int evaluar(ASTNode *n);

Utiliza memoria dinámica para crear los nodos.
La evaluación debe ser recursiva y soportar los operadores
+, - y *.
Comprueba si los apuntadores son NULL antes de utilizarlos.
Si evaluar recibe NULL, debe devolver -1.
No agregues una función main().
El código debe compilar con:
gcc -std=c11 -Wall -Wextra -Werror.

OUTPUT:
Entrega el código completo para ia_sesion04_c.c,
con comentarios y una implementación diferente a la manual.
Incluye una breve explicación de su funcionamiento
y de la complejidad temporal y espacial.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion04.h"
#include <stdlib.h>

/* Reserva un nodo numerico */
ASTNode *crear_numero(int v) {
    ASTNode *nuevo = calloc(1, sizeof(ASTNode));

    if (nuevo != NULL) {
        nuevo->tipo = NODO_NUM;
        nuevo->valor = v;
    }

    return nuevo;
}

/* Enlaza los dos nodos de una operacion */
ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der) {
    ASTNode *nuevo = calloc(1, sizeof(ASTNode));

    if (nuevo != NULL) {
        nuevo->tipo = NODO_BINOP;
        nuevo->op = op;
        nuevo->izq = izq;
        nuevo->der = der;
    }

    return nuevo;
}

/* Obtiene el valor representado por el arbol */
int evaluar(ASTNode *n) {
    int a;
    int b;

    if (n == NULL) {
        return -1;
    }

    if (n->tipo == NODO_NUM) {
        return n->valor;
    }

    a = evaluar(n->izq);
    b = evaluar(n->der);

    if (n->op == '+') {
        return a + b;
    } else if (n->op == '-') {
        return a - b;
    } else if (n->op == '*') {
        return a * b;
    } else if (n->op == '/') {
        return (b != 0) ? a / b : -1;
    }

    return -1;
}

/* Destruye todos los nodos del arbol */
void liberar_arbol(ASTNode *n) {
    if (n != NULL) {
        liberar_arbol(n->izq);
        liberar_arbol(n->der);
        free(n);
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Árbol vacío y evaluación de nodo raíz

### Salida del harness
```text
=== AUDITORIA SESION 04 - Nodo binario de AST ===
Caso                               esperado:              obtenido:              resultado
[ 1] evaluar(nodo 42)               esperado: 42           obtenido: 42           OK
[ 2] evaluar(3+5*2)                 esperado: 13           obtenido: 13           OK
[ 3] hijo izquierdo de (* 5 2)      esperado: 5            obtenido: 5            OK
[ 4] operador guardado              esperado: *            obtenido: *            OK
[ 5] evaluar(NULL)                  esperado: -1           obtenido: -1           OK
[ 6] evaluar(10-4)                  esperado: 6            obtenido: 6            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
La IA no consideró explícitamente todos los casos borde, principalmente cuando se recibe un operador desconocido o cuando alguno de los hijos de un nodo binario es NULL. Aunque la función evaluar() ya comprueba si el nodo recibido es NULL y devuelve -1, agregué estas situaciones como aspectos que deben revisarse para evitar errores durante la evaluación. También se comprobó que liberar_arbol() tenga una condición para no intentar liberar un apuntador NULL. Finalmente, ejecuté el harness y comprobé que tanto el Artesano como el Científico obtuvieron 6 de 6 pruebas satisfactorias.

## Caso borde omitido por la IA
La IA no consideró explícitamente el caso en el que un nodo binario recibe uno de sus hijos como NULL. Esto podría provocar una evaluación incorrecta de la expresión. Para evitar problemas, se debe comprobar que ambos hijos existan antes de realizar la operación. También es importante validar los operadores desconocidos.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
