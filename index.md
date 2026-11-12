---
tags: [indice, meta]
---

# índice — Catálogo de Temas

> Catálogo de las notas de `temas/`. Se actualiza cada vez que se procesa un PDF.
> Ver [[MAP]] para el índice de las fuentes originales.

## Estado

- PDFs en el vault: **21**
- Notas creadas: **21**
- Tarea: `NutritionistApp` — Entrega 1 (GUI) y Entrega 2 (DAO de `Patient` con JPA y `DaoException`) completadas y corregidas. Ver [[estado]].
- Rúbrica ya procesada: [[enunciado_rubrica]]. Antes era un bloqueante; ya no.

## Bloque 1 — Teoría y conceptos

- ✅ [[POO_Abstraccion]] — Abstracción
- ✅ [[POO_Metodos]] — Métodos
- ✅ [[POO_Herencia]] — Herencia
- ✅ [[POO_HerenciaMultiple_Interfaces]] — Interfaces y herencia múltiple
- ✅ [[POO_Polimorfismo]] — Polimorfismo
- ✅ [[POO_Equals]] — `equals`, `hashCode`, identidad
- ✅ [[Errores_y_Excepciones_Java]] — Errores y excepciones en Java
- ✅ [[POO_Repaso_y_Diagrama_de_Clases]] — Repaso y diagrama de clases
- ✅ [[SOLID_Principios]] — Principios SOLID
- ✅ [[Codigo_Limpio_Nombres_y_Estructura]] — Código limpio, nombres y estructura
- ✅ [[Diagrama_Casos_de_Uso_UML]] — Diagrama de casos de uso UML
- ✅ [[DiagramaClases_Ej2_Biblioteca]] — Diagrama de clases — Ejercicio biblioteca
- ✅ [[Diagrama_de_Secuencia_UML]] — Diagrama de secuencia UML
- ✅ [[Javadoc_en_Java]] — Javadoc y documentación en Java

## Bloque 2 — Práctica y herramientas

- ✅ [[Swing_MVC_Interfaces_Java]] — Swing, MVC e interfaces
- ✅ [[Listas_y_ComboBox_Swing]] — Listas y ComboBox en Swing
- ✅ [[JTable_en_Swing]] — JTable
- ✅ [[Checkbox_y_RadioButton_Swing]] — Checkbox y RadioButton
- ✅ [[JDBC_DAO_MySQL]] — JDBC, DAO y MySQL
- ✅ [[JPA_Hibernate_DAO]] — Mapeo objeto-relacional con JPA, Hibernate y DAO

## Tarea

### Entregas (metodología incremental)

- ✅ [[entrega-01-interfaces-graficas]] — GUI de dos entidades (`Patient` y `Aliment`)
- ✅ [[entrega-02-dao-una-entidad]] — DAO de una entidad (`Patient` con JPA y `DaoException`)

### Notas de trabajo

- ✅ [[estado]] — Estado general del código y de la tarea
- ✅ [[hoja-de-ruta-7-a-9]] — Checklist de bloques 0 a G (incluye la escalada de 7–9 a 10) con los PDFs de la cátedra para cada subtarea
- ✅ [[tutorial-dao-desde-cero]] — Guía paso a paso del patrón DAO (JPA y JDBC) para el equipo
- ✅ [[ejemplo-de-excepciones]] — Mensajes personalizados por método del DAO (`DaoException`) y su llegada al usuario
- ✅ [[plan-cableado-patient]] — Plan de cableado Vista ↔ Controlador ↔ Modelo para `Patient` (paso a paso, con código y trampas)
- ✅ [[plan-cableado-aliment]] — El mismo plan aplicado a `Aliment`: getters, `AlimentDAOInMemory`, `AlimentTableModel`, controlador y trampas
- ✅ [[plan-cableado-diet]] — El tercer cableado: ajustar la `DietView` que hizo Caro (§2 bis), `DietTableModel`, `DietController` completo con el panel de alimentos (§4 bis), la navegación desde el botón Diets y el JPQL de `DietDAOJPA`
- ✅ [[plan-crud-bd]] — Plan para dejar el CRUD de `Patient` y `Aliment` andando contra MySQL: jars, esquema, los 5 JPQL, 25 pruebas y commit
- ✅ [[vistas-manuales]] — Contingencia: vistas Swing a mano (compilan) y
  alternativa de `setVisible(false)` a `dispose()`
- ✅ [[convenciones]] — Convenciones de código: getters para exponer los
  componentes al controlador, abreviaturas, Javadoc
- ✅ [[bug-jpql-dao]] — Bug de las consultas JPQL de los DAO: nombres de tabla en vez de entidades (5 consultas, archivo y línea exactos, estado al `1f785b6`)
- ✅ [[enunciado_rubrica]] — Enunciado y rúbrica del trabajo práctico

## Navegación

[[ME]] · [[MAP]] · [[SKILLS]] · `log.md`