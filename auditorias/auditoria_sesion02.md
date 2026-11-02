# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Gestión en GitHub Web y Nodos Dinámicos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Jose Alfredo Nahuat Canul  
**Matrícula:** 24531399
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lista enlazada de tokens en Heap  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gestión de memoria dinámica C99/Rust |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Pega aquí el prompt formal que redactaste para la IA]
```

---

## 3. Código Devuelto por el Modelo
```c
// Código devuelto por el LLM
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Prevención de Use-After-Free y cadenas nulas

### Salida del harness
```text
Pega aquí la salida de la celda del harness. Debe terminar con:
RESULTADO: n/n pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
>
Al revisar el código del Artesano, identifiqué que no contempla la validación del apuntador lexema cuando su valor es NULL. Esto podría provocar un error al utilizar la función strncpy. Para solucionarlo, se puede agregar una condición que verifique que el apuntador no esté vacío antes de copiar la cadena. También comprobé que el programa pueda agregar elementos cuando la lista está vacía, contar correctamente los tokens y liberar la memoria utilizada para evitar fugas de memoria.

## Caso borde omitido por la IA
La IA omitió validar si el apuntador lexema era NULL antes de utilizar la función strncpy. Esto puede ocasionar un error durante la ejecución del programa. Para solucionarlo, agregué una condición que verifica si el apuntador contiene una cadena válida antes de copiarla. También revisé que la función pudiera trabajar con una lista vacía y que la memoria se liberara correctamente al finalizar el programa.


--- Jose Alfredo Nahuat Canul Unidad: Ignacio Zaragosa
**Firma digital:** [Usuario de GitHub]
