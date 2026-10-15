---
fuente: enunciado_rubrica.pdf
tags: [tarea, enunciado, rubrica, evaluacion]
estado: completo
---

# Enunciado y Rúbrica del Trabajo Práctico Integrador

> [!info] Fuente
> `enunciado_rubrica.pdf` — pág. 1 y 2
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

El trabajo práctico integrador consiste en el desarrollo en equipo (tres integrantes) de un sistema orientado a objetos estructurado en capas utilizando el patrón MVC. Debe gestionar por lo menos **tres entidades**, donde **dos de ellas se combinan en una tercera**, persistidas en una base de datos SQL mediante el patrón DAO, con interfaz gráfica en Swing y manejo de excepciones, Javadoc y diagramas UML.

---

## Lo que dice el profesor

### 1. Visión general y restricciones generales

> pág. 1 — enunciado_rubrica.pdf

- **Modalidad:** Equipos de tres integrantes. Todos los integrantes deben poder defender oralmente todo lo desarrollado.
- **Problemática:** Identificar una problemática a resolver donde se requiera la realización de un pequeño sistema.
- **Entidades:** El proyecto permitirá gestionar por lo menos **tres entidades**, dos de ellas deben combinarse en una tercera.
- **Arquitectura:** Debe ser desarrollado en **capas** y utilizando **MVC**.
- **Autenticación:** *No* es objetivo del proyecto la implementación de un sistema de autenticación y autorización.

### 2. Elementos particulares

> pág. 1 — enunciado_rubrica.pdf

- **CRUD en capas:** Cada entidad debe tener su propio CRUD implementado en capas, aprovechando las características de la Programación Orientada a Objetos.
- **Persistencia (DAO):** Las entidades se deben persistir en una base de datos SQL utilizando el patrón **DAO**. El motor de bases de datos queda a elección de cada equipo.
- **Interfaz Gráfica (GUI):** Cada entidad debe tener por lo menos una pantalla que permita implementar CRUD a nivel de la vista. Se debe utilizar **Swing** como librería gráfica de base. Se pueden utilizar componentes gráficos externos que agreguen funcionalidad.
- **Excepciones:** Deben hacer uso y gestión de excepciones para asegurar el funcionamiento del programa y resolver los problemas que puedan ser resueltos con ellas.
- **Javadoc:** Será necesaria la realización de Javadoc para los métodos y clases públicas.
- **Diagramas UML:** Deberán desarrollar el diagrama de **Casos de Uso General** del sistema y el **diagrama de Secuencia** para un proceso a elección del equipo.

### 3. Criterios de evaluación y rúbrica

> pág. 1 y 2 — enunciado_rubrica.pdf

#### Requisitos por nota
1. **Nota 4:** Desarrollar la totalidad de lo expuesto anteriormente para **una entidad** y el **20% de la segunda**.
2. **Nota 7:** Desarrollar la totalidad para **dos entidades** y el **20% de la tercera**.
3. **Nota 10:** Desarrollar la totalidad para **las tres entidades** + incorporación de alguna característica extra (investigación) a lo trabajado en clase.

#### Rúbrica detallada (Criterios y Niveles)

| Criterio | Insuficiente (0-2) | Aceptable (3) | Bueno (4-6) | Excelente (7-9) |
|---|---|---|---|---|
| **Arquitectura en Capas y MVC** | No sigue el patrón MVC o no existe separación en capas. | Implementa MVC solamente en una entidad. | Implementa MVC con buena separación de capas en una entidad. En la segunda cumple parcial o completamente. | Excelente separación en capas en por lo menos tres entidades, con una clara y efectiva implementación de MVC. |
| **Interfaz Gráfica de Usuario (GUI)** | La GUI es confusa, presenta errores, o no cumple con los requisitos mínimos. | La GUI es funcional para una entidad, pero poco intuitiva o presenta errores menores. | La GUI es funcional y clara para una entidad. En la segunda cumple una interfaz parcial o totalmente desarrollada. | La GUI es intuitiva, estéticamente agradable y funcional para todas las entidades sin errores. |
| **Implementación de DAO (JDBC/JPA)** | No se usa DAO, o la implementación no es funcional o es incorrecta. | Implementación básica de DAO para una entidad, con algunos problemas en el uso de JDBC o JPA. | Implementación correcta de DAO para una entidad, con buena funcionalidad y manejo de JDBC o JPA. | Excelente implementación de DAO para todas las entidades, con un diseño bien estructurado y buenas prácticas de JDBC o JPA. |
| **Manejo de Excepciones** | Sin manejo de excepciones o manejo inadecuado, resultando en errores no controlados o caídas. | No se realiza resolución de excepciones. Solamente el bloque try tiene contenido relevante. | Buen manejo de excepciones para una entidad, con mensajes de error claros y manejo adecuado de errores críticos. | Excelente manejo de excepciones en todas las entidades, con excepciones personalizadas y mensajes claros que mejoran la experiencia. |
| **Documentación y Código Limpio** | Código difícil de entender, sin comentarios ni convenciones de estilo. | Código comprensible para una entidad, pero con documentación y estilo mejorables. | Buen estilo y documentación en el código de una entidad, con comentarios explicativos y nombres descriptivos. | Código bien estructurado y documentado en todas las entidades, fácil de leer, con buenas prácticas de programación. |

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Conexión con el proyecto actual (`NutritionistApp`) y estrategia para alcanzar la Nota 10.

- **Entidades actuales en `NutritionistApp`:** `Patient` y `Aliment`.
- **Tercera entidad faltante:** Para cumplir con la regla de que *dos entidades deben combinarse en una tercera*, el sistema de nutrición necesita una entidad como `Diet` (Dieta) o `MealPlan` / `PatientAliment` que relacione a un paciente con los alimentos que consume, sus porciones y calorías totales.
- **Estrategia DAO + JDBC:** Implementar conexión MySQL mediante `java.sql` siguiendo `JDBC_DAO_MySQL.pdf`.
- **Diagramas UML:** Generar Casos de Uso y Secuencia usando herramientas estándar (PlantUML, Draw.io o Mermaid).

---

## Dudas y vacíos

- [ ] ¿El profesor exige un motor SQL específico (ej. MySQL Workbench con XAMPP) o es libre? (El enunciado dice: *El motor de bases de datos queda a elección de cada equipo*).
- [ ] ¿Qué característica extra se puede implementar para asegurar el 10 (ej. reportes PDF, gráficos estadísticos con JFreeChart o encriptación)?

## Relacionado

- [[00-estado]]
- [[03-vistas-manuales]]
- [[JDBC_DAO_MySQL]]
- [[Swing_MVC_Interfaces_Java]]

---

**Navegación:** [[index]] · [[MAP]]
