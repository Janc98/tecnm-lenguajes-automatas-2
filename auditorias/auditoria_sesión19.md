# Bitácora de Requerimientos y Auditoría IA — Sesión 19
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Estructuras (struct) y Miembros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Offsets de miembros en estructuras  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual, identifiqué que la principal dificultad fue calcular correctamente la posición de cada miembro dentro de una estructura (struct). Tuve que considerar que cada tipo de dato necesita una alineación específica y que, cuando el espacio disponible no coincide con esa alineación, se debe agregar padding (espacios de relleno). También fue necesario calcular el tamaño total de la estructura considerando el padding final. La parte más complicada fue evitar errores al calcular los offsets de los campos cuando se combinan diferentes tipos de datos, como char, int y double.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Alineación de memoria por palabras |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en programación
de sistemas y desarrollo de compiladores.

CONTEXTO:
Estoy trabajando en la Sesión 19 de Lenguajes y Autómatas II.
Necesito implementar funciones en lenguaje C99 utilizando GCC,
respetando el contrato definido en api_sesion19.h.

ROL:
Actúa como un programador experto en estructuras de datos,
organización de memoria y alineación de estructuras.

ESPECIFICACIÓN:
Implementa las siguientes funciones:

int offset_campo(const Campo *campos, int total, const char *nombre);
int tamano_struct(const Campo *campos, int total);

La función offset_campo debe calcular la posición de un miembro
dentro de una estructura, considerando la alineación de cada tipo.

La función tamano_struct debe calcular el tamaño total de la
estructura, incluyendo el padding entre campos y el padding final.

Considera los siguientes tipos:
char, short, int, float, double, long, long long,
int8_t, uint8_t, int16_t, uint16_t, int32_t, uint32_t,
int64_t, uint64_t y bool.

RESTRICCIONES:
- Respetar exactamente las firmas del archivo api_sesion19.h.
- No incluir la función main().
- No utilizar bibliotecas no estándar.
- Validar punteros NULL y tipos desconocidos.
- Devolver -1 cuando exista un campo inexistente o un tipo inválido.
- Devolver 0 cuando la estructura no tenga campos.
- Utilizar código modular y legible.
- Evitar advertencias con -Wall -Wextra -Werror.

OUTPUT:
Devuelve únicamente el código fuente en C99 y un análisis
de complejidad temporal y espacial.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion19.h"
#include <stddef.h>
#include <string.h>

typedef struct {
    const char *nombre;
    int tamano;
    int alineacion;
} TipoInfo;

static const TipoInfo TABLA_TIPOS[] = {
    { "char",      1, 1 },
    { "int8_t",    1, 1 },
    { "uint8_t",   1, 1 },
    { "bool",      1, 1 },
    { "short",     2, 2 },
    { "int16_t",   2, 2 },
    { "uint16_t",  2, 2 },
    { "int",       4, 4 },
    { "float",     4, 4 },
    { "int32_t",   4, 4 },
    { "uint32_t",  4, 4 },
    { "double",    8, 8 },
    { "long",      8, 8 },
    { "long long", 8, 8 },
    { "int64_t",   8, 8 },
    { "uint64_t",  8, 8 }
};

static const TipoInfo *buscar_tipo(const char *tipo) {
    if (tipo == NULL) {
        return NULL;
    }

    size_t n = sizeof(TABLA_TIPOS) / sizeof(TABLA_TIPOS[0]);

    for (size_t i = 0; i < n; i++) {
        if (strcmp(TABLA_TIPOS[i].nombre, tipo) == 0) {
            return &TABLA_TIPOS[i];
        }
    }

    return NULL;
}

static int siguiente_multiplo(int actual, int alineacion) {
    if (alineacion <= 1) {
        return actual;
    }

    int resto = actual % alineacion;

    if (resto == 0) {
        return actual;
    }

    return actual + (alineacion - resto);
}

int offset_campo(const Campo *campos, int total, const char *nombre) {
    if (campos == NULL || total <= 0 || nombre == NULL) {
        return -1;
    }

    int cursor = 0;

    for (int i = 0; i < total; i++) {
        const TipoInfo *info = buscar_tipo(campos[i].tipo);

        if (info == NULL || campos[i].nombre == NULL) {
            return -1;
        }

        int offset_real = siguiente_multiplo(cursor, info->alineacion);

        if (strcmp(campos[i].nombre, nombre) == 0) {
            return offset_real;
        }

        cursor = offset_real + info->tamano;
    }

    return -1;
}

int tamano_struct(const Campo *campos, int total) {
    if (campos == NULL || total < 0) {
        return -1;
    }

    if (total == 0) {
        return 0;
    }

    int cursor = 0;
    int mayor_alineacion = 1;

    for (int i = 0; i < total; i++) {
        const TipoInfo *info = buscar_tipo(campos[i].tipo);

        if (info == NULL) {
            return -1;
        }

        int offset_real = siguiente_multiplo(cursor, info->alineacion);

        cursor = offset_real + info->tamano;

        if (info->alineacion > mayor_alineacion) {
            mayor_alineacion = info->alineacion;
        }
    }

    return siguiente_multiplo(cursor, mayor_alineacion);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Acceso a miembros inexistentes

### Salida del harness
```text
=== AUDITORIA SESION 19 - Offsets de miembros y tamano de struct ===
Caso                               esperado:              obtenido:              resultado
[ 1] struct char e int: b               esperado: 4            obtenido: 4            OK
[ 2] struct de tres campos: c           esperado: 8            obtenido: 8            OK
[ 3] tamano total con padding           esperado: 12           obtenido: 12           OK
[ 4] double seguido de char             esperado: 16           obtenido: 16           OK
[ 5] miembro inexistente                esperado: -1           obtenido: -1           OK
[ 6] struct sin campos                  esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la inteligencia artificial, identifiqué que una situación que necesitaba mayor atención era la validación de los datos de entrada, especialmente cuando se reciben punteros NULL, nombres de campos inexistentes o tipos de datos desconocidos. Aunque la IA contempló varias de estas condiciones, observé que en la función tamano_struct() era importante validar también que el nombre de cada campo no fuera NULL. Esto permitiría mantener la misma validación que se realiza al buscar los offsets.
Para mejorar la solución, se debe agregar esa validación antes de procesar cada campo. También es importante considerar que el harness proporcionado únicamente comprueba seis casos específicos, por lo que superar las pruebas no garantiza que se hayan cubierto todas las situaciones posibles. Con esta revisión comprendí que no basta con que un programa funcione en los casos normales, sino que también debe controlar correctamente las entradas incorrectas.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
