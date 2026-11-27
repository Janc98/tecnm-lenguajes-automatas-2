# Bitácora de Requerimientos y Auditoría IA — Sesión 20
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 2: Analizador Semántico y Scopes  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Tabla de Símbolos y AST  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual tuve dificultad para recorrer correctamente la cadena de ámbitos y comprobar si una variable ya había sido declarada en un nivel más interno. También tuve que cuidar los límites del arreglo de símbolos utilizando AMBITO_MAX, evitar acceder a posiciones con valores NULL y controlar que los nombres repetidos no se contaran varias veces. Lo más complicado fue implementar el shadowing, porque cuando un nombre aparece en varios ámbitos solamente debe considerarse la declaración más cercana al bloque actual.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato de Validación Semántica Integral |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy trabajando en la Sesión 20 de la materia Lenguajes y Autómatas II.
Necesito implementar funciones para resolver nombres y contar símbolos
visibles dentro de una estructura de ámbitos en lenguaje C.

ROL:
Actúa como un ingeniero de software especializado en programación de
sistemas, tablas de símbolos y desarrollo de compiladores.

ESPECIFICACIÓN:
Implementa las siguientes funciones respetando exactamente sus firmas:

int profundidad_de(const Ambito *actual, const char *nombre);
int contar_visibles(const Ambito *actual);

Utiliza la estructura Ambito declarada en api_sesion20.h.

La función profundidad_de debe recorrer todos los ámbitos desde el
actual hasta el padre y devolver la profundidad donde encuentra el
nombre. El ámbito actual corresponde al nivel 0.

Si el nombre no existe debe devolver -1.
Si algún argumento es NULL debe devolver -2.

La función contar_visibles debe contar todos los nombres diferentes
que pueden observarse desde el ámbito actual.

Debe respetar el shadowing, dando prioridad al símbolo declarado
en el ámbito más interno. No debe contar nombres repetidos.
También debe evitar accesos fuera de los límites del arreglo
AMBITO_MAX y comprobar los valores NULL.

El código debe ser compatible con GCC y las opciones:
-std=c11 -Wall -Wextra -Werror

No incluyas main(), porque el programa será ejecutado mediante
un harness externo.

OUTPUT:
Devuelve únicamente el código completo en C, incluyendo los
encabezados necesarios y comentarios breves que expliquen
las funciones implementadas.]
```

---

## 3. Código Devuelto por el Modelo
```c
/*
 * ia_sesion20_c.c
 * Fase 2: El Cientifico
 * Resolucion de nombres y visibilidad.
 */

#include "api_sesion20.h"
#include <string.h>

/* Comprueba si un ambito contiene determinado nombre */
static int buscar_simbolo(const Ambito *a, const char *nombre) {

    int i;

    if (a == NULL || nombre == NULL) {
        return 0;
    }

    for (i = 0; i < a->total && i < AMBITO_MAX; i++) {

        if (a->simbolos[i] != NULL &&
            strcmp(a->simbolos[i], nombre) == 0) {
            return 1;
        }
    }

    return 0;
}

int profundidad_de(const Ambito *actual, const char *nombre) {

    int nivel = 0;

    if (actual == NULL || nombre == NULL) {
        return -2;
    }

    for (; actual != NULL; actual = actual->padre) {

        if (buscar_simbolo(actual, nombre)) {
            return nivel;
        }

        nivel++;
    }

    return -1;
}

int contar_visibles(const Ambito *actual) {

    const Ambito *a;
    const Ambito *b;
    int i;
    int j;
    int contador = 0;
    int encontrado;

    if (actual == NULL) {
        return -2;
    }

    for (a = actual; a != NULL; a = a->padre) {

        for (i = 0; i < a->total && i < AMBITO_MAX; i++) {

            if (a->simbolos[i] == NULL) {
                continue;
            }

            encontrado = 0;

            /* Buscar si el nombre ya fue declarado antes */
            for (b = actual; b != a && !encontrado; b = b->padre) {

                if (buscar_simbolo(b, a->simbolos[i])) {
                    encontrado = 1;
                }
            }

            /* Comprobar nombres repetidos en el mismo ambito */
            for (j = 0; j < i && !encontrado; j++) {

                if (a->simbolos[j] != NULL &&
                    strcmp(a->simbolos[j], a->simbolos[i]) == 0) {
                    encontrado = 1;
                }
            }

            if (!encontrado) {
                contador++;
            }
        }
    }

    return contador;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Casos complejos de shadowing y ámbito

### Salida del harness
```text
=== AUDITORIA SESION 20 - Resolucion de nombres y visibilidad ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre en el ambito actual         esperado: 0            obtenido: 0            OK
[ 2] nombre dos niveles arriba          esperado: 2            obtenido: 2            OK
[ 3] shadowing gana el interno          esperado: 0            obtenido: 0            OK
[ 4] nombre no visible                  esperado: -1           obtenido: -1           OK
[ 5] visibles sin repetir               esperado: 3            obtenido: 3            OK
[ 6] ambito NULL                        esperado: -2           obtenido: -2           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la IA identifiqué que no contemplaba explícitamente todos los casos de nombres repetidos dentro del mismo ámbito ni situaciones donde existieran valores NULL en las posiciones del arreglo de símbolos. Aunque el código revisaba estos valores, el harness solamente probaba seis escenarios básicos y no comprobaba todas las combinaciones posibles.
También observé que era importante validar el límite AMBITO_MAX para evitar recorrer posiciones fuera del arreglo. Para comprobar el funcionamiento, revisé los ciclos y las condiciones utilizadas para recorrer los ámbitos y comparar los nombres. Las pruebas del harness finalizaron correctamente con 6 de 6 casos superados.
Como mejora, sería conveniente agregar pruebas adicionales con símbolos NULL, nombres duplicados dentro de un mismo ámbito y arreglos con el máximo de elementos permitido.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
