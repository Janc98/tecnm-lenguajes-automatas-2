# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Gestión en GitHub Web y Nodos Dinámicos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul  
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lista enlazada de tokens en Heap  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
Al realizar la lista enlazada de tokens, identifiqué que uno de los principales problemas es el manejo de memoria dinámica, ya que cada nodo debe reservarse utilizando la función malloc. También tuve que considerar que el último nodo debe apuntar a NULL para indicar el final de la lista. Otra dificultad fue recorrer la lista para agregar nuevos elementos al final sin perder la dirección de la cabeza. Además, fue necesario utilizar strncpy para copiar los lexemas respetando el límite de 15 caracteres y agregar el carácter nulo al final de la cadena.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gestión de memoria dinámica C99/Rust |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en sistemas
y compiladores.

CONTEXTO:
Estoy trabajando en C99 utilizando GCC y bibliotecas estándar.
Necesito implementar una lista enlazada de tokens que utilice
memoria dinámica en el heap.

ROL:
Compórtate como un programador experto en lenguaje C y manejo
de estructuras de datos dinámicas.

ESPECIFICACIONES:
1. Implementa la función agregar_token, que recibe la cabeza
   de una lista y un lexema, reserva memoria con malloc y
   agrega el nuevo nodo al final.
2. Implementa contar_tokens para recorrer la lista y devolver
   el número total de nodos.
3. Implementa liberar_lista para liberar toda la memoria
   reservada utilizando free.
4. Respeta exactamente las firmas declaradas en api_sesion02.h.
5. Cada nodo contiene un arreglo char lexema[16] y un apuntador
   al siguiente nodo.
6. El último nodo debe apuntar a NULL.
7. Si la lista está vacía, agregar_token debe devolver
   el nuevo nodo como cabeza.
8. contar_tokens(NULL) debe devolver 0.
9. liberar_lista(NULL) debe ejecutarse sin provocar errores.
10. Evita fugas de memoria y accesos a memoria liberada.
11. Utiliza strncpy para copiar los lexemas y garantiza
    la terminación nula de las cadenas.

OUTPUT:
Entrega únicamente el código modular en C, sin main(),
con los encabezados necesarios y una explicación breve
de la complejidad temporal de cada función.
El código debe compilar con:
gcc -std=c11 -Wall -Wextra -Werror]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion02.h"
#include <stdlib.h>
#include <string.h>

NodoToken *agregar_token(NodoToken *cabeza, const char *lexema) {
    NodoToken *nuevo = malloc(sizeof(NodoToken));

    if (nuevo == NULL) return cabeza;

    strncpy(nuevo->lexema, lexema, 15);
    nuevo->lexema[15] = '\0';
    nuevo->siguiente = NULL;

    if (cabeza == NULL) return nuevo;

    NodoToken *actual = cabeza;

    while (actual->siguiente != NULL)
        actual = actual->siguiente;

    actual->siguiente = nuevo;

    return cabeza;
}

int contar_tokens(NodoToken *cabeza) {
    int total = 0;

    while (cabeza != NULL) {
        total++;
        cabeza = cabeza->siguiente;
    }

    return total;
}

void liberar_lista(NodoToken *cabeza) {
    while (cabeza != NULL) {
        NodoToken *temporal = cabeza;
        cabeza = cabeza->siguiente;
        free(temporal);
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Prevención de Use-After-Free y cadenas nulas

### Salida del harness
```text
=== AUDITORIA SESION 02 - Lista enlazada de tokens en heap ===
Caso                               esperado:              obtenido:              resultado
[ 1] agregar_token(NULL,"if")           esperado: if           obtenido: if           OK
[ 2] contar_tokens(3 tokens)            esperado: 3            obtenido: 3            OK
[ 3] orden: segundo token               esperado: dos          obtenido: dos          OK
[ 4] ultimo->siguiente == NULL          esperado: 1            obtenido: 1            OK
[ 5] contar_tokens(NULL)                esperado: 0            obtenido: 0            OK
[ 6] liberar_lista(NULL)                esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion02.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
>
Al revisar el código generado por la IA, identifiqué que no contempla la validación del apuntador lexema cuando su valor es NULL. Esto podría provocar un error al utilizar la función strncpy, porque necesita recibir una cadena válida. Para solucionarlo, se puede agregar una condición que compruebe que el apuntador no sea NULL antes de copiar el contenido. También revisé que la lista pudiera comenzar vacía, que los tokens se agregaran en el orden correcto y que la memoria se liberara correctamente utilizando free.

## Caso borde omitido por la IA
La IA omitió validar si el apuntador lexema era NULL antes de utilizar strncpy. Este caso no está contemplado en las seis pruebas del harness, pero es importante considerarlo para evitar errores durante la ejecución. La solución consiste en comprobar que lexema no sea NULL antes de realizar la copia. De esta manera, se mejora la seguridad del programa y se evita trabajar con un apuntador inválido.


--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
