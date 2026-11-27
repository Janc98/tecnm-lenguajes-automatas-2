# Bitácora de Requerimientos y Auditoría IA — Sesión 24
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Sentencia if-else  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Saltos condicionales y evaluación booleana  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la función manualmente, identifiqué que era necesario controlar correctamente el espacio disponible en el arreglo de instrucciones para evitar escribir fuera de sus límites. También tuve que utilizar snprintf para generar las instrucciones de manera segura. Otra dificultad fue identificar qué tipo de salto correspondía a cada operador lógico: IFFALSE para AND y IFTRUE para OR. Finalmente, consideré el caso donde el segundo operando es NULL, porque en esa situación solamente se debe generar un salto.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cortocircuito lógico en expresiones |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[**Contexto:** Estoy trabajando en la Sesión 24 de Lenguajes y Autómatas II. Necesito implementar una función en lenguaje C que genere instrucciones de salto condicional para expresiones lógicas.

**Rol:** Actúa como un ingeniero de software especializado en compiladores y generación de código intermedio.

**Especificación:** Implementa la función `emitir_condicion()` respetando exactamente esta firma:

`int emitir_condicion(char lineas[][64], int max, const char *a, const char *b, char op, const char *etiqueta);`

Requisitos:
- Si el operador es `&`, generar instrucciones `IFFALSE`.
- Si el operador es `|`, generar instrucciones `IFTRUE`.
- Cada operando debe generar su propio salto hacia la etiqueta recibida.
- Si el segundo operando es NULL, generar únicamente un salto.
- Validar los parámetros y evitar accesos fuera de los límites del arreglo.
- Utilizar `snprintf` para generar las instrucciones.
- No incluir la función `main()`.
- El código debe compilar con `gcc -std=c11 -Wall -Wextra -Werror`.

**Output:** Devuelve únicamente el código completo en C, compatible con `api_sesion24.h`.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion24.h"
#include <stdio.h>

int emitir_condicion(char lineas[][64], int max,
                     const char *a, const char *b,
                     char op, const char *etiqueta) {
    int total = 0;
    char *instruccion;

    if (lineas == NULL || a == NULL ||
        etiqueta == NULL || max < 1) {
        return 0;
    }

    switch (op) {
        case '&':
            instruccion = "IFFALSE";
            break;

        case '|':
            instruccion = "IFTRUE";
            break;

        default:
            return 0;
    }

    const char *operandos[2] = {a, b};

    for (int i = 0; i < 2 && i < max; i++) {
        if (operandos[i] == NULL) {
            break;
        }

        snprintf(lineas[total], 64,
                 "%s %s GOTO %s",
                 instruccion,
                 operandos[i],
                 etiqueta);

        total++;
    }

    return total;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Condiciones compuestas con AND y OR

### Salida del harness
```text
=== AUDITORIA SESION 24 - Saltos condicionales con cortocircuito ===
Caso                               esperado:              obtenido:              resultado
[ 1] a && b (primer salto)
     esperado: IFFALSE a GOTO L9
     obtenido: IFFALSE a GOTO L9
     -> OK
[ 2] a && b (segundo salto)
     esperado: IFFALSE b GOTO L9
     obtenido: IFFALSE b GOTO L9
     -> OK
[ 3] a && b (dos saltos)
     esperado: 2            obtenido: 2            OK
[ 4] a || b (primer salto)
     esperado: IFTRUE a GOTO L9
     obtenido: IFTRUE a GOTO L9
     -> OK
[ 5] a || b (segundo salto)
     esperado: IFTRUE b GOTO L9
     obtenido: IFTRUE b GOTO L9
     -> OK
[ 6] condicion simple (b NULL)
     esperado: IFFALSE a GOTO L0
     obtenido: IFFALSE a GOTO L0
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
La IA no consideró explícitamente en su explicación el caso borde donde el segundo operando es NULL. Sin embargo, su implementación sí contempla esta situación mediante una condición que verifica si el operando es NULL antes de generar el segundo salto.
Al revisar el código y ejecutar el harness, comprobé que cuando el segundo operando no existe solamente se genera una instrucción. Por lo tanto, no fue necesario modificar el código, ya que pasó correctamente las seis pruebas. Este caso permitió comprobar que la función puede trabajar tanto con condiciones simples como compuestas.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
