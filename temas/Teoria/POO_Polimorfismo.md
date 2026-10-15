---
fuente: POO_Polimorfismo.pdf
tags: [poo, polimorfismo, overriding, overloading, tiempo-ejecucion, tiempo-compilacion]
estado: completo
---

# Polimorfismo

> [!info] Fuente
> `POO_Polimorfismo.pdf` — pág. 1 a 15
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

El polimorfismo ("muchas formas") es el cuarto pilar de la POO. Permite que un mismo mensaje produzca múltiples comportamientos según el objeto receptor en tiempo de ejecución. El material distingue entre polimorfismo en tiempo de compilación (Sobrecarga / Overloading) y en tiempo de ejecución (Sobreescritura / Overriding), así como su aplicación mediante clases abstractas e interfaces.

---

## Lo que dice el profesor

### 1. Concepto fundamental

> pág. 2 y 3 — POO_Polimorfismo.pdf

- **Definición:** Capacidad de un objeto de tomar distintas formas. Un mismo método puede comportarse de manera diferente según el tipo real del objeto que lo ejecuta en tiempo de ejecución.
- **Relación con Herencia:** Si `B` extiende `A`, entonces `B` es un `A`. Esto habilita tratar un `B` como si fuera un `A` (ej. `Figura f = new Circulo(5);`).

### 2. Tipos de polimorfismo en Java

> pág. 5 a 10 — POO_Polimorfismo.pdf

1. **Tiempo de compilación (Sobrecarga / Overloading):**
   - Múltiples métodos con el mismo nombre pero diferente firma (parámetros).
   - Se resuelve en compilación.
2. **Tiempo de ejecución (Sobreescritura / Overriding):**
   - Una subclase redefine un método heredado (`@Override`).
   - Se resuelve en ejecución por la JVM (*dynamic dispatch*).

### 3. Polimorfismo con Interfaces

> pág. 11 a 13 — POO_Polimorfismo.pdf

- Permite polimorfismo sin herencia de clase (ej. `List<String> lista = new ArrayList<>();` o `LinkedList<>();`). El código cliente no cambia ante distintas implementaciones.

### 4. Beneficios

> pág. 14 — POO_Polimorfismo.pdf

- Código reutilizable, extensibilidad, bajo acoplamiento y cumplimiento del **Principio Abierto/Cerrado** (abierto para extensión, cerrado para modificación).

> pág. 15 — POO_Polimorfismo.pdf
> **Resumen del profesor:** "Un programa polimórfico está preparado para el futuro."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Detalles técnicos de enlace estático vs dinámico.

- El enlace estático (*early binding*) ocurre en la sobrecarga (compilación), mientras que el enlace dinámico (*late binding* / *dynamic method dispatch*) ocurre en la sobreescritura (ejecución).

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[POO_Abstraccion]]
- [[POO_Herencia]]
- [[POO_HerenciaMultiple_Interfaces]]

---

**Navegación:** [[index]] · [[MAP]]
