# log — Bitácora

Registro append-only de cada tanda de trabajo. Entradas nuevas arriba.

---

## 2026-11-10 — [[bug-jpql-dao]] reescrita con el estado post-rebase de RomiM

**Contexto:** RomiM hizo un rebase y un push (4 commits, rama `master`, hasta `0f060f5`) y avisó que `PatientDAOInMemory` le había dado un error. Se auditó el remoto antes de que el Estudiante hiciera el pull: **fast-forward limpio, sin conflictos**, `ahead 0`, y tanto la copia local como `origin/master` compilan con `javac --release 25` (`ExitCode: 0`). El commit propio `8c7e94c` ya estaba en el remoto. `controller/`, `view/` y `AppNutritionist` **no fueron tocados**.

**Lo que cambió en la nota:** la versión anterior describía los JPQL con nombres en **español** (`Paciente`, `apellido`, `Alimento`). RomiM los "corrigió" y los dejó con nombres de **tabla** (`patients`, `aliments`, `diets`, `p.id_dieta`), que sigue siendo JPQL inválido. Se reescribió la nota completa con:

- Tabla de las **5** consultas rotas (antes eran 4): se sumó `PatientDAOJPA:26` (`buscarPorDieta`, nuevo).
- `DietDAOJPA:19` ahora tiene **dos** errores: RomiM cambió `d.nombre` por `d.name` (nombre correcto en la tabla SQL, incorrecto en la entidad). Es el mejor ejemplo del error conceptual del commit.
- `DietDetailDAOJPA:17-18`: el JPQL ya no existe, quedó un `UnsupportedOperationException`. El modelo, en cambio, **sí** tiene las dos relaciones, así que el arreglo es de una línea.
- Sección nueva: **`Sql/nutricion.sql` no coincide con las entidades** (`diets.name` vs `Diet.nombre`, `aliments.aliment` vs `Aliment.name`). Y `schema-generation.database.action=create` en el `persistence.xml` **borra y recrea el esquema en cada arranque**, así que los `INSERT` del script se pierden.
- Sección nueva: los otros `UnsupportedOperationException` (`PatientDAOInMemory:66-67`, `DietDetailDAOJPA:18`) y la hipótesis de cuál fue el error que vio RomiM después del rebase.
- Sección nueva: los cinco `{@link}` que apuntan a clases inexistentes (`Alimento`, `Dieta`, `Paciente`, `DetalleDieta`) rompen `Generate Javadoc`.
- Deudas cerradas respecto de la nota anterior: `DietDetailDAO.buscarPorDieta` ya es `Integer` (no `Long`) y `DietDetail` ya tiene las relaciones.
- Checklist de 12 ítems y comando de `grep` para listar los JPQL inválidos en dos segundos.

**Pendiente de decisión:** el campo `cantidad` de `DietDetail` (el commit `797b39c` lo definía como *un alimento + cantidad*; hoy es un pivot puro `Diet`–`Aliment` con `UniqueConstraint`, así que un alimento no puede repetirse en una dieta ni llevar gramaje). Hay que hablarlo con RomiM antes de cablear dietas.

---

## 2026-11-10 — Plan de cableado de `Aliment` creado

**Contexto:** con `Patient` cerrado (entrada de abajo), el Estudiante pidió el mismo plan de cableado para la segunda entidad, para implementarla sin volver al chat.

**Hecho en el vault:**
- Nota nueva [[plan-cableado-aliment]] con la misma estructura que [[plan-cableado-patient]]: estado de partida verificado contra el código, mapa de nombres del `.form` a getters, `AlimentDAOInMemory` completo (6 métodos, con `buscarPorCategoria`), `AlimentTableModel` de 4 columnas, `AlimentController` con los 4 listeners y `selectRow()` con `convertRowIndexToModel`, la tabla de 8 trampas, el orden de trabajo y 10 pruebas manuales.
- Actualizados `index.md` y [[MAP]] con la nota nueva.

**Decisiones que quedaron planteadas en la nota:**
- `Category` puede seguir como `JTextField` o pasar a `JComboBox` (`Listas_y_ComboBox_Swing.pdf`, pág. 1 a 15). Hay que elegirlo **antes** de escribir el controlador, porque cambia los getters.
- La firma del controlador pasa a `(AlimentDAO, PatientDAO, AlimentView)` y se elimina el `Aliment model` que nunca se usaba.
- `AppNutritionist` pasa a crear **los dos DAO una sola vez** y `PatientController` los recibe a ambos, para que la navegación no reinicie los datos. Se deja anotado que, cuando llegue `Diet`, conviene un `AppContext`.
- El JPQL de `AlimentDAOJPA:19` (`Alimento`, `categoria`) se arregla en la Etapa B, junto con `<class>appNutritionist.model.Aliment</class>` en el `persistence.xml`.

---

## 2026-11-10 — Cableado de `Patient` terminado (A0 a A7) y checklist B.2 cerrada

**Contexto:** el Estudiante implementó a mano, en NetBeans, todo el plan de [[plan-cableado-patient]] y fue pidiendo revisión paso a paso. El proyecto pasó de no compilar a tener el CRUD de `Patient` funcionando contra un DAO en memoria.

**Qué se implementó (revisado leyendo el código, no contado por el Estudiante):**
- `appNutritionist.dao.PatientDAOInMemory` — 6 métodos (`insert`, `findById`, `findAll`, `update`, `delete`, `buscarPorApellido`), con las firmas nuevas del DAO genérico de RomiM. Sin header de NetBeans.
- `appNutritionist.view.PatientTableModel` — `AbstractTableModel` con 7 columnas, `isCellEditable()` en `false`, `getPatient(fila)` e IMC como `Double` redondeado. Sin header.
- `PatientController` — firma `(PatientDAO, PatientView)`, listeners de los 4 botones CRUD, `selectRow()` con `convertRowIndexToModel`, helpers `leerFormulario()`, `mostrarPaciente()`, `refrescarTabla()` y `resetFields()`, más `insert`, `modify`, `delete` y `search`.
- `PatientView` — `clearForm()`, `confirmInsert()`, `confirmModify()` y `confirmDelete()`; getters/setters completos.
- `AlimentController` — recibe el `PatientDAO` y lo pasa de vuelta al abrir Patients: así hay **una sola instancia** del DAO y los pacientes no desaparecen al navegar.
- Diseñador — `rowCount` de `patientsTable` en **0** (ya no se regeneran las 4 filas de `null`).

**Verificado:** `javac --release 25 -cp "lib/jakarta.persistence-2.2.3.jar;lib/flatlaf-3.5.4.jar"` sobre todo `src` → **0 errores, 0 warnings**.

**Decisiones que se resolvieron durante la implementación:**
- Los helpers del A3 van **dentro** del controlador como `private`, no en una clase aparte (necesitan `view`, `dao` y `actual`).
- `PatientTableModel` sí es clase propia, en `view`, porque es un componente de Swing.
- El reset del formulario y el de `actual` van juntos en `resetFields()`: evita que `Modify` pise al paciente que estaba seleccionado antes de un Insert.
- Se agregó `confirmModify()`, que no estaba en el plan.

**Hallazgos para el Estudiante:**
- Dos veces el cambio estaba en el editor y **no en disco**: `confirmInsert{` sin paréntesis y, después, la validación de nombre/apellido sin mover. `javac` compilaba la versión vieja. NetBeans tiene *Compile on Save*, pero compila desde memoria: no escribe el archivo.
- Quedaron dos deudas menores: falta el **botón Clear** en la vista (el método existe) y en `modify()` conviene `actual = p;` después de `dao.update(p)`, porque el DAO en memoria reemplaza el elemento de la lista.
- El `"Informacion"` de `showInfo` sigue sin tilde.

**Hecho en el vault:**
- [[hoja-de-ruta-7-a-9]]: **B.2.1 a B.2.6 marcados como completos** (con el detalle real de cada uno), B.1.2 y B.1.4 actualizados, D.1 anotado, tabla G.1 con `Patient` cableado y el aviso de "el build está roto" reemplazado por el estado real. Se agregó además la receta **C.6** para pasar `Patient` de `PatientDAOInMemory` a `PatientDAOJPA` en 7 pasos ordenados (jars, base, `persistence.xml`, `JPAUtil`, JPQL, la línea de `AppNutritionist` y prueba), con la tabla de "qué significa cada mensaje de error" y los dos riesgos de configuración (`create` vs `update` y la contraseña en el repo); se unificó el nombre de la base a `nutritionistdb`.
- [[plan-cableado-patient]]: frontmatter a `estado: implementado`, callout nuevo con la tabla de diferencias entre el plan y el código, y la tabla de "Orden de trabajo" con la columna de estado (1 a 6 hechos, 7 pendiente).
- Ninguna nota de `temas/` se tocó: esto es trabajo de la tarea, no material de la cátedra.

**Qué sigue:** commit de los 8 archivos (van juntos porque el cambio de firma acopla `PatientController` y `AlimentController`), después el Bloque B.3 (`Aliment`) o la Etapa B de MySQL.

---

## 2026-11-09 — Hoja de ruta auditada contra el código + bloque nuevo para el 10

**Contexto:** el Estudiante preguntó si el CRUD en memoria (Etapa A del cableado) suma para el 10 y recordó un comentario del profesor sobre testear sin tocar la base. Se auditó el proyecto completo para marcar lo ya hecho y se agregó lo que falta para escalar de 7–9 a 10.

**Auditoría (contra commits `008edbf`, `b357923` y cambios locales):**
- **Completos:** excepción propia y `throws` en `DAO` (Bloque 0), `Patient` y `Aliment` con anotaciones JPA y constructores (A.1.1, A.1.2, A.2.1–A.2.4), campo ID en `AlimentView`, tablas read-only en las dos vistas, IMC no editable, textos "Height (m)"/"Weight (kg)"/"Patient ID", títulos y centrado (B.1), todos los getters/setters de `PatientView` (B.2.1, con pulido pendiente), `lib/` en la raíz y `persistence.xml` (con errores).
- **Pendientes detectados:** `equals`/`hashCode` (A.1.3, A.2.5); `Diet` sin relaciones (A.3.1); `DietDetail` sin `servings` ni `@ManyToOne` (A.3.2); `DietDetailController` vacío (A.3.3); `PatientController` y `AlimentController` sin cablear (B.2.2 a B.4); proveedor JPA y driver MySQL ausentes (C.4); 7 archivos con encabezados de NetBeans (D.1); `Patient` sin Javadoc (D.3).

**Hecho en el vault:**
- Reescrita [[hoja-de-ruta-7-a-9]] con los 24 ítems completados marcados y el estado real de cada pendiente.
- **Bloque G nuevo: "Escalar de 7–9 a 10"**, con la cita de la rúbrica, la tabla de las tres entidades (falta `Diet`), la tabla de extras candidatos, el guion de defensa del DAO en memoria (citas `SOLID_Principios.pdf` pág. 28 a 33 y `JDBC_DAO_MySQL.pdf` pág. 11, 12, 19 y 20) y el warning de que el DAO en memoria **no reemplaza** la base.
- Corregida en [[plan-cableado-patient]] la decisión pendiente sobre la unidad de la altura: quedó resuelta en metros por el propio diseñador.

**Hallazgos para el Estudiante:**
- `PatientView.java:478` tiene `confirmInsert{` **sin paréntesis**: el proyecto no compila y el fix no está guardado en disco.
- El commit `b357923` unió el renombrado de `AlimentView` y el cambio `aliment` → `name` en un solo commit (se había sugerido separarlos).
- Respuesta a la consulta: el DAO en memoria **no es** la "característica extra" que pide el 10, pero sí sostiene los criterios DAO y MVC, y es lo que permite mostrar el CRUD si MySQL falla en la defensa.

---

## 2026-11-09 — Plan de cableado actualizado con los nombres nuevos y el DAO de RomiM

**Contexto:** terminado el renombrado de `AlimentView` y la entidad `Aliment`, arranca el cableado de `Patient`. La nota [[plan-cableado-patient]] estaba escrita con los nombres viejos (`jTextField1`, `getBtnAliment()`) y con un `PatientDAOInMemory` que ya no compilaría, porque el commit `797b39c` cambió las firmas del DAO.

**Hecho en el vault (reescritura de la nota):**
- **Paso A0 nuevo:** tabla de mapeo entre los componentes reales de `PatientView`, sus campos camelCase y el getter/setter que hay que exponer.
- Todo el código actualizado a los nombres reales: `getButtonInsert()`, `getTablePatients()`, `getButtonAliments()`, `idTextField`, `patientsTable`.
- `PatientDAOInMemory` reescrito para la interfaz actual: `insert`/`update` devuelven `Patient` y suma `buscarPorApellido` (6 métodos, no 5).
- Se reemplazó el `getImc()` inventado por `calcularIMC()`, que es el que existe en `Patient` desde el commit de RomiM.
- `PatientTableModel` con `isCellEditable()` en `false`: el override vive hoy en el `DefaultTableModel` anónimo del `.form` y se pierde al cambiar de modelo.
- Tabla nueva "qué se reutiliza de RomiM": `DAO` sirve para las dos etapas; `DAOJpa`/`PatientDAOJPA`/`JPAUtil` sólo para la Etapa B; `DAOException` ya deja compilar el `catch` hoy.
- Etapa B reescrita como checklist de configuración, con las tres bombas de RomiM: nombre de la PU (`nutricionPU` vs `persistence.xml`), base de datos `mysql` (catálogo del sistema) y `schema-generation=create` (borra en cada arranque).

**Hallazgos nuevos anotados:**
- La altura tiene que estar en metros para que `calcularIMC()` dé bien; quedó como decisión pendiente (metros vs. centímetros).
- `dietComboBox` tiene 4 ítems de relleno ("Item 1"…"Item 4").
- `imcTextField` ya tiene `editable=false` y tooltip explicativo; `idTextField` sí es editable, así que el flujo de búsqueda por ID funciona.
- `AppNutritionist.java:22` todavía hace `new PatientController(new Patient(), vista)`: hay que cambiar la firma cuando se cablee el DAO.

---

## 2026-11-09 — Revisión del renombrado de `AlimentView` y del atributo `Aliment.aliment` → `name`

**Contexto:** tras el renombrado de `PatientView` a camelCase, el Estudiante renombró las 22 variables de `AlimentView` con los nombres sugeridos por rol y cambió el atributo `aliment` de la entidad `Aliment` por `name`. Se pidió revisar typos, errores de compilación e incoherencias antes de commitear.

**Verificado en el proyecto NetBeans (`NutritionistApp`):**
- `javac --release 25` → **exit 0**. Las 22 variables de `AlimentView` quedaron en camelCase (`mainPanel`, `actionsPanel`, `alimentsTable`, `tableScrollPane`, `titleLabel`…), `.form` y `.java` sincronizados, **cero** restos de `jPanel`/`jButton`/`jTextField`/`idTx`, y los eventos del diseñador renombrados junto con los campos.
- Getters de navegación simétricos entre las dos vistas: `getButtonPatients()` / `getButtonDiets()`, y `AlimentController:17` ya usa `view.getButtonPatients()`.
- `Aliment`: campo, getter, setter y `toString` pasados a `name`; no queda ningún `getAliment()`/`setAliment()` en el proyecto.

**Correcciones aplicadas por el Estudiante (revisadas y OK):**
- `Aliment(String aliment, …)` → `Aliment(String name, …)`.
- Tabla de `AlimentView` en `editable="false"` para las 4 columnas + `isCellEditable()` con `canEdit[]`, igual que `PatientView`.

**Pendientes marcados al Estudiante (no bloquean el commit):**
- Tooltip del botón Search: `"Input and ID and click…"` → `"Input an ID and click…"`.
- Los `name` properties de NetBeans siguen con el esquema viejo (`insertTx`, `modifyBt`, `deleteTx`, `searchTx`, `alimentTx`, `categoryTx`, `caloriesPerServingTx`). Cosmético: nadie llama a `getName()`.

**Hecho en el vault:**
- Creada [[bug-jpql-dao]] en `tarea/`: las 4 consultas JPQL de los `*DAOJPA` usan nombres de entidad en español (`Alimento`, `Dieta`, `DetalleDieta`, `Paciente`) y campos inexistentes (`categoria`, `dieta`, `apellido`). Va con archivo, línea exacta, consulta corregida, por qué `javac` no lo detecta y quién lo dispara al cablear los buscadores.
- Agregada en `index.md` (Tarea → Notas de trabajo) y en el árbol de `MAP.md`.

**Cita usada:** «JPQL trabaja sobre entidades y atributos en lugar de tablas y columnas» — pág. 13 a 16, `JPA_Hibernate_DAO.pdf`.

---

## 2026-11-07 — Nota nueva: plan de cableado de `Patient` (Vista ↔ Controlador ↔ Modelo)

**Contexto:** se pidió el plan para cablear la entidad `Patient` usando `PatientController` como puente entre la vista y el modelo, guiándose por los PDFs de la cátedra, para implementarlo a mano.

**Hecho en el vault:**
- Creada [[plan-cableado-patient]] en `tarea/` con el plan completo: contrato de getters de la vista, `PatientDAOInMemory` como doble de prueba, `PatientTableModel` (`AbstractTableModel`), helpers del controlador, listeners de los 4 botones, limpieza de handlers de NetBeans y la Etapa B de persistencia.
- Agregada en `index.md` (Tarea → Notas de trabajo) y en el árbol de `MAP.md`.
- Registradas las citas usadas: MVC (pág. 20 a 23) y eventos (pág. 14) de `Swing_MVC_Interfaces_Java.pdf`; `AbstractTableModel` y configuración de tabla (pág. 4 a 18) de `JTable_en_Swing.pdf`; `persistence.xml` y `EntityManagerFactory` (pág. 8 y 9) de `JPA_Hibernate_DAO.pdf`.

**Hallazgos del relevamiento (2026-11-07):**
- El repo local está **sincronizado** con `origin/master`: sin commits nuevos de compañeros ni cambios sin commitear.
- **No existe** `src/META-INF/persistence.xml`, ni proveedor JPA (Hibernate), ni driver MySQL en `lib/`. La capa de persistencia no puede ejecutarse todavía: por eso el plan separa Etapa A (sin base) y Etapa B (con base).
- La vista sólo expone `getBtnAliment()` y `getBtnDiet()`: falta todo el contrato de lectura/escritura del formulario, los 4 botones de acción y la tabla.

---

## 2026-11-06 — Nota nueva: ejemplo de excepciones (mensajes personalizados en el DAO)

**Contexto:** quedó pendiente el paso de `DaoException` en el proyecto, y el equipo armó un borrador explicando por qué cada método del DAO lleva su propio mensaje. Se pidió formatearlo y guardarlo en el vault.

**Hecho en el vault:**
- Creada [[ejemplo-de-excepciones]] en `tarea/`: qué pide la rúbrica en el criterio *Manejo de Excepciones* (pág. 2 — `enunciado_rubrica.pdf`), el patrón de mensajes del profesor (pág. 16 a 18 — `JDBC_DAO_MySQL.pdf`; pág. 18 a 22 — `JPA_Hibernate_DAO.pdf`), tabla de los 5 mensajes × 4 entidades, código completo de `DaoException`, `DAO` y `PatientDAOJPA`, la conexión con `JOptionPane` desde el controlador, y un checklist de aplicación.
- Agregada la nota en `index.md` (sección Tarea → Notas de trabajo) y en el árbol de `MAP.md`.

**Correcciones detectadas al revisar el código real (van marcadas en la nota):**
- `DaoException.java` **no existe** en `src/appNutritionist/dao/` (sólo está en `Desktop\Entrega`); el borrador asumía que ya estaba.
- `Aliment` **ya tiene `getId()`** desde el commit `d8e40df`, así que el mensaje de `update` sí puede incluir el id (el borrador lo omitía por considerarla vacía).
- El borrador usaba `SELECT p FROM Aliment p` (alias `p` para un alimento); se sugirió `a`.

**Decisión pendiente que surgió de la nota:** en qué idioma van los mensajes de excepción, dado que la UI está en inglés y el profesor escribe los mensajes en español.

---

## 2026-11-06 — Rediseño de la UI con FlatLaf, autozoom y sincronización de `PatientView` con `AlimentView`

**Contexto:** tras la reestructuración a PascalCase quedaban dos frentes abiertos: (1) las vistas seguían con la estética por defecto de Swing, sin la librería sugerida por el profesor Santini; (2) `PatientView` se había quedado atrás respecto de `AlimentView`, que ya estaba rediseñada. En paralelo, la entrega de `DaoException` quedó reservada para otro integrante del equipo (para que el historial de GitHub muestre aportes de todos).

**Hecho en el proyecto NetBeans (`NutritionistApp`):**
- Movidas las librerías a `lib/` en la **raíz** del proyecto: `flatlaf-3.5.4.jar` y `jakarta.persistence-2.2.3.jar`, ambas declaradas en `nbproject/project.properties` (`javac.classpath`). Se abandonó `javax.persistence-api-2.2` a favor de la implementación `jakarta.persistence-2.2.3`.
- `AppNutritionist.java`: inicialización de **FlatLaf** con `FlatLightLaf.setup()`, arcos globales de la UI (`Button.arc = 999`, `TextComponent.arc = 15`, `Component.arc = 15`), y `--enable-native-access=ALL-UNNAMED` en las VM Options para evitar los *warnings* del JDK.
- Layout general: la UI se dividió en `JPanel` independientes (navegación / formulario / acciones / tabla) y el `JFrame` usa `GridBagLayout` para centrar el contenido. La ventana mantiene proporción y centrado al abrirse.
- `AlimentView` rediseñada como **vista de referencia**: `TitledBorder` en "Aliment Data" y "Actions", paleta de colores (nav `#00695C`, Search `#00838F`, Insert `#2E7D32`, Modify `#1565C0`, Delete `#C62828`, fondo `#E8F5E9`), `JTable` con `autoCreateRowSorter`, `SINGLE_SELECTION`, grilla visible y modelo `ID / Name / Category / Calories`, dentro de un `JScrollPane` de Swing.
- Implementados en ambas vistas `enableAutoZoom()` y `collectComponents(...)`: recorren el árbol de componentes, guardan la fuente y el tamaño base de cada uno y, al redimensionar la ventana, escalan fuentes, alturas de fila y tamaños preferidos (con tope inferior en `1.0` para no achicar).
- `PatientView.java` + `.form` **sincronizados con `AlimentView`**: mismo header de navegación (`[Aliments] — Patients — [Diets]`), `TitledBorder` centrado en "Actions", botón `Search` en turquesa con tooltip, corrección del cruce entre `idTx` y `lastNameTx` en el panel de datos, `horizontalAlignment = CENTER` en todos los `JTextField`, tabla reconfigurada con las columnas `ID / Name / Last Name / Weight / Height / IMC / Diet` y eliminación del `jPanel5` intermedio que dejaba la tabla clavada en 100 px.
- Renombres propagados en los 4 DAO JPA: `save` → `insert` y `listAll` → `findAll`, con las consultas JPQL corregidas (`FROM Patient`, `FROM Aliment`, `FROM Diet`, `FROM DietDetail`).
- Verificada la compilación completa con `javac --release 25 -cp "lib/jakarta.persistence-2.2.3.jar;lib/flatlaf-3.5.4.jar"` → exit code 0, 0 errores.

**Hecho en Git:**
- El push fue rechazado porque un compañero (`turquoise28`) había subido el commit `d8e40df` (*"Aliment.java hecho/Diet.java y DietDetail.java en proceso"*, sólo 3 archivos de `model`). Como no había archivos en común con los cambios de UI, se resolvió con **Rebase** en lugar de Merge: historial lineal, sin commit de unión ni conflictos.
- El commit de UI quedó como `be82843` sobre `d8e40df` y ya está en `origin/master` (rama `master` sincronizada).
- Sin commitear todavía: `DaoException` y el reemplazo de `RuntimeException` en los DAO, a la espera de que lo suba otro integrante.

**Hecho en el vault:**
- Entrada nueva en esta bitácora; sin cambios de contenido en [[estado]], [[hoja-de-ruta-7-a-9]] ni [[entrega-02-dao-una-entidad]] (se mantienen vigentes tal como quedaron el 2026-11-03).

**Pendiente / próximos pasos:**
- Bloque 0 de [[hoja-de-ruta-7-a-9]]: `DaoException` en la interfaz `DAO` y en los 4 `*DAOJPA` (a cargo de un compañero).
- Completar `model/Aliment.java`, `model/Diet.java` y `model/DietDetail.java` (Bloque A).
- Definir si se elimina `DietDetailController` (vacío) o se absorbe en `DietController`.

---

## 2026-11-03 — Corrección de la Entrega 2 (DAO de `Patient` con JPA), limpieza de nombres y tutorial DAO

**Contexto:** el profesor Esteban Santini devolvió dos correcciones sobre la Entrega 2 (carpeta `Desktop\Entrega`):
1. *"Traten de mejorar los nombres de las clases: `modelPatient` es en realidad la clase `Patient` en el paquete `model` (algo similar con el `GenericDAO`)."*
2. *"Implementen una excepción para el DAO."*

**Hecho en el proyecto NetBeans (`NutritionistApp`):**
- Reestructurados todos los paquetes en minúscula y singular bajo `appNutritionist`:
  - `appNutritionist.model` (`Patient.java`, `Aliment.java`)
  - `appNutritionist.dao` (`DAO.java`, `DaoException.java`, `PatientDAO.java`, `PatientDAOJPA.java`)
  - `appNutritionist.view` (`PatientView.java`, `PatientView.form`, `AlimentView.java`, `AlimentView.form`)
  - `appNutritionist.controller` (`PatientController.java`, `AlimentController.java`)
- Eliminados los paquetes viejos (`Models`, `Views`, `Controller`) y actualizadas las referencias en `AppNutritionist.java`.
- Implementada `DaoException` (`extends Exception`) siguiendo `JDBC_DAO_MySQL.pdf` (pág. 13) y `JPA_Hibernate_DAO.pdf` (pág. 20 a 22).
- Implementada la interfaz genérica `DAO<T, ID>` con métodos `insert`, `findById`, `findAll`, `update`, `delete` que declaran `throws DaoException`.
- Implementada `PatientDAO extends DAO<Patient, Integer>` agregando `findByLastName(String lastName) throws DaoException`.
- Implementada `PatientDAOJPA` envolviendo `PersistenceException` en `DaoException` con `tx.rollback()` cuando corresponde, y corrigiendo las consultas JPQL a `"SELECT p FROM Patient p"`.
- Descargada la librería `javax.persistence-api-2.2.jar` en `NutritionistApp/lib/` y vinculada en `nbproject/project.properties` (`javac.classpath`).
- Verificada la compilación completa con `javac --release 25 -Xlint:all -cp "lib/javax.persistence-api-2.2.jar"` → exit code 0, 0 errores.

**Hecho en la carpeta `Desktop\Entrega`:**
- Reemplazados los archivos viejos por los 5 archivos corregidos: `Patient.java`, `DAO.java`, `DaoException.java`, `PatientDAO.java` y `PatientDAOJPA.java`.

**Hecho en el vault:**
- Creada [[tutorial-dao-desde-cero]] en `tarea/` con la explicación paso a paso del patrón DAO desde cero tanto en JPA como en JDBC para el equipo.
- Creada [[hoja-de-ruta-7-a-9]] en `tarea/` con el checklist detallado (Bloques 0 a E) vinculado a los PDFs y páginas específicas de la cátedra para cada subtarea.
- Actualizadas [[entrega-02-dao-una-entidad]], [[estado]], `ME.md`, `MAP.md` e `index.md` reflejando la estructura actual de `C:\Users\games\Desktop\Proyectos\NutritionistApp` tras el refactor manual sobre el clon de GitHub (dejando el commit de `DaoException` listo para que lo suba otro integrante del equipo).

---

## 2026-10-16 — Incorporación de los PDFs de Diagrama de Secuencia y Javadoc

**Hecho:**
- Añadidos a `fuentes/pdf/` los dos PDFs faltantes: `Diagrama_de_Secuencia_UML.pdf` y `Javadoc_en_Java.pdf`.
- Creada la nota de síntesis [[Diagrama_de_Secuencia_UML]] en `temas/Teoria/` (pág. 1 a 20), cubriendo casos de uso vs. secuencia, componentes, tipos de mensajes, fragmentos combinados (`alt`, `opt`, `loop`) y el flujo en arquitectura MVC + DAO con ejemplos del sistema de biblioteca.
- Creada la nota de síntesis [[Javadoc_en_Java]] en `temas/Teoria/` (pág. 1 a 13), cubriendo la herramienta oficial del JDK, estructura de comentarios `/** ... */`, etiquetas esenciales (`@param`, `@return`, `@throws`, `@see`), comparación con otros lenguajes y generación de documentación HTML.
- Actualizados los contadores del vault (21 PDFs, 21 notas) en `MAP.md`, `index.md` y `ME.md`.

---

## 2026-10-15 — Correcciones: getter `Diets`, `src.zip` y guía de sync

**Hecho:**
- Eliminado `src.zip` del repo `NutritionistApp` (era un zip manual de `src/`, sin trackear; el repo quedó limpio).
- [[convenciones]]: corregida la afirmación obsoleta «el botón `Diets` no tiene getter». Ahora documenta la **excepción deliberada**: ambas vistas exponen `getBtnDiet()` (commit `8fc5218`) aunque la entidad `Diet` no exista. Nueva subsección «El caso del botón `Diets`» y duda pendiente sobre el nombre (`getBtnDiet` vs campo `dietsBt`/texto «Diets»).
- [[estado]]: callout que aclara que el getter `getBtnDiet()` ya existe en las dos vistas.
- [[Sincronizacion_Notebook]] (Fase D): reescrita. Se aclaró que `community-plugins.json` **no** auto-instala el plugin: hay que commitear también su código (`main.js`, `manifest.json`, `styles.css`). Se documentó excluir `data.json`.
- `.gitignore`: agregado `.obsidian/plugins/*/data.json` (evita filtrar config/token de plugins).
- Instalado el plugin **Obsidian Git** (Vinzent03) en esta PC; su código quedó versionado.

**Fuentes verificadas (no es contenido del profesor):**
- Foro de Obsidian: `community-plugins.json` no reinstala plugins automáticamente al clonar.
- `obsidian-git` (Vinzent03): en escritorio la autenticación la maneja el git del sistema, no `data.json`.

---

## 2026-10-04 — Reestructuración de `tarea/`: esquema por entregas

**Contexto:** el profesor entrega el proyecto de forma **incremental**; cada trabajo agrega funcionalidad hasta la entrega final (rúbrica). El eje de organización de `tarea/` pasa a ser **la entrega**.

**Hecho:**
- Creada la carpeta `tarea/entregas/` con una nota por entrega:
  - [[entrega-01-interfaces-graficas]] (✅ entregada): GUI de `Patient` y `Aliment`.
  - [[entrega-02-dao-una-entidad]] (⬜ pendiente): DAO de una entidad con JDBC o JPA (consigna textual del profesor incluida).
- Renombradas las notas de trabajo (sin número, con `aliases` al nombre viejo y H1 sin prefijo numérico):
  - `00-estado` → [[estado]]
  - `03-vistas-manuales` → [[vistas-manuales]]
  - `04-convenciones` → [[convenciones]]
- Eliminado `02-plan.md` (vacío, mal ubicado en la raíz).
- Reapuntados todos los `[[enlaces]]` a las notas renombradas (13 archivos). `log.md` se dejó intacto (append-only; los enlaces viejos resuelven por `aliases`).
- Limpiadas las referencias muertas a `01-modelo` y `02-plan`.
- Actualizados `index.md` (sección Tarea dividida en Entregas / Notas de trabajo), `ME.md` (tabla de Estado al día) y `MAP.md` (árbol de `tarea/` y tabla de notas inexistentes).

---

## 2026-10-04 — Guía de sincronización en la notebook

**Hecho:**
- Creada [[Sincronizacion_Notebook]] con los pasos para clonar el vault en la notebook (Fase C) y configurar el plugin Obsidian Git en ambas máquinas (Fase D).

---

## 2026-10-04 — Vault versionado y sincronizado con GitHub

**Hecho:**
- Inicializado el vault como repositorio git (rama `main`) y publicado en el repo **privado** `francoemmanuelmartinez/vault-programacion-oo`.
- Creado `.gitignore` del vault excluyendo lo específico de cada máquina: `.obsidian/workspace.json`, `.obsidian/workspace-mobile.json`, `.obsidian/cache/` y `.trash/`.
- Commit inicial `4a2ab7d` (55 archivos) pusheado a `origin/main`.
- Objetivo: copiar el vault a la notebook clonando el repo y mantenerlo sincronizado con el plugin **Obsidian Git**.

---

## 2026-10-03 — Reformulación del orden de procesamiento sugerido

**Hecho:**
- Reformulado el "Orden de procesamiento sugerido" en `MAP.md`: el paso 2 ahora habla de "teoría y conceptos" (incluyendo POO, SOLID y código limpio) y el paso 4 aclara que los diagramas UML viven en `temas/Teoria/` (Bloque 1), coherente con la reorganización de carpetas.

---

## 2026-10-03 — Diagramas UML movidos a `temas/Teoria`

**Hecho:**
- Movidas `Diagrama_Casos_de_Uso_UML.md` y `DiagramaClases_Ej2_Biblioteca.md` de `temas/practica/` a `temas/Teoria/`.
- Actualizados `index.md` y `MAP.md`: ambas notas reubicadas en el Bloque 1 (Teoría y conceptos); comentarios del árbol de carpetas ajustados.
- `temas/practica/` queda sólo con las notas de código (Swing, JDBC y JPA).

---

## 2026-10-03 — Reorganización de carpetas: `temas/POO` → `temas/Teoria`

**Hecho:**
- Renombrada la carpeta `temas/POO/` a `temas/Teoria/`.
- Movidas dos notas de `temas/practica/` a `temas/Teoria/` por ser conceptuales, no de código:
  - [[SOLID_Principios]] y [[Codigo_Limpio_Nombres_y_Estructura]].
- Actualizados `index.md` y `MAP.md`: Bloque 1 renombrado a "Teoría y conceptos" y las dos notas reubicadas en su tabla; árbol de carpetas de `MAP.md` corregido.
- Los `[[links]]` de Obsidian no se tocaron (resuelven por nombre de archivo).

---

## 2026-10-03 — Expansión completa de notas de componentes de Swing (JTable, Listas y Checkboxes)

**Hecho:**
- Ampliadas y enriquecidas las tres notas restantes de componentes gráficos y modelos de Swing en `temas/practica/` para dejarlas al mismo nivel de detalle que JPA y JDBC:
  - [[JTable_en_Swing]]: incorporadas las tres formas de carga (array, `DefaultTableModel`, y `AbstractTableModel` propio), configuración avanzada (`setAutoCreateRowSorter`, ancho de columnas) y el ejemplo MVC completo de calificaciones de alumnos (`Alumno`, `CalificacionesModelo`, `AlumnosTableModel`, `VistaAlumnos`, `CalificacionesControlador`).
  - [[Listas_y_ComboBox_Swing]]: incorporados `JList`, `DefaultListModel`, `JComboBox`, eventos (`ListSelectionListener` vs `ActionListener`), selección múltiple y el ejemplo MVC completo del catálogo por categoría (`CatalogoModelo` con `Map`, `CatalogoVista`, `CatalogoControlador`).
  - [[Checkbox_y_RadioButton_Swing]]: incorporados `JCheckBox`, `JRadioButton`, el rol fundamental de `ButtonGroup` para la exclusividad lógica, `setActionCommand()` y los dos ejemplos MVC completos: el de ingredientes de un pedido (`Ingrediente`, `PedidoModelo`, `PedidoVista`, `PedidoControlador`) y el de métodos de pago (`MetodoPago`, `PagoModelo`, `PagoVista`, `PagoControlador`).

---

## 2026-10-03 — Expansión de notas prácticas con código completo de la cátedra

**Hecho:**
- Ampliadas y enriquecidas dos notas clave del Bloque 2 (`temas/practica/`) para que tengan el mismo nivel de detalle técnico, fragmentos de código completos y ejemplos paso a paso que la nota de JPA:
  - [[JDBC_DAO_MySQL]]: incorporados los bloques de código completos de la entidad `Persona`, la excepción `DaoException`, la interfaz genérica `DAO<T, ID>`, la especialización `PersonaDAO` y la implementación completa `PersonaDAOMySQL` (con `PreparedStatement`, `try-with-resources`, `RETURN_GENERATED_KEYS` y recorrido de `ResultSet`).
  - [[Swing_MVC_Interfaces_Java]]: incorporados los ejemplos completos del ciclo de Swing, el patrón Observer con lambdas, los layout managers y el ejemplo completo del "Saludador MVC" (`SaludadorModelo`, `SaludadorVista`, `SaludadorControlador` con `SwingUtilities.invokeLater`), además de la variante avanzada de la *Action API* (`AbstractAction`, `SaludarAction`).

---

## 2026-10-03 — Incorporación de la fuente 19: JPA_Hibernate_DAO.pdf

**Hecho:**
- Detectado que el PDF `JPA_Hibernate_DAO.pdf` (505 KB) estaba en la carpeta de trabajo del profesor (`Desktop\poo`) pero faltaba en el vault. Los otros 18 eran idénticos byte a byte.
- Copiado a `fuentes/pdf/JPA_Hibernate_DAO.pdf`.
- Creada la nota de síntesis [[JPA_Hibernate_DAO]] en `temas/practica/` (pág. 1 a 24), cubriendo JPA, Jakarta Persistence, Hibernate, configuración con `persistence.xml`, entidades anotadas, operaciones CRUD con `EntityManager`, JPQL y el patrón `PersonaDAOJPA`.
- Actualizados los contadores del vault (19 PDFs, 19 notas):
  - `MAP.md`: agregado al Bloque 2 y actualizado contador de archivos y notas.
  - `ME.md` e `index.md`: actualizado contador a 19.

**Por qué es clave para el proyecto:**
- La rúbrica exige persistencia en base de datos mediante el patrón DAO. Hasta ahora solo contábamos con JDBC puro (`JDBC_DAO_MySQL.pdf`). Este PDF aporta la segunda opción (JPA/Hibernate), que simplifica el mapeo y califica como la característica de investigación avanzada para la Nota 10.

---

## 2026-10-03 — Convención de getters en las vistas

**Hecho:**
- Nueva nota [[04-convenciones]] en `tarea/`. Fija la regla de exponer los
  componentes de la vista por getters y dejar los campos `private`, "siempre que
  sea posible".
- La nota distingue dos niveles, porque no todos los getters deberían devolver el
  componente:
  - **Nivel 1**, devuelve el componente. Alcanza para colgar listeners.
    `getBtnAliment()` → `JButton`.
  - **Nivel 2**, devuelve el dato. Preferido en campos de formulario.
    `getNombre()` → `String`, no el `JTextField`.
  - Regla de elección: si el controlador solo agrega listeners, Nivel 1. Si lee o
    escribe valores, Nivel 2.
- También deja asentado que la regla de no abreviar aplica a los comentarios y no
  a los nombres, que era la duda que había dejado `getBtnAliment()` a medias.

**Hallazgo relevante:**
- La rúbrica **ya estaba procesada** (`temas/tarea/enunciado_rubrica.md`,
  `estado: completo`). `00-estado.md` y el `log.md` la trataban todavía como un
  bloqueante sin leer. Ya no.
- Revisé el texto: **no pide getters, encapsulamiento ni nivel de acceso**. Así
  que la convención se anotó explícitamente como decisión propia del proyecto, no
  como regla de la cátedra. El apoyo real es indirecto:
  - pág. 1 — Javadoc obligatorio en métodos y clases públicas → los getters
    públicos heredan la obligación.
  - pág. 1 y 2 — criterio *Documentación y Código Limpio*: "nombres descriptivos"
    → `jButton1` es justo el tipo de nombre que penaliza en *Insuficiente*.
  - pág. 3 a 7 — `Codigo_Limpio_Nombres_y_Estructura.pdf`: revelar intención.

**Riesgo 2 de `00-estado.md`, reevaluado:**
-_decía_ que las vistas del GUI Builder podían ser un problema si la rúbrica
  penaliza el designer. Ya se leyó: el criterio *Interfaz Gráfica de Usuario*
  evalúa claridad, ausencia de errores, intuición y completitud para todas las
  entidades. **No menciona cómo se construyeron las ventanas.** El riesgo baja a
  bajo. El riesgo real es otro: *Documentación y Código Limpio* pide código
  propio, y `initComponents()` no se puede documentar.

**Limpieza de contenido desactualizado:**
- `00-estado.md`: se sacaron los números de commit fijos y ahora apunta a
  `log.md` como fuente de verdad, para que no vuelva a pudrirse. Se reemplazó el
  "Próximo paso: leer la rúbrica" por las preguntas que siguen abiertas.
- `MAP.md`: el bloque de estructura listaba `tarea/` con `00-enunciado.md`,
  `01-modelo.md` y `02-plan.md`, que no existen. Corregido, con una advertencia
  que aclara que esas dos últimas están pendientes de escribir.
- `index.md`: alta de [[04-convenciones]] y aclaración de que la rúbrica ya no es
  un bloqueante.
- `00-estado.md`: el link de "Relacionado" apuntaba a `[[00-enunciado]]`, que no
  existe. Apunta ahora a `[[enunciado_rubrica]]`, que sí.

**Decisiones:**
- La nota va en `tarea/` (capa "Aplicación") y no en `temas/practica/`. Las notas
  de `temas/` sintetizan al profesor; mezclar ahí una convención propia rompería
  la separación de fuentes que pide `AGENTS.md`.
- No se renombró ningún getter. `getBtnAliment()` y `getBtnPatient()` quedan.

**Pendiente:**
- `01-modelo` y `02-plan` siguen sin escribirse. Los referencia `00-estado.md` e
  `index.md` como `⬜`.

**Fuera de scope:** el getter y el Javadoc de `getBtnAliment()` todavía no están
en el código. La nota dice que hacen falta; el cambio en
`NutritionistApp` quedó pendiente y sin commitear.

---

## 2026-10-03 — Encapsular los botones de navegación detrás de getters

**Hecho:**
- Los botones de navegación dejó de exponerse como campos públicos.
  - `Views/viewPatient.java`: `alimentBt` y `dietsBt` ahora son `private`. Se
    agregó `getBtnAliment()`.
  - `Views/viewAliment.java`: `jButton1` volvió a `private`. Se agregó
    `getBtnPatient()`.
  - `Controller/controllerPatient.java` y `Controller/controllerAliment.java` ahora
    llaman `view.getBtnAliment()` y `view.getBtnPatient()`.
- Se quitó de los tres `.form` el `AuxValue` con
  `JavaCodeGenerator_VariableModifier = 1`, que era lo que forzaba los campos a
  `public`. Sin eso, NetBeans los volvería a generar públicos en la próxima
  regeneración.
- Verificado: `javac --release 25 -Xlint:all` sobre los 7 `.java` → exit 0,
  6 warnings, 0 campos públicos en las vistas, 0 `AuxValue` en los `.form`,
  ambos `.form` con XML válido.

**Por qué (corrección de una decisión propia):**
- El plan original de la transición especificaba getters, pero la implementación los
  reemplazó por campos públicos para replicar el estilo que ya tenía
  `viewPatient`. Fue un error: ataba el controlador a nombres autogenerados por
  NetBeans. Si el designer renombra el componente, el controlador deja de
  compilar.
- Los getters van **fuera** de las regiones `GEN` (`//GEN-BEGIN` /
  `//GEN-END`), que es la única forma de que NetBeans no los borre al regenerar
  el `.java`. El `.form` no tiene forma de declarar métodos, sólo campos.

**Base en la wiki:**
- `> pág. 3 a 7 — Codigo_Limpio_Nombres_y_Estructura.pdf`: revelar intención en
  los nombres. `jButton1` no revela nada.
- `> pág. 4 a 9 — SOLID_Principios.pdf`: una única razón para cambiar. Si el
  controlador toca los campos del designer, un renombre en la vista rompe al
  controlador.

> [!warning] Ampliación de la IA, no del profesor
> Que los widgets de Swing se expongan con getters sale de OOP general. Ninguno de
> los dos PDFs dice eso explícitamente.

**Decisiones:**
- No se agregó `getBtnDiets()`. Sería código sin usar: el botón `Diets` no tiene
  listener porque no existe la entidad `Diet`. Se suma cuando exista.
- Se mantiene el nombre `getBtnAliment()` del plan original y de
  `03-vistas-manuales.md`. **Corrección posterior:** había anotado que
  `getAlimentButton()` sería mejor por no abreviar "button". El Estudiante
  aclaró que la regla de no abreviar aplica a los **comentarios**, no a los
  nombres de identificadores, así que `getBtnAliment()` queda como está.
  Queda asentado en [[04-convenciones]].

**Pendiente:**
- Commit de este cambio: **no hecho**, esperando aprobación del Estudiante.

---

## 2026-10-03 — Navegación entre vistas Patient ↔ Aliment

**Hecho:**
- Implementada la transición recíproca entre las dos ventanas usando los botones
  de la parte superior. Cada controlador cierra la vista actual con `dispose()` y
  crea modelo, vista y controlador del destino dentro de
  `SwingUtilities.invokeLater`.
  - `Controller/controllerPatient.java` → listener sobre `view.alimentBt`.
  - `Controller/controllerAliment.java` → listener sobre `view.jButton1`.
- `Views/viewAliment.form`: se agregó
  `JavaCodeGenerator_VariableModifier = 1` al componente `jButton1` para que el
  campo quede `public` **y siga así** cuando NetBeans regenere el `.java`.
- Limpieza de código muerto: los dos `main()` y los dos `logger` de las vistas,
  el handler vacío `jButton3ActionPerformed` (y su bloque `<Events>` en el
  `.form`), dos imports de `ActionEvent` sin uso, `setLabel` deprecado → `setText`,
  `setToolTipText("")` inútil, y líneas en blanco sueltas.
- Commit `7fc3180` — "Code cleanup and navigation between Patient and Aliment
  buttons". 10 archivos, +46 / −90. El repo queda con 2 commits y el working
  tree limpio.

**Base en la wiki:**
- `> pág. 20 a 23 — Swing_MVC_Interfaces_Java.pdf`: la vista solo compone la
  interfaz; el controlador registra los listeners y coordina las acciones.
- `> pág. 28 — Swing_MVC_Interfaces_Java.pdf`: la interfaz debe crearse en el
  hilo de eventos con `SwingUtilities.invokeLater(() -> ...)`.
- `> pág. 2 — Codigo_Limpio_Nombres_y_Estructura.pdf`: "Dejá el código un poco
  mejor de como lo encontraste."

**Aprendizaje sobre el build (importante):**
- Compilar a mano con el **JDK 26 sin `--release 25`** produce `.class` versión
  70. NetBeans corre con JDK 25, que sólo reconoce hasta la 69, y Ant no
  recompila porque los `.java` son más viejos que los `.class`. Resultado:
  `UnsupportedClassVersionError` con un mensaje engañoso de "no se ha encontrado
  el método principal".
- siempre con `javac --release 25`, y sobre los **7 `.java`**, no sólo
  `AppNutritionist.java`: `controllerAliment` y `modelAliment` todavía no los
  referencia nadie y quedan sin verificar si se compila sólo desde el entry point.
- Comando de verificación usado:
  `javac --release 25 -Xlint:all -d build/classes <los 7 .java>` → exit 0,
  6 warnings (2 `[serial]` + 4 `[this-escape]`, inherentes a heredar de `JFrame`).

**Otro error propio (anotado para no repetirlo):**
- Al reescribir archivos enteros se perdieron los saltos de línea finales y
  quedaron con finales LF. Git avisaba con `LF will be replaced by CRLF`.
  Corregido a CRLF. Verificado que ningún archivo quede sin salto final.

**Decisiones:**
- El botón `Diets` queda **sin listener** en las dos vistas: no existe entidad
  `Diet` todavía, y no se va a inventar código para una vista que no está
  especificada.
- Estrategia de transición: `dispose()` y nueva instancia. La alternativa de
  `setVisible(false)` quedó documentada como contingencia en `03-vistas-manuales`.

**Pendiente:**
- Botón `Diets` inerte en las dos vistas. Falta la entidad `Diet` completa.
- Las ventanas destino no llevan `setLocationRelativeTo(null)`, así que pueden
  aparecer pegadas en el borde superior izquierdo. Es comportamiento de NetBeans,
  no introducido por este cambio.
- El campo `model` de los dos controladores se asigna pero no se lee: es el
  contrato del constructor que la cátedra teach, no se tocó.
- Sigue bloqueante leer `enunciado_rubrica.pdf`.

---

## 2026-10-03 — Creación de la LLM Wiki completa (18 PDFs procesados)

**Hecho:**
- Procesados los **18 PDFs** oficiales de la cátedra de Programación Orientada a Objetos (Prof. Esteban Santini, ISFT 204).
- Creadas todas las notas de síntesis en `temas/POO/` (Bloque 1: Abstracción, Métodos, Herencia, Herencia Múltiple e Interfaces, Polimorfismo, Equals, Excepciones, Repaso y Diagrama de Clases).
- Creadas todas las notas de síntesis en `temas/practica/` (Bloque 2: Principios SOLID, Código Limpio, Swing MVC, Listas y ComboBox, JTable, Checkbox y RadioButton, JDBC & DAO, Casos de Uso UML, Diagrama de Clases - Biblioteca).
- Actualizados `MAP.md` e `index.md` con el estado completo de la wiki (18 notas creadas).

**Pendiente:**
- Esperar instrucciones del Estudiante para avanzar con la capa de aplicación (`tarea/`).

---

## 2026-10-03 — Procesamiento de enunciado_rubrica.pdf

**Hecho:**
- Procesado `enunciado_rubrica.pdf` y creada la nota de síntesis `temas/tarea/enunciado_rubrica.md`.
- Extraídos los requerimientos para la nota 10: 3 entidades (dos combinadas en una tercera), capas MVC, DAO con SQL, Swing GUI, excepciones, Javadoc y diagramas UML (Casos de Uso y Secuencia).
- Actualizados `MAP.md` e `index.md`.

**Pendiente:**
- Diseñar el modelo de clases (`01-modelo.md`) incorporando la tercera entidad (ej. `Diet`) para cumplir con la combinación de entidades.
- Plan de implementación (`02-plan.md`).

---

## 2026-10-03 — Contingencia de vistas Swing

**Decisiones del Estudiante:**
- El código **se queda** en `Desktop\Proyectos\NutritionistApp`, fuera del vault.
- Sobre el GUI Builder: probablemente el profesor lo permite, pero se prepara
  el replacement por las dudas.

**Hecho:**
- Analizadas las dos vistas generadas y extraídos los campos reales que
  escondían detrás de `jTextField1..8`:
  - `Patient`: id, name, lastName, weight, height, imc, observations.
  - `Aliment`: name, category, caloriesPerServing.
- Escritas y **verificadas** las versiones manuales en `tarea/03-vistas-manuales.md`.
  Compilan con `javac --release 25 -Xlint:all -Werror`, exit code 0.
- Cambios respecto a las generadas: `GridBagLayout` en vez de `GroupLayout`,
  nombres con significado (`txtName`, `btnInsert`), clases en PascalCase,
  `private final` agrupadas, IMC en `setEditable(false)`.

**Bug encontrado al compilar (importante):**
- El getter `getHeight()` **no compila**: colisiona con
  `java.awt.Component.getHeight()`, que devuelve `int` (píxeles).
  Renombrado a `getHeightMeters()`, y `getWeightKg()` por simetría.

**Hallazgos nuevos que hay que verificar contra la rúbrica:**
- Hay botón `Diets` en las dos vistas, pero **no existe** `viewDiet`,
  `modelDiet` ni `controllerDiet`. Si la tarea pide dietas, falta una entidad
  entera.
- **No hay ningún `JTable` en el proyecto**, pero existe `JTable_en_Swing.pdf`.
  Casi seguro la tarea pide listar pacientes y alimentos en tabla.

**Pendiente:**
- Leer `enunciado_rubrica.pdf`. Sigue siendo lo bloqueante.

---

## 2026-10-03 — Descubrimiento del código de la tarea

**Hecho:**
- El Estudiante tiene código de la tarea en
  `C:\Users\games\Desktop\Proyectos\NutritionistApp\NutritionistApp`
  (proyecto NetBeans/Ant, Java 25, git con 1 commit).
- Auditoría completa registrada en `tarea/00-estado.md`.

**Diagnóstico:** es un **esqueleto**. Hay `main` y dos ventanas, pero
`modelPatient`, `modelAliment`, `controllerPatient` y `controllerAliment` están
vacíos. No existe capa de persistencia.

**Riesgos marcados para verificar contra la rúbrica:**
1. Nombres de clase en minúscula (`modelPatient`, `viewPatient`).
2. Las vistas son generadas por el NetBeans GUI Builder (`initComponents()`),
   y los PDFs de Swing enseñan armado manual.
3. `javac.target=25` con JDK 26 instalado.
4. Sin capa JDBC/DAO, pese a que existe `JDBC_DAO_MySQL.pdf`.

**Decisiones:**
- El código **permanece fuera del vault**, en su carpeta de proyecto con su
  propio git. El vault lo documenta y lo referencia por ruta.
- `00-estado.md` es la nota puente entre el wiki y el proyecto de código.

**Pendiente:**
- Leer `enunciado_rubrica.pdf`: es lo bloqueante. Define campos de las
  entidades, si hay relación entre `Patient` y `Aliment`, y si la rúbrica
  acepta el GUI Builder o exige Swing manual.

---

## 2026-10-03 — Estructura inicial del vault

**Hecho:**
- Creado el vault en `G:\Apps\Obsidian\vaults\Programacion Orientada a Objetos`.
- Copiados los **18 PDFs** del profesor desde `C:\Users\games\Desktop\poo`
  a `fuentes/pdf/`.
- Creadas las carpetas `temas/POO`, `temas/practica`, `temas/tarea`, `tarea/`,
  `plantillas/`.
- Creados los archivos núcleo:
  - `ME.md` — contexto del Estudiante y la materia.
  - `MAP.md` — mapa del vault e índice de fuentes.
  - `SKILLS.md` — capacidades del asistente.
  - `index.md` — catálogo de temas.
  - `AGENTS.md` — reglas para assistants.
  - `plantillas/tema.md` — template de notas.
- Configurada la carpeta de plantillas de Obsidian (`Templates`) en
  `plantillas/`.
- Configurado `opencode` con `google/gemini-flash-latest` como modelo por
  defecto, porque el modelo anterior no acepta PDFs adjuntos.

**Decisiones:**
- Orden de procesamiento: primero `enunciado_rubrica.pdf`, después los
  conceptos, después las herramientas.
- Separación estricta entre lo que dice el profesor y la ampliación de la IA.
- `Fuentes/pdf/` es intocable: los originales no se editan ni se renombran.

**Pendiente:**
- Procesar los 18 PDFs en `temas/`.
- Analizar el enunciado y la rúbrica de la tarea.
- Incorporar los videos del instituto (requiere transcripción previa).

---

## Plantilla de entrada

```markdown
## YYYY-MM-DD — Título corto

**Hecho:**
- ...

**Decisiones:**
- ...

**Pendiente:**
- ...
```