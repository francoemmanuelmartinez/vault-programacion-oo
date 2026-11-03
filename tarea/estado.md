---
tags: [tarea, estado, auditoria]
fuente: inspección directa del repositorio
aliases: [00-estado]
---

# Estado actual del código

> Actualizado el 2026-11-03 tras aplicar las correcciones de la Entrega 2 y reestructurar paquetes y nombres de clases.
> **No** es contenido del profesor: es un relevamiento de lo que hay escrito.

## Ubicación

El código vive en el proyecto NetBeans:

```text
C:\Users\games\Desktop\Proyectos\NutritionistApp\NutritionistApp\
├── build.xml, manifest.mf, nbproject/    ← proyecto NetBeans (Ant)
├── lib/
│   └── javax.persistence-api-2.2.jar     ← librería JPA configurada en javac.classpath
└── src/appNutritionist/
    ├── AppNutritionist.java               ← main, cablea MVC de Patient
    ├── controller/  PatientController.java  AlimentController.java
    ├── model/       Patient.java            Aliment.java
    ├── dao/         DAO.java  DaoException.java  PatientDAO.java  PatientDAOJPA.java
    └── view/        PatientView.java        AlimentView.java (+ .form)
```

- **Java:** `javac.source=25` / `javac.target=25`. JDK instalado: **26**.
  Ojo: compilar a mano **siempre** con `--release 25`, si no los `.class`
  salen versión 70 y NetBeans (JDK 25) no los puede cargar.
- **Entry point:** `appNutritionist.AppNutritionist`
- **Git:** la fuente de verdad es `log.md`, y el historial real está en el proyecto con `git log`.

## Inventario

| Archivo | Estado |
|---|---|
| `AppNutritionist.java` | Funciona. Instancia `Patient` → `PatientView` → `PatientController` |
| `model/Patient.java` | **Completa.** Entidad JPA (`@Entity`, `@Table(name="Patients")`, `@Id`, `@GeneratedValue`) con `id`, `name`, `lastName`, `weight`, `height`, `observations`, constructores, getters/setters y Javadoc |
| `model/Aliment.java` | **Vacía.** Pendiente de completar atributos |
| `dao/DaoException.java` | **Completa.** Excepción *checked* (`extends Exception`) con Javadoc |
| `dao/DAO.java` | **Completa.** Interfaz genérica `DAO<T, ID>` (`insert`, `findById`, `findAll`, `update`, `delete` con `throws DaoException`) |
| `dao/PatientDAO.java` | **Completa.** Extiende `DAO<Patient, Integer>` y declara `findByLastName` |
| `dao/PatientDAOJPA.java` | **Completa.** Implementación JPA con `EntityManager`, transacciones, `rollback()` y traducción de `PersistenceException` a `DaoException` |
| `controller/PatientController.java` | Cablea el modelo y la vista. Listener de navegación a `AlimentView` |
| `controller/AlimentController.java` | Cablea el modelo y la vista. Listener de navegación a `PatientView` |
| `view/PatientView.java` | GUI armada. Componentes privados; `getBtnAliment()` y `getBtnDiet()` con Javadoc |
| `view/AlimentView.java` | GUI armada. Componentes privados; `getBtnPatient()` y `getBtnDiet()` con Javadoc |

**Navegación entre vistas:** funcionando (`PatientView` ↔ `AlimentView` con `dispose()` y `SwingUtilities.invokeLater()`).

## Lo que falta, según lo que implican los PDFs

| Capa | Estado | PDF que la gobierna |
|---|---|---|
| Modelo `Patient` + DAO | ✅ Completo con JPA | `JPA_Hibernate_DAO.pdf`, `Errores_y_Excepciones_Java.pdf` |
| Modelo `Aliment` + DAO | ❌ Modelo vacío, sin DAO | `POO_Abstraccion.pdf`, `JPA_Hibernate_DAO.pdf` / `JDBC_DAO_MySQL.pdf` |
| Controllers (lógica CRUD) | ⚠️ Solo navegación, falta conectar botones CRUD al DAO | `Swing_MVC_Interfaces_Java.pdf` |
| Navegación entre vistas | ✅ `PatientView` ↔ `AlimentView` | `Swing_MVC_Interfaces_Java.pdf` |
| Entidad `Diet` (3ra entidad) | ❌ No existe | `enunciado_rubrica.pdf` |
| Diagramas UML | ❌ No hay | `Diagrama_Casos_de_Uso_UML.pdf`, `Diagrama_de_Secuencia_UML.pdf` |

## Riesgos verificados contra la rúbrica

1. **Nombres de clase y paquetes:** ✅ **Resuelto.** Se renombraron todos los paquetes a minúscula singular (`model`, `view`, `controller`, `dao`) y todas las clases a PascalCase sin prefijos redundantes (`Patient`, `Aliment`, `PatientView`, `AlimentView`, `PatientController`, `AlimentController`, `DAO`, `PatientDAO`, `PatientDAOJPA`, `DaoException`), tal como indicó el profesor en la corrección de la Entrega 2.
2. **Las vistas son código generado por el NetBeans GUI Builder.** Riesgo bajo; los getters públicos fuera de las secciones generadas ya tienen Javadoc según [[convenciones]].
3. **Java 25 con JDK 26 instalado.** Compilar siempre con `--release 25`.
4. **Ejecución real con base de datos (JPA vs JDBC):** Para compilar alcanza con `lib/javax.persistence-api-2.2.jar` (ya incluido), pero para conectar con MySQL en tiempo de ejecución hay que decidir si se agrega el stack de Hibernate/EclipseLink + `persistence.xml` o si se pasa la implementación a JDBC (`mysql-connector-j-9.6.0.jar`). Ver [[tutorial-dao-desde-cero]].

## Entregas (metodología incremental)

- ✅ [[entrega-01-interfaces-graficas]] — GUI de `Patient` y `Aliment`.
- ✅ [[entrega-02-dao-una-entidad]] — DAO de `Patient` corregido e integrado.

## Relacionado

[[convenciones]] · [[enunciado_rubrica]] · [[vistas-manuales]] · [[tutorial-dao-desde-cero]] · [[ME]] · [[MAP]]
