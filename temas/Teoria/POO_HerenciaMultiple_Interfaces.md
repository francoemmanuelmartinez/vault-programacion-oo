---
fuente: POO_HerenciaMultiple_Interfaces.pdf
tags: [poo, interfaces, herencia-multiple, contratos, implements]
estado: completo
---

# Herencia Múltiple e Interfaces

> [!info] Fuente
> `POO_HerenciaMultiple_Interfaces.pdf` — pág. 1 a 12
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

Java no permite herencia múltiple de clases para evitar el "Problema del Diamante" y la ambigüedad. La solución que adopta el lenguaje son las **interfaces**, que permiten a una clase implementar múltiples contratos simultáneamente (`implements`), definiendo el qué pero no el cómo.

---

## Lo que dice el profesor

### 1. El Problema del Diamante y la restricción en Java

> pág. 2 y 3 — POO_HerenciaMultiple_Interfaces.pdf

- **El problema:** Si dos clases padre implementan el mismo método y una clase hija hereda de ambas, el compilador no puede determinar qué versión heredar (ambigüedad).
- **Herencia simple en Java:** Una clase solo puede extender (`extends`) UNA clase padre directa.
- **La solución:** Las interfaces solo declaran contratos sin implementación propia conflictiva.

### 2. ¿Qué es una Interfaz?

> pág. 4 a 8 — POO_HerenciaMultiple_Interfaces.pdf

- **Definición:** Estructura que define un conjunto de métodos que cualquier clase implementante DEBE proveer.
- **Características:**
  - Métodos implícitamente `public abstract` (salvo `default` desde Java 8).
  - Atributos implícitamente `public static final` (constantes).
  - No tiene constructor ni estado de instancia propio.
- **Sintaxis:** `class Pato implements Volador, Nadador { ... }`

### 3. Cuadro comparativo

> pág. 10 — POO_HerenciaMultiple_Interfaces.pdf

| Característica | Hereda clase concreta | Hereda clase abstracta | Implementa interfaz |
|---|---|---|---|
| ¿Cuántas se pueden usar? | Solo una (`extends`) | Solo una (`extends`) | Múltiples (`implements`) |
| ¿Hereda implementación? | Sí, completa | Sí, parcial (métodos abstract) | No (salvo `default`) |
| ¿Obliga a implementar? | No | Sí, los métodos abstract | Sí, TODOS los métodos |
| ¿Tiene atributos de instancia? | Sí | Sí | No (solo constantes) |
| ¿Es instanciable directamente? | Sí | No | No |

### 4. Beneficios

> pág. 11 y 12 — POO_HerenciaMultiple_Interfaces.pdf

- **Desacoplamiento:** El cliente depende del contrato, no de la implementación concreta.
- **Herencia múltiple:** Comportamientos de múltiples fuentes sin el Problema del Diamante.
- **Testabilidad y Extensibilidad.**

> pág. 12 — POO_HerenciaMultiple_Interfaces.pdf
> **Las interfaces son el contrato del diseño:** "La clase implementa el acuerdo. La interfaz lo define."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Aspectos avanzados de interfaces.

- Desde Java 8, las interfaces admiten métodos `default` y `static` con cuerpo, y desde Java 9 admiten métodos `private`, rompiendo parcialmente la regla de que no tienen implementación.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[POO_Herencia]]
- [[POO_Polimorfismo]]
- [[Swing_MVC_Interfaces_Java]]

---

**Navegación:** [[index]] · [[MAP]]
