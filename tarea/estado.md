---
tags: [tarea, estado, auditoria]
fuente: inspección directa del repositorio
aliases: [00-estado]
---

# Estado actual del código

> Actualizado el 2026-11-05 tras clonar el repositorio compartido de GitHub y aplicar la reestructuración manual de paquetes, nombres de clases y consultas JPQL.
> **No** es contenido del profesor: es un relevamiento de lo que hay escrito.

## Ubicación

El código vive en el proyecto NetBeans:

```text
C:\Users\games\Desktop\Proyectos\NutritionistApp\
├── build.xml, manifest.mf, nbproject/    ← proyecto NetBeans (Ant)
├── lib/ (o src/lib/)
│   └── jakarta.persistence-2.2.3.jar     ← librería JPA configurada en javac.classpath
└── src/appNutritionist/
    ├── AppNutritionist.java               ← main, cablea MVC de Patient
    ├── controller/  PatientController.java  AlimentController.java  DietController.java  DietDetailController.java
    ├── model/       Patient.java            Aliment.java            Diet.java            DietDetail.java
    ├── dao/         DAO.java  PatientDAO.java  PatientDAOJPA.java  AlimentDAO.java  AlimentDAOJPA.java  DietDAO.java  DietDAOJPA.java  DietDetailDAO.java  DietDetailDAOJPA.java
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
| `model/Patient.java` | **Completa.** Entidad JPA (`@Entity`, `@Table(name="Patients")`, `@Id`, `@GeneratedValue`) con `id`, `name`, `lastName`, `weight`, `height`, `observations`, constructores y getters/setters |
| `model/Aliment.java` | **Vacía.** Pendiente de completar atributos (ver [[hoja-de-ruta-7-a-9]], Bloque A.2) |
| `model/Diet.java` | **Vacía.** Pendiente de completar atributos y relaciones (ver [[hoja-de-ruta-7-a-9]], Bloque A.3) |
| `model/DietDetail.java` | **Vacía.** Pendiente de completar atributos y relaciones (ver [[hoja-de-ruta-7-a-9]], Bloque A.3) |
| `dao/DAO.java` | Interfaz genérica `DAO<T, ID>` con métodos `insert`, `findById`, `findAll`, `update`, `delete` (pendiente agregar `throws DaoException` por un compañero) |
| `dao/PatientDAO.java`, `AlimentDAO.java`, `DietDAO.java`, `DietDetailDAO.java` | Interfaces específicas que extienden `DAO<Entidad, Integer>` |
| `dao/PatientDAOJPA.java`, `AlimentDAOJPA.java`, `DietDAOJPA.java`, `DietDetailDAOJPA.java` | Implementaciones JPA con consultas JPQL corregidas (`FROM Patient`, `FROM Aliment`, `FROM Diet`, `FROM DietDetail`). Pendiente reemplazar `RuntimeException` por `DaoException` (ver Bloque 0 de [[hoja-de-ruta-7-a-9]]) |
| `controller/PatientController.java` | Cablea modelo y vista; listener de navegación a `AlimentView` |
| `controller/AlimentController.java` | Cablea modelo y vista; listener de navegación a `PatientView` |
| `controller/DietController.java`, `DietDetailController.java` | **Vacíos.** Esqueletos creados |
| `view/PatientView.java` | GUI armada. Componentes privados; `getBtnAliment()` y `getBtnDiet()` |
| `view/AlimentView.java` | GUI armada. Componentes privados; `getBtnPatient()` y `getBtnDiet()` |

**Navegación entre vistas:** funcionando (`PatientView` ↔ `AlimentView` con `dispose()` y `SwingUtilities.invokeLater()`).

## Lo que falta, según lo que implican los PDFs

Ver el desglose completo con subtareas y páginas de cada PDF en **[[hoja-de-ruta-7-a-9]]**:

| Bloque | Estado | PDFs que lo gobiernan |
|---|---|---|
| **Bloque 0:** Excepción `DaoException` en los DAOs | ⏳ Listo para que lo commitee un compañero (ya está hecho en `Desktop\Entrega`) | `Errores_y_Excepciones_Java.pdf`, `JDBC_DAO_MySQL.pdf`, `JPA_Hibernate_DAO.pdf` |
| **Bloque A:** Modelos `Aliment`, `Diet` y `DietDetail` | ❌ Vacíos | `POO_Abstraccion.pdf`, `POO_Metodos.pdf`, `JPA_Hibernate_DAO.pdf` |
| **Bloque B:** Conectar Vistas + Controladores + DAOs (CRUD) | ⚠️ Solo navegación, falta conectar botones CRUD | `Swing_MVC_Interfaces_Java.pdf`, `Listas_y_ComboBox_Swing.pdf`, `JTable_en_Swing.pdf` |
| **Bloque C:** Conexión real SQL (`persistence.xml`) | ❌ Falta configurar | `JPA_Hibernate_DAO.pdf`, `JDBC_DAO_MySQL.pdf` |
| **Bloque D:** Javadoc en clases y métodos públicos | ⚠️ Parcial | `Javadoc_en_Java.pdf`, `Codigo_Limpio_Nombres_y_Estructura.pdf` |
| **Bloque E:** Diagramas UML (Casos de Uso y Secuencia) | ❌ No hay | `Diagrama_Casos_de_Uso_UML.pdf`, `Diagrama_de_Secuencia_UML.pdf` |

## Riesgos verificados contra la rúbrica

1. **Nombres de clase y paquetes:** ✅ **Resuelto.** Paquetes en minúscula singular (`model`, `view`, `controller`, `dao`) y clases en PascalCase sin prefijos redundantes (`Patient`, `Aliment`, `Diet`, `DietDetail`, `PatientView`, `AlimentView`, `PatientController`, `AlimentController`, `DietController`, `DietDetailController`, `DAO`, `*DAO`, `*DAOJPA`).
2. **Las vistas son código generado por el NetBeans GUI Builder.** Riesgo bajo; los getters públicos van fuera de las secciones generadas según [[convenciones]].
3. **Java 25 con JDK 26 instalado.** Compilar siempre con `--release 25`.

## Entregas (metodología incremental)

- ✅ [[entrega-01-interfaces-graficas]] — GUI de `Patient` y `Aliment`.
- 🔄 [[entrega-02-dao-una-entidad]] — Corrección de nombres y `Patient` lista; `DaoException` lista en `Desktop\Entrega` y preparada para que otro integrante haga el commit en GitHub.

## Relacionado

[[hoja-de-ruta-7-a-9]] · [[tutorial-dao-desde-cero]] · [[convenciones]] · [[enunciado_rubrica]] · [[vistas-manuales]] · [[ME]] · [[MAP]]
