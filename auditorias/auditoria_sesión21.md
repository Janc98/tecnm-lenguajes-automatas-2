# Bitácora de Requerimientos y Auditoría IA — Sesión 21
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Fundamentos de Código de 3 Direcciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Representación en Cuádruplos (op,a,b,r)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual identifiqué que era necesario respetar el formato exacto de los cuádruplos y controlar correctamente el uso de los buffers de memoria. Una dificultad fue manejar los argumentos que podían recibir valores NULL, ya que debían representarse con un guion bajo (_). También tuve que cuidar que el contador de temporales se incrementara después de utilizar su valor, para generar nombres únicos como _t0, _t1 y _t2. Para evitar escribir fuera de los límites del buffer utilicé la función snprintf().*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Generación de temporales únicos (_t0..) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[```text
CONTEXTO:
Estoy desarrollando una práctica de Lenguajes y Autómatas II sobre los fundamentos del Código de Tres Direcciones (TAC). Necesito implementar dos funciones en lenguaje C para generar temporales únicos y representar instrucciones mediante cuádruplos.

ROL:
Actúa como un ingeniero de software especializado en programación en C, sistemas y desarrollo de compiladores.

ESPECIFICACIÓN:
Desarrolla las siguientes funciones respetando exactamente sus firmas:

void nuevo_temporal(char *dst, int cap, int *contador);

void emitir_cuadruplo(char *dst, int cap,
                      const char *op,
                      const char *a1,
                      const char *a2,
                      const char *res);

REQUISITOS:
1. Generar nombres de temporales comenzando desde _t0.
2. Incrementar el contador después de utilizarlo.
3. Permitir generar temporales únicos como _t0, _t7 y _t10.
4. Representar las instrucciones con el formato (op, a1, a2, res).
5. Si algún argumento es NULL, sustituirlo por un guion bajo (_).
6. Escribir directamente en el buffer recibido.
7. Validar los argumentos y la capacidad del buffer.
8. No utilizar memoria dinámica ni variables globales.
9. Utilizar snprintf() para controlar las escrituras.
10. No incluir la función main().
11. El código debe compilar sin advertencias con:
gcc -std=c11 -Wall -Wextra -Werror

OUTPUT:
Entrega el código completo y modular para ia_sesion21_c.c, incluyendo las cabeceras necesarias, comentarios breves y una explicación de la complejidad de las funciones.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion21.h"
#include <stdio.h>
#include <stddef.h>

/* Devuelve "_" cuando el argumento es NULL */
static const char *valor_seguro(const char *texto) {
    if (texto == NULL) {
        return "_";
    }

    return texto;
}

void nuevo_temporal(char *dst, int cap, int *contador) {
    int numero;

    if (dst == NULL || contador == NULL || cap <= 0) {
        return;
    }

    numero = *contador;
    (void)snprintf(dst, (size_t)cap, "_t%d", numero);
    *contador = numero + 1;
}

void emitir_cuadruplo(char *dst, int cap,
                      const char *op,
                      const char *a1,
                      const char *a2,
                      const char *res) {

    if (dst == NULL || cap <= 0) {
        return;
    }

    (void)snprintf(dst, (size_t)cap,
                   "(%s, %s, %s, %s)",
                   valor_seguro(op),
                   valor_seguro(a1),
                   valor_seguro(a2),
                   valor_seguro(res));
}
```
---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones aritméticas compuestas

### Salida del harness
```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
[ 1] nuevo_temporal(_t0)       esperado: _t0 obtenido: _t0 OK
[ 2] nuevo_temporal(_t7)       esperado: _t7 obtenido: _t7 OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple
     esperado: (=, a, _, x)
     obtenido: (=, a, _, x)
     -> OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _)
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
La IA no contempló directamente el caso de utilizar un buffer con capacidad insuficiente para almacenar completamente el nombre del temporal o el cuádruplo. Aunque utilizó snprintf() para evitar escribir fuera de los límites de memoria, no verificó su valor de retorno para saber si el texto había sido truncado. Identifiqué esta situación al revisar el código y analizar qué sucedería si el buffer fuera demasiado pequeño. Como mejora, se puede comprobar el valor devuelto por snprintf() y agregar pruebas con diferentes capacidades de buffer para detectar resultados incompletos.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
