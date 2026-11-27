# Bitácora de Requerimientos y Auditoría IA — Sesión 26
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** TAC para Llamadas a Funciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Instrucciones param, call y return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al realizar el código manualmente, identifiqué que debía controlar correctamente el espacio disponible en el arreglo de instrucciones lineas[][64] para evitar escribir fuera de sus límites. También tuve que utilizar snprintf() para generar las instrucciones PARAM, CALL y RETURN de manera segura. Otra dificultad fue controlar el contador de parámetros y actualizar correctamente la variable n para no perder el orden de las instrucciones, especialmente cuando se realizan llamadas anidadas.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Convención de paso de argumentos TAC |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Contexto:
Estoy desarrollando una práctica de Lenguajes y Autómatas II
sobre generación de código de tres direcciones (TAC) en C.

Rol:
Actúa como un ingeniero de software especializado en
programación en C y desarrollo de compiladores.

Especificación:
Implementa las funciones emitir_llamada() y emitir_return()
respetando exactamente las firmas del archivo api_sesion26.h.

La función emitir_llamada debe:
- Generar PARAM para cada argumento que no sea NULL.
- Mantener el orden de los argumentos a1 y a2.
- Generar CALL indicando la cantidad real de parámetros.
- Permitir llamadas con cero, uno o dos argumentos.
- Asignar el resultado solamente cuando resultado no sea NULL.
- Verificar los límites del arreglo y evitar accesos fuera
  de memoria.
- Conservar correctamente el contador n.

La función emitir_return debe:
- Generar RETURN seguido del valor cuando exista.
- Generar únicamente RETURN cuando valor sea NULL.
- Respetar la capacidad del búfer de destino.

Output:
Entrega código completo en C, compatible con C11,
sin función main(), utilizando snprintf() y respetando
la interfaz original.
El código debe compilar con:
gcc -std=c11 -Wall -Wextra -Werror]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion26.h"
#include <stdio.h>

static int agregar_linea(char lineas[][64], int max,
                         int *n, const char *texto)
{
    if (lineas == NULL || n == NULL || texto == NULL) {
        return 0;
    }

    if (*n < 0 || *n >= max) {
        return 0;
    }

    snprintf(lineas[*n], 64, "%s", texto);
    *n = *n + 1;

    return 1;
}

int emitir_llamada(char lineas[][64], int max, int *n,
                   const char *funcion,
                   const char *a1,
                   const char *a2,
                   const char *resultado)
{
    char instruccion[64];
    int cantidad = 0;
    int necesarios;

    if (lineas == NULL || n == NULL || funcion == NULL) {
        return 0;
    }

    if (max < 0 || *n < 0 || *n > max) {
        return 0;
    }

    if (a1 != NULL) cantidad++;
    if (a2 != NULL) cantidad++;

    necesarios = cantidad + 1;

    if (max - *n < necesarios) {
        return 0;
    }

    if (a1 != NULL) {
        snprintf(instruccion, sizeof(instruccion),
                 "PARAM %s", a1);

        if (!agregar_linea(lineas, max, n, instruccion)) {
            return 0;
        }
    }

    if (a2 != NULL) {
        snprintf(instruccion, sizeof(instruccion),
                 "PARAM %s", a2);

        if (!agregar_linea(lineas, max, n, instruccion)) {
            return 0;
        }
    }

    if (resultado != NULL) {
        snprintf(instruccion, sizeof(instruccion),
                 "%s = CALL %s, %d",
                 resultado, funcion, cantidad);
    } else {
        snprintf(instruccion, sizeof(instruccion),
                 "CALL %s, %d", funcion, cantidad);
    }

    return agregar_linea(lineas, max, n, instruccion);
}

void emitir_return(char *dst, int cap, const char *valor)
{
    if (dst == NULL || cap <= 0) {
        return;
    }

    if (valor != NULL) {
        (void)snprintf(dst, (size_t)cap, "RETURN %s", valor);
    } else {
        (void)snprintf(dst, (size_t)cap, "RETURN");
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas anidadas f(g(x))

### Salida del harness
```text
=== AUDITORIA SESION 26 - Llamadas, parametros y retorno ===
Caso                               esperado:              obtenido:              resultado
[ 1] PARAM x                            esperado: PARAM x      obtenido: PARAM x      OK
[ 2] PARAM y                            esperado: PARAM y      obtenido: PARAM y      OK
[ 3] CALL con dos argumentos
     esperado: _t0 = CALL f, 2
     obtenido: _t0 = CALL f, 2
     -> OK
[ 4] llamada anidada f(g(x))
     esperado: _t1 = CALL f, 1
     obtenido: _t1 = CALL f, 1
     -> OK
[ 5] CALL sin argumentos
     esperado: _t0 = CALL p, 0
     obtenido: _t0 = CALL p, 0
     -> OK
[ 6] RETURN con valor                   esperado: RETURN _t0   obtenido: RETURN _t0   OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. 
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Identifiqué que el harness original no incluye una prueba específica para comprobar un RETURN sin valor. Aunque la implementación de inteligencia artificial contempla esta condición mediante una comparación con NULL, no existe un caso que la verifique directamente. Para comprobarlo propuse una prueba adicional utilizando emitir_return(b, 32, NULL), cuyo resultado esperado es únicamente RETURN. De esta manera se verifica que la función pueda generar correctamente un retorno sin valor. También considero importante probar los límites del arreglo cuando no existe espacio suficiente para agregar todas las instrucciones de una llamada.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
