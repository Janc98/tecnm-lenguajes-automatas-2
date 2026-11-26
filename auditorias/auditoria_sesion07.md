# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Sistema de Tipos Primitivos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Chequeo de compatibilidad de tipos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar el código manualmente, la principal dificultad fue implementar la evaluación recursiva de los árboles de expresiones aritméticas. Tuve que tener cuidado al evaluar primero los nodos hijos y después realizar la operación del nodo principal. También identifiqué que era necesario comprobar si el divisor era cero antes de realizar una división o módulo, para evitar errores durante la ejecución. Otro detalle importante fue controlar los errores y evitar que el programa continuara evaluando cuando encontraba un nodo inválido.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Matriz de compatibilidad estática |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en
compiladores y lenguaje C.

Necesito implementar una función llamada evaluar()
para evaluar expresiones aritméticas representadas
mediante un árbol sintáctico abstracto (AST).

Requisitos:
1. Respetar la firma int evaluar(ASTNode *n, int *error).
2. Evaluar recursivamente los nodos del árbol.
3. Soportar las operaciones suma, resta, multiplicación,
   división y módulo.
4. Detectar división entre cero y establecer error = 1.
5. Detectar nodos NULL o inválidos y establecer error = 2.
6. No realizar divisiones ni módulos antes de comprobar
   que el divisor sea diferente de cero.
7. Propagar los errores de los nodos hijos hacia los padres.
8. No continuar evaluando si ya existe un error.
9. Utilizar código compatible con C99/C11 y GCC.
10. No incluir main(), porque el harness proporciona
    su propia función principal.

Entrega únicamente el código modular y asegúrate de
que compile con -Wall -Wextra -Werror.]
```

---

## 3. Código Devuelto por el Modelo
```c

#include "api_sesion07.h"

int evaluar(ASTNode *n, int *error) {
    int izquierdo;
    int derecho;

    if (error == NULL) {
        return 0;
    }

    if (n == NULL) {
        *error = 2;
        return 0;
    }

    if (n->tipo == NODO_NUM) {
        *error = 0;
        return n->valor;
    }

    if (n->tipo != NODO_BINOP) {
        *error = 2;
        return 0;
    }

    izquierdo = evaluar(n->izq, error);

    if (*error != 0) {
        return 0;
    }

    derecho = evaluar(n->der, error);

    if (*error != 0) {
        return 0;
    }

    switch (n->op) {
        case '+':
            *error = 0;
            return izquierdo + derecho;

        case '-':
            *error = 0;
            return izquierdo - derecho;

        case '*':
            *error = 0;
            return izquierdo * derecho;

        case '/':
            if (derecho == 0) {
                *error = 1;
                return 0;
            }
            *error = 0;
            return izquierdo / derecho;

        case '%':
            if (derecho == 0) {
                *error = 1;
                return 0;
            }
            *error = 0;
            return izquierdo % derecho;

        default:
            *error = 2;
            return 0;
    }
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Rechazo de asignaciones incompatibles

### Salida del harness
```text
=== AUDITORIA SESION 07 - Evaluador recursivo con deteccion de errores ===

[ 1] (2+3)*4                            esperado: 20   obtenido: 20   OK
[ 2] 7/0 -> error 1                     esperado: 1    obtenido: 1    OK
[ 3] 7%0 -> error 1                     esperado: 1    obtenido: 1    OK
[ 4] arbol NULL -> error 2              esperado: 2    obtenido: 2    OK
[ 5] 10-2-3                             esperado: 5    obtenido: 5    OK
[ 6] (8/2)+1                            esperado: 5    obtenido: 5    OK

RESULTADO: 6/6 pruebas superadas

TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
La IA no contemplaba inicialmente todos los casos de error que podían presentarse durante la evaluación de las expresiones. Revisé que una división o módulo entre cero debía detectarse antes de realizar la operación. También comprobé que, cuando un nodo hijo presenta un error, el nodo padre no debe continuar evaluándose. Para solucionarlo, agregué condiciones que revisan el valor de la variable error después de evaluar cada hijo y antes de realizar las operaciones. Finalmente, ejecuté nuevamente el arnés y obtuve las 6 de 6 pruebas superadas.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
