# Bitácora de Requerimientos y Auditoría IA — Sesión 08
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Inferencia y Promoción de Tipos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul  
**Matrícula:** 24531399  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Reglas de casteo implícito  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual, tuve que identificar cómo comparar los tipos de datos para determinar cuál tenía mayor jerarquía. La principal dificultad fue comprender que CHAR tiene menor rango que INT y que FLOAT es el tipo de mayor rango. También tuve que considerar el tipo TIPO_ERROR para evitar que una operación con un tipo inválido devolviera un resultado incorrecto. Para solucionarlo utilicé condiciones if y comparaciones entre los valores del tipo enumerado, además de validar que los tipos estuvieran dentro del rango permitido.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Promoción entera y ensanchamiento |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Actúa como un ingeniero de software especializado en compiladores y programación en C11.

CONTEXTO:
Estoy trabajando en la Sesión 08 de Lenguajes y Autómatas II. Debo implementar un sistema de promoción de tipos y detección de pérdida de información utilizando el contrato api_sesion08.h.

ROL:
Compórtate como un programador experto en C y explica las decisiones de implementación de manera sencilla.

ESPECIFICACIÓN:
1. Implementa Tipo promover(Tipo a, Tipo b).
2. La jerarquía de tipos es CHAR < INT < FLOAT.
3. Si alguno de los tipos es TIPO_ERROR, devuelve TIPO_ERROR.
4. Implementa int hay_perdida(Tipo destino, Tipo origen), que devuelva 1 cuando el destino tenga menor rango que el origen y 0 en caso contrario.
5. Implementa const char *nombre_tipo(Tipo t), que devuelva el nombre correspondiente al tipo.
6. Respeta exactamente las firmas declaradas en api_sesion08.h.
7. No utilices main(), memoria dinámica ni bibliotecas externas.
8. El código debe compilar con gcc -std=c11 -Wall -Wextra -Werror.

OUTPUT:
Entrega únicamente el código fuente C completo, modular, ordenado y compatible con el harness de la sesión 08.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion08.h"

Tipo promover(Tipo a, Tipo b) {

    if (a == TIPO_ERROR) {
        return TIPO_ERROR;
    }

    if (b == TIPO_ERROR) {
        return TIPO_ERROR;
    }

    if (a == TIPO_FLOAT || b == TIPO_FLOAT) {
        return TIPO_FLOAT;
    }

    if (a == TIPO_INT || b == TIPO_INT) {
        return TIPO_INT;
    }

    if (a == TIPO_CHAR && b == TIPO_CHAR) {
        return TIPO_CHAR;
    }

    return TIPO_ERROR;
}

int hay_perdida(Tipo destino, Tipo origen) {

    switch (destino) {

        case TIPO_CHAR:
            return origen == TIPO_INT || origen == TIPO_FLOAT;

        case TIPO_INT:
            return origen == TIPO_FLOAT;

        case TIPO_FLOAT:
            return 0;

        default:
            return 0;
    }
}

const char *nombre_tipo(Tipo t) {

    switch (t) {

        case TIPO_CHAR:
            return "char";

        case TIPO_INT:
            return "int";

        case TIPO_FLOAT:
            return "float";

        case TIPO_ERROR:
            return "error";

        default:
            return "desconocido";
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Mezcla de tipos enteros y flotantes

### Salida del harness
```text
=== AUDITORIA SESION 08 - Reglas de promocion y perdida ===
Caso                               esperado:              obtenido:              resultado
[ 1] promover(CHAR,INT)                 esperado: int          obtenido: int          OK
[ 2] promover(INT,FLOAT)                esperado: float        obtenido: float        OK
[ 3] promover(FLOAT,FLOAT)              esperado: float        obtenido: float        OK
[ 4] promover(ERROR,INT)                esperado: error        obtenido: error        OK
[ 5] hay_perdida(CHAR,INT)              esperado: 1            obtenido: 1            OK
[ 6] hay_perdida(INT,CHAR)              esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Durante la revisión del código detecté que la función hay_perdida() no validaba explícitamente todos los valores posibles del tipo de origen, especialmente cuando se recibía un tipo inválido o el valor TIPO_ERROR. Aunque el código funcionaba correctamente con los casos del harness, consideré importante agregar una validación para evitar resultados incorrectos cuando se recibieran valores fuera del rango permitido.

También comprobé las conversiones de FLOAT a INT y de FLOAT a CHAR, ya que ambas pueden provocar pérdida de información. Para mejorar la implementación tomé como referencia mi código artesanal, donde agregué condiciones para validar tanto el tipo de destino como el de origen antes de realizar la comparación.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragoza 
**Firma digital:** [Usuario de GitHub]
