---
fuente: POO_Abstraccion.pdf
tags: [poo, abstraccion, clases, atributos, metodos]
estado: completo
---

# Abstracción

> [!info] Fuente
> `POO_Abstraccion.pdf` — pág. 1 a 7
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

La abstracción es uno de los pilares de la Programación Orientada a Objetos. Consiste en quedarse con lo que importa para el sistema e ignorar el resto, permitiendo modelar objetos del mundo real simplificando su complejidad mediante atributos y métodos relevantes.

---

## Lo que dice el profesor

### 1. El concepto de Abstracción y el ejemplo del automóvil

> pág. 2 — POO_Abstraccion.pdf

Todos usamos un automóvil sin saber cómo funciona el motor internamente. Para manejarlo, solo necesitamos conocer algunas características (marca, modelo, año, color) y las acciones que podemos realizar (arrancar, acelerar, frenar, apagar).

> La Abstracción es exactamente eso: quedarse con lo que importa para el sistema e ignorar el resto. No modelamos la bomba de nafta ni los pistones: esos son detalles de implementación.
> 
> **Pregunta clave:** ¿Qué necesita saber el sistema sobre este objeto para cumplir su función?

### 2. Del objeto real a la clase

> pág. 3 — POO_Abstraccion.pdf

| Objeto Real (Auto) | Clase Abstraída (`Auto.java`) |
|---|---|
| **Lo que existe:** Motor, pistones, correa de distribución, inyectores, caja de cambios... | **Lo que modelamos:** Ignoramos los detalles mecánicos internos del auto. |
| **Datos identificatorios:** Marca, modelo, año de fabricación, color, número de patente... | **Atributos:** `marca: String`, `modelo: String`, `anio: int`, `color: String` |
| **Acciones posibles:** Arrancar, frenar, acelerar, cambiar marcha, encender luces... | **Métodos:** `arrancar()`, `frenar()`, `acelerar(int vel)` |

### 3. Diagrama de clases UML y visibilidad

> pág. 4 y 5 — POO_Abstraccion.pdf

- **Atributos (privados `-`):** `- marca : String`, `- modelo : String`, `- anio : int`, `- color : String`. Solo accesibles dentro de la clase.
- **Métodos (públicos `+`):** `+ arrancar() : void`, `+ frenar() : void`, `+ acelerar(vel : int) : void`, `+ apagar() : void`. Accesibles desde fuera.

> **La Regla de Oro:** Solo modelamos lo que el sistema necesita. La abstracción elimina la complejidad innecesaria.

### 4. Implementación en Java

> pág. 6 — POO_Abstraccion.pdf

```java
public class Auto {
    // Atributos (privados)
    private String marca;
    private String modelo;
    private int anio;
    private String color;

    // Métodos (públicos)
    public void arrancar() { ... }
    public void frenar() { ... }
    public void acelerar(int vel) { ... }
    public void apagar() { ... }
}
```

> pág. 7 — POO_Abstraccion.pdf
> **Abstraer es decidir qué importa:** "No modelamos todo el objeto real, solo lo que el sistema necesita conocer." La Abstracción nos permite construir modelos simples de un mundo complejo. Es el primer paso para pensar en Objetos.

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Conceptos adicionales de abstracción en Java y diseño OO.

- En Java, la abstracción también se manifiesta a nivel estructural mediante clases abstractas (`abstract class`) y contratos (`interface`), aunque aquí el profesor introduce el concepto fundamental de abstracción como modelado de entidades.

## Dudas y vacíos

- [ ] Ninguna detectada para este tema introductorio.

## Relacionado

- [[POO_Metodos]]
- [[POO_Herencia]]
- [[POO_Repaso_y_Diagrama_de_Clases]]

---

**Navegación:** [[index]] · [[MAP]]
