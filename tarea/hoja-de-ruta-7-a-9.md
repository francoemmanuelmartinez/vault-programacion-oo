---
fuente: síntesis de enunciado_rubrica.pdf, auditoría de vistas actuales y cruce con los 21 PDFs de la cátedra
tags: [tarea, plan, checklist, rubrica, hoja-de-ruta, gui, mvc, dao]
estado: vigente
---

# Hoja de Ruta y Checklist del Proyecto (`NutritionistApp`) para Asegurar Nota 7–9

> [!info] Propósito de esta nota
> Checklist operativo dividido en bloques (**0 a F**) para organizar el trabajo entre los tres integrantes del equipo y **blindar la nota 7–9 (Excelente)** en los 5 criterios de la rúbrica, indicando en cada subtarea **qué PDF del profesor y qué páginas** explican cómo resolverla.
>
> **Referencia general del trabajo:** `enunciado_rubrica.pdf` (pág. 1 y 2) → [[enunciado_rubrica]]

---

## Bloque 0: Cerrar Corrección de Entrega 2 (`DaoException`)

**PDFs de la cátedra para este bloque:**
- 📄 **`Errores_y_Excepciones_Java.pdf`** (`> pág. 4 a 6`: `try/catch/finally` y relanzamiento; `> pág. 9 y 10`: cómo crear una excepción personalizada extendiendo `Exception`) → [[Errores_y_Excepciones_Java]]
- 📄 **`JDBC_DAO_MySQL.pdf`** (`> pág. 13`: código exacto de `DaoException`; `> pág. 14`: interfaz `DAO<T, ID>` con `throws DaoException`) → [[JDBC_DAO_MySQL]]
- 📄 **`JPA_Hibernate_DAO.pdf`** (`> pág. 18 a 22`: cómo capturar `PersistenceException`, hacer `tx.rollback()` y lanzar `new DaoException(...)` en el DAO con JPA) → [[JPA_Hibernate_DAO]]
- 📓 **Guía del equipo:** [[tutorial-dao-desde-cero]]

### Subtareas:
- [ ] **0.1.** Crear `DaoException.java` en `appNutritionist.dao` extendiendo de `Exception` (*ver `JDBC_DAO_MySQL.pdf`, pág. 13*).
- [ ] **0.2.** Agregar `throws DaoException` a las 5 firmas de métodos en `DAO.java` (*ver `JPA_Hibernate_DAO.pdf`, pág. 18*).
- [ ] **0.3.** Actualizar `PatientDAOJPA.java`: importar `PersistenceException`, agregar `throws DaoException` y envolver los errores con mensajes claros (*ver `JPA_Hibernate_DAO.pdf`, pág. 20 a 22*).
- [ ] **0.4.** Actualizar `AlimentDAOJPA.java` con `throws DaoException` y sus mensajes personalizados.
- [ ] **0.5.** Actualizar `DietDAOJPA.java` y `DietDetailDAOJPA.java` con `throws DaoException` y sus mensajes personalizados.

---

## Bloque A: Completar las Clases del Modelo y Definir las 3 Entidades (`appNutritionist.model`)

**PDFs de la cátedra para este bloque:**
- 📄 **`POO_Abstraccion.pdf`** (`> pág. 1 a 10`: cómo abstraer atributos relevantes de una entidad y encapsularlos con `private`) → [[POO_Abstraccion]]
- 📄 **`POO_Metodos.pdf`** (`> pág. 1 a 12`: constructores, sobrecarga de constructores, getters y setters) → [[POO_Metodos]]
- 📄 **`JPA_Hibernate_DAO.pdf`** (`> pág. 9 a 11`: anotaciones `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column` y regla del constructor vacío obligatorio; `> pág. 12`: relaciones `@ManyToOne`, `@OneToMany`, `@JoinColumn`) → [[JPA_Hibernate_DAO]]
- 📄 **`POO_Equals.pdf`** (`> pág. 1 a 10`: sobrescribir `equals`, `hashCode` y `toString()` para que los objetos `Patient` y `Aliment` se muestren legibles dentro de un `JComboBox` o lista) → [[POO_Equals]]
- 📄 **`POO_Repaso_y_Diagrama_de_Clases.pdf`** y **`DiagramaClases_Ej2_Biblioteca.pdf`** (cómo modelar relaciones entre clases) → [[POO_Repaso_y_Diagrama_de_Clases]] · [[DiagramaClases_Ej2_Biblioteca]]

### A.1. Entidad `Patient.java` (1ra Entidad)
- [x] **A.1.1.** Definir atributos (`id`, `name`, `lastName`, `weight`, `height`, `observations`), constructores, getters/setters y anotaciones JPA (*`JPA_Hibernate_DAO.pdf`, pág. 9 a 11*).
- [ ] **A.1.2.** Agregar método de dominio `getImc()` o `calculateImc()` (`weight / (height * height)`) para que el campo IMC de la vista se calcule automáticamente sin que el usuario lo escriba a mano (*`POO_Metodos.pdf`*).
- [ ] **A.1.3.** Sobrescribir `toString()` (ej. `return id + " - " + lastName + ", " + name;`) y `equals`/`hashCode` por `id` para poder cargar pacientes en el `JComboBox` de la ventana de Dietas (*`POO_Equals.pdf` y `Listas_y_ComboBox_Swing.pdf`*).

### A.2. Entidad `Aliment.java` (2da Entidad)
- [ ] **A.2.1.** Agregar anotaciones `@Entity`, `@Table(name = "Aliments")` e `implements Serializable` (*`JPA_Hibernate_DAO.pdf`, pág. 9 y 11*).
- [ ] **A.2.2.** Declarar los atributos privados (`id`, `name`, `category`, `caloriesPerServing`).
- [ ] **A.2.3.** Crear el constructor vacío `public Aliment() {}` y el constructor con parámetros (*`JPA_Hibernate_DAO.pdf`, pág. 10*).
- [ ] **A.2.4.** Crear todos los `getters` y `setters`.
- [ ] **A.2.5.** Sobrescribir `toString()` (ej. `return name + " (" + caloriesPerServing + " kcal)";`) y `equals`/`hashCode` para usar `Aliment` en el selector de Dietas.
- [ ] **A.2.6.** Actualizar el mensaje del `update` en `AlimentDAOJPA.java` agregando `+ aliment.getId()`.

### A.3. Entidad `Diet.java` y `DietDetail.java` (3ra Entidad que combina Paciente + Alimentos)
> [!warning] Cuidado con la rúbrica (3 entidades vs 4 entidades)
> La columna **Excelente (7–9)** evalúa que **todas** las entidades tengan GUI, MVC, DAO y Javadoc completos.
> - Las **3 entidades principales** con ventana propia deben ser **`Patient`**, **`Aliment`** y **`Diet`**.
> - Si mantienen `DietDetail` como clase auxiliar para los renglones de una dieta, **se gestiona dentro de `DietView` y `DietController`**. Conviene **eliminar `DietDetailController.java`** (que hoy está vacío) para no tener un 4to controlador muerto sin vista que baje puntos en *Arquitectura MVC* y *Código Limpio*.

- [ ] **A.3.1.** En `Diet.java`: agregar `@Entity`, `@Id`, atributos de la dieta (ej. `name`, `dayOfWeek` o `observations`), relación `@ManyToOne @JoinColumn(name = "patient_id") private Patient patient;` y `@OneToMany(mappedBy = "diet") private List<DietDetail> details;` (*`JPA_Hibernate_DAO.pdf`, pág. 12*).
- [ ] **A.3.2.** En `DietDetail.java`: agregar `@Entity`, `@Id`, atributo `servings` (porciones) y relaciones `@ManyToOne` hacia `Diet` y `@ManyToOne` hacia `Aliment` (*`JPA_Hibernate_DAO.pdf`, pág. 12*).
- [ ] **A.3.3.** Eliminar `DietDetailController.java` si `DietDetail` se maneja desde `DietController`.

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
- 📓 **Convenciones del equipo:** [[convenciones]] · [[vistas-manuales]]

### B.1. Correcciones de Usabilidad y Estética en las Vistas actuales (para cumplir *"intuitiva, estéticamente agradable y sin errores"*)
- [ ] **B.1.1. Resolver el ID faltante en `AlimentView`**: Hoy `AlimentView` solo tiene `Aliment`, `Category` y `Calories per serving` (no tiene campo `Id`), por lo que es imposible hacer `Search`, `Modify` o `Delete` por ID. Agregar el campo `Id` en `AlimentView` (igual que en `PatientView`) y/o selección desde tabla.
- [ ] **B.1.2. Agregar `JTable` en las vistas para mostrar `findAll()`**: Sin una tabla, el usuario no puede ver qué pacientes o alimentos existen si no sabe el ID de memoria. Agregar un `JTable` dentro de un `JScrollPane` en `PatientView`, `AlimentView` y `DietView` que cargue la lista con `dao.findAll()`, y que al hacer clic en una fila copie los datos a los campos de texto (*ver `JTable_en_Swing.pdf`, pág. 5 a 14*).
- [ ] **B.1.3. Bloquear edición manual del IMC en `PatientView`**: Poner `jTextField5.setEditable(false)` para que el usuario no pueda escribir texto inválido en el IMC; calcularlo y mostrarlo automáticamente cuando se carga o guarda el peso y la altura.
- [ ] **B.1.4. Corregir textos, títulos y centrado de ventanas**:
  - En `PatientView`: cambiar la etiqueta `"High"` por `"Height"` y `"Patient´s Id"` por `"Patient ID"`.
  - En `AlimentView`: cambiar la etiqueta repetida `"Aliment"` al lado del campo de texto (`jLabel1`) por `"Name"`.
  - Agregar título a las ventanas (`setTitle("NutritionistApp - Patients")`, etc.) y centrarlas en pantalla con `setLocationRelativeTo(null)` (o activando `generateCenter` en el `.form`).
  - Cambiar el campo `Observations` de `JTextField` gigante a `JTextArea` (o `JTextField` de altura normal) para que el texto no quede flotando en el medio.
  - Agregar un botón **`Clear`** (Limpiar campos) en las vistas para vaciar el formulario tras una búsqueda.

### B.2. Conectar `PatientView.java` y `PatientController.java` con `PatientDAO`
- [ ] **B.2.1.** En `PatientView.java` (debajo de `//GEN-END:variables`), agregar getters para los botones CRUD (`getBtnInsert()`, `getBtnModify()`, `getBtnDelete()`, `getBtnSearch()`, `getBtnClear()`) y métodos para leer/escribir los campos de texto y la tabla (*ver [[convenciones]]*).
- [ ] **B.2.2.** En `PatientController.java`, recibir `PatientDAO` en el constructor (en lugar de depender solo de un `new Patient()` vacío que no hace nada) (*`Swing_MVC_Interfaces_Java.pdf`, pág. 26 y 27*).
- [ ] **B.2.3.** Programar el botón **Insert**: validar que nombre/apellido no estén vacíos, parsear peso y altura con `try/catch (NumberFormatException)` mostrando `JOptionPane` claro si ingresan letras, llamar a `dao.insert(patient)` dentro de `try/catch (DaoException)`, actualizar el IMC en pantalla y refrescar la `JTable`.
- [ ] **B.2.4.** Programar el botón **Search**: validar que el ID sea un entero, llamar a `dao.findById(id)` y cargar los datos en los campos (o avisar con `JOptionPane` si no existe).
- [ ] **B.2.5.** Programar el botón **Modify**: leer el ID y los campos modificados, llamar a `dao.update(patient)` y refrescar la tabla.
- [ ] **B.2.6.** Programar el botón **Delete**: leer el ID, pedir confirmación (`JOptionPane.showConfirmDialog`), llamar a `dao.delete(id)`, limpiar campos y refrescar la tabla.

### B.3. Conectar `AlimentView.java` y `AlimentController.java` con `AlimentDAO`
- [ ] **B.3.1.** Exponer con getters los botones CRUD, los campos (`id`, `name`, `category`, `caloriesPerServing`) y la `JTable`. *(Pueden usar un `JComboBox` para `Category`, como Frutas, Verduras, Carnes, Lácteos, Cereales, aplicando `Listas_y_ComboBox_Swing.pdf`)*.
- [ ] **B.3.2.** Programar en `AlimentController.java` los listeners de **Insert**, **Search**, **Modify**, **Delete** y **Clear** con validación de números para las calorías y manejo de `DaoException`.

### B.4. Crear `DietView.java` y conectar `DietController.java` (3ra Entidad)
- [ ] **B.4.1.** Crear `DietView.java` en `appNutritionist.view` con:
  - Barra superior de navegación hacia `Patient` y `Aliment`.
  - `JComboBox<Patient>` para elegir el paciente (*`Listas_y_ComboBox_Swing.pdf`*).
  - `JComboBox<Aliment>` y campo de porciones (`servings`) para agregar alimentos a la dieta.
  - `JTable` mostrando las dietas/alimentos cargados y calorías totales (*`JTable_en_Swing.pdf`*).
  - Botones CRUD (`Insert`, `Search`, `Modify`, `Delete`, `Clear`).
- [ ] **B.4.2.** Conectar el botón `getBtnDiet()` en `PatientViewController` y `AlimentController` para que abra `DietView`.
- [ ] **B.4.3.** Programar el CRUD completo en `DietController.java` usando `DietDAO`, `PatientDAO` y `AlimentDAO`.

---

## Bloque C: Configurar la Conexión Real a la Base de Datos SQL

**PDFs de la cátedra para este bloque:**
- 📄 **`JPA_Hibernate_DAO.pdf`** (`> pág. 4 a 8`: librerías de JPA, estructura exacta de `META-INF/persistence.xml` e inicialización con `Persistence.createEntityManagerFactory("...")`) → [[JPA_Hibernate_DAO]]
- 📄 **`JDBC_DAO_MySQL.pdf`** (`> pág. 4 a 6`: cómo agregar el `.jar` de MySQL Connector en proyectos Ant en la carpeta `lib/`) → [[JDBC_DAO_MySQL]]

### Subtareas:
- [ ] **C.1.** Mover la carpeta `lib/` afuera de `src/` (a la raíz del proyecto `NutritionistApp/lib/`) y actualizar la ruta en *Project Properties → Libraries*.
- [ ] **C.2.** Crear la base de datos en MySQL (`CREATE DATABASE nutritionist_db;`).
- [ ] **C.3.** Crear `src/META-INF/persistence.xml` con la unidad de persistencia y los datos de conexión a MySQL (*copiar plantilla de `JPA_Hibernate_DAO.pdf`, pág. 8*).
- [ ] **C.4.** Agregar en `lib/` el conector `mysql-connector-j-9.6.0.jar` y el proveedor JPA (EclipseLink o Hibernate) para que funcione en tiempo de ejecución (*`JDBC_DAO_MySQL.pdf`, pág. 4 y `JPA_Hibernate_DAO.pdf`, pág. 4 a 6*).
- [ ] **C.5.** Inicializar `EntityManagerFactory` en `AppNutritionist.java`, pasarlo a los DAOs y cerrar el `EntityManagerFactory` al cerrar la aplicación (*`JPA_Hibernate_DAO.pdf`, pág. 7 y 8*).

---

## Bloque D: Documentación con Javadoc y Código Limpio

**PDFs de la cátedra para este bloque:**
- 📄 **`Javadoc_en_Java.pdf`** (`> pág. 1 a 13`: sintaxis `/** ... */`, qué se documenta, etiquetas `@param`, `@return`, `@throws`, `@see` y cómo generar el HTML de Javadoc) → [[Javadoc_en_Java]]
- 📄 **`Codigo_Limpio_Nombres_y_Estructura.pdf`** (`> pág. 1 a 13`: nombres con intención, funciones pequeñas de una sola responsabilidad, estructura vertical, eliminar comentarios basura autogenerados por NetBeans como `Click nbfs://...` y `@author Usuario`) → [[Codigo_Limpio_Nombres_y_Estructura]]
- 📄 **`SOLID_Principios.pdf`** (`> pág. 1 a 14`: responsabilidad única e inversión de dependencias programando contra las interfaces `PatientDAO`, `AlimentDAO`, `DietDAO` y no contra las clases `*DAOJPA`) → [[SOLID_Principios]]

### Subtareas:
- [ ] **D.1.** Borrar los encabezados basura de plantilla de NetBeans (`Click nbfs://...` y `@author Usuario`) en todos los archivos `.java`.
- [ ] **D.2.** Agregar Javadoc (`/** ... */` con `@param`, `@return`, `@throws`) a todas las interfaces y clases de `appNutritionist.dao` (*`Javadoc_en_Java.pdf`, pág. 4 a 8*).
- [ ] **D.3.** Agregar Javadoc a las entidades de `appNutritionist.model` (`Patient`, `Aliment`, `Diet`, `DietDetail`).
- [ ] **D.4.** Agregar Javadoc a los controladores de `appNutritionist.controller` y a `AppNutritionist.java`.
- [ ] **D.5.** Agregar Javadoc a todos los getters y métodos públicos agregados en `PatientView`, `AlimentView` y `DietView` (*ver ejemplo en [[convenciones]]*).
- [ ] **D.6.** Generar el Javadoc desde NetBeans (*Run → Generate Javadoc*) y verificar que se cree la carpeta `dist/javadoc/` sin errores (*`Javadoc_en_Java.pdf`, pág. 10 y 11*).

---

## Bloque E: Los 2 Diagramas UML Obligatorios

**PDFs de la cátedra para este bloque:**
- 📄 **`Diagrama_Casos_de_Uso_UML.pdf`** (`> pág. 1 a 14`: actores, casos de uso, límite del sistema y diferencia exacta entre `<<include>>` y `<<extend>>`) → [[Diagrama_Casos_de_Uso_UML]]
- 📄 **`Diagrama_de_Secuencia_UML.pdf`** (`> pág. 1 a 20`: líneas de vida, cajas de activación, mensajes síncronos vs retorno, fragmentos `alt`/`opt`/`loop`, y **en las págs. 14 a 17 incluye un ejemplo completo de MVC + DAO**) → [[Diagrama_de_Secuencia_UML]]

### E.1. Diagrama de Casos de Uso General del Sistema
- [ ] **E.1.1.** Dibujar el rectángulo del sistema (`"Sistema NutritionistApp"`) y el actor `Nutricionista` (*`Diagrama_Casos_de_Uso_UML.pdf`, pág. 4 a 6*).
- [ ] **E.1.2.** Dibujar los casos de uso para gestionar Pacientes, Alimentos y Dietas, usando `<<include>>` entre *Crear Dieta* y *Buscar Paciente* / *Buscar Alimento* (*`Diagrama_Casos_de_Uso_UML.pdf`, pág. 8 a 11*).

### E.2. Diagrama de Secuencia de un Proceso (ej. "Registrar Paciente")
- [ ] **E.2.1.** Colocar arriba de izquierda a derecha los participantes: Actor `:Nutricionista`, `:PatientView`, `:PatientController`, `:PatientDAOJPA`, `:EntityManager` (*copiar la estructura de `Diagrama_de_Secuencia_UML.pdf`, pág. 14 y 15*).
- [ ] **E.2.2.** Dibujar las flechas de llamada y retorno con un fragmento combinado **`alt`** para mostrar el camino de éxito (`commit`) y el camino de error (`rollback` + `DaoException`) (*`Diagrama_de_Secuencia_UML.pdf`, pág. 11 y 16*).

---

## Bloque F: Preparación para la Defensa Oral (Requisito Excluyente)

> `> pág. 1 — enunciado_rubrica.pdf`: *"Todos los integrantes del equipo deben estar en condiciones de defender (oralmente) todo lo desarrollado para el proyecto."*

- [ ] **F.1.** Que los 3 integrantes puedan explicar qué hace cada capa de **MVC + DAO** y por qué la Vista nunca llama al DAO directamente (*ver [[Swing_MVC_Interfaces_Java]] y [[SOLID_Principios]]*).
- [ ] **F.2.** Que los 3 integrantes puedan explicar los 5 archivos del patrón DAO (`Entidad`, `DaoException`, `DAO<T, ID>`, `PatientDAO`, `PatientDAOJPA`) usando [[tutorial-dao-desde-cero]].
- [ ] **F.3.** Que los 3 integrantes sepan explicar por qué `DaoException` es una excepción *checked* (`extends Exception`) y por qué se hace `tx.rollback()` cuando se atrapa `PersistenceException` (*ver [[Errores_y_Excepciones_Java]] y [[JPA_Hibernate_DAO]]*).
- [ ] **F.4.** Que los 3 integrantes sepan explicar los 2 diagramas UML (diferencia entre `<<include>>` y `<<extend>>` en Casos de Uso, y líneas de vida/mensajes en el de Secuencia).

---

## Relacionado

[[enunciado_rubrica]] · [[estado]] · [[entrega-02-dao-una-entidad]] · [[tutorial-dao-desde-cero]] · [[convenciones]] · [[MAP]]
