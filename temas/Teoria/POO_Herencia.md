---
fuente: POO_Herencia.pdf
tags: [poo, herencia, subclasses, superclass, abstract, override, super]
estado: completo
---

# Herencia

> [!info] Fuente
> `POO_Herencia.pdf` — pág. 1 a 14
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

La herencia es el mecanismo de reutilización más poderoso de la POO. Permite que una clase hija o subclase adquiera los atributos y métodos de una clase padre o superclase utilizando la palabra clave `extends`, modelando relaciones del tipo "ES UN". El material abarca herencia entre clases concretas, clases abstractas, visibilidad (`protected`), uso de `super` y tratamiento de métodos (sobreescritura total, parcial o sin sobreescritura).

---

## Lo que dice el profesor

### 1. Concepto y beneficios

> pág. 1 y 2 — POO_Herencia.pdf

- **Definición:** Mecanismo donde una subclase adquiere atributos y métodos de una superclase. Modela relaciones **"ES UN"** (un Auto *es un* Vehículo).
- **Beneficios:** Reutilización de código (evita duplicación), extensibilidad (agregar atributos/métodos sin modificar el padre), jerarquías claras y mantenibilidad.

### 2. Herencia entre clases concretas y Clases Abstractas

> pág. 3 a 7 — POO_Herencia.pdf

- **Clases concretas:** Pueden instanciarse directamente (`class Auto extends Vehiculo`).
- **Clases abstractas (`abstract class`):** No pueden instanciarse directamente. Definen una estructura común que las subclases deben completar. Pueden tener métodos concretos y métodos abstractos (`public abstract double calcularArea();`).

### 3. Visibilidad en herencia y la referencia `super`

> pág. 8 y 9 — POO_Herencia.pdf

- **`protected`:** Permite que la subclase acceda directamente al atributo del padre sin romper la encapsulación.
- **`super`:**
  1. En constructor: `super(args)` invoca al constructor del padre (debe ser la primera línea).
  2. En métodos: `super.metodo()` invoca la versión del método en el padre (fundamental para sobreescritura parcial).

### 4. Tratamiento de métodos en herencia

> pág. 10 a 13 — POO_Herencia.pdf

- **Sin sobreescritura (Herencia directa):** El método se hereda tal cual del padre.
- **Sobreescritura total (`@Override`):** Se reescribe el cuerpo del método desde cero, ignorando al padre.
- **Sobreescritura parcial (`@Override` + `super.metodo()`):** Se reutiliza el comportamiento del padre y se le agrega lógica propia.

> pág. 14 — POO_Herencia.pdf
> **La herencia es reutilizar y extender:** "Una subclase no repite lo que ya existe. Lo hereda, lo adapta y lo supera."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Conceptos adicionales sobre herencia en Java.

- Java soporta herencia simple de clases (una clase solo puede extender un `extends`), pero permite herencia múltiple de comportamiento mediante interfaces.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[POO_Abstraccion]]
- [[POO_HerenciaMultiple_Interfaces]]
- [[POO_Polimorfismo]]

---

**Navegación:** [[index]] · [[MAP]]
