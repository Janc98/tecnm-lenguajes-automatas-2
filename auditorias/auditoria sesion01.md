# Bitácora de Requerimientos y Auditoría IA — Sesión 01
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Inversión de buffer in-place (char*)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> 
Al realizar la inversión del buffer manualmente, identifiqué que debía trabajar directamente con punteros y posiciones de la cadena. La principal dificultad fue encontrar el último carácter sin contar el terminador nulo \0. También tuve que controlar que los índices no se salieran de los límites de memoria. Para resolverlo utilicé dos índices, uno al inicio y otro al final, intercambiando los caracteres mediante una variable auxiliar. La función modifica la misma cadena sin utilizar memoria adicional.
---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Punteros, O(n) tiempo, O(1) memoria |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en C.

Necesito implementar una función llamada invertir que reciba
una cadena de caracteres mediante un puntero char *s.

La función debe invertir los caracteres directamente en el
buffer original, sin utilizar memoria dinámica ni arreglos
auxiliares.

Requisitos:
1. Respetar la firma void invertir(char *s).
2. Manejar correctamente un puntero NULL.
3. Manejar cadenas vacías y de un solo carácter.
4. Utilizar dos índices para recorrer la cadena.
5. Intercambiar los caracteres utilizando una variable temporal.
6. No utilizar malloc ni crear copias de la cadena.
7. Mantener complejidad O(n) en tiempo y O(1) en memoria.
8. Entregar código modular y compatible con C99.
9. Explicar brevemente la complejidad del algoritmo.]
```

---

## 3. Código Devuelto por el Modelo
```c
// Código devuelto por el LLM
```
#include "api_sesion01.h"

void invertir(char *s) {
    if (s == NULL) {
        return;
    }

    int i = 0;
    int j = 0;
    char aux;

    while (s[j] != '\0') {
        j++;
    }

    if (j > 0) {
        j--;

        while (i < j) {
            aux = s[i];
            s[i] = s[j];
            s[j] = aux;
            i++;
            j--;
        }
    }
}
---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Valores NULL, buffers vacios y longitudes impares

### Salida del harness
```text
=== AUDITORIA SESION 01 - Inversion de buffer in-place ===
Caso                               esperado:              obtenido:              resultado
[ 1] invertir("HOLA")                   esperado: ALOH         obtenido: ALOH         OK
[ 2] invertir("abc")                    esperado: cba          obtenido: cba          OK
[ 3] invertir("")                       esperado:              obtenido:              OK
[ 4] invertir("AB")                     esperado: BA           obtenido: BA           OK
[ 5] invertir("a")                      esperado: a            obtenido: a            OK
[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion01.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
>
Al revisar la implementación generada por la IA, comprobé que cumpliera con todos los requisitos establecidos. Verifiqué especialmente el manejo de punteros NULL, cadenas vacías y cadenas de diferentes longitudes. No encontré casos borde omitidos, ya que la implementación contempla las situaciones solicitadas. También revisé que utilizara una variable auxiliar para intercambiar los caracteres y que no necesitara memoria adicional.

Finalmente, ejecuté el arnés de pruebas tanto para mi implementación manual como para la implementación de la IA. Ambas obtuvieron un resultado de 6/6 pruebas superadas. Esto me permitió comprobar que las dos soluciones funcionan correctamente con los casos establecidos.



## Caso borde omitido por la IA
Al revisar el código generado por la inteligencia artificial, no encontré ningún caso borde omitido de los solicitados en el contrato. La implementación contempla correctamente el puntero NULL, las cadenas vacías y las cadenas de un solo carácter. También comprobé que funciona con cadenas de longitud par e impar. Por lo tanto, no fue necesario realizar correcciones en la lógica del código, ya que tanto la implementación manual como la de la IA superaron las seis pruebas del arnés, obteniendo un resultado de 6/6.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragoza
**Firma digital:** [Usuario de GitHub]
