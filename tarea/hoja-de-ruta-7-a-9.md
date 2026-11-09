---
fuente: síntesis de enunciado_rubrica.pdf, auditoría de las vistas actuales y cruce con los 21 PDFs de la cátedra
tags: [tarea, plan, checklist, rubrica, hoja-de-ruta, gui, mvc, dao]
estado: vigente
actualizado: 2026-11-09
---

# Hoja de Ruta y Checklist del Proyecto (`NutritionistApp`) — Nota 7–9 y escalada al 10

> [!info] Propósito de esta nota
> Checklist operativo dividido en bloques (**0 a G**) para organizar el trabajo entre los tres integrantes y **blindar la nota 7–9 (Excelente)** en los 5 criterios de la rúbrica, indicando en cada subtarea **qué PDF del profesor y qué páginas** explican cómo resolverla. El **Bloque G** agrega lo que falta para el 10.
>
> **Referencia general del trabajo:** `enunciado_rubrica.pdf` (pág. 1 y 2) → [[enunciado_rubrica]]
>
> **Estado auditado el 2026-11-09** contra el código real (commits `008edbf`, `b357923` y cambios locales sin commitear). Los `[x]` son de esa auditoría; los `[ ]` siguen pendientes.

> [!danger] El build está roto ahora mismo
> `PatientView.java:478` tiene `public boolean confirmInsert{` **sin los paréntesis**. `javac` falla con `';' expected`. Es el único error del proyecto: guardar el archivo con `confirmInsert()` lo resuelve.

---

## Bloque 0: Corrección de Entrega 2 (`DAOException`)

**PDFs de la cátedra para este bloque:**
- 📄 **`Errores_y_Excepciones_Java.pdf`** (`> pág. 4 a 6`: `try/catch/finally` y relanzamiento; `> pág. 9 y 10`: cómo crear una excepción personalizada extendiendo `Exception`) → [[Errores_y_Excepciones_Java]]
- 📄 **`JDBC_DAO_MySQL.pdf`** (`> pág. 13`: código exacto de `DaoException`; `> pág. 14`: interfaz `DAO<T, ID>` con `throws DaoException`) → [[JDBC_DAO_MySQL]]
- 📄 **`JPA_Hibernate_DAO.pdf`** (`> pág. 18 a 22`: cómo capturar `PersistenceException`, hacer `tx.rollback()` y lanzar `new DaoException(...)` en el DAO con JPA) → [[JPA_Hibernate_DAO]]
- 📓 **Guía del equipo:** [[tutorial-dao-desde-cero]]

### Subtareas:
- [x] **0.1.** Excepción propia ✔ (*`JDBC_DAO_MySQL.pdf`, pág. 13*).
  - ⚠️ Quedó como **`excepciones.DAOException`** (paquete en español, siglas en mayúscula), no `appNutritionist.dao.DaoException`. Funciona, pero va contra la convención de nombres; está anotado como deuda en [[convenciones]].
- [x] **0.2.** `throws DAOException` en las 5 firmas de `DAO.java` ✔ (*`JPA_Hibernate_DAO.pdf`, pág. 18*).
- [x] **0.3.** Mensajes claros en los DAO de JPA ✔.
  - ⚠️ Se resolvió distinto a lo previsto: el commit `797b39c` creó `DAOJpa<T, ID>` genérico, que concentra el `try/catch`, el `rollback` y los mensajes (`"Error al crear Patient"`, `"Error al buscar Patient con id 3"`, `"Patient con id 3 no existe"`…). Los cuatro `*DAOJPA` heredan y ya no repiten código. **Es mejor que lo que pedía el plan.**
- [x] **0.4.** `AlimentDAOJPA` ✔ (hereda de `DAOJpa`).
- [x] **0.5.** `DietDAOJPA` y `DietDetailDAOJPA` ✔ (idem).

> [!warning] Lo que este bloque dejó abierto
> Las **4 consultas JPQL** de los DAO están escritas con nombres en español que no existen (`Alimento`, `categoria`, `Dieta`, `DetalleDieta`, `dieta`, `Paciente`, `apellido`). Compilan y explotan en runtime. Archivo y línea exactos en [[bug-jpql-dao]].

---

## Bloque A: Completar las Clases del Modelo y Definir las 3 Entidades (`appNutritionist.model`)

**PDFs de la cátedra para este bloque:**
- 📄 **`POO_Abstraccion.pdf`** (`> pág. 1 a 10`: cómo abstraer atributos relevantes de una entidad y encapsularlos con `private`) → [[POO_Abstraccion]]
- 📄 **`POO_Metodos.pdf`** (`> pág. 1 a 12`: constructores, sobrecarga de constructores, getters y setters) → [[POO_Metodos]]
- 📄 **`JPA_Hibernate_DAO.pdf`** (`> pág. 9 a 11`: anotaciones `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column` y regla del constructor vacío obligatorio; `> pág. 12`: relaciones `@ManyToOne`, `@OneToMany`, `@JoinColumn`) → [[JPA_Hibernate_DAO]]
- 📄 **`POO_Equals.pdf`** (`> pág. 1 a 10`: sobrescribir `equals`, `hashCode` y `toString()` para que los objetos `Patient` y `Aliment` se muestren legibles dentro de un `JComboBox` o lista) → [[POO_Equals]]
- 📄 **`POO_Repaso_y_Diagrama_de_Clases.pdf`** y **`DiagramaClases_Ej2_Biblioteca.pdf`** (cómo modelar relaciones entre clases) → [[POO_Repaso_y_Diagrama_de_Clases]] · [[DiagramaClases_Ej2_Biblioteca]]

### A.1. Entidad `Patient.java` (1ra Entidad)
- [x] **A.1.1.** Atributos (`id`, `name`, `lastName`, `weight`, `height`, `observations`), constructores, getters/setters y anotaciones JPA ✔ (*`JPA_Hibernate_DAO.pdf`, pág. 9 a 11*).
- [x] **A.1.2.** Método de dominio para el IMC ✔ (*`POO_Metodos.pdf`*).
  - Quedó como **`calcularIMC()`**, no `getImc()`. Es el nombre correcto para una operación y coincide con `calcularCaloriasTotales()` de `DietDetail`.
- [ ] **A.1.3.** `toString()` ✔ (`lastName + ", " + name`), pero **faltan `equals` y `hashCode` por `id`** (*`POO_Equals.pdf`*). Hacen falta para cargar pacientes en el `JComboBox` de la pantalla de Dietas.

### A.2. Entidad `Aliment.java` (2da Entidad)
- [x] **A.2.1.** `@Entity`, `@Table(name = "Aliments")` e `implements Serializable` ✔.
- [x] **A.2.2.** Atributos privados (`id`, `name`, `category`, `caloriesPerServing`) ✔.
  - ⚠️ El atributo se llamaba `aliment` y el 2026-11-09 se renombró a `name` (commit `b357923`), por consistencia con `Patient.name` y `DietDetail`/`Diet`.
- [x] **A.2.3.** Constructor vacío y constructor con parámetros ✔.
- [x] **A.2.4.** Getters y setters ✔ (incluidos `getName()` / `setName()`).
- [ ] **A.2.5.** `toString()` ✔ (`name + " (" + category + ")"`), pero **faltan `equals` y `hashCode`** (*`POO_Equals.pdf`*).
- [ ] **A.2.6.** El mensaje de `update` no incluye el id: hoy `DAOJpa` dice `"Error al actualizar " + clase.getSimpleName()`. Se arregla en **un solo lugar** (el genérico), no en cada DAO.

### A.3. Entidad `Diet.java` y `DietDetail.java` (3ra Entidad que combina Paciente + Alimentos)
> [!warning] Cuidado con la rúbrica (3 entidades vs 4 entidades)
> La columna **Excelente (7–9)** evalúa que **todas** las entidades tengan GUI, MVC, DAO y Javadoc completos.
> - Las **3 entidades principales** con ventana propia deben ser **`Patient`**, **`Aliment`** y **`Diet`**.
> - Si mantienen `DietDetail` como clase auxiliar para los renglones de una dieta, **se gestiona dentro de `DietView` y `DietController`**. Conviene **eliminar `DietDetailController.java`** (que hoy está vacío) para no tener un 4to controlador muerto sin vista que baje puntos en *Arquitectura MVC* y *Código Limpio*.

- [ ] **A.3.1.** `Diet` **sólo tiene `id` y `nombre`**. Faltan los atributos de la dieta y las relaciones `@ManyToOne @JoinColumn(name = "patient_id") private Patient patient;` y `@OneToMany(mappedBy = "diet") private List<DietDetail> details;` (*`JPA_Hibernate_DAO.pdf`, pág. 12*).
- [ ] **A.3.2.** `DietDetail` tiene `nombre` + `List<Aliment> aliments` (decisión de RomiM, distinta de la del plan). Faltan `servings` (porciones), `@ManyToOne Diet diet` y `@ManyToOne Aliment aliment`. Sin la cantidad no se pueden calcular porciones (*`JPA_Hibernate_DAO.pdf`, pág. 12*).
- [ ] **A.3.3.** `DietDetailController.java` **sigue existiendo y vacío**: borrarlo o darle uso.

---

## Bloque B: Pulido de la GUI y Conexión MVC + DAO (Criterio GUI 7–9)

> [!important] Qué exige la rúbrica para 7–9 en GUI y MVC (`pág. 1 y 2 — enunciado_rubrica.pdf`)
> - *"Cada entidad debe tener por lo menos una pantalla que permita implementar **CRUD** a nivel de la vista."*
> - *"La GUI es **intuitiva**, **estéticamente agradable** y **funcional para todas las entidades sin errores**."*
> - *"Excelente separación en capas en por lo menos tres entidades, con una clara y efectiva implementación de MVC."*

**PDFs de la cátedra para este bloque:**
- 📄 **`Swing_MVC_Interfaces_Java.pdf`** (`> pág. 20 a 28`: arquitectura MVC en Swing, cómo el Controlador escucha eventos de los botones de la Vista con `addActionListener(e -> ...)` y coordina Modelo/DAO y Vista) → [[Swing_MVC_Interfaces_Java]]
- 📄 **`JTable_en_Swing.pdf`** (`> pág. 1 a 16`: cómo mostrar la lista de `findAll()` en una `JTable` usando `DefaultTableModel` o `AbstractTableModel`) → [[JTable_en_Swing]]
- 📄 **`Listas_y_ComboBox_Swing.pdf`** (`> pág. 1 a 15`: uso de `JComboBox` y `JList` para seleccionar categorías, pacientes o alimentos) → [[Listas_y_ComboBox_Swing]]
- 📄 **`Checkbox_y_RadioButton_Swing.pdf`** (para filtros u opciones excluyentes con `ButtonGroup`) → [[Checkbox_y_RadioButton_Swing]]
- 📓 **Convenciones del equipo:** [[convenciones]] · [[vistas-manuales]] · **Plan paso a paso:** [[plan-cableado-patient]]

### B.1. Correcciones de Usabilidad y Estética en las Vistas actuales (para cumplir *"intuitiva, estéticamente agradable y sin errores"*)
- [x] **B.1.1. Campo ID en `AlimentView`** ✔: existe `idTextField` con su etiqueta `alimentIdLabel`, así que Search/Modify/Delete por ID ya son posibles.
- [x] **B.1.2. `JTable` en las vistas para mostrar `findAll()`** ✔ parcial: `PatientView` (`patientsTable`) y `AlimentView` (`alimentsTable`) ya la tienen, read-only y dentro de `tableScrollPane`. **Falta `DietView`.**
- [x] **B.1.3. IMC no editable** ✔: `imcTextField` tiene `editable="false"` y el tooltip *"Calculated automatically from Weight and Height"*.
- [ ] **B.1.4. Textos, títulos y centrado** — estado por ítem:
  - [x] `"High"` → **`"Height (m)"`** ✔ (y `"Weight (kg)"` agregado).
  - [x] `"Patient´s Id"` → **`"Patient ID"`** ✔.
  - [x] En `AlimentView`, la etiqueta repetida `"Aliment"` → **`"Name"`** ✔.
  - [x] Títulos de ventana ✔: `"NutritionistApp - Patients Management"` y `"NutritionistApp - Aliments Management"`.
  - [x] Centrado ✔: `setLocationRelativeTo(null)` en las dos vistas.
  - [ ] `Observations` sigue siendo `JTextField`: pasarlo a `JTextArea` dentro de un `JScrollPane`.
  - [ ] Botón **`Clear`** (limpiar formulario): en las dos vistas falta.

### B.2. Conectar `PatientView.java` y `PatientController.java` con `PatientDAO`
- [x] **B.2.1.** Getters de la vista ✔ (casi completo): `getButtonAliments()`, `getButtonDiets()`, `getIdText()`, `getNameText()`, `getLastNameText()`, `getWeightText()`, `getHeightText()`, `getObservationsText()`, `getImcText()`, los seis setters, `getButtonInsert()`, `getButtonSearch()`, `getButtonModify()`, `getButtonDelete()`, `getPatientstable()`, `showError()`, `showInfo()` y `confirmInsert()`.
  - ⚠️ Pendientes de pulido, todos de una línea:
    - `confirmInsert` **sin `()`** → no compila (lo primero que hay que arreglar).
    - `getPatientstable()` → `getTablePatients()` (la `s` de `Table` y el orden, como en [[plan-cableado-patient]]).
    - `setnameText()` → `setNameText()`.
    - `"Informacion"` → `"Información"` en `showInfo`.
    - Falta `clearForm()` para el botón Clear.
  - ⚠️ `confirmInsert()` no estaba en el plan: usaste `OK_CANCEL_OPTION`. Está bien; sólo hay que llamarlo desde el controlador.
- [ ] **B.2.2.** `PatientController` sigue recibiendo `new Patient()`; falta la firma `PatientController(PatientDAO dao, PatientView view)` (*`Swing_MVC_Interfaces_Java.pdf`, pág. 26 y 27*).
- [ ] **B.2.3.** Insert: validar nombre/apellido, `try/catch (NumberFormatException)` al parsear peso y altura, `dao.insert(patient)`, refrescar la tabla.
- [ ] **B.2.4.** Search: validar ID entero, `dao.findById(id)`, cargar los campos o avisar si no existe.
- [ ] **B.2.5.** Modify: leer ID + campos, `dao.update(patient)`, refrescar.
- [ ] **B.2.6.** Delete: leer ID, pedir confirmación (`showConfirmDialog`), `dao.delete(id)`, limpiar campos y refrescar.

> [!tip] Ya está el código exacto de todo esto
> [[plan-cableado-patient]] tiene los métodos completos (con `PatientDAOInMemory`, `PatientTableModel`, `convertRowIndexToModel` y las trampas). Acá sólo se sigue el estado.

### B.3. Conectar `AlimentView.java` y `AlimentController.java` con `AlimentDAO`
- [ ] **B.3.1.** Exponer con getters los botones CRUD, los campos (`idTextField`, `nameTextField`, `categoryTextField`, `caloriesTextField`) y `alimentsTable`. *(Sugerencia: `JComboBox` para `Category` — Frutas, Verduras, Carnes, Lácteos, Cereales — según `Listas_y_ComboBox_Swing.pdf`)*.
- [ ] **B.3.2.** Listeners de **Insert**, **Search**, **Modify**, **Delete** y **Clear** en `AlimentController`, con validación numérica de calorías y manejo de `DAOException`.

### B.4. Crear `DietView.java` y conectar `DietController.java` (3ra Entidad)
- [ ] **B.4.1.** Crear `DietView.java` en `appNutritionist.view` con:
  - Barra superior de navegación hacia `Patient` y `Aliment`.
  - `JComboBox<Patient>` para elegir el paciente (*`Listas_y_ComboBox_Swing.pdf`*).
  - `JComboBox<Aliment>` y campo de porciones (`servings`) para agregar alimentos.
  - `JTable` con los alimentos de la dieta y las calorías totales (*`JTable_en_Swing.pdf`*).
  - Botones CRUD (`Insert`, `Search`, `Modify`, `Delete`, `Clear`).
- [ ] **B.4.2.** Conectar `getButtonDiets()` en `PatientController` y en `AlimentController` para que abra `DietView`.
- [ ] **B.4.3.** CRUD completo en `DietController.java` (hoy está vacío) usando `DietDAO`, `PatientDAO` y `AlimentDAO`.

---

## Bloque C: Configurar la Conexión Real a la Base de Datos SQL

**PDFs de la cátedra para este bloque:**
- 📄 **`JPA_Hibernate_DAO.pdf`** (`> pág. 4 a 8`: librerías de JPA, estructura exacta de `META-INF/persistence.xml` e inicialización con `Persistence.createEntityManagerFactory("...")`) → [[JPA_Hibernate_DAO]]
- 📄 **`JDBC_DAO_MySQL.pdf`** (`> pág. 4 a 6`: cómo agregar el `.jar` de MySQL Connector en proyectos Ant en la carpeta `lib/`) → [[JDBC_DAO_MySQL]]

### Subtareas:
- [x] **C.1.** `lib/` en la raíz del proyecto ✔ (`flatlaf-3.5.4.jar`, `jakarta.persistence-2.2.3.jar`), declarada en `nbproject/project.properties`.
- [ ] **C.2.** Crear la base en MySQL (`CREATE DATABASE nutritionist_db;`). *No se puede verificar desde el código.*
- [x] **C.3.** `src/META-INF/persistence.xml` ✔, pero **con tres errores de configuración**:
  - La unidad se llama `"persistence.xml"` y `JPAUtil.java:12` pide `"nutricionPU"` → nunca va a encontrarla.
  - La URL apunta a `jdbc:mysql://localhost:3306/mysql` (el **catálogo del sistema**, no una base propia).
  - `javax.persistence.schema-generation.database.action` está en **`create`**: borra y recrea las tablas en cada arranque. Para desarrollo va `update`.
- [ ] **C.4.** Faltan en `lib/` el **proveedor JPA** (EclipseLink, que es el declarado en el XML) y el **driver MySQL** (*`JDBC_DAO_MySQL.pdf`, pág. 4 y `JPA_Hibernate_DAO.pdf`, pág. 4 a 6*). Sin ellos, `PatientDAOJPA` no arranca.
- [ ] **C.5.** `Persistencia` inicializada desde `AppNutritionist` ✔ parcial: existe `util.JPAUtil` (singleton del `EntityManagerFactory`), pero **nadie llama a `JPAUtil.cerrar()`** al cerrar la aplicación y el `EntityManagerFactory` se crea de forma perezosa en el primer DAO (*`JPA_Hibernate_DAO.pdf`, pág. 7 y 8*).

---

## Bloque D: Documentación con Javadoc y Código Limpio

**PDFs de la cátedra para este bloque:**
- 📄 **`Javadoc_en_Java.pdf`** (`> pág. 1 a 13`: sintaxis `/** ... */`, qué se documenta, etiquetas `@param`, `@return`, `@throws`, `@see` y cómo generar el HTML de Javadoc) → [[Javadoc_en_Java]]
- 📄 **`Codigo_Limpio_Nombres_y_Estructura.pdf`** (`> pág. 1 a 13`: nombres con intención, funciones pequeñas de una sola responsabilidad, estructura vertical, eliminar comentarios basura autogenerados por NetBeans como `Click nbfs://...` y `@author Usuario`) → [[Codigo_Limpio_Nombres_y_Estructura]]
- 📄 **`SOLID_Principios.pdf`** (`> pág. 1 a 14`: responsabilidad única e inversión de dependencias programando contra las interfaces `PatientDAO`, `AlimentDAO`, `DietDAO` y no contra las clases `*DAOJPA`) → [[SOLID_Principios]]

### Subtareas:
- [ ] **D.1.** Borrar los encabezados basura de NetBeans (`Click nbfs://...` y `@author Usuario`). **Quedan 7 archivos**: `DietController`, `DietDetailController`, `AlimentDAO`, `DietDAO`, `DietDetailDAO`, `AlimentView` y `DAOException`. El resto ya está limpio ✔.
- [x] **D.2.** Javadoc en `appNutritionist.dao` ✔ parcial: `DAO`, `DAOJpa`, las 4 interfaces, `DAOException` y `JPAUtil` están documentados; a los cuatro `*DAOJPA` les falta el bloque de clase completo (`@param`/`@return`/`@throws` de sus métodos).
- [x] **D.3.** Javadoc en el modelo ✔ parcial: `Aliment`, `Diet` y `DietDetail` tienen Javadoc de clase; **`Patient` no tiene ninguno**.
- [ ] **D.4.** Javadoc en los controladores y en `AppNutritionist`: **cero** hoy (los dos controladores vacíos sólo tienen `@author Usuario`).
- [ ] **D.5.** Javadoc en los getters/métodos públicos nuevos de `PatientView` y `AlimentView` (*ver ejemplo en [[convenciones]]*).
- [ ] **D.6.** Generar el Javadoc desde NetBeans (*Run → Generate Javadoc*) y verificar `dist/javadoc/` sin errores (*`Javadoc_en_Java.pdf`, pág. 10 y 11*).

---

## Bloque E: Los 2 Diagramas UML Obligatorios

**PDFs de la cátedra para este bloque:**
- 📄 **`Diagrama_Casos_de_Uso_UML.pdf`** (`> pág. 1 a 14`: actores, casos de uso, límite del sistema y diferencia exacta entre `<<include>>` y `<<extend>>`) → [[Diagrama_Casos_de_Uso_UML]]
- 📄 **`Diagrama_de_Secuencia_UML.pdf`** (`> pág. 1 a 20`: líneas de vida, cajas de activación, mensajes síncronos vs retorno, fragmentos `alt`/`opt`/`loop`, y **en las págs. 14 a 17 incluye un ejemplo completo de MVC + DAO**) → [[Diagrama_de_Secuencia_UML]]

> No se pueden auditar desde el repo: se hacen fuera del código.

### E.1. Diagrama de Casos de Uso General del Sistema
- [ ] **E.1.1.** Rectángulo del sistema (`"Sistema NutritionistApp"`) y actor `Nutricionista` (*`Diagrama_Casos_de_Uso_UML.pdf`, pág. 4 a 6*).
- [ ] **E.1.2.** Casos de uso para gestionar Pacientes, Alimentos y Dietas, con `<<include>>` entre *Crear Dieta* y *Buscar Paciente* / *Buscar Alimento* (*`Diagrama_Casos_de_Uso_UML.pdf`, pág. 8 a 11*).

### E.2. Diagrama de Secuencia de un Proceso (ej. "Registrar Paciente")
- [ ] **E.2.1.** Participantes: `:Nutricionista`, `:PatientView`, `:PatientController`, `:PatientDAOJPA`, `:EntityManager` (*`Diagrama_de_Secuencia_UML.pdf`, pág. 14 y 15*).
- [ ] **E.2.2.** Flechas de llamada y retorno con un fragmento **`alt`** para el camino de éxito (`commit`) y el de error (`rollback` + `DAOException`) (*`Diagrama_de_Secuencia_UML.pdf`, pág. 11 y 16*).

---

## Bloque F: Preparación para la Defensa Oral (Requisito Excluyente)

> `> pág. 1 — enunciado_rubrica.pdf`: *"Todos los integrantes del equipo deben estar en condiciones de defender (oralmente) todo lo desarrollado para el proyecto."*

- [ ] **F.1.** Que los 3 integrantes puedan explicar qué hace cada capa de **MVC + DAO** y por qué la Vista nunca llama al DAO directamente (*ver [[Swing_MVC_Interfaces_Java]] y [[SOLID_Principios]]*).
- [ ] **F.2.** Que los 3 integrantes puedan explicar los 5 archivos del patrón DAO (`Entidad`, `DAOException`, `DAO<T, ID>`, `PatientDAO`, `PatientDAOJPA`) usando [[tutorial-dao-desde-cero]].
- [ ] **F.3.** Que los 3 integrantes sepan explicar por qué `DAOException` es una excepción *checked* (`extends Exception`) y por qué se hace `tx.rollback()` cuando se atrapa `PersistenceException` (*ver [[Errores_y_Excepciones_Java]] y [[JPA_Hibernate_DAO]]*).
- [ ] **F.4.** Que los 3 integrantes sepan explicar los 2 diagramas UML (diferencia entre `<<include>>` y `<<extend>>` en Casos de Uso, y líneas de vida/mensajes en el de Secuencia).

---

## Bloque G: Escalar de 7–9 a 10

> pág. 1 y 2 — `enunciado_rubrica.pdf`
> - **Nota 10:** *"Desarrollar la totalidad para **las tres entidades** + incorporación de alguna **característica extra (investigación)** a lo trabajado en clase."*

### G.1. Las tres entidades completas (piso, no extra)

| Entidad | Modelo | Vista | Controlador cableado | DAO | Estado |
|---|---|---|---|---|---|
| `Patient` | ✅ | ✅ | ⬜ | ✅ | falta cablear (Bloque B.2) |
| `Aliment` | ✅ | ✅ | ⬜ | ✅ | falta cablear (Bloque B.3) |
| `Diet` | ⚠️ sólo `id` + `nombre` | ❌ no existe `DietView` | ❌ `DietController` vacío | ✅ (JPQL roto) | falta todo (A.3 + B.4) |

Sin las tres, el 10 no existe: la rúbrica lo pide como **piso**. El extra se suma **después**.

### G.2. El extra: qué rinde y qué no

| Candidato | Esfuerzo | Se ve en pantalla | Riesgo |
|---|---|---|---|
| **Tests JUnit sobre `PatientDAOInMemory`** | bajo | no | el profe puede decir "eso es una técnica, no una feature" |
| **Reporte PDF de la dieta** (JasperReports/iText) | medio | **sí** | suma un jar |
| **Gráfico de IMC / evolución de peso** (JFreeChart) | medio | **sí** | suma un jar |
| **Exportar la tabla a CSV** | bajo | **sí** | ninguno |
| **Búsqueda por apellido / categoría** | bajo | **sí** | depende de [[bug-jpql-dao]] |
| FlatLaf + autozoom | ya hecho | sí | **el profesor la sugirió**: como "investigación propia" queda flojo |

**Recomendación:** elegir **uno visible** (reporte PDF o gráfico de IMC) y, si quieren que el DAO en memoria cuente, **agregarle tests JUnit** con evidencia en la defensa.

> [!warning] El DAO en memoria NO reemplaza la base
> > pág. 1 — `enunciado_rubrica.pdf`: *"Las entidades **se deben persistir en una base de datos SQL** utilizando el patrón DAO."*
>
> Si sólo hay DAO en memoria, el criterio **DAO** cae a "no es funcional o es incorrecta". El en memoria es un **doble de prueba**: complementa, no sustituye.

### G.3. Cómo defender el DAO en memoria sin que baje puntos

Guion de tres frases, con el respaldo del PDF:

1. **El controlador depende de la abstracción, no de la implementación.** `PatientController` recibe un `PatientDAO`; hoy le inyectamos `PatientDAOInMemory`, mañana `PatientDAOJPA`, y el controlador no cambia una línea.
   > pág. 28 a 33 — `SOLID_Principios.pdf`: *"Los módulos de alto nivel no deben depender de los módulos de bajo nivel; ambos deben depender de abstracciones."*
2. **Eso permite testear sin base.**
   > pág. 19 — `JDBC_DAO_MySQL.pdf`: *"Testeabilidad: Permite mocks e implementaciones en memoria para pruebas unitarias."*
   > pág. 11 y 12 — `JDBC_DAO_MySQL.pdf`: *"…facilitando la reutilización, el mantenimiento y los tests (se puede simular un DAO en memoria)."*
3. **Y no es un adorno: es lo que nos deja mostrar el CRUD aunque MySQL no esté disponible.** Cerrar con:
   > pág. 20 — `JDBC_DAO_MySQL.pdf`: *"Separar el qué del cómo es la esencia del buen diseño."*

> [!warning] No atribuirle al profesor lo que no dijo
> El PDF habla de **pruebas unitarias**, nunca de "base de producción" ni de "peligroso". Que testear sin la base evite tocar datos reales es una **consecuencia** que se puede explicar, pero no es una cita.

### G.4. Checklist del 10

- [ ] `Diet` completa (relaciones, `DietView`, `DietController`, `DietDAO` andando).
- [ ] Los 3 CRUD cableados contra su DAO (Bloque B completo).
- [ ] Base MySQL real funcionando (Bloque C) y [[bug-jpql-dao]] corregido.
- [ ] **Un extra visible** elegido e implementado + documentado en una nota del vault.
- [ ] Si el extra es "testeo": tests JUnit reales sobre `PatientDAOInMemory`, no sólo la clase.
- [ ] Javadoc + UML + Javadoc generado sin errores (Bloques D y E).
- [ ] Los 3 integrantes pueden defender el extra y el DAO en memoria (Bloque F).

---

## Relacionado

- [[plan-cableado-patient]] — el paso a paso del cableado, con el código de cada método
- [[bug-jpql-dao]] — las 4 consultas JPQL rotas (bloquea el Bloque C y el criterio DAO)
- [[estado]] · [[convenciones]] · [[tutorial-dao-desde-cero]]
- [[entrega-01-interfaces-graficas]] · [[entrega-02-dao-una-entidad]]
- [[enunciado_rubrica]] · [[MAP]]
