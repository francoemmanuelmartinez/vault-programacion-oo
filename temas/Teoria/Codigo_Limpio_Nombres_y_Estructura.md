---
fuente: Codigo_Limpio_Nombres_y_Estructura.md (Codigo_Limpio_Nombres_y_Estructura.pdf)
tags: [practica, codigo-limpio, clean-code, nombres, funciones]
estado: completo
---

# Código Limpio: Nombres y Estructura

> [!info] Fuente
> `Codigo_Limpio_Nombres_y_Estructura.pdf` — pág. 1 a 13
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

Basado en las prácticas de Robert C. Martin ("Uncle Bob"), el material enfatiza que la legibilidad es la característica más importante del software, dado que el código se lee mucho más de lo que se escribe. Se establecen reglas para nombres con intención, funciones pequeñas de una sola responsabilidad y estructura vertical legible como un diario.

---

## Lo que dice el profesor

### 1. El concepto de Código Limpio y la Regla del Boy Scout

> pág. 2 — Codigo_Limpio_Nombres_y_Estructura.pdf

- **Legibilidad:** El código se lee más de lo que se escribe. Debe ser fácil de entender sin adivinar.
- **Regla del Boy Scout:** "Dejá el código un poco mejor de como lo encontraste."

### 2. Reglas para nombrar

> pág. 3 a 7 — Codigo_Limpio_Nombres_y_Estructura.pdf

- Revelar intención (`int elapsedTimeInDays;` en lugar de `int d;`).
- Nombres pronunciables y buscables.
- Un nombre por concepto (ej. usar siempre `get` o `fetch`, sin mezclar).
- Clases y atributos como sustantivos (`Cliente`, `Pedido`); métodos como verbos (`calcularTotal()`).
- Evitar notación húngara y prefijos (`m_`, `str`).

### 3. Funciones limpias

> pág. 8 a 10 — Codigo_Limpio_Nombres_y_Estructura.pdf

- **Una sola cosa:** Las funciones deben ser pequeñas y hacer una sola tarea (si en la descripción aparece un "y", hace dos cosas).
- Pocos argumentos (ideal 0 o 1, evitar booleanos como parámetros *flags*).
- Sin efectos secundarios. Comando o consulta, nunca ambos.

### 4. Estructura vertical del código

> pág. 11 a 13 — Codigo_Limpio_Nombres_y_Estructura.pdf

- Un archivo se lee como un diario: funciones de alto nivel arriba, detalles y funciones auxiliares abajo.

> pág. 13 — Codigo_Limpio_Nombres_y_Estructura.pdf
> **Mensaje final:** "El código limpio siempre parece que fue escrito por alguien a quien le importa."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Herramientas de análisis estático.

- Lintas y formateadores automáticos (como Checkstyle, PMD o Prettier/Google Java Format) ayudan a mantener estas convenciones de forma automática en el equipo de desarrollo.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[SOLID_Principios]]
- [[enunciado_rubrica]]

---

**Navegación:** [[index]] · [[MAP]]
