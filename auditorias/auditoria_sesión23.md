# Bitácora de Requerimientos y Auditoría IA — Sesión 23
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Etiquetas y Saltos Incondicionales  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul 
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Generador de etiquetas e instrucciones goto  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:Al desarrollar las funciones tuve que cuidar que las etiquetas se generaran de manera consecutiva, comenzando desde L0, y que el contador aumentara después de utilizar su valor. También tuve que considerar el tamaño del búfer para evitar escribir fuera de sus límites. Otro problema fue validar los punteros NULL para evitar errores y asegurar que la función dejara una cadena vacía cuando no recibiera una etiqueta válida.*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Trazabilidad del flujo de control |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[CONTEXTO:
Estoy trabajando en la materia Lenguajes y Autómatas II.
Necesito implementar dos funciones en lenguaje C para generar
etiquetas y representar saltos incondicionales.

ROL:
Actúa como un ingeniero de software especializado en sistemas
y compiladores.

ESPECIFICACIÓN:
Implementa las funciones:
void nueva_etiqueta(char *dst, int cap, int *contador);
void emitir_flujo(char *dst, int cap, const char *etiqueta, int es_salto);

La función nueva_etiqueta debe generar etiquetas consecutivas
con el formato L0, L1, L2, etc., utilizando el contador actual
y aumentándolo después.

La función emitir_flujo debe escribir GOTO Lx cuando es_salto
sea diferente de cero y Lx: cuando sea cero.

Si la etiqueta es NULL, debe producir una cadena vacía.
Se deben validar los punteros y la capacidad del búfer.
Utiliza snprintf para evitar escrituras fuera de los límites.

OUTPUT:
Entrega únicamente código C modular, incluyendo las funciones
solicitadas, sin utilizar main(), respetando las firmas originales
y evitando advertencias con -Wall -Wextra -Werror.]
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion23.h"
#include <stdio.h>

void nueva_etiqueta(char *dst, int cap, int *contador) {
    if (dst == NULL || contador == NULL || cap <= 0) {
        return;
    }

    int numero = *contador;
    *contador = numero + 1;

    (void)snprintf(dst, (size_t)cap, "L%d", numero);
}

void emitir_flujo(char *dst, int cap,
                  const char *etiqueta, int es_salto) {
    int resultado;

    if (dst == NULL || cap <= 0) {
        return;
    }

    if (etiqueta == NULL) {
        dst[0] = '\0';
    } else if (es_salto) {
        resultado = snprintf(dst, (size_t)cap,
                             "GOTO %s", etiqueta);
        (void)resultado;
    } else {
        resultado = snprintf(dst, (size_t)cap,
                             "%s:", etiqueta);
        (void)resultado;
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Saltos adelante y atrás estructurados

### Salida del harness
```text
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> 
La IA omitió considerar que cuando el contador es NULL y el búfer contiene información anterior, la función nueva_etiqueta termina sin limpiar el destino. Esto puede provocar que permanezca información anterior en la cadena. Lo identifiqué al revisar manualmente las condiciones de la función y comparar ambas implementaciones. Mi versión artesanal primero inicializa el destino como una cadena vacía y después verifica si el contador es NULL. De esta manera se evita conservar datos anteriores en ese caso especial.

---Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
