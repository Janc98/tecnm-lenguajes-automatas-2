# Bitácora de Requerimientos y Auditoría IA — Sesión 13
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Manejo de Ámbitos (Scope Global/Local)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul  
**Matrícula:** 24531399  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Pila de tablas de símbolos (Stack Scopes)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar el ejercicio manualmente, una de las principales dificultades fue comprender cómo recorrer los ámbitos enlazados para encontrar una variable. También tuve que considerar que los nombres solamente deben registrarse en el ámbito actual y que pueden repetirse en diferentes niveles. Otra dificultad fue controlar el límite máximo de ocho identificadores por ámbito y evitar declaraciones duplicadas. Para resolverlo utilicé ciclos y condiciones que permiten revisar los nombres registrados y subir al ámbito padre cuando no se encuentra el identificador.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Ocultamiento de variables (Shadowing) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy desarrollando una práctica de la materia Lenguajes
y Autómatas II en lenguaje C99. Necesito implementar
el manejo de ámbitos globales y locales mediante una
estructura de ámbitos enlazados.

ROL:
Actúa como un ingeniero de software especializado en
programación de compiladores y estructuras de datos.

ESPECIFICACIÓN:
Implementa las funciones declarar() y buscar() respetando
exactamente las firmas del archivo api_sesion13.h.

La estructura Ambito contiene:
- nombre: nombre del ámbito.
- nombres: arreglo de identificadores.
- total: cantidad de identificadores registrados.
- padre: apuntador al ámbito superior.

La función declarar() debe:
1. Validar los datos recibidos.
2. Insertar nombres únicamente en el ámbito actual.
3. Evitar nombres duplicados dentro del mismo ámbito.
4. Permitir el mismo identificador en otros ámbitos.
5. Devolver 1 cuando inserta correctamente.
6. Devolver 0 cuando el nombre ya existe.
7. Devolver -1 cuando recibe datos inválidos o el
   ámbito está lleno.

La función buscar() debe:
1. Buscar primero en el ámbito actual.
2. Continuar buscando en los ámbitos padre.
3. Respetar el ocultamiento de variables (Shadowing).
4. Devolver el nombre del ámbito donde se encuentra
   el identificador.
5. Devolver NULL cuando el nombre no existe.

RESTRICCIONES:
- Utilizar C99.
- Incluir api_sesion13.h y string.h.
- No modificar las firmas originales.
- No incluir la función main().
- Evitar advertencias con -Wall -Wextra -Werror.
- No utilizar bibliotecas externas.
- Mantener el código modular.

OUTPUT:
Entregar el código completo de ambas funciones,
comentado y organizado, junto con su análisis de
complejidad temporal y espacial.]
```

---

## 3. Código Devuelto por el Modelo
```c
/*
 * ia_sesion13_c.c
 * Fase 2: El Cientifico
 * Manejo de ambitos globales y locales.
 */

#include "api_sesion13.h"
#include <string.h>

int declarar(Ambito *ambito, const char *nombre) {

    if (!ambito || !nombre || !*nombre) {
        return -1;
    }

    if (ambito->total >= AMBITO_MAX || ambito->total < 0) {
        return -1;
    }

    int posicion = 0;

    while (posicion < ambito->total) {

        if (strcmp(ambito->nombres[posicion], nombre) == 0) {
            return 0;
        }

        posicion++;
    }

    ambito->nombres[posicion] = nombre;
    ambito->total += 1;

    return 1;
}

const char *buscar(const Ambito *ambito, const char *nombre) {

    if (ambito == NULL || nombre == NULL || *nombre == '\0') {
        return NULL;
    }

    const Ambito *actual = ambito;

    while (actual) {

        int posicion = 0;

        while (posicion < actual->total &&
               posicion < AMBITO_MAX) {

            if (strcmp(actual->nombres[posicion], nombre) == 0) {
                return actual->nombre;
            }

            posicion++;
        }

        actual = actual->padre;
    }

    return NULL;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Búsqueda jerárquica de identificadores

### Salida del harness
```text
=== AUDITORIA SESION 13 - Pila de ambitos y busqueda jerarquica ===
Caso                               esperado:              obtenido:              resultado
[ 1] declarar(&g, "x")                  esperado: 1            obtenido: 1            OK
[ 2] buscar en el mismo ambito          esperado: global       obtenido: global       OK
[ 3] buscar subiendo al padre           esperado: global       obtenido: global       OK
[ 4] shadowing en el hijo               esperado: bloque       obtenido: bloque       OK
[ 5] tres niveles hasta la raiz         esperado: raiz         obtenido: raiz         OK
[ 6] nombre no declarado                esperado: (nulo)       obtenido: (nulo)       OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la inteligencia artificial, identifiqué que no se contempló completamente el caso de encontrar un apuntador NULL dentro del arreglo de identificadores. La función utiliza strcmp() para comparar los nombres, por lo que si una posición contiene NULL podría provocar un error durante la ejecución. Para prevenir esta situación, se puede agregar una condición que verifique que el apuntador no sea NULL antes de realizar la comparación. Aunque las seis pruebas proporcionadas por el docente fueron superadas, esta revisión permitió identificar una situación adicional que no estaba contemplada en el harness.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
