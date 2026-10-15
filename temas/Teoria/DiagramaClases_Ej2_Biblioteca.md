---
fuente: DiagramaClases_Ej2_Biblioteca.pdf
tags: [practica, uml, diagrama-clases, relaciones, asociacion, agregacion, composicion]
estado: completo
---

# Diagrama de Clases UML: Relaciones (Ejercicio Biblioteca)

> [!info] Fuente
> `DiagramaClases_Ej2_Biblioteca.pdf` — pág. 1 a 11
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

El material explica en detalle las relaciones entre clases en UML (asociación, agregación y composición) utilizando como caso de estudio el sistema de gestión de préstamos de una biblioteca universitaria con las clases `Libro` y `Socio`.

---

## Lo que dice el profesor

### 1. Tipos de relaciones en UML

> pág. 6 a 9 — DiagramaClases_Ej2_Biblioteca.pdf

- **Asociación:** Relación general donde un objeto conoce o utiliza a otro (`1` socio se asocia con `0..*` libros).
- **Agregación (Diamante vacío `◇`):** Relación "tiene" o "contiene" débil. La parte puede existir de forma independiente al todo (si se elimina un `Socio`, los `Libro` del catálogo siguen existiendo).
- **Composición (Diamante relleno `◆`):** Relación "parte de" fuerte. Si el todo se destruye, las partes también (ej. un `Pedido` y sus `ItemPedido`).

### 2. Caso de Estudio: Biblioteca Universitaria

> pág. 2 a 5, 10 — DiagramaClases_Ej2_Biblioteca.pdf

- **Clase `Libro`:** Atributos (`titulo`, `autor`, `anioPublicacion`, `ISBN`, `cantEjemplaresDisponibles`) y métodos (`tieneEjemplaresDisponibles()`, `registrarDevolucion()`).
- **Clase `Socio`:** Atributos (`nombre`, `nroSocio`, `correoElectronico`, `librosEnPrestamo : List<Libro>`) y métodos (`registrarPrestamo(libro)`, `registrarDevolucion(libro)`).
- Relación de **agregación** con multiplicidad `1` (Socio) a `0..*` (Libro).

> pág. 11 — DiagramaClases_Ej2_Biblioteca.pdf
> **Cierre:** "Identificar relaciones entre clases es el primer paso para diseñar sistemas robustos y mantenibles."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Navegabilidad de asociaciones.

- En UML, las asociaciones pueden ser unidireccionales (con una flecha en un extremo) o bidireccionales (sin flecha, ambas clases se conocen), lo cual impacta directamente en cómo se diseñan los atributos de navegación en Java.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[POO_Repaso_y_Diagrama_de_Clases]]
- [[Diagrama_Casos_de_Uso_UML]]
- [[enunciado_rubrica]]

---

**Navegación:** [[index]] · [[MAP]]
