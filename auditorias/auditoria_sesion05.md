# Bitácora de Requerimientos y Auditoría IA — Sesión 05
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Recorridos del AST y Notación Polaca  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul  
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Impresión en Postorden / RPN  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*Al realizar la función manualmente, la principal dificultad fue controlar el recorrido recursivo del árbol y respetar el orden correcto de postorden, que consiste en visitar primero el hijo izquierdo, después el derecho y finalmente la raíz. También tuve que cuidar la posición del buffer para evitar escribir fuera de su capacidad, agregar los espacios entre los números y operadores y colocar correctamente el carácter nulo '\0' al finalizar la cadena.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariantes de recorrido recursivo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Contexto:
Estoy trabajando en la Sesión 05 de Lenguajes y Autómatas II.
Necesito implementar una función en lenguaje C que realice
el recorrido postorden de un Árbol de Sintaxis Abstracta (AST)
y genere su representación en Notación Polaca Inversa (RPN).

Rol:
Actúa como un ingeniero de software especializado en
programación en C y desarrollo de compiladores.

Especificación:
Implementa la función:
void postorden(ASTNode *n, char *buf, int cap);

Debes cumplir las siguientes condiciones:
1. Recorrer el árbol en orden izquierdo, derecho y raíz.
2. Convertir los nodos numéricos a texto.
3. Agregar los operadores después de sus operandos.
4. Separar los elementos con un solo espacio.
5. No dejar espacios al final de la cadena.
6. Manejar correctamente un árbol NULL.
7. Respetar la capacidad del buffer y reservar espacio
   para el carácter terminador '\0'.
8. No utilizar printf dentro de la función.
9. No modificar las estructuras ni las firmas del archivo
   api_sesion05.h.
10. No incluir la función main().
11. Evitar advertencias con -Wall -Wextra -Werror.

Output:
Entrega el código completo y modular en C, listo para
compilar con el harness_sesion05_c.c.
Explica brevemente la complejidad temporal y espacial.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion05.h"
#include <stdio.h>

void agregar(char *buf, int cap, int *pos, const char *texto) {
    int i = 0;

    if (*pos > 0 && *pos < cap - 1) {
        buf[*pos] = ' ';
        (*pos)++;
    }

    while (texto[i] != '\0' && *pos < cap - 1) {
        buf[*pos] = texto[i];
        (*pos)++;
        i++;
    }

    buf[*pos] = '\0';
}

void convertir(ASTNode *n, char *buf, int cap, int *pos) {
    char texto[20];

    if (n == NULL || *pos >= cap - 1) {
        return;
    }

    if (n->tipo == NODO_BINOP) {
        convertir(n->izq, buf, cap, pos);
        convertir(n->der, buf, cap, pos);

        texto[0] = n->op;
        texto[1] = '\0';
        agregar(buf, cap, pos, texto);
    } else if (n->tipo == NODO_NUM) {
        snprintf(texto, sizeof(texto), "%d", n->valor);
        agregar(buf, cap, pos, texto);
    }
}

void postorden(ASTNode *n, char *buf, int cap) {
    int pos = 0;

    if (buf == NULL || cap <= 0) {
        return;
    }

    buf[0] = '\0';
    convertir(n, buf, cap, &pos);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Validación contra desbordamiento de pila

### Salida del harness
```text
=== AUDITORIA SESION 05 - Recorrido postorden y RPN ===
Caso                               esperado:              obtenido:              resultado
[ 1] hoja 42                            esperado: 42           obtenido: 42           OK
[ 2] 3+(5*2)                            esperado: 3 5 2 * +    obtenido: 3 5 2 * +    OK
[ 3] arbol NULL                         esperado:              obtenido:              OK
[ 4] (1+2)*(3+4)
     esperado: 1 2 + 3 4 + *
     obtenido: 1 2 + 3 4 + *
     -> OK
[ 5] 8-3                                esperado: 8 3 -        obtenido: 8 3 -        OK

RESULTADO: 5/5 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
La IA omitió considerar completamente el caso de un buffer con capacidad insuficiente para almacenar toda la expresión RPN. Para solucionarlo, agregué condiciones que verifican el espacio disponible antes de escribir cada carácter, evitando escribir fuera de los límites de memoria. También comprobé que, cuando el árbol es NULL, la función genere una cadena vacía y que el buffer siempre termine correctamente con '\0'.

## Caso borde omitido por la IA
La IA omitió considerar el caso de un buffer con capacidad insuficiente para almacenar toda la expresión RPN. Esto podría provocar que se intenten escribir caracteres fuera de los límites de memoria. Lo solucioné agregando validaciones para comprobar el espacio disponible antes de escribir, asegurando también que el árbol NULL produzca una cadena vacía y que el buffer siempre termine correctamente con '\0'.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
