# Bitácora de Requerimientos y Auditoría IA — Sesión 18
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arreglos y Verificación de Límites  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de tamaño de arreglo en memoria  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar el código manualmente, tuve que identificar el tamaño que ocupa cada tipo de dato utilizando la instrucción sizeof. Una de las dificultades fue comprender que no todos los tipos de datos ocupan la misma cantidad de memoria, por lo que era necesario multiplicar el tamaño correspondiente por el número de elementos. También tuve que validar que la cantidad de elementos no fuera negativa y que los índices no sobrepasaran los límites del arreglo.

*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Indexación entera no negativa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en lenguaje C
y desarrollo de compiladores.

CONTEXTO:
Estoy trabajando en la sesión 18 de Lenguajes y Autómatas II.
Necesito desarrollar funciones para calcular el tamaño de memoria
de arreglos y verificar sus límites estáticos.

ROL:
Actúa como un programador experto en C, proporcionando una
solución sencilla, clara y adecuada para un estudiante.

ESPECIFICACIÓN:
Implementa las siguientes funciones respetando exactamente
sus firmas:

long bytes_arreglo(const char *tipo, int elementos);
int indice_en_rango(int indice, int elementos);

La función bytes_arreglo debe reconocer los tipos:
char, short, int, float, double y puntero.

Debe multiplicar el tamaño del tipo de dato por la cantidad
de elementos utilizando sizeof.

Si el tipo es NULL, desconocido o la cantidad de elementos
es negativa, debe devolver -1.
Si la cantidad de elementos es cero, debe devolver 0.

La función indice_en_rango debe devolver 1 cuando el índice
sea válido y 0 cuando esté fuera de los límites.

Debe rechazar índices negativos, índices iguales o mayores
que la cantidad de elementos y arreglos sin elementos.

RESTRICCIONES:
Utiliza C11 y bibliotecas estándar.
Incluye api_sesion18.h y string.h.
No agregues la función main().
No modifiques las firmas originales.
El código debe compilar con:
gcc -std=c11 -Wall -Wextra -Werror

OUTPUT:
Entrega el código completo, modular y comentado de ambas
funciones, listo para guardarse en ia_sesion18_c.c.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion18.h"
#include <string.h>

long bytes_arreglo(const char *tipo, int elementos) {

    long tamano = 0;

    if (tipo == NULL || elementos < 0) {
        return -1;
    }

    if (strcmp(tipo, "char") == 0) {
        tamano = sizeof(char);
    }
    else if (strcmp(tipo, "short") == 0) {
        tamano = sizeof(short);
    }
    else if (strcmp(tipo, "int") == 0) {
        tamano = sizeof(int);
    }
    else if (strcmp(tipo, "float") == 0) {
        tamano = sizeof(float);
    }
    else if (strcmp(tipo, "double") == 0) {
        tamano = sizeof(double);
    }
    else if (strcmp(tipo, "puntero") == 0) {
        tamano = sizeof(void *);
    }
    else {
        return -1;
    }

    return tamano * elementos;
}

int indice_en_rango(int indice, int elementos) {

    return (elementos > 0 &&
            indice >= 0 &&
            indice < elementos) ? 1 : 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Límites estáticos y dimensiones inválidas

### Salida del harness
```text
=== AUDITORIA SESION 18 - Tamano de arreglo y limites ===
Caso                               esperado:              obtenido:              resultado
[ 1] int[10]                       esperado: 40           obtenido: 40           OK
[ 2] double[3]                     esperado: 24           obtenido: 24           OK
[ 3] int[-4]                       esperado: -1           obtenido: -1           OK
[ 4] indice 0 de 5                 esperado: 1            obtenido: 1            OK
[ 5] indice 5 de 5                 esperado: 0            obtenido: 0            OK
[ 6] indice negativo               esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la inteligencia artificial, observé que el harness no incluye pruebas para todos los casos posibles, como un arreglo con cero elementos, un tipo de dato desconocido o un índice igual a cero cuando la cantidad de elementos también es cero.
La función de IA ya contempla estas situaciones mediante sus condiciones de validación, pero no todas fueron comprobadas directamente por el harness original. Por ejemplo, cuando se utiliza indice_en_rango(0, 0), el resultado debe ser 0 porque el arreglo no contiene elementos.
Para comprobar estos casos adicionales, revisé las condiciones del código y confirmé que rechazan correctamente los índices inválidos y las dimensiones negativas. Considero que agregar pruebas para estos escenarios permite tener mayor seguridad sobre el funcionamiento de las funciones.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragoza
**Firma digital:** [Usuario de GitHub]
