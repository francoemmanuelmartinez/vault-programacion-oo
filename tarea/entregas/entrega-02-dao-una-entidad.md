---
fuente: consigna de la entrega y correcciones del profesor (texto provisto por el Estudiante)
tags: [tarea, entrega, dao, jdbc, jpa, persistencia]
estado: corregida
---

# Entrega 2 — Implementación DAO de una entidad

## Consigna

> Tome una de las entidades simples de su problema. Escriba la clase que la
> representa. Escriba el DAO para persistir esa entidad utilizando JDBC o JPA
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

## Correcciones del profesor sobre la primera versión entregada

> *"Traten de mejorar los nombres de las clases: `modelPatient` es en realidad la clase `Patient` en el paquete `model` (algo similar con el `GenericDAO`).*
>
> *Implementen una excepción para el DAO."*

### Cómo se resolvió cada punto

1. **Nombres de clases y paquetes (`pág. 3 a 7 — Codigo_Limpio_Nombres_y_Estructura.pdf`, `pág. 18 a 22 — JPA_Hibernate_DAO.pdf`):**
   - `appNutritionist.Models.modelPatient` → **`appNutritionist.model.Patient`**
   - `Dao.GenericDAO` → **`appNutritionist.dao.DAO<T, ID>`** (con métodos `insert`, `findById`, `findAll`, `update`, `delete`)
   - `Dao.PatientDAO` → **`appNutritionist.dao.PatientDAO`** (agregando `findByLastName(String lastName)`)
   - `DaoImpl.PatientDAOImpl` → **`appNutritionist.dao.PatientDAOJPA`** (y consulta JPQL corregida a `"SELECT p FROM Patient p"`)
   - Además se aplicó la misma limpieza a todo el proyecto NetBeans: `Aliment`, `PatientView`, `AlimentView`, `PatientController`, `AlimentController` en los paquetes `model`, `view` y `controller`.

2. **Excepción propia para el DAO (`pág. 13 — JDBC_DAO_MySQL.pdf`, `pág. 20 a 22 — JPA_Hibernate_DAO.pdf`):**
   - Creada **`appNutritionist.dao.DaoException`** (`extends Exception`).
   - Todos los métodos de `DAO<T, ID>` y `PatientDAO` declaran `throws DaoException`.
   - En `PatientDAOJPA`, se captura `PersistenceException`, se hace `tx.rollback()` si la transacción está activa y se relanza envuelta en `new DaoException("...", e)`.

## Archivos finales (en `NutritionistApp` y en `Desktop\Entrega`)

- `src/appNutritionist/model/Patient.java`
- `src/appNutritionist/dao/DaoException.java`
- `src/appNutritionist/dao/DAO.java`
- `src/appNutritionist/dao/PatientDAO.java`
- `src/appNutritionist/dao/PatientDAOJPA.java`

## Estado: ✅ Corregida e integrada en `NutritionistApp`

## Mapeo a la rúbrica

| Criterio | Aporte de esta entrega |
|---|---|
| Implementación de DAO (JDBC/JPA) | `DAO<T, ID>`, `PatientDAO` y `PatientDAOJPA` completos |
| Manejo de Excepciones | Excepción propia *checked* (`DaoException`) envolviendo `PersistenceException` con `rollback()` |
| Documentación y Código Limpio | Paquetes en minúscula (`model`, `dao`, `view`, `controller`), clases en PascalCase sin prefijos redundantes y Javadoc en todas las clases y métodos públicos |

## Relacionado

- [[tutorial-dao-desde-cero]] — guía paso a paso de DAO en JPA y JDBC para el equipo
- [[entrega-01-interfaces-graficas]] — etapa anterior
- [[JDBC_DAO_MySQL]] — implementación de DAO con JDBC
- [[JPA_Hibernate_DAO]] — variante con JPA/Hibernate
- [[enunciado_rubrica]] — criterios de evaluación
- [[estado]] — estado general del proyecto
