---
fuente: POO_Equals.pdf
tags: [poo, equals, objeto, comparacion, overriding]
estado: completo
---

# Método Equals

> [!info] Fuente
> `POO_Equals.pdf` — pág. 1 a 8
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

El método `equals` compara el estado interno de dos objetos en lugar de sus referencias en memoria. Todas las clases heredan `equals` de `Object`, pero para comparar contenido por valor es necesario sobrescribirlo (`@Override`) siguiendo una estructura lógica rigurosa.

---

## Lo que dice el profesor

### 1. Firma y Objetivo

> pág. 2 — POO_Equals.pdf

- **Firma:** `public boolean equals(Object o)`
- **Objetivo:** Retorna `true` si ambos objetos son de la misma clase y absolutamente todos sus atributos tienen el mismo valor; `false` en cualquier otro caso.

### 2. Estructura lógica del método

> pág. 3 a 7 — POO_Equals.pdf

1. **Mismo objeto (`this`):** `if (o == this) return true;` (atajo rápido y eficiente).
2. **Referencia nula (`null`):** `if (o == null) return false;` (evita `NullPointerException`).
3. **Misma clase (`getClass()`):** `if (o.getClass() != this.getClass()) return false;` (asegura que se comparen objetos del mismo tipo).
4. **Castear y comparar atributos:**
   - Castear: `Person p = (Person) o;`
   - Primitivos: `if (this.age != p.age) return false;`
   - Objetos (`String`, etc.): `if (!this.firstName.equals(p.firstName)) return false;`

### 3. Ejemplo completo en Java

> pág. 4 — POO_Equals.pdf

```java
public class Person {
    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (o == null) return false;
        if (o.getClass() != this.getClass()) return false;
        Person p = (Person) o;
        if (!this.firstName.equals(p.firstName)) return false;
        if (!this.lastName.equals(p.lastName)) return false;
        if (this.age != p.age) return false;
        else return true;
    }
}
```

> pág. 8 — POO_Equals.pdf
> **Equals: comparar estado interno:** "Dos objetos son iguales cuando su estado interno es idéntico, no cuando ocupan el mismo lugar en memoria." Sin `equals()`, Java compara referencias (`==`), no contenido.

---

## Ampliación de la IA

> [!warning] **Contrato de equals y hashCode**
> En Java, si sobrescribís `equals`, **siempre** debés sobrescribir `hashCode`. Si dos objetos son iguales según `equals`, deben tener el mismo `hashCode`, de lo contrario colecciones basadas en hash (`HashMap`, `HashSet`) fallarán.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[POO_Abstraccion]]
- [[POO_Metodos]]

---

**Navegación:** [[index]] · [[MAP]]
