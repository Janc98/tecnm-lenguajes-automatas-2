# Bitácora de Requerimientos y Auditoría IA — Sesión 25
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Bucles while y for  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24531399  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de ciclo en instrucciones TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Durante esta fase realicé manualmente las funciones emitir_while() y emitir_for() en lenguaje C. Tuve que organizar las instrucciones TAC, controlar las etiquetas de inicio y fin y cuidar que no se repitieran entre ciclos. También identifiqué la importancia de respetar el límite de 64 caracteres por línea y actualizar correctamente el contador de posiciones. La parte que requirió más atención fue colocar la actualización del iterador antes del salto GOTO para que el ciclo pudiera regresar al inicio.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Garantía de actualización de iterador |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy trabajando en la Sesión 25 de Lenguajes y Autómatas II. Necesito generar código TAC para representar ciclos while y for utilizando lenguaje C.

ROL:
Actúa como un ingeniero de software especializado en programación en C y desarrollo de compiladores.

ESPECIFICACIÓN:
Implementa las funciones emitir_while() y emitir_for() respetando las firmas de api_sesion25.h.

El while debe generar una etiqueta de inicio, una condición IFFALSE, un salto GOTO al inicio y una etiqueta de fin.

El for debe generar una inicialización, etiqueta de inicio, condición IFFALSE, actualización del iterador, salto GOTO y etiqueta de fin.

Cada ciclo debe consumir dos etiquetas diferentes. Si inicio o post es NULL, la instrucción correspondiente debe omitirse. También debes respetar los límites del arreglo y evitar accesos fuera de memoria.

OUTPUT:
Entrega código C modular, sin main(), compatible con GCC y las opciones -std=c11 -Wall -Wextra -Werror. Incluye una explicación breve de la complejidad.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion25.h"
#include <stdio.h>

static void agregar(char lineas[][64], int max, int *n,
                    const char *formato, int etiqueta,
                    const char *texto) {
    if (*n >= max) {
        return;
    }

    if (texto != NULL) {
        snprintf(lineas[*n], 64, formato, texto, etiqueta);
    } else {
        snprintf(lineas[*n], 64, formato, etiqueta);
    }

    (*n)++;
}

void emitir_while(char lineas[][64], int max, int *n,
                  const char *cond, int *etiquetas) {
    int primero = *etiquetas;
    int salida = primero + 1;

    *etiquetas = salida + 1;

    agregar(lineas, max, n, "L%d:", primero, NULL);

    if (*n < max) {
        snprintf(lineas[*n], 64, "IFFALSE %s GOTO L%d",
                 cond, salida);
        (*n)++;
    }

    agregar(lineas, max, n, "GOTO L%d", primero, NULL);
    agregar(lineas, max, n, "L%d:", salida, NULL);
}

void emitir_for(char lineas[][64], int max, int *n,
                const char *inicio, const char *cond,
                const char *post, int *etiquetas) {
    int primero = *etiquetas;
    int salida = primero + 1;

    *etiquetas += 2;

    if (inicio != NULL && *n < max) {
        snprintf(lineas[*n], 64, "%s", inicio);
        (*n)++;
    }

    agregar(lineas, max, n, "L%d:", primero, NULL);

    if (*n < max) {
        snprintf(lineas[*n], 64, "IFFALSE %s GOTO L%d",
                 cond, salida);
        (*n)++;
    }

    if (post != NULL && *n < max) {
        snprintf(lineas[*n], 64, "%s", post);
        (*n)++;
    }

    agregar(lineas, max, n, "GOTO L%d", primero, NULL);
    agregar(lineas, max, n, "L%d:", salida, NULL);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Bucles anidados y condiciones de parada

### Salida del harness
```text
=== AUDITORIA SESION 25 - Esqueleto TAC de while y for ===
Caso                               esperado:              obtenido:              resultado
[ 1] while: marca de inicio             esperado: L0:          obtenido: L0:          OK
[ 2] while: condicion
     esperado: IFFALSE a < b GOTO L1
     obtenido: IFFALSE a < b GOTO L1
     -> OK
[ 3] dos ciclos sin colision            esperado: L2:          obtenido: L2:          OK
[ 4] for: inicializacion                esperado: i = 0        obtenido: i = 0        OK
[ 5] for: actualizacion                 esperado: i = i + 1    obtenido: i = i + 1    OK
[ 6] for sin post (NULL)                esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Durante la revisión del código generado por la inteligencia artificial, observé que las funciones emitir_while() y emitir_for() cumplen con las instrucciones solicitadas y pasan las seis pruebas del harness.
Sin embargo, considero que el código podría mejorarse en algunos aspectos. Por ejemplo, sería conveniente validar que los parámetros recibidos no sean NULL cuando sea necesario, comprobar que el contador de instrucciones no tenga valores incorrectos y verificar que las etiquetas no se desborden.
También observé que la función auxiliar agregar() facilita la escritura de las etiquetas y los saltos, aunque su implementación podría simplificarse porque solamente se utiliza para generar instrucciones con números de etiqueta.
Una situación que no se comprueba directamente en las seis pruebas es qué sucede cuando el arreglo de instrucciones llega a su capacidad máxima o cuando se reciben parámetros inválidos.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
