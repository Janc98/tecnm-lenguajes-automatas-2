# Bitácora de Requerimientos y Auditoría IA — Sesión 09
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Reporte Formal de Errores Semánticos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Diagnóstico con línea y columna  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar la implementación manual, identifiqué que uno de los principales cuidados era controlar el tamaño del buffer donde se almacena el mensaje de error. La función debía generar el diagnóstico con el formato exacto de GCC, incluyendo el nombre del archivo, número de línea, columna y mensaje. También tuve que considerar que el buffer podía tener una capacidad pequeña, por lo que utilicé la función snprintf() para evitar escribir más caracteres de los permitidos. Otra dificultad fue comprobar que los parámetros no fueran nulos y que la salida siempre terminara correctamente con el carácter '\0'.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Estructura estándar de errores GCC |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy trabajando en una práctica de Lenguajes y Autómatas II,
utilizando el lenguaje C y el compilador GCC. Necesito
implementar una función para generar reportes formales
de errores semánticos.

ROL:
Actúa como un ingeniero de software especializado en
programación en C y desarrollo de compiladores.

ESPECIFICACIÓN:
Desarrolla la función reportar() respetando exactamente
la siguiente firma:

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap);

La función debe cumplir los siguientes requisitos:

- Generar el mensaje con el formato exacto:
  archivo:línea:columna: error: mensaje.
- No agregar espacios adicionales ni punto final.
- Utilizar snprintf() para evitar desbordamientos.
- Nunca escribir más de cap-1 caracteres.
- Garantizar la terminación con '\0' cuando exista
  un buffer válido.
- Comprobar que salida no sea NULL y que cap sea mayor
  que cero.
- Manejar archivo y mensaje cuando sean NULL.
- Mantener compatibilidad con C11.
- Compilar con -Wall -Wextra -Werror.
- No incluir una función main().
- Incluir la cabecera api_sesion09.h.

OUTPUT:
Proporciona únicamente la implementación completa del
archivo ia_sesion09_c.c, respetando la firma original,
e incluye una breve explicación de su funcionamiento
y complejidad.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion09.h"

#include <stdio.h>

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap) {
    if (salida == NULL || cap <= 0) {
        return;
    }

    salida[0] = '\0';

    if (archivo == NULL) {
        archivo = "";
    }

    if (mensaje == NULL) {
        mensaje = "";
    }

    snprintf(salida, (size_t)cap,
             "%s:%d:%d: error: %s",
             archivo, linea, columna, mensaje);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Múltiples errores sin abortar parser

### Salida del harness
```text
=== AUDITORIA SESION 09 - Diagnostico con linea y columna ===
Caso                               esperado:              obtenido:              resultado
[ 1] main.c:14:5
     esperado: main.c:14:5: error: tipo incompatible
     obtenido: main.c:14:5: error: tipo incompatible
     -> OK
[ 2] a.c:1:1
     esperado: a.c:1:1: error: x
     obtenido: a.c:1:1: error: x
     -> OK
[ 3] linea 0
     esperado: x.c:0:0: error: y
     obtenido: x.c:0:0: error: y
     -> OK
[ 4] cap=10 trunca
     esperado: main.c:14
     obtenido: main.c:14
     OK

RESULTADO: 4/4 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
El caso que identifiqué como no comprobado directamente por la IA fue cuando el buffer tiene una capacidad de solamente un carácter (cap = 1), así como cuando los parámetros de entrada son nulos. El código generado contemplaba estas situaciones mediante las validaciones de NULL, cap <= 0 y el uso de snprintf(), pero el harness original no incluía pruebas específicas para todos esos escenarios. Revisé las condiciones de la función y comprobé que, con una capacidad de uno, solamente se almacena el carácter terminador '\0', sin provocar un desbordamiento. No fue necesario modificar el código para superar las cuatro pruebas originales, pero identifiqué que sería conveniente ampliar el harness para comprobar estos casos adicionales.

--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa.
**Firma digital:** [Usuario de GitHub]
