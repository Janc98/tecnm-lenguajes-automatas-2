# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Atributos de Símbolos y Memoria  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de registro de símbolos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Durante la elaboración manual tuve que identificar cómo calcular el espacio que ocupa cada tipo de dato y cómo asignar correctamente su posición en memoria. La principal dificultad fue comprender la alineación, ya que no basta con sumar el tamaño de las variables, sino que también se debe ajustar el cursor para que el siguiente dato comience en una dirección múltiplo del tamaño de su tipo. También tuve que validar que el registro no fuera NULL, que el cursor no fuera negativo y que el tipo de dato existiera. Para resolverlo utilicé comparaciones con strcmp() y operaciones aritméticas para calcular el desplazamiento correspondiente.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cálculo de desplazamientos y alineación |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en sistemas y compiladores.

CONTEXTO:
Estoy desarrollando la Sesión 12 de Lenguajes y Autómatas II.
El programa debe trabajar en lenguaje C11 y compilar con GCC
utilizando las opciones -Wall -Wextra -Werror.

ROL:
Compórtate como un ingeniero experto en programación de sistemas,
administración de memoria y diseño de compiladores.

ESPECIFICACIÓN:
Implementa las siguientes funciones respetando exactamente las
firmas declaradas en api_sesion12.h:

1. int ancho_de_tipo(const char *tipo);
   - char debe devolver 1.
   - short debe devolver 2.
   - int debe devolver 4.
   - float debe devolver 4.
   - double debe devolver 8.
   - puntero debe devolver 8.
   - Cualquier tipo desconocido debe devolver -1.
   - También debe validar que el tipo no sea NULL.

2. int asignar_offset(Simbolo *simbolo, int cursor);
   - Validar que el registro no sea NULL.
   - Rechazar cursores negativos.
   - Obtener el tamaño mediante ancho_de_tipo().
   - Calcular el desplazamiento respetando la alineación natural.
   - Guardar el desplazamiento calculado en simbolo->offset.
   - Devolver el cursor donde comenzará la siguiente variable.
   - Evitar desbordamientos de enteros.
   - Devolver -1 cuando exista algún dato inválido.

RESTRICCIONES:
- Incluir api_sesion12.h.
- Utilizar bibliotecas estándar de C.
- No modificar las firmas de las funciones.
- No incluir main(), porque el harness ya contiene esa función.
- No utilizar memoria dinámica innecesaria.
- El código debe compilar sin advertencias.

OUTPUT:
Entrega únicamente el código C completo, modular y comentado,
con ambas funciones implementadas.
Explica brevemente la complejidad temporal y espacial.]
```

---

## 3. Código Devuelto por el Modelo
```c
/*
 * ia_sesion12_c.c
 * Fase 2: El Cientifico
 * Registro de simbolos y calculo de memoria.
 */

#include "api_sesion12.h"
#include <string.h>
#include <limits.h>

int ancho_de_tipo(const char *tipo) {

    if (tipo == NULL) {
        return -1;
    }

    if (strcmp(tipo, "char") == 0) {
        return 1;
    }
    else if (strcmp(tipo, "short") == 0) {
        return 2;
    }
    else if (strcmp(tipo, "int") == 0 ||
             strcmp(tipo, "float") == 0) {
        return 4;
    }
    else if (strcmp(tipo, "double") == 0 ||
             strcmp(tipo, "puntero") == 0) {
        return 8;
    }

    return -1;
}

int asignar_offset(Simbolo *simbolo, int cursor) {

    int tamano;
    int residuo;
    int desplazamiento;

    if (simbolo == NULL || cursor < 0) {
        return -1;
    }

    tamano = ancho_de_tipo(simbolo->tipo);

    if (tamano <= 0) {
        return -1;
    }

    residuo = cursor % tamano;

    if (residuo == 0) {
        desplazamiento = cursor;
    }
    else {
        int ajuste = tamano - residuo;

        if (cursor > INT_MAX - ajuste) {
            return -1;
        }

        desplazamiento = cursor + ajuste;
    }

    if (desplazamiento > INT_MAX - tamano) {
        return -1;
    }

    simbolo->offset = desplazamiento;

    return desplazamiento + tamano;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Alineación de tipos y tamaños en bytes

### Salida del harness
```text
=== AUDITORIA SESION 12 - Registro de simbolo y offsets alineados ===
Caso                               esperado:              obtenido:              resultado
[ 1] ancho_de_tipo("int")          esperado: 4            obtenido: 4            OK
[ 2] ancho_de_tipo("double")       esperado: 8            obtenido: 8            OK
[ 3] int en cursor 0               esperado: 4            obtenido: 4            OK
[ 4] char e int: offset de b       esperado: 4            obtenido: 4            OK
[ 5] double desde cursor 4         esperado: 8            obtenido: 8            OK
[ 6] registro NULL                 esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion12.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Aunque la IA generó correctamente las funciones y logró superar las seis pruebas del harness, identifiqué que las pruebas proporcionadas no verificaban todos los casos posibles. Por ejemplo, no se probaban los tipos short, float y puntero, tampoco los cursores negativos, los tipos desconocidos ni los posibles desbordamientos de enteros. Para atender estos casos revisé las validaciones de la implementación y comprobé que se rechazaran los datos incorrectos mediante condiciones de seguridad. También verifiqué que el cálculo del desplazamiento utilizara el residuo de la división para respetar la alineación. Esto me permitió comprender que pasar las pruebas no significa que el programa esté libre de todos los posibles errores.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
