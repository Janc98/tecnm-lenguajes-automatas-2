# Bitácora de Requerimientos y Auditoría IA — Sesión 14
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Declaración vs Uso de Variables  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul  
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Detección de variables no declaradas  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual, la principal dificultad fue controlar correctamente los nombres almacenados en la tabla de símbolos y evitar que una variable se declarara más de una vez. También tuve que considerar que la tabla tiene una capacidad máxima de 16 elementos, por lo que era necesario verificar que existiera espacio antes de insertar un nuevo nombre. Otro detalle importante fue utilizar correctamente strcmp() para comparar cadenas completas y no confundir nombres diferentes como "x" y "xy". Además, tuve que validar los valores NULL y los nombres vacíos para evitar errores durante la ejecución.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariante de definición previa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy desarrollando una tabla de símbolos en lenguaje C para la
materia Lenguajes y Autómatas II. Necesito implementar el control
de declaración previa de variables.

ROL:
Actúa como un ingeniero de software especializado en lenguaje C,
compiladores y análisis semántico.

ESPECIFICACIÓN:
Desarrolla las funciones declarar() y usar(), respetando exactamente
las siguientes firmas:

int declarar(Tabla *tabla, const char *nombre);
int usar(const Tabla *tabla, const char *nombre);

La función declarar() debe:
- Devolver 1 si inserta correctamente un nombre.
- Devolver 0 si el nombre ya existe.
- Devolver -1 si recibe NULL, un nombre vacío o si la tabla está llena.
- No permitir nombres duplicados.
- No incrementar el total cuando existe una redeclaración.

La función usar() debe:
- Devolver 1 si el nombre ya fue declarado.
- Devolver 0 si el nombre no existe o los datos son inválidos.
- Realizar comparaciones exactas utilizando strcmp().
- No modificar el contenido de la tabla.

La tabla tiene una capacidad máxima de 16 elementos.
El código debe incluir api_sesion14.h y no debe contener main().

SALIDA:
Entrega el código completo para ia_sesion14_c.c, con comentarios
explicativos, funciones modulares y compatible con:

gcc -std=c11 -Wall -Wextra -Werror

Explica brevemente la complejidad temporal de las funciones.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion14.h"
#include <string.h>

/* Funcion auxiliar para buscar un nombre */
static int buscar_nombre(const Tabla *tabla, const char *nombre) {
    int posicion = 0;

    while (posicion < tabla->total) {
        if (strcmp(tabla->nombres[posicion], nombre) == 0) {
            return posicion;
        }
        posicion++;
    }

    return -1;
}

/* Declarar una variable */
int declarar(Tabla *tabla, const char *nombre) {

    if (tabla == NULL || nombre == NULL || nombre[0] == '\0') {
        return -1;
    }

    if (buscar_nombre(tabla, nombre) != -1) {
        return 0;
    }

    if (tabla->total >= TABLA_MAX) {
        return -1;
    }

    tabla->nombres[tabla->total] = nombre;
    tabla->total += 1;

    return 1;
}

/* Utilizar una variable */
int usar(const Tabla *tabla, const char *nombre) {

    if (tabla == NULL || nombre == NULL || nombre[0] == '\0') {
        return 0;
    }

    if (buscar_nombre(tabla, nombre) >= 0) {
        return 1;
    }

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Uso antes de declarar y redeclaración

### Salida del harness
```text
=== AUDITORIA SESION 14 - Declaracion previa y redeclaracion ===
Caso                               esperado:              obtenido:              resultado
[ 1] usar sin declarar                  esperado: 0            obtenido: 0            OK
[ 2] usar despues de declarar           esperado: 1            obtenido: 1            OK
[ 3] redeclarar el mismo nombre         esperado: 0            obtenido: 0            OK
[ 4] total tras redeclarar              esperado: 1            obtenido: 1            OK
[ 5] prefijo no es el mismo nombre      esperado: 0            obtenido: 0            OK
[ 6] declarar nombre vacio              esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
La IA no incluyó en el harness una prueba específica para comprobar qué sucede cuando la tabla alcanza su capacidad máxima de 16 elementos. Aunque el código generado sí contiene una condición para verificar si la tabla está llena, fue necesario revisar esa parte de la implementación para asegurar que no se intentara insertar un elemento fuera de los límites establecidos.
También identifiqué que faltaban pruebas específicas para comprobar el comportamiento cuando se reciben punteros NULL. Para atender estos casos, revisé las validaciones de entrada en ambas funciones y comprobé que declarar() devuelve -1 y usar() devuelve 0 cuando recibe datos inválidos. De esta manera, aunque las seis pruebas originales fueron superadas, reconocí que todavía existen casos adicionales que pueden utilizarse para fortalecer la auditoría.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
