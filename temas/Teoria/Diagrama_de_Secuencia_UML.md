---
fuente: Diagrama_de_Secuencia_UML.pdf
tags: [tema, uml, teoria]
estado: completo
---

# Diagrama de Secuencia UML

> [!info] Fuente
> `Diagrama_de_Secuencia_UML.pdf` — pág. 1 a 20
> **Profesor:** Esteban Santini — ISFT 204

## Resumen

El Diagrama de Secuencia es un diagrama de comportamiento de UML que detalla cómo un conjunto de objetos colabora e intercambia mensajes a lo largo del tiempo para cumplir un caso de uso o escenario puntual. Actúa como puente entre el análisis (qué hace el sistema) y el diseño de clases (cómo lo hace).

## Lo que dice el profesor

### 1. De Casos de Uso a Secuencia
- **Casos de Uso (El Qué):** Muestran la funcionalidad completa desde afuera, sin entrar en detalles internos. Son una vista general estática y el punto de partida del análisis. > pág. 3 — Diagrama_de_Secuencia_UML.pdf
- **Secuencia (El Cómo):** Detalla paso a paso qué objetos colaboran, qué mensajes se envían y en qué orden. Es una vista detallada del interior de un caso de uso puntual y sirve de puente al diseño. > pág. 3 — Diagrama_de_Secuencia_UML.pdf

### 2. ¿Qué es y para qué sirve?
Es un diagrama de comportamiento ordenado en el tiempo: el eje vertical representa el tiempo que avanza de arriba hacia abajo. > pág. 4 — Diagrama_de_Secuencia_UML.pdf
- Detalla la lógica interna y el orden temporal exacto (condiciones y repeticiones). > pág. 5 — Diagrama_de_Secuencia_UML.pdf
- Ayuda a identificar las operaciones (métodos) que necesitará cada clase en el Diagrama de Clases. > pág. 5 — Diagrama_de_Secuencia_UML.pdf

### 3. Componentes del Diagrama
- **Objeto / Actor:** Participante de la interacción (actor externo o instancia de clase escrita como `:NombreClase`). > pág. 6 — Diagrama_de_Secuencia_UML.pdf
- **Línea de vida:** Línea vertical punteada que indica la existencia del objeto a lo largo del tiempo. > pág. 6 — Diagrama_de_Secuencia_UML.pdf
- **Foco de control:** Barra angosta sobre la línea de vida que muestra cuándo un objeto está activo ejecutando una operación. > pág. 6 — Diagrama_de_Secuencia_UML.pdf
- **Mensaje:** Flecha horizontal entre líneas de vida (llamada a un método o envío de información). > pág. 6 — Diagrama_de_Secuencia_UML.pdf

### 4. Tipos de mensajes y notación
- **Síncrono:** El emisor espera la respuesta antes de continuar. Línea llena y punta rellena (ej. `buscarSocio(id)`). > pág. 8, 9 — Diagrama_de_Secuencia_UML.pdf
- **De retorno:** Devuelve un valor al emisor. Línea punteada (ej. `return socio`). > pág. 8, 9 — Diagrama_de_Secuencia_UML.pdf
- **Asíncrono:** El emisor sigue su ejecución de inmediato sin esperar. Línea llena y punta abierta (ej. `notificarEvento()`). > pág. 8, 9 — Diagrama_de_Secuencia_UML.pdf
- **Autollamada:** Un objeto se invoca a sí mismo (ej. `validarDatos()`). > pág. 9 — Diagrama_de_Secuencia_UML.pdf
- **Creación / Destrucción:** Crea (`«create»`) o destruye (`«destroy»`) una instancia; la línea de vida nace o termina con una `X`. > pág. 8 — Diagrama_de_Secuencia_UML.pdf

### 5. Fragmentos combinados (`alt`, `opt`, `loop`)
Para flujos condicionales o repetitivos: > pág. 10 — Diagrama_de_Secuencia_UML.pdf
- **`alt` (Alternativa):** Ejecuta una de varias ramas con condición de guarda entre corchetes `[ ]`, separadas por línea punteada (`[else]`). > pág. 10, 11 — Diagrama_de_Secuencia_UML.pdf
- **`opt` (Opcional):** Una única rama que se ejecuta solo si se cumple la condición (equivalente a un `alt` sin `else`). > pág. 10 — Diagrama_de_Secuencia_UML.pdf
- **`loop` (Repetición):** Repite mensajes mientras se cumpla la condición. > pág. 10 — Diagrama_de_Secuencia_UML.pdf

### 6. MVC + DAO en el Diagrama de Secuencia
Cada línea de vida pertenece a una capa de la arquitectura: > pág. 12 — Diagrama_de_Secuencia_UML.pdf
- **Vista:** Recibe eventos del usuario, sin lógica de negocio.
- **Controlador:** Coordina, recibe peticiones e invoca al Modelo o a los DAO.
- **Modelo:** Aplica reglas de negocio (clases de dominio).
- **DAO:** Encapsula el acceso a datos sin exponer JDBC.
- **Regla de oro:** Los mensajes respetan las capas. La Vista nunca habla directamente a un DAO, y el Modelo no conoce a la Vista. > pág. 12 — Diagrama_de_Secuencia_UML.pdf

### 7. Ejemplos del sistema de biblioteca
- **Ejemplo 1 (Consultar Socio):** Flujo simple de solo lectura (Bibliotecario ➔ Vista ➔ Controlador ➔ SocioDAO ➔ Base de datos). > pág. 14, 15 — Diagrama_de_Secuencia_UML.pdf
- **Ejemplo 2 (Registrar Préstamo):** Flujo condicional con `alt`. El controlador busca el libro y el socio por sus DAO; si `libro.tieneEjemplaresDisponibles()` es verdadero, inserta el préstamo con `PrestamoDAO` y confirma; si no (`[else]`), muestra error. > pág. 16, 17 — Diagrama_de_Secuencia_UML.pdf
- **Del diagrama al código:** Cada mensaje del diagrama de secuencia se traduce casi literalmente en una línea de código del método del Controlador. > pág. 18 — Diagrama_de_Secuencia_UML.pdf

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Estándares de modelado UML y su relación con herramientas de diseño (como Enterprise Architect o PlantUML).

- Los diagramas de secuencia son fundamentales para generar stubs de métodos y verificar contratos de interfaces antes de codificar.
- En arquitecturas MVC, los controladores actúan como orquestadores centrales que reciben llamadas de la Vista y distribuyen trabajo entre Dominio y Persistencia.

## Dudas y vacíos

- [ ] ¿Exige la cátedra herramientas CASE formales para dibujar los diagramas en la entrega final o se aceptan esquemas limpios?

## Relacionado

- [[Diagrama_Casos_de_Uso_UML]]
- [[DiagramaClases_Ej2_Biblioteca]]
- [[Swing_MVC_Interfaces_Java]]
- [[JDBC_DAO_MySQL]]

---

**Navegación:** [[index]] · [[MAP]]
