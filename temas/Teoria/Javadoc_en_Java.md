---
fuente: Javadoc_en_Java.pdf
tags: [tema, javadoc, practica, herramientas]
estado: completo
---

# Javadoc — Documentando el Código

> [!info] Fuente
> `Javadoc_en_Java.pdf` — pág. 1 a 13
> **Profesor:** Esteban Santini — ISFT 204

## Resumen

Javadoc es la herramienta oficial del JDK que genera documentación en HTML navegable a partir de comentarios especiales (`/** ... */`) escritos directamente en el código fuente. Constituye el contrato legible sobre qué hace cada clase, interfaz o método sin necesidad de abrir su implementación.

## Lo que dice el profesor

### 1. ¿Qué es Javadoc?
- Es la herramienta oficial del JDK que produce páginas HTML navegables (idénticas a la API oficial de Java) a partir de comentarios incrustados en el código. > pág. 2 — Javadoc_en_Java.pdf
- No es un comentario interno: es un **contrato legible** sobre el comportamiento del código para quien lo utiliza. > pág. 2 — Javadoc_en_Java.pdf

### 2. Javadoc vs. Comentarios Comunes
- **Comentarios comunes (`//` o `/*...*/`):** Explican decisiones internas al programador que lee el código (algoritmos complejos, decisiones de diseño, `TODO`). No siguen formato fijo y no generan artefactos externos. > pág. 3 — Javadoc_en_Java.pdf
- **Comentarios Javadoc (`/** ... */`):** Documentan la **API pública** (clases, métodos, atributos públicos/protegidos). Usan etiquetas estructuradas (`@param`, `@return`) y generan documentación HTML navegable. > pág. 3 — Javadoc_en_Java.pdf

### 3. Estructura y Etiquetas Básicas
- **Formato:** Inicia con `/**` y termina con `*/`. La primera oración es el resumen que aparece en las tablas de índice y debe terminar en punto. > pág. 6 — Javadoc_en_Java.pdf
- **Orden convencional de tags:** `@author`, `@param`, `@return`, `@throws` / `@exception`, `@see`, `@since`. > pág. 6 — Javadoc_en_Java.pdf
- **Etiquetas principales:**
  - `@param <nombre>`: Describe cada parámetro en el orden de la firma (obligatorio). > pág. 7 — Javadoc_en_Java.pdf
  - `@return`: Describe el valor devuelto (obligatorio salvo en `void`). > pág. 7 — Javadoc_en_Java.pdf
  - `@throws` / `@exception`: Documenta las excepciones lanzadas y bajo qué condición. > pág. 7 — Javadoc_en_Java.pdf
  - `@see`: Referencia cruzada a otra clase o método. > pág. 7 — Javadoc_en_Java.pdf
  - Etiquetado en línea (`{@code ...}`, `{@link ...}`): Dan formato de código o insertan hipervínculos internos. > pág. 8 — Javadoc_en_Java.pdf

### 4. Ejemplos en la práctica (Sistema de Biblioteca y librerías)
- En métodos como `public boolean tieneEjemplaresDisponibles()` o `public void registrarPrestamo(Libro libro)`, el Javadoc describe claramente la regla de negocio que implementan. > pág. 9 — Javadoc_en_Java.pdf
- Se aplica también en librerías estándar y de terceros (ej. Commons Lang `StringUtils`, Guava `Lists.partition`, JUnit 5 `assertThrows`). > pág. 10, 11 — Javadoc_en_Java.pdf

### 5. Cómo generar la documentación
- El comando `javadoc` del JDK recorre los `.java` y genera el sitio HTML. > pág. 12 — Javadoc_en_Java.pdf
- Integrado en IDEs (NetBeans / Eclipse: *Project ➔ Generate Javadoc*; IntelliJ: *Tools ➔ Generate JavaDoc*; Maven: `maven-javadoc-plugin`). > pág. 12 — Javadoc_en_Java.pdf

## Ampliación de la IA

> [!warning] Estándar de la industria
> Requisito de la rúbrica de la materia.

- La rúbrica del trabajo práctico de la materia (`enunciado_rubrica.pdf`) exige explícitamente Javadoc obligatorio en clases y métodos públicos. Sin Javadoc, la calificación en el criterio de *Documentación y Código Limpio* desciende a *Insuficiente*.

## Dudas y vacíos

- [ ] ¿Exige la cátedra exportar e incluir el sitio HTML generado por Javadoc en la entrega o alcanza con tener los comentarios `/** ... */` bien escritos en el código fuente?

## Relacionado

- [[Codigo_Limpio_Nombres_y_Estructura]]
- [[enunciado_rubrica]]
- [[estado]]

---

**Navegación:** [[index]] · [[MAP]]
