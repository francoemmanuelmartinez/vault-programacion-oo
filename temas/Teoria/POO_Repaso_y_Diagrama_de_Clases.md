---
fuente: POO_Repaso_y_Diagrama_de_Clases.pdf
tags: [poo, repaso, clases, objetos, uml, diagrama-clases]
estado: completo
---

# Repaso de POO y Diagrama de Clases UML

> [!info] Fuente
> `POO_Repaso_y_Diagrama_de_Clases.pdf` — pág. 1 a 15
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

El material repasa los conceptos fundamentales de la Programación Orientada a Objetos (clase, objeto, instancia, atributos, estado, métodos y comportamiento) e introduce la notación estándar en UML para el Diagrama de Clases (rectángulo de tres compartimentos: nombre, atributos y métodos con su respectiva visibilidad y tipos).

---

## Lo que dice el profesor

### 1. Repaso de Conceptos Fundamentales

> pág. 2 a 8 — POO_Repaso_y_Diagrama_de_Clases.pdf

- **Clase:** Plantilla o molde (analógicamente, el plano de un edificio) que define la estructura y comportamiento común.
- **Objeto / Instancia:** Entidad concreta creada a partir de una clase que ocupa memoria y posee un estado particular.
- **Atributos y Estado:** Características o propiedades (`visibilidad nombre : tipo`). El estado es la foto de los valores de los atributos en un instante dado.
- **Métodos y Comportamiento:** Acciones u operaciones que el objeto sabe realizar (`visibilidad nombre(parámetros) : tipoDeRetorno`).

### 2. Diagrama de Clases en UML

> pág. 9 a 14 — POO_Repaso_y_Diagrama_de_Clases.pdf

- **Estructura del rectángulo UML:**
  1. **Compartimento superior:** Nombre de la clase (centrado, en negrita, sustantivo en singular).
  2. **Compartimento intermedio:** Atributos (`- nombre : String`, `- edad : int`). Privado `-`, Público `+`, Protegido `#`.
  3. **Compartimento inferior:** Métodos (`+ saludar() : void`).

> pág. 15 — POO_Repaso_y_Diagrama_de_Clases.pdf
> **Cierre de clase:** "Modelar con claridad hoy es programar con claridad mañana."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Estándares de modelado UML adicionales.

- En UML avanzado, los atributos estáticos (`static`) se subrayan, y los métodos abstractos se escriben en cursiva.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[POO_Abstraccion]]
- [[POO_Metodos]]
- [[DiagramaClases_Ej2_Biblioteca]]

---

**Navegación:** [[index]] · [[MAP]]
