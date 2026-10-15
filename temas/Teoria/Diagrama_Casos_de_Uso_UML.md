---
fuente: Diagrama_Casos_de_Uso_UML.pdf
tags: [practica, uml, casos-de-uso, requisitos, analisis]
estado: completo
---

# Diagrama de Casos de Uso UML

> [!info] Fuente
> `Diagrama_Casos_de_Uso_UML.pdf` — pág. 1 a 16
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

El material introduce el modelado del comportamiento del sistema desde la perspectiva del usuario mediante los Diagramas de Casos de Uso en UML. Explica sus componentes (actores, casos de uso, límite del sistema) y relaciones avanzadas como `«include»` y `«extend»`, aplicados al caso de estudio de una biblioteca.

---

## Lo que dice el profesor

### 1. Concepto y objetivos

> pág. 3 y 4 — Diagrama_Casos_de_Uso_UML.pdf

- **Caso de uso:** Secuencia de acciones que realiza un sistema para producir un resultado de valor para un actor externo. Describe el *qué* y no el *cómo*.
- **Propósito:** Relevar requisitos funcionales, servir de puente entre usuario y desarrollador, y delimitar el alcance del sistema.

### 2. Componentes y notación gráfica

> pág. 7 y 8 — Diagrama_Casos_de_Uso_UML.pdf

- **Actor:** Rol externo (persona, dispositivo, otro sistema) representado por un monigote de palitos.
- **Caso de uso:** Elipse con un verbo en infinitivo (`Gestionar Socios`, `Registrar Préstamo`).
- **Límite del sistema:** Rectángulo que delimita lo que pertenece al sistema.
- **Asociación:** Línea continua sin flecha entre actor y caso de uso.

### 3. Relaciones avanzadas (`«include»` y `«extend»`)

> pág. 12 y 13 — Diagrama_Casos_de_Uso_UML.pdf

- **`«include»` (Inclusión):** Un caso de uso base *siempre* incorpora la ejecución de otro (ej. las operaciones CRUD incluidas en "Gestionar Socios").
- **`«extend»` (Extensión):** Un caso de uso opcional agrega comportamiento adicional a otro bajo ciertas condiciones (ej. "Enviar bienvenida por email" al registrar un socio).

> pág. 16 — Diagrama_Casos_de_Uso_UML.pdf
> **Cierre:** "Comprender qué necesita el usuario es el primer paso para diseñar cualquier sistema."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Especificación de casos de uso (UCE).

- En proyectos grandes, cada caso de uso gráfico suele estar acompañado de una especificación textual detallada (curso normal de eventos, flujos alternativos, precondiciones y postcondiciones).

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[enunciado_rubrica]]
- [[DiagramaClases_Ej2_Biblioteca]]

---

**Navegación:** [[index]] · [[MAP]]
