# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Comprobación Semántica de Retornos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Validación de tipo de sentencia return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al desarrollar manualmente la función diagnosticar_retorno(), la principal dificultad fue organizar las condiciones para comparar correctamente los tipos de datos y verificar que todas las rutas de ejecución tuvieran un retorno. También fue necesario utilizar strcmp() para comparar las cadenas de caracteres y comprobar que los parámetros no fueran NULL. Otra dificultad fue establecer el orden correcto de las validaciones, porque primero se debe comprobar la compatibilidad de tipos y después verificar las rutas de retorno.

2. Fase 2: El Científico (In*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Rutas de ejecución con retorno garantizado |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy trabajando en la materia Lenguajes y Autómatas II,
en la sesión 17, cuyo tema es la comprobación semántica
de retornos y rutas de control. Necesito desarrollar una
función en lenguaje C que valide los tipos de retorno
de una función y compruebe que todas sus rutas tengan
un retorno válido.

ROL:
Actúa como un ingeniero de software especializado en
lenguaje C, compiladores y análisis semántico.

ESPECIFICACIÓN:
Desarrolla la función diagnosticar_retorno() respetando
exactamente la siguiente firma:

int diagnosticar_retorno(
    const char *tipo_funcion,
    const char *tipo_expresion,
    int rutas_totales,
    int rutas_con_return
);

La función debe cumplir las siguientes condiciones:

1. Devolver -1 si alguno de los tipos es NULL, si las
   rutas totales son cero o negativas, si los retornos
   son negativos o si hay más retornos que rutas.

2. Devolver 1 cuando exista incompatibilidad entre el
   tipo de la función y el tipo de la expresión devuelta.

3. Una función void solamente puede devolver una
   expresión de tipo void.

4. Aceptar conversiones cuando el tipo destino sea
   igual o más ancho que el tipo de origen.

5. Devolver 2 cuando existan rutas de ejecución sin
   una instrucción return.

6. Devolver 0 cuando todas las comprobaciones sean
   correctas.

7. Comparar primero los tipos y después las rutas.

8. Utilizar código C estándar e incluir las bibliotecas
   necesarias.

9. No incluir main(), porque el harness proporciona
   la función principal.

OUTPUT:
Entrega el código completo, modular y funcional,
incluyendo las funciones auxiliares necesarias.
Debe compilar con:

gcc -std=c11 -Wall -Wextra -Werror]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion17.h"
#include <string.h>

int obtener_tamano(const char *tipo) {

    if (tipo == NULL) {
        return -1;
    }

    if (strcmp(tipo, "char") == 0) {
        return 1;
    }

    if (strcmp(tipo, "int") == 0) {
        return 2;
    }

    if (strcmp(tipo, "float") == 0) {
        return 3;
    }

    if (strcmp(tipo, "double") == 0) {
        return 4;
    }

    return -1;
}

int diagnosticar_retorno(const char *tipo_funcion,
                         const char *tipo_expresion,
                         int rutas_totales,
                         int rutas_con_return) {

    int destino;
    int origen;

    if (tipo_funcion == NULL || tipo_expresion == NULL) {
        return -1;
    }

    if (rutas_totales <= 0 || rutas_con_return < 0 ||
        rutas_con_return > rutas_totales) {
        return -1;
    }

    if (strcmp(tipo_funcion, "void") == 0) {
        if (strcmp(tipo_expresion, "void") == 0) {
            return 0;
        }
        return 1;
    }

    destino = obtener_tamano(tipo_funcion);
    origen = obtener_tamano(tipo_expresion);

    if (destino == -1 || origen == -1) {
        return 1;
    }

    if (destino < origen) {
        return 1;
    }

    if (rutas_con_return < rutas_totales) {
        return 2;
    }

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Funciones sin retorno o retornos mixtos

### Salida del harness
```text
=== AUDITORIA SESION 17 - Validacion de retornos y rutas ===
Caso                               esperado:              obtenido:              resultado
[ 1] int con int y dos rutas            esperado: 0            obtenido: 0            OK
[ 2] void devuelve valor                esperado: 1            obtenido: 1            OK
[ 3] ruta sin retorno                   esperado: 2            obtenido: 2            OK
[ 4] double acepta int                  esperado: 0            obtenido: 0            OK
[ 5] int no acepta double               esperado: 1            obtenido: 1            OK
[ 6] cero rutas de ejecucion            esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
Al revisar el código generado por la inteligencia artificial, identifiqué que el harness no contempla algunos casos límite adicionales. Por ejemplo, no prueba específicamente cuando el número de rutas con retorno es mayor que el número de rutas totales, cuando los parámetros son NULL o cuando los conteos son negativos.
La IA sí incluyó condiciones para detectar estos problemas, pero no presentó pruebas específicas para comprobarlos. Por eso considero importante realizar pruebas adicionales y no confiar únicamente en que los seis casos del harness hayan sido superados.
También observé que la función auxiliar obtener_tamano() utiliza valores numéricos para representar los tipos de datos. Esta solución funciona para los casos establecidos, aunque la compatibilidad de tipos en C debe considerar las reglas reales del lenguaje y no solamente el tamaño de los tipos.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
