# Bitácora de Requerimientos y Auditoría IA — Sesión 10
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 1: Mini-Evaluador de Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul
**Matrícula:** 24531399 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración AST y Tipado Estático  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al desarrollar manualmente el mini-evaluador de expresiones, una de las principales dificultades fue comprender cómo separar las operaciones matemáticas para respetar la prioridad de los operadores. Tuve que organizar las funciones para que primero se resolvieran las multiplicaciones y divisiones, y después las sumas y restas. También se me dificultó controlar la lectura de los caracteres, reconocer los números enteros, ignorar los espacios y detectar correctamente los paréntesis. Otro aspecto importante fue identificar cuándo una expresión estaba incompleta o contenía una división entre cero, para asignar el código de error correspondiente sin provocar que el programa fallara.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato formal de Frontend Semántico |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[# Prompt estructurado C-R-E-O — Sesión 10

**C — Contexto:**

Estoy desarrollando la actividad de la materia Lenguajes y Autómatas II (SCD-1016), correspondiente a la Sesión 10: Hito 1, Mini-Evaluador de Expresiones. El proyecto utiliza lenguaje C y el compilador GCC. Se debe trabajar con el archivo `api_sesion10.h`, respetando exactamente la interfaz establecida. La compilación se realiza con las opciones `-std=c11 -Wall -Wextra -Werror`.

**R — Rol:**

Actúa como un ingeniero de software especializado en lenguaje C, análisis sintáctico y desarrollo de compiladores. Tu tarea es desarrollar una implementación clara, modular y eficiente, utilizando conceptos de análisis sintáctico descendente.

**E — Especificación:**

Desarrolla un mini-evaluador de expresiones aritméticas que cumpla con los siguientes requisitos:

1. Incluir el archivo `api_sesion10.h`.
2. Respetar exactamente la siguiente firma:
   `long evaluar_expresion(const char *fuente, int *error);`
3. No modificar la firma de la función ni agregar una función `main()`.
4. Reconocer números enteros y los operadores de suma (+), resta (-), multiplicación (*) y división (/).
5. Reconocer paréntesis y espacios en blanco.
6. Respetar la precedencia matemática: primero multiplicación y división, después suma y resta.
7. Utilizar una estructura de funciones para analizar las expresiones, separando la lectura de números y paréntesis, los términos y las expresiones.
8. No implementar variables ni operadores unarios, ya que no forman parte del contrato de esta sesión.
9. Reportar el error 1 cuando se intente dividir entre cero.
10. Reportar el error 2 cuando exista una expresión incompleta, inválida o con paréntesis incorrectos.
11. Evitar accesos fuera de los límites, variables sin inicializar y errores de compilación.
12. Utilizar únicamente bibliotecas estándar de C.
13. El programa debe ser compatible con el harness proporcionado por el docente.

**O — Output (Salida esperada):**

Entrega lo siguiente:

- Código completo del archivo `ia_sesion10_c.c`.
- Implementación modular, organizada y con comentarios que expliquen las funciones principales.
- Sin función `main()`, porque el harness proporciona su propia función principal.
- Código compatible con GCC y las opciones `-std=c11 -Wall -Wextra -Werror`.
- Breve explicación del funcionamiento del evaluador.
- Análisis de complejidad temporal y espacial.

El objetivo final es conseguir que el programa supere las seis pruebas del harness de la Sesión 10, incluyendo precedencia de operadores, paréntesis, espacios en blanco, división entre cero y errores de sintaxis.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion10.h"
#include <ctype.h>

/* Estructura para controlar el analisis */
typedef struct {
    const char *texto;
    int posicion;
    int error;
} Analizador;

/* Declaracion de funciones */
static long leer_expresion(Analizador *a);

/* Ignorar espacios en blanco */
static void saltar_espacios(Analizador *a) {
    while (isspace((unsigned char)a->texto[a->posicion])) {
        a->posicion++;
    }
}

/* Leer numeros y parentesis */
static long leer_factor(Analizador *a) {
    long valor = 0;
    int signo = 1;

    saltar_espacios(a);

    /* Revisar signo */
    if (a->texto[a->posicion] == '-') {
        signo = -1;
        a->posicion++;
    } else if (a->texto[a->posicion] == '+') {
        a->posicion++;
    }

    saltar_espacios(a);

    /* Revisar parentesis */
    if (a->texto[a->posicion] == '(') {
        a->posicion++;

        valor = leer_expresion(a);

        saltar_espacios(a);

        if (a->texto[a->posicion] != ')') {
            a->error = 2;
            return 0;
        }

        a->posicion++;

        return signo * valor;
    }

    /* Comprobar que exista un numero */
    if (!isdigit((unsigned char)a->texto[a->posicion])) {
        a->error = 2;
        return 0;
    }

    /* Leer los digitos */
    while (isdigit((unsigned char)a->texto[a->posicion])) {
        valor = valor * 10 +
                (a->texto[a->posicion] - '0');

        a->posicion++;
    }

    return signo * valor;
}

/* Resolver multiplicaciones y divisiones */
static long leer_termino(Analizador *a) {
    long resultado = leer_factor(a);
    char operador;

    while (a->error == 0) {
        saltar_espacios(a);

        operador = a->texto[a->posicion];

        if (operador != '*' && operador != '/') {
            break;
        }

        a->posicion++;

        long siguiente = leer_factor(a);

        if (a->error != 0) {
            return 0;
        }

        if (operador == '*') {
            resultado = resultado * siguiente;
        } else {
            if (siguiente == 0) {
                a->error = 1;
                return 0;
            }

            resultado = resultado / siguiente;
        }
    }

    return resultado;
}

/* Resolver sumas y restas */
static long leer_expresion(Analizador *a) {
    long resultado = leer_termino(a);
    char operador;

    while (a->error == 0) {
        saltar_espacios(a);

        operador = a->texto[a->posicion];

        if (operador != '+' && operador != '-') {
            break;
        }

        a->posicion++;

        long siguiente = leer_termino(a);

        if (a->error != 0) {
            return 0;
        }

        if (operador == '+') {
            resultado = resultado + siguiente;
        } else {
            resultado = resultado - siguiente;
        }
    }

    return resultado;
}

/* Funcion solicitada por el contrato */
long evaluar_expresion(const char *fuente, int *error) {
    Analizador a;
    long resultado = 0;

    if (error != NULL) {
        *error = 0;
    }

    /* Verificar la cadena */
    if (fuente == NULL) {
        if (error != NULL) {
            *error = 2;
        }

        return 0;
    }

    /* Inicializar analizador */
    a.texto = fuente;
    a.posicion = 0;
    a.error = 0;

    /* Evaluar expresion */
    resultado = leer_expresion(&a);

    saltar_espacios(&a);

    /* Verificar caracteres sobrantes */
    if (a.texto[a.posicion] != '\0' && a.error == 0) {
        a.error = 2;
    }

    if (error != NULL) {
        *error = a.error;
    }

    if (a.error != 0) {
        return 0;
    }

    return resultado;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Batería de 10 expresiones mixtas complejas

### Salida del harness
```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                               esperado:              obtenido:              resultado

[ 1] "2+3*4"                        esperado: 14           obtenido: 14           OK
[ 2] "(2+3)*4"                      esperado: 20           obtenido: 20           OK
[ 3] "10/0" -> error 1              esperado: 1            obtenido: 1            OK
[ 4] "2+" -> error 2                esperado: 2            obtenido: 2            OK
[ 5] "  7 "                         esperado: 7            obtenido: 7            OK
[ 6] "5-10"                         esperado: -5           obtenido: -5           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Un problema que identifiqué al revisar el código generado por la IA fue que no se contemplaban algunas entradas especiales, como una cadena vacía, una expresión con paréntesis incompletos o números demasiado grandes para el tipo de dato long. Aunque el programa logró superar las seis pruebas del arnés, estos casos adicionales no fueron comprobados. Como parte de la auditoría, identifiqué que sería necesario agregar validaciones para evitar resultados incorrectos y mejorar el manejo de entradas inválidas. Esto me permitió comprender que superar las pruebas establecidas no significa que el programa esté libre de todos los posibles errores.
---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragoza 
**Firma digital:** [Usuario de GitHub]
