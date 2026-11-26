# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Sistema de Tipos Primitivos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Chequeo de compatibilidad de tipos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*Al realizar el código manualmente, tuve que identificar cómo comparar los tipos de datos y establecer cuáles asignaciones eran permitidas. La principal dificultad fue comprender la jerarquía CHAR, INT y FLOAT, ya que un tipo puede convertirse a uno de mayor capacidad, pero no a uno más pequeño sin una conversión explícita. También tuve que asegurarme de que TIPO_ERROR siempre fuera rechazado y que cada tipo tuviera su nombre correspondiente.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Matriz de compatibilidad estática |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy trabajando en la Sesión 06 de Lenguajes y Autómatas II.
Necesito implementar una tabla de compatibilidad de tipos
primitivos en lenguaje C.

ROL:
Actúa como un programador experto en C y compiladores.

ESPECIFICACIÓN:
Implementa las funciones compatible(Tipo destino, Tipo origen)
y nombre_tipo(Tipo t), respetando las firmas de api_sesion06.h.

La jerarquía de tipos es CHAR < INT < FLOAT.
Se deben aceptar asignaciones cuando el destino tiene el mismo
rango o uno mayor que el origen.
Se deben rechazar las asignaciones que puedan perder datos.
TIPO_ERROR nunca debe ser compatible con ningún tipo.

La función nombre_tipo debe devolver char, int, float o error.
El código debe utilizar estructuras de control sencillas y
contemplar valores de tipos desconocidos.

OUTPUT:
Entrega el código completo en C, sin función main(),
sin modificar las firmas originales y compatible con:
gcc -std=c11 -Wall -Wextra -Werror.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion06.h"

int compatible(Tipo destino, Tipo origen) {

    if (destino == TIPO_ERROR || origen == TIPO_ERROR) {
        return 0;
    }

    switch (destino) {

        case TIPO_CHAR:
            if (origen == TIPO_CHAR) {
                return 1;
            }
            break;

        case TIPO_INT:
            if (origen == TIPO_CHAR || origen == TIPO_INT) {
                return 1;
            }
            break;

        case TIPO_FLOAT:
            if (origen == TIPO_CHAR || origen == TIPO_INT ||
                origen == TIPO_FLOAT) {
                return 1;
            }
            break;

        default:
            return 0;
    }

    return 0;
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
*Pruebas ejecutadas:* Rechazo de asignaciones incompatibles

### Salida del harness
```text
=== AUDITORIA SESION 06 - Tabla de compatibilidad de tipos ===
Caso                               esperado:              obtenido:              resultado
[ 1] compatible(INT,INT)                esperado: 1            obtenido: 1            OK
[ 2] compatible(FLOAT,INT)              esperado: 1            obtenido: 1            OK
[ 3] compatible(INT,FLOAT)              esperado: 0            obtenido: 0            OK
[ 4] compatible(INT,ERROR)              esperado: 0            obtenido: 0            OK
[ 5] nombre_tipo(INT)                   esperado: int          obtenido: int          OK
[ 6] nombre_tipo(ERROR)                 esperado: error        obtenido: error        OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
La IA omitió considerar explícitamente qué sucedería cuando se recibiera un tipo de dato desconocido, es decir, un valor que no estuviera definido en el enumerado Tipo. Para solucionarlo, agregué una condición default en la estructura switch de la función compatible, para rechazar esos valores y devolver 0. También agregué un default en nombre_tipo para devolver "desconocido". De esta manera, el programa puede manejar valores no contemplados sin aceptar asignaciones incorrectas. Finalmente, ejecuté el arnés y comprobé que las seis pruebas solicitadas funcionaron correctamente, obteniendo un resultado de 6/6.

## Caso borde omitido por la IA
La IA omitió considerar qué sucedería cuando se recibiera un tipo de dato desconocido, es decir, un valor que no estuviera definido en el enumerado Tipo. Para solucionarlo, agregué una condición default en la función compatible para rechazar cualquier tipo no reconocido y devolver 0. También agregué un default en nombre_tipo para devolver "desconocido". De esta manera, se evitan asignaciones incorrectas y el programa puede manejar valores no contemplados. Finalmente, ejecuté el harness y obtuve un resultado de 6/6 pruebas superadas.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragoza 
**Firma digital:** [Usuario de GitHub]
