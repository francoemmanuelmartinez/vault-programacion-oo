---
fuente: POO_Metodos.pdf
tags: [poo, metodos, constructores, getters, setters, visibilidad]
estado: completo
---

# Métodos

> [!info] Fuente
> `POO_Metodos.pdf` — pág. 1 a 10
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

Un método es un bloque de código que define el comportamiento de un objeto y representa una acción que puede realizar sobre sus propios datos o sobre el mundo exterior. El material detalla su estructura, la firma, los niveles de visibilidad y su clasificación en métodos del ciclo de vida, encapsulación y funcionalidad.

---

## Lo que dice el profesor

### 1. Concepto y firma del método

> pág. 2 y 3 — POO_Metodos.pdf

- **Definición:** Bloque de código que define el comportamiento de un objeto. Pertenece a una clase, tiene un nombre, puede recibir parámetros, puede retornar un valor y tiene visibilidad.
- **Estructura de la firma:** `[visibilidad] tipo_retorno nombre([parámetros])`

Ejemplos de firma en Java:
```java
// Sin parámetros, sin retorno:
public void arrancar() {
    System.out.println("El auto arrancó.");
}

// Con retorno:
public String getMarca() {
    return this.marca;
}

// Con parámetro:
public void setAnio(int a) {
    if (a > 1885) this.anio = a;
}
```

### 2. Tipos de visibilidad

> pág. 4 — POO_Metodos.pdf

- **`private`:** Accesible solo dentro de la propia clase. El nivel más restrictivo, usado para atributos.
- **`public`:** Accesible desde cualquier clase. Usado para constructores, getters, setters y métodos de comportamiento.
- **`protected`:** Accesible desde la propia clase y sus subclases (herencia).
- **`package` (default):** Sin modificador explícito. Accesible desde clases del mismo paquete.

### 3. Clasificación de métodos

> pág. 5 a 9 — POO_Metodos.pdf

1. **Del ciclo de vida (Constructores):** Controlan el nacimiento del objeto.
   ```java
   // Constructor por defecto
   public Auto() {
       this.marca = "Sin marca";
       this.anio = 2000;
   }
   // Constructor con parámetros
   public Auto(String m, int a) {
       this.marca = m;
       this.anio = a;
   }
   ```
   *Nota sobre destructores:* En Java no hay destructores explícitos; el **Garbage Collector (GC)** libera memoria automáticamente. El método `finalize()` está obsoleto desde Java 9 y eliminado en Java 18.

2. **De encapsulación (Getters y Setters):** Controlan el acceso y modificación de atributos privados con validación.
   ```java
   public String getMarca() { return this.marca; }
   public void setMarca(String m) {
       if (m != null) this.marca = m;
   }
   ```

3. **De funcionalidad:** Implementan el comportamiento real del objeto en el sistema (ej. `arrancar()`, `acelerar()`, `frenar()`).

> pág. 10 — POO_Metodos.pdf
> **Un método es una promesa de comportamiento:** "Un método bien escrito hace una sola cosa, la hace bien y su nombre lo dice todo." Los métodos son el contrato entre un objeto y el mundo. Un objeto sin métodos no tiene voz — solo existe, pero no actúa.

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Conceptos adicionales sobre sobrecarga de métodos (*method overloading*).

- Java permite la sobrecarga de métodos (varios métodos con el mismo nombre pero distinta firma/parámetros), muy común en constructores.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[POO_Abstraccion]]
- [[POO_Herencia]]
- [[POO_Polimorfismo]]

---

**Navegación:** [[index]] · [[MAP]]
