---
fuente: consigna de la entrega (texto provisto por el Estudiante)
tags: [tarea, entrega, dao, jdbc, jpa, persistencia]
estado: pendiente
---

# Entrega 2 — Implementación DAO de una entidad

## Consigna

> Tome una de las entidades simples de su problema. Escriba la clase que la
> representa. Escriba la el DAO para persistir esa entidad utilizando JDBC o JPA
> (según ustedes quieran).

**Archivos deseados:**
- Entidad
- Interfaz DAO usando genéricos
- Interfaz DAO especificada para la entidad
- Implementación del DAO con JDBC o JPA

**Archivos aceptables:**
- Entidad
- Interfaz DAO usando genéricos
- Implementación especificada de DAO para esa entidad usando JDBC o JPA

## Decisiones a tomar

- [ ] **Entidad**: ¿`Patient` o `Aliment`? (Aliment es más simple: 3 campos.)
- [ ] **Tecnología**: ¿JDBC ([[JDBC_DAO_MySQL]]) o JPA/Hibernate ([[JPA_Hibernate_DAO]])?
- [ ] **Alcance**: apuntar a la versión "deseada" (4 archivos): entidad +
  `DAO<T, ID>` genérico + DAO específico + implementación.

## Estado: ⬜ Pendiente

## Mapeo a la rúbrica

| Criterio | Aporte de esta entrega |
|---|---|
| Implementación de DAO (JDBC/JPA) | Núcleo de esta entrega |
| Manejo de Excepciones | Excepción propia (`DaoException`) y `try-with-resources` |
| Documentación y Código Limpio | Javadoc en clases y métodos públicos |

## Relacionado

- [[entrega-01-interfaces-graficas]] — etapa anterior
- [[JDBC_DAO_MySQL]] — implementación de DAO con JDBC
- [[JPA_Hibernate_DAO]] — variante con JPA/Hibernate
- [[enunciado_rubrica]] — criterios de evaluación
- [[estado]] — estado general del proyecto
