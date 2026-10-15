---
fuente: Errores_y_Excepciones_Java.pdf
tags: [poo, excepciones, errores, try, catch, throw, checked, unchecked]
estado: completo
---

# Errores y Excepciones en Java

> [!info] Fuente
> `Errores_y_Excepciones_Java.pdf` — pág. 1 a 11
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

El material aborda la gestión de problemas en tiempo de ejecución en Java. Distingue entre `Error` (fallas graves e irrecuperables de la JVM) y `Exception` (manejables mediante `try`/`catch`, divididas en *checked* y *unchecked* / `RuntimeException`), así como la creación de excepciones personalizadas y buenas prácticas.

---

## Lo que dice el profesor

### 1. Jerarquía: Error vs Exception

> pág. 2 y 3 — Errores_y_Excepciones_Java.pdf

- **`Error`:** Condición grave e irrecuperable (ej. `OutOfMemoryError`, `StackOverflowError`). Indica fallas de la JVM; no deben capturarse.
- **`Exception` (Manejable):**
  - **Checked:** El compilador obliga a manejarlas (`IOException`, `SQLException`).
  - **Unchecked (`RuntimeException`):** Subclases como `NullPointerException`, `IllegalArgumentException`. No es obligatorio capturarlas.

### 2. Ciclo de vida (Lanzamiento, Relanzamiento y Tratamiento)

> pág. 4 a 6 — Errores_y_Excepciones_Java.pdf

- **Lanzamiento (`throw` / `throws`):** Crear y enviar un objeto de excepción (`throw new IllegalArgumentException(...)`).
- **Relanzamiento:** Capturar y volver a lanzar (ej. traduciendo excepciones técnicas en excepciones de dominio).
- **Tratamiento (`try` / `catch` / `finally`):**
  - `try`: Código susceptible de fallar.
  - `catch`: Captura específica (del más específico al más general).
  - `finally`: Se ejecuta siempre (ideal para liberar recursos), o usar `try-with-resources`.

### 3. Excepciones personalizadas

> pág. 9 y 10 — Errores_y_Excepciones_Java.pdf

Para crear una excepción propia, se extiende `Exception` (checked) o `RuntimeException` (unchecked):
```java
public class SaldoInsuficienteException extends Exception {
    private final double monto;
    public SaldoInsuficienteException(double monto) {
        super("Saldo insuficiente: $" + monto);
        this.monto = monto;
    }
}
```

> pág. 11 — Errores_y_Excepciones_Java.pdf
> **Resumen del profesor:** "throw lanza · catch atrapa · finally siempre se ejecuta. Diseñá excepciones que comuniquen problemas reales y tratá solo lo que podés resolver."

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Buenas prácticas adicionales de logging.

- Es fundamental registrar (*log*) las excepciones con un framework como SLF4J o Log4j en lugar de solo imprimir `e.printStackTrace()` en aplicaciones de producción.

## Dudas y vacíos

- [ ] Ninguna.

## Relacionado

- [[JDBC_DAO_MySQL]]
- [[Codigo_Limpio_Nombres_y_Estructura]]

---

**Navegación:** [[index]] · [[MAP]]
