---
fuente: SOLID_Principios.pdf
tags: [practica, solid, diseño, principios, srp, ocp, lsp, isp, dip]
estado: completo
---

# Principios SOLID

> [!info] Fuente
> `SOLID_Principios.pdf` — pág. 1 a 34
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

SOLID es un acrónimo que agrupa cinco principios de diseño orientado a objetos formulados por Robert C. Martin ("Uncle Bob"). Su objetivo es reducir el acoplamiento, aumentar la cohesión y facilitar que el software crezca y se mantenga sin romper lo existente.

---

## Lo que dice el profesor

### 1. S — Single Responsibility Principle (Principio de Responsabilidad Única)

> pág. 4 a 9 — SOLID_Principios.pdf

- **Definición:** Una clase debe tener una única razón para cambiar (una sola responsabilidad claramente definida).
- **Code smells:** Clases con nombres genéricos (`Manager`, `Util`), dificultad para describirlas sin usar la "y" (calcula *y* guarda), mezcla de persistencia y UI.
- **Solución:** Delegar responsabilidades en clases especializadas (ej. separar `Factura` de `ImpresoraFactura` y `RepositorioFactura`).

### 2. O — Open / Closed Principle (Principio Abierto / Cerrado)

> pág. 10 a 15 — SOLID_Principios.pdf

- **Definición:** El software debe estar abierto a extensión pero cerrado a modificación (agregar comportamiento nuevo sin alterar el código existente).
- **Solución:** Uso de polimorfismo e interfaces (ej. `EstrategiaDescuento`) en lugar de largos bloques `if/else` o `switch`.

### 3. L — Liskov Substitution Principle (Principio de Sustitución de Liskov)

> pág. 16 a 21 — SOLID_Principios.pdf

- **Definición:** Los objetos de una subclase deben poder sustituir a los de su clase base sin alterar el comportamiento correcto del programa.
- **Code smells:** Métodos sobreescritos que lanzan `UnsupportedOperationException`, uso de `instanceof`.

### 4. I — Interface Segregation Principle (Principio de Segregación de Interfaces)

> pág. 22 a 27 — SOLID_Principios.pdf

- **Definición:** Ningún cliente debería verse obligado a depender de métodos que no usa. Es preferible tener varias interfaces específicas que una "gorda".
- **Solución:** Dividir interfaces grandes en contratos pequeños y cohesivos (`Trabajable`, `Alimentable`).

### 5. D — Dependency Inversion Principle (Principio de Inversión de Dependencias)

> pág. 28 a 33 — SOLID_Principios.pdf

- **Definición:** Los módulos de alto nivel no deben depender de módulos de bajo nivel; ambos deben depender de abstracciones. Las dependencias se inyectan (Inyección de Dependencias).
- **Code smells:** Uso directo de `new ClaseConcreta()` dentro de la lógica de negocio.

> pág. 34 — SOLID_Principios.pdf
> **Cierre:** "Diseñar con SOLID es diseñar para el cambio. Menos acoplamiento, mayor claridad, código que dura."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Conexión con frameworks modernos.

- Los principios SOLID son la base teórica de frameworks como Spring, donde la Inyección de Dependencias (DIP) y el polimorfismo se gestionan automáticamente mediante contenedores IoC.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[Codigo_Limpio_Nombres_y_Estructura]]
- [[Swing_MVC_Interfaces_Java]]

---

**Navegación:** [[index]] · [[MAP]]
