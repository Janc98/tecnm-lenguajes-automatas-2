# Bitácora de Requerimientos y Auditoría IA — Sesión 15
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Bloques Anidados y Ciclo de Vida  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24531399  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Operaciones Push y Pop de ámbito  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual, tuve que tener cuidado con la reserva y liberación de memoria dinámica. Uno de los principales puntos fue asegurar que el nuevo ámbito conservara correctamente el apuntador hacia su padre. También fue necesario comprobar que no se intentara eliminar el ámbito raíz, ya que este representa el inicio del programa. Utilicé malloc() para reservar memoria y free() para liberar únicamente el ámbito actual, evitando perder la referencia al padre.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Liberación de memoria al cerrar bloque |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy desarrollando una práctica de Lenguajes y Autómatas II
en lenguaje C. Necesito implementar un sistema de ámbitos
anidados utilizando una estructura llamada Ambito, que contiene
un nombre y un apuntador hacia su ámbito padre.

ROL:
Actúa como un ingeniero de software especializado en lenguaje C,
estructuras de datos enlazadas y administración de memoria dinámica.

ESPECIFICACIÓN:
Implementa las siguientes funciones respetando exactamente
las firmas establecidas en api_sesion15.h:

Ambito *push_ambito(Ambito *actual, const char *nombre);
Ambito *pop_ambito(Ambito *actual);

La función push_ambito debe:
1. Comprobar que actual y nombre no sean NULL.
2. Reservar memoria para un nuevo ámbito.
3. Guardar el nombre recibido.
4. Enlazar el nuevo ámbito con su padre.
5. Devolver NULL si ocurre algún error.

La función pop_ambito debe:
1. Comprobar que el ámbito actual no sea NULL.
2. Evitar eliminar el ámbito raíz.
3. Guardar el apuntador al padre antes de liberar memoria.
4. Liberar únicamente el ámbito actual.
5. Devolver el ámbito padre.

El código debe ser compatible con C11 y compilar con:
-Wall -Wextra -Werror.

No incluyas main() ni modifiques las firmas originales.

OUTPUT:
Entrega el código completo para ia_sesion15_c.c,
incluyendo las bibliotecas necesarias y las dos funciones.
Agrega un breve análisis de su funcionamiento y complejidad.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion15.h"
#include <stdlib.h>

Ambito *push_ambito(Ambito *actual, const char *nombre) {
    Ambito *nuevo;

    if (actual == NULL || nombre == NULL) {
        return NULL;
    }

    nuevo = calloc(1, sizeof(Ambito));

    if (nuevo == NULL) {
        return NULL;
    }

    nuevo->padre = actual;
    nuevo->nombre = nombre;

    return nuevo;
}

Ambito *pop_ambito(Ambito *actual) {
    Ambito *anterior;

    if (actual == NULL || actual->padre == NULL) {
        return NULL;
    }

    anterior = actual->padre;

    free(actual);

    return anterior;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Aislamiento de variables fuera de bloque

### Salida del harness
```text
```text
=== AUDITORIA SESION 15 - Push y pop de ambitos ===
[ 1] nombre del ambito nuevo            esperado: if           obtenido: if           OK
[ 2] profundidad con dos push           esperado: 3            obtenido: 3            OK
[ 3] pop devuelve el padre              esperado: 1            obtenido: 1            OK
[ 4] pop sobre la raiz                  esperado: 1            obtenido: 1            OK
[ 5] push con padre NULL                esperado: 1            obtenido: 1            OK
[ 6] profundidad tras pop               esperado: 2            obtenido: 2            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la inteligencia artificial, observé que guarda directamente el apuntador del nombre recibido en lugar de realizar una copia independiente de la cadena. Esto significa que el nombre debe permanecer válido mientras exista el ámbito. Si la memoria original deja de estar disponible, podría producirse un acceso inválido. Para evitar este problema, se debe garantizar que la cadena original permanezca disponible o realizar una copia dinámica del nombre y liberarla al eliminar el ámbito. También identifiqué que el harness no comprueba de manera independiente algunos casos como pop_ambito(NULL) y push_ambito() con nombre NULL, aunque el código sí contempla estas condiciones.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
