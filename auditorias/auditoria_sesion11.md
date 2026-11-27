# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arquitectura de la Tabla de Símbolos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Tabla Hash con encadenamiento  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual, una de las principales dificultades fue comprender cómo manejar las colisiones cuando dos nombres diferentes obtienen la misma posición en la tabla hash. También tuve que tener cuidado con el uso de apuntadores y memoria dinámica para crear los nodos de la lista enlazada. Otra dificultad fue evitar que se insertaran nombres duplicados y asegurar que cada símbolo conservara correctamente sus datos, como el tipo y el offset. Además, fue necesario validar los argumentos y controlar que los nombres no superaran el tamaño permitido del arreglo.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Función de dispersión y colisiones O(1) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en estructuras de datos y desarrollo de compiladores.

CONTEXTO:
Estoy trabajando en la materia Lenguajes y Autómatas II, en una práctica de arquitectura de tablas de símbolos utilizando lenguaje C99 y compilador GCC.

ROL:
Desempeña el papel de un ingeniero de software experto en tablas hash, listas enlazadas, administración de memoria y programación en C.

ESPECIFICACIONES:
1. Implementa la función hash_djb2(const char *clave), utilizando el valor inicial 5381 y multiplicando el resultado por 33 antes de sumar cada carácter.
2. Implementa insertar_simbolo(Simbolo *tabla[], unsigned int cubetas, const char *nombre, int tipo, int offset).
3. Utiliza encadenamiento mediante listas enlazadas para resolver colisiones.
4. Evita insertar nombres duplicados y devuelve 0 cuando el símbolo ya existe.
5. Devuelve 1 cuando se inserta correctamente.
6. Devuelve -1 cuando la tabla es NULL, el nombre es NULL o el número de cubetas es cero.
7. Respeta el tamaño de 16 caracteres del campo nombre, incluyendo el carácter nulo.
8. Conserva correctamente el tipo, offset y apuntador al siguiente nodo.
9. Comprueba que malloc no falle antes de utilizar la memoria reservada.
10. Respeta exactamente las firmas declaradas en api_sesion11.h.
11. No incluyas la función main(), porque el programa será evaluado mediante un harness externo.
12. El código debe compilar con gcc -std=c11 -Wall -Wextra -Werror.

OUTPUT:
Entrega código modular en C99, sin bibliotecas externas, y explica brevemente la complejidad temporal y espacial de la implementación.]
```

---

## 3. Código Devuelto por el Modelo
```c

/*
 * ia_sesion11_c.c
 * Fase 2: El Cientifico
 * Tabla hash con encadenamiento
 */

#include "api_sesion11.h"
#include <stdlib.h>
#include <string.h>

unsigned int hash_djb2(const char *clave) {
    unsigned int resultado = 5381;

    if (clave == NULL) {
        return resultado;
    }

    for (; *clave != '\0'; clave++) {
        resultado = (resultado * 33) + (unsigned char)*clave;
    }

    return resultado;
}

int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas,
                     const char *nombre, int tipo, int offset) {

    unsigned int indice;
    Simbolo *recorrer;
    Simbolo *nuevo;

    if (tabla == NULL || nombre == NULL || cubetas == 0) {
        return -1;
    }

    if (strlen(nombre) >= sizeof(tabla[0]->nombre)) {
        return -1;
    }

    indice = hash_djb2(nombre) % cubetas;
    recorrer = tabla[indice];

    while (recorrer != NULL) {
        if (strcmp(recorrer->nombre, nombre) == 0) {
            return 0;
        }

        if (recorrer->sig == NULL) {
            break;
        }

        recorrer = recorrer->sig;
    }

    nuevo = (Simbolo *)malloc(sizeof(Simbolo));

    if (nuevo == NULL) {
        return -1;
    }

    strcpy(nuevo->nombre, nombre);
    nuevo->tipo = tipo;
    nuevo->offset = offset;
    nuevo->sig = NULL;

    if (tabla[indice] == NULL) {
        tabla[indice] = nuevo;
    } else {
        recorrer->sig = nuevo;
    }

    return 1;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Manejo de claves duplicadas y colisiones

### Salida del harness
```text

=== AUDITORIA SESION 11 - Tabla hash con encadenamiento ===
Caso                               esperado:              obtenido:              resultado
[ 1] hash_djb2("")                      esperado: 5381         obtenido: 5381         OK
[ 2] hash_djb2("abc")                   esperado: 193485963    obtenido: 193485963    OK
[ 3] insertar x dos veces               esperado: 0            obtenido: 0            OK
[ 4] dos claves, una cubeta             esperado: 2            obtenido: 2            OK
[ 5] offset de beta en cadena           esperado: 4            obtenido: 4            OK
[ 6] insertar en tabla NULL             esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.

```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la IA, observé que no se incluyó una función específica para liberar la memoria dinámica de los nodos que se van creando en la tabla hash. Aunque la implementación utiliza correctamente malloc para reservar memoria, es importante liberar esos espacios cuando la tabla deja de utilizarse. Para solucionar este problema, identifiqué que se debe recorrer cada cubeta, avanzar por los nodos de las listas enlazadas y utilizar free para liberar cada elemento. También consideré importante probar nombres que superen los 15 caracteres permitidos, para evitar desbordamientos en el arreglo nombre. Las pruebas proporcionadas por el harness fueron superadas, aunque estos casos adicionales no fueron evaluados.
--- Jose Alfredo Nahuat Canul Unidad: Igancio Zaragosa
**Firma digital:** [Usuario de GitHub]
