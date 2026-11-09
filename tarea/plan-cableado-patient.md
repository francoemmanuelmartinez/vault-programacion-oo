---
fuente: síntesis basada en Swing_MVC_Interfaces_Java.pdf (pág. 14, 20 a 23), JTable_en_Swing.pdf (pág. 4 a 18), JPA_Hibernate_DAO.pdf (pág. 8 y 9), enunciado_rubrica.pdf (pág. 1 y 2) y convenciones propias del proyecto
tags: [tarea, plan, mvc, swing, cableado, patient]
estado: vigente
actualizado: 2026-11-09
aliases: [plan-cableado, plan-patient]
---

# Plan de cableado: `Patient` (Vista ↔ Controlador ↔ Modelo)

> [!info] Qué es esta nota
> El plan completo para conectar `PatientView` con `Patient` a través de `PatientController`, con el código de los métodos, los tips y las trampas. Escrita para poder implementarla **sin depender del chat**.
> **Actualizada el 2026-11-09** después de dos cosas: el renombrado de `PatientView` a camelCase y el commit `797b39c` de RomiM (DAO genérico + entidad `Patient` completa). Todo el código de acá usa **los nombres reales que hay hoy en el proyecto**.

> [!warning] Procedencia del contenido
> - **Del profesor:** las reglas de MVC, el registro de eventos, la configuración de `JTable` y la unidad de persistencia (citas en cada sección).
> - **Criterio de equipo (NO del profesor):** el `PatientDAOInMemory`, el `PatientTableModel`, el `JOptionPane` en la vista y el orden de trabajo. Van marcados con `[!warning]` o `[!tip]`.

---

## 0. Qué cambió desde la versión del 2026-11-07

| Antes | Ahora |
|---|---|
| Componentes con nombres de NetBeans (`jTextField1`, `jButton3`) | camelCase por rol: `nameTextField`, `insertButton`, `patientsTable`… |
| Getters: `getBtnAliment()`, `getBtnDiet()` | `getButtonAliments()`, `getButtonDiets()`, `getIdText()` |
| `Patient` con datos pero sin lógica | `Patient` con `calcularIMC()` y `toString()` (commit de RomiM) |
| No existía `persistence.xml` | Existe, con EclipseLink como proveedor (`src/META-INF/persistence.xml`) |
| No existía DAO genérico | `DAO<T,ID>` + `DAOJpa<T,ID>` + `PatientDAOJPA` |
| `DaoException` no existía | Existe como `excepciones.DAOException` (checked) |

**Consecuencia práctica:** la Etapa B ya casi no tiene código que escribir (lo hizo RomiM), pero **sigue sin poder ejecutarse** porque faltan los jars del proveedor y del driver. Ver sección 3.

---

## 1. El estado real de la persistencia (verificado el 2026-11-09)

```powershell
git status -sb
git fetch --all --prune
git log --oneline --left-right HEAD...origin/master
```

| Pieza | Estado |
|---|---|
| `src/META-INF/persistence.xml` | ✅ Existe (proveedor EclipseLink, base `mysql`, usuario `root`) |
| `DAO<T,ID>`, `DAOJpa<T,ID>`, `PatientDAOJPA` | ✅ Existen y compilan |
| `excepciones.DAOException` | ✅ Existe, es **checked** (`extends Exception`) |
| `util.JPAUtil` | ⚠️ Existe, pero pide la unidad `"nutricionPU"` |
| Proveedor JPA (EclipseLink/Hibernate) en `lib/` | ❌ **No está** |
| Driver MySQL en `lib/` | ❌ **No está** |
| `lib/` actual | `flatlaf-3.5.4.jar`, `jakarta.persistence-2.2.3.jar` (sólo la API) |

`jakarta.persistence-2.2.3.jar` son **sólo las interfaces**. Sin proveedor, `Persistence.createEntityManagerFactory(...)` tira `No Persistence provider for EntityManager named ...` y `PatientDAOJPA` no se puede ejecutar.

> [!danger] Tres bombas en la configuración de RomiM
> 1. **Nombre de la unidad de persistencia:** `JPAUtil` pide `"nutricionPU"` y el XML declara `name="persistence.xml"`. No coinciden → excepción al arrancar, incluso con los jars puestos.
> 2. **Base de datos:** la URL apunta a `jdbc:mysql://localhost:3306/mysql`, que es **el catálogo del sistema** de MySQL. Hay que crear una base propia (p. ej. `nutritionistdb`) y cambiar la URL.
> 3. **`schema-generation.database.action=create`:** crea las tablas **borrando lo que había en cada arranque**. Para desarrollo va `update`.
>
> Y hay una cuarta, escondida: las consultas JPQL de los cuatro DAO están rotas (ver [[bug-jpql-dao]]).

**Por eso el plan sigue siendo: Etapa A completa primero.** No se tira nada: en la Etapa B sólo cambia qué implementación de `PatientDAO` se inyecta.

---

## 2. Etapa A — Cableado Vista ↔ Controlador ↔ Modelo

> pág. 20 a 23 — Swing_MVC_Interfaces_Java.pdf
> Las tres reglas:
> - **Modelo:** lógica y datos. **No conoce Swing** (ni un `import javax.swing`).
> - **Vista:** componentes gráficos. *"Solo muestra y captura datos; expone getters y métodos de actualización."*
> - **Controlador:** *"Conecta la vista con el modelo. Escucha los eventos de la vista, invoca al modelo y actualiza la vista."*

`Patient` ya cumple la regla del modelo (no importa Swing, y ahora además tiene la regla de negocio del IMC). El trabajo está en la vista y en el controlador.

### Paso A0 — Mapa de nombres (lo que hay hoy)

| Componente | Campo en `PatientView` | Getter / setter a exponer |
|---|---|---|
| Campo ID | `idTextField` | `getIdText()` ✅ ya existe · `setIdText()` |
| Campo nombre | `nameTextField` | `getNameText()` · `setNameText()` |
| Campo apellido | `lastNameTextField` | `getLastNameText()` · `setLastNameText()` |
| Campo peso | `weightTextField` | `getWeightText()` · `setWeightText()` |
| Campo altura | `heightTextField` | `getHeightText()` · `setHeightText()` |
| Campo IMC | `imcTextField` | `setImcText()` (el campo es **read-only**) |
| Campo observaciones | `observationsTextField` | `getObservationsText()` · `setObservationsText()` |
| Combo dieta | `dietComboBox` | *(sin cablear todavía)* |
| Botón Insert | `insertButton` | `getButtonInsert()` |
| Botón Modify | `modifyButton` | `getButtonModify()` |
| Botón Delete | `deleteButton` | `getButtonDelete()` |
| Botón Search | `searchButton` | `getButtonSearch()` |
| Tabla | `patientsTable` | `getTablePatients()` |
| Botón Aliments | `alimentsButton` | `getButtonAliments()` ✅ ya existe |
| Botón Diets | `dietsButton` | `getButtonDiets()` ✅ ya existe |
| Paneles | `mainPanel`, `navigationPanel`, `contentPanel`, `patientDataPanel`, `actionsPanel`, `observationsPanel` | **no se exponen** |

> [!tip] La convención que quedó
> `getButton<Acción>()` para botones, `get<Propiedad>Text()` para campos de texto, `getTable<Entidad en plural>()` para la tabla. Los getters de navegación ya están hechos y son el modelo a copiar.

### Paso A1 — Completar el contrato de `PatientView`

Hoy la vista expone `getButtonAliments()`, `getButtonDiets()` y `getIdText()`. Falta el resto del formulario. Va todo dentro de `PatientView`, **fuera** del bloque `//GEN-BEGIN:variables`, al lado de los getters que ya están.

**a) Lectura del formulario** (devuelven `String`, como `getNombre()` del ejemplo del profesor):

```java
public String getNameText()         { return nameTextField.getText(); }
public String getLastNameText()     { return lastNameTextField.getText(); }
public String getWeightText()       { return weightTextField.getText(); }
public String getHeightText()       { return heightTextField.getText(); }
public String getObservationsText() { return observationsTextField.getText(); }
```

**b) Escritura en el formulario** (equivalentes a `mostrarSaludo`):

```java
public void setIdText(String s)           { idTextField.setText(s); }
public void setNameText(String s)         { nameTextField.setText(s); }
public void setLastNameText(String s)     { lastNameTextField.setText(s); }
public void setWeightText(String s)       { weightTextField.setText(s); }
public void setHeightText(String s)       { heightTextField.setText(s); }
public void setImcText(String s)          { imcTextField.setText(s); }
public void setObservationsText(String s) { observationsTextField.setText(s); }

public void clearForm() {
    idTextField.setText("");
    nameTextField.setText("");
    lastNameTextField.setText("");
    weightTextField.setText("");
    heightTextField.setText("");
    imcTextField.setText("");
    observationsTextField.setText("");
}
```

**c) Botones, tabla y mensajes:**

```java
public javax.swing.JButton getButtonInsert() { return insertButton; }
public javax.swing.JButton getButtonModify() { return modifyButton; }
public javax.swing.JButton getButtonDelete() { return deleteButton; }
public javax.swing.JButton getButtonSearch() { return searchButton; }

public javax.swing.JTable getTablePatients() { return patientsTable; }

public void showError(String mensaje) {
    javax.swing.JOptionPane.showMessageDialog(
        this, mensaje, "Error", javax.swing.JOptionPane.ERROR_MESSAGE);
}

public void showInfo(String mensaje) {
    javax.swing.JOptionPane.showMessageDialog(
        this, mensaje, "Información", javax.swing.JOptionPane.INFORMATION_MESSAGE);
}

public boolean confirmDelete() {
    return javax.swing.JOptionPane.showConfirmDialog(
        this, "¿Eliminar este paciente?", "Confirmar",
        javax.swing.JOptionPane.YES_NO_OPTION) == javax.swing.JOptionPane.YES_OPTION;
}
```

> [!tip] Por qué el `JOptionPane` va en la vista
> El profesor hace que la **vista** sea la que muestra (`mostrarSaludo`). Si el `JOptionPane` vive en la vista, el controlador no importa nada de Swing y queda testeable. El controlador pasa el texto; la vista decide cómo mostrarlo.

> [!note] Dos detalles ya resueltos en el diseñador
> - `imcTextField` tiene `editable=false` y el tooltip *"Calculated automatically from Weight and Height"*: el controlador lo llena, el usuario no lo toca.
> - `idTextField` **sí** es editable, así que el flujo "escribo un ID y busco" funciona sin cambios.

### Paso A2 — A quién le habla el controlador (y qué se reutiliza de RomiM)

Hoy el controlador recibe un `Patient` suelto, que alcanza para navegar pero no para un CRUD:

```java
private final Patient model;   // hoy
```

Para un CRUD necesita **el paciente en edición** y **el repositorio**. El repositorio va como **interfaz** `PatientDAO`, no como la implementación JPA:

```java
public class PatientController {

    private Patient actual = new Patient();   // el paciente en edición
    private final PatientDAO dao;             // el contrato, no la implementación
    private final PatientView view;
}
```

**Qué del commit `797b39c` de RomiM sirve para cada etapa:**

| Pieza de RomiM | Etapa A (sin base) | Etapa B (con MySQL) |
|---|---|---|
| `DAO<T, ID>` | ✅ Define las firmas que hay que implementar | ✅ Igual |
| `DAOJpa<T, ID>` | ❌ Necesita proveedor JPA para arrancar | ✅ CRUD completo gratis |
| `PatientDAOJPA` | ❌ Idem | ✅ Ya está escrito (1 línea en `AppNutritionist`, pero hay que arreglarle el JPQL) |
| `util.JPAUtil` | ❌ Idem | ⚠️ Sirve, pero el nombre de la PU no coincide |
| `excepciones.DAOException` | ✅ **El `catch` ya compila hoy** | ✅ Igual |
| `persistence.xml` | ❌ | ⚠️ Existe pero está mal configurado (sección 3) |
| `Patient.calcularIMC()` | ✅ **Usarlo tal cual** | ✅ Igual |

O sea: **RomiM ya escribió la Etapa B**. Lo que falta para poder ver la pantalla funcionando hoy es la Etapa A, que no tiene DAO en memoria y hay que hacerlo.

**El DAO en memoria** (va en `src/appNutritionist/dao/PatientDAOInMemory.java`). Ojo: tiene que implementar **las 6 operaciones** de `PatientDAO` — las 5 de `DAO` **más `buscarPorApellido`**, y con las firmas nuevas (`insert` y `update` devuelven `Patient`):

```java
package appNutritionist.dao;

import appNutritionist.model.Patient;
import java.util.ArrayList;
import java.util.List;

/**
 * Implementación en memoria de {@link PatientDAO}, para cablear la vista
 * sin base de datos (Etapa A).
 */
public class PatientDAOInMemory implements PatientDAO {

    private final List<Patient> pacientes = new ArrayList<>();
    private int proximoId = 1;

    @Override
    public Patient insert(Patient p) {
        p.setId(proximoId++);
        pacientes.add(p);
        return p;
    }

    @Override
    public Patient findById(Integer id) {
        for (Patient p : pacientes) {
            if (p.getId() == id) return p;
        }
        return null;
    }

    @Override
    public List<Patient> findAll() {
        return new ArrayList<>(pacientes);
    }

    @Override
    public Patient update(Patient p) {
        for (int i = 0; i < pacientes.size(); i++) {
            if (pacientes.get(i).getId() == p.getId()) {
                pacientes.set(i, p);
                return p;
            }
        }
        return p;
    }

    @Override
    public void delete(Integer id) {
        pacientes.removeIf(p -> p.getId() == id);
    }

    @Override
    public List<Patient> buscarPorApellido(String apellido) {
        List<Patient> encontrados = new ArrayList<>();
        for (Patient p : pacientes) {
            if (p.getLastName().toLowerCase().contains(apellido.toLowerCase())) {
                encontrados.add(p);
            }
        }
        return encontrados;
    }
}
```

> [!tip] Por qué no hace falta `throws DAOException` acá
> `DAOException` es *checked*, pero una implementación **puede no declararla**: el contrato dice "puede lanzar", no "debe". El DAO en memoria nunca falla, así que no la declara. El controlador igual necesita el `catch` porque la variable es de tipo `PatientDAO`.

En `AppNutritionist` (`AppNutritionist.java:22`):

```java
PatientView vista = new PatientView();
PatientDAO dao = new PatientDAOInMemory();      // Etapa A
// PatientDAO dao = new PatientDAOJPA();        // Etapa B: una sola línea
new PatientController(dao, vista);
vista.setVisible(true);
```

> [!tip] El premio de programar contra la interfaz
> Ese cambio de una línea es todo el beneficio. Y el doble de prueba sirve como justificación de diseño ("poder testear sin base") en la defensa.

### Paso A3 — Helpers del controlador

```java
// 1) Del formulario al objeto
private Patient leerFormulario() {
    Patient p = new Patient();
    p.setName(view.getNameText().trim());
    p.setLastName(view.getLastNameText().trim());
    p.setWeight(Double.parseDouble(view.getWeightText().trim()));
    p.setHeight(Double.parseDouble(view.getHeightText().trim()));
    p.setObservations(view.getObservationsText().trim());
    return p;
}

// 2) Del objeto al formulario
private void mostrarPaciente(Patient p) {
    view.setIdText(String.valueOf(p.getId()));
    view.setNameText(p.getName());
    view.setLastNameText(p.getLastName());
    view.setWeightText(String.valueOf(p.getWeight()));
    view.setHeightText(String.valueOf(p.getHeight()));
    view.setImcText(String.format("%.2f", p.calcularIMC()));
    view.setObservationsText(p.getObservations());
}

// 3) Refrescar la tabla
private void refrescarTabla() {
    try {
        tableModel.setPacientes(dao.findAll());
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}
```

`Double.parseDouble` es **el** punto donde el formulario explota si el usuario escribe "ochenta" o deja el campo vacío. Va con `try/catch (NumberFormatException e)` y `view.showError("El peso debe ser un número")`. Esto alimenta el criterio *Manejo de Excepciones* de la rúbrica (pág. 2), no sólo el DAO.

> [!success] El IMC ya está resuelto en el modelo
> ```java
> // Patient.java — ya existe, lo agregó RomiM
> public double calcularIMC() {
>     return height > 0 ? weight / (height * height) : 0;
> }
> ```
> **No** hay que inventar un `getImc()`: usá `calcularIMC()`, que además coincide con el estilo `calcularCaloriasTotales()` de `DietDetail`. La regla de negocio queda en el modelo, que es lo que pide la cátedra.

> [!success] La unidad de la altura ya está resuelta
> El diseñador ya dice **"Height (m)"** y **"Weight (kg)"**, así que `calcularIMC()` (que divide por `height * height`) recibe metros y funciona tal cual. No hay que tocar nada, sólo **respetar la unidad al cargar datos de prueba**: 1.75, no 175.

### Paso A4 — La tabla

> pág. 11 a 18 — JTable_en_Swing.pdf (ejemplo MVC con `AbstractTableModel`)
> pág. 8 y 9 — JTable_en_Swing.pdf (`setAutoCreateRowSorter`, `SINGLE_SELECTION`)

El `.form` ya tiene las 7 columnas (`ID`, `Name`, `Last Name`, `Weight`, `Height`, `IMC`, `Diet`) y ya están en `editable="false"`, pero eso vive dentro del `DefaultTableModel` anónimo que genera NetBeans. **Al reemplazar el modelo, esa protección se pierde** si no la reimplementás.

**Recomendación: `AbstractTableModel`**, en `appNutritionist.view.PatientTableModel`:

```java
package appNutritionist.view;

import appNutritionist.model.Patient;
import java.util.ArrayList;
import java.util.List;
import javax.swing.table.AbstractTableModel;

public class PatientTableModel extends AbstractTableModel {

    private final String[] columnas =
        {"ID", "Name", "Last Name", "Weight", "Height", "IMC", "Diet"};
    private List<Patient> pacientes = new ArrayList<>();

    public void setPacientes(List<Patient> pacientes) {
        this.pacientes = pacientes;
        fireTableDataChanged();
    }

    public Patient getPaciente(int fila) {
        return pacientes.get(fila);
    }

    @Override public int getRowCount()    { return pacientes.size(); }
    @Override public int getColumnCount() { return columnas.length; }
    @Override public String getColumnName(int columna) { return columnas[columna]; }

    // Sin esto, la tabla vuelve a ser editable al cambiar de modelo.
    @Override public boolean isCellEditable(int fila, int columna) { return false; }

    @Override
    public Object getValueAt(int fila, int columna) {
        Patient p = pacientes.get(fila);
        return switch (columna) {
            case 0 -> p.getId();
            case 1 -> p.getName();
            case 2 -> p.getLastName();
            case 3 -> p.getWeight();
            case 4 -> p.getHeight();
            case 5 -> Math.round(p.calcularIMC() * 100.0) / 100.0;  // Double: ordena bien
            case 6 -> "";              // Diet: todavía no se cablea
            default -> null;
        };
    }
}
```

En el controlador:

```java
private final PatientTableModel tableModel = new PatientTableModel();
// en el constructor:
view.getTablePatients().setModel(tableModel);
```

> [!warning] Borrar el modelo de ejemplo
> Si no se quita el `DefaultTableModel` de prueba del `.form` (las 4 filas de `null`), NetBeans lo vuelve a generar en cada edición del diseño y pisa el modelo del controlador.

### Paso A5 — Los listeners (el corazón del cableado)

> pág. 14 — Swing_MVC_Interfaces_Java.pdf (registrar eventos con `addActionListener` y lambda)

```java
public PatientController(PatientDAO dao, PatientView view) {
    this.dao = dao;
    this.view = view;

    view.getTablePatients().setModel(tableModel);
    refrescarTabla();                                    // arranca con datos

    view.getButtonInsert().addActionListener(e -> insertar());
    view.getButtonModify().addActionListener(e -> modificar());
    view.getButtonDelete().addActionListener(e -> eliminar());
    view.getButtonSearch().addActionListener(e -> buscar());

    view.getTablePatients().addMouseListener(new java.awt.event.MouseAdapter() {
        @Override
        public void mouseClicked(java.awt.event.MouseEvent e) {
            seleccionarFila();
        }
    });

    view.getButtonAliments().addActionListener(e -> irAAliments());   // lo que ya funcionaba
}
```

Cada acción:

```java
private void insertar() {
    try {
        Patient p = leerFormulario();
        dao.insert(p);              // Etapa B: acá aparece "Error al crear Patient"
        refrescarTabla();
        view.clearForm();
        view.showInfo("Paciente guardado con id " + p.getId() + ".");
    } catch (NumberFormatException ex) {
        view.showError("Peso y altura deben ser números (usá punto decimal).");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}

private void modificar() {
    if (actual.getId() == 0) {
        view.showError("Seleccioná un paciente de la tabla primero.");
        return;
    }
    try {
        Patient p = leerFormulario();
        p.setId(actual.getId());     // el id no se edita a mano
        dao.update(p);
        refrescarTabla();
        view.showInfo("Paciente actualizado.");
    } catch (NumberFormatException ex) {
        view.showError("Peso y altura deben ser números (usá punto decimal).");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}

private void eliminar() {
    if (actual.getId() == 0) {
        view.showError("Seleccioná un paciente de la tabla primero.");
        return;
    }
    if (!view.confirmDelete()) return;
    try {
        dao.delete(actual.getId());
        actual = new Patient();
        refrescarTabla();
        view.clearForm();
        view.showInfo("Paciente eliminado.");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}

private void buscar() {
    try {
        int id = Integer.parseInt(view.getIdText().trim());
        Patient p = dao.findById(id);
        if (p == null) {
            view.showError("No existe un paciente con id " + id);   // "no existe" ≠ "falló"
        } else {
            actual = p;
            mostrarPaciente(p);
        }
    } catch (NumberFormatException ex) {
        view.showError("El ID debe ser un número.");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}

private void seleccionarFila() {
    int filaVista = view.getTablePatients().getSelectedRow();
    if (filaVista < 0) return;
    int filaModelo = view.getTablePatients().convertRowIndexToModel(filaVista);
    actual = tableModel.getPaciente(filaModelo);
    mostrarPaciente(actual);
}
```

> [!warning] El bug del ordenamiento (`convertRowIndexToModel`)
> Con `autoCreateRowSorter(true)`, al ordenar por columna la fila **visual** deja de coincidir con la del modelo. Sin `convertRowIndexToModel` se edita el paciente equivocado, **sólo cuando la tabla está ordenada**. Es un bug intermitente y difícil de reproducir.

> [!warning] `DAOException`, no `DaoException`
> La clase que existe en el proyecto es `excepciones.DAOException` (siglas en mayúscula, paquete en español). El borrador viejo de [[ejemplo-de-excepciones]] decía `DaoException`. **Usá el nombre real** o no compila. El renombrado a `DaoException` quedó como deuda de naming a charlar con el equipo.

### Paso A6 — Limpiar los handlers que dejó NetBeans

`PatientView` tiene `weightTextFieldActionPerformed`, `heightTextFieldActionPerformed`, `nameTextFieldActionPerformed`, `lastNameTextFieldActionPerformed`, `dietComboBoxActionPerformed` y `observationsTextFieldActionPerformed`, todos vacíos. **No poner lógica ahí**: esos listeners los registra la vista sobre sí misma, que es justo lo que MVC separa. Dejarlos vacíos o borrarlos desde el diseñador; la lógica va en el controlador.

### Paso A7 — Navegación

`getButtonAliments()` ya funciona: el listener actual de `PatientController` hace `dispose()` y abre `AlimentView` + `AlimentController`. Conviene extraerlo a un método `irAAliments()` para que el constructor quede legible.

`getButtonDiets()` todavía no tiene destino (no existe `DietView`): dejarlo sin listener hasta el Bloque B/C de [[hoja-de-ruta-7-a-9]], o deshabilitarlo con `view.getButtonDiets().setEnabled(false)` para que no parezca roto.

En la pantalla de alimientos pasa lo mismo al revés: `AlimentController` ya cablea `getButtonPatients()`, y `getButtonDiets()` está sin uso.

---

## 3. Etapa B — Cuando se quiera enchufar MySQL

Casi todo el código ya está (RomiM). Lo que falta es **configuración**:

- [ ] **Jars en `lib/`**: proveedor EclipseLink (`eclipselink.jar`, o `org.eclipse.persistence.jar` + `jakarta.persistence.jar`) y driver MySQL (`mysql-connector-j-*.jar`). Con Ant hay que agregarlos a mano en `nbproject/project.properties` (`javac.classpath`), igual que se hizo con FlatLaf.
- [ ] **Alinear el nombre de la unidad de persistencia.** Hoy `JPAUtil.java:12` dice `"nutricionPU"` y `persistence.xml:3` declara `name="persistence.xml"`. Elegir uno (recomendado: `nutritionistPU`) y cambiarlo en los dos lados.
- [ ] **Crear la base.** `CREATE DATABASE nutritionistdb;` y cambiar la URL a `jdbc:mysql://localhost:3306/nutritionistdb`. La actual (`.../mysql`) apunta al catálogo del sistema.
- [ ] **`schema-generation.database.action`**: `create` **borra y recrea las tablas en cada arranque**. Para desarrollo usar `update`; `create` sólo para una demo desde cero.
- [ ] **Arreglar el JPQL** de los cuatro DAO: ver [[bug-jpql-dao]] (archivo y línea exactos).
- [ ] **`DietDetail`**: falta el mapeo `@ManyToOne Diet diet`; sin eso no hay detalle de dieta ni historial.
- [ ] **Seguridad**: la contraseña de MySQL está hardcodeada en `persistence.xml` y va a quedar en el repo. Avisar al equipo.
- [ ] **Una sola línea** en `AppNutritionist`: `new PatientController(new PatientDAOJPA(), vista)`.

> [!question] Decisión pendiente para la Etapa B (sin cambios)
> EclipseLink son ~2 jars y arrastra poco; Hibernate arrastra ~8. La alternativa es **JDBC** (`mysql-connector-j.jar`, un solo jar), que es el otro PDF de la cátedra ([[JDBC_DAO_MySQL]]). La Etapa A es independiente de esa decisión.

---

## 4. Orden de trabajo

| # | Paso | Dónde | Se puede probar |
|---|---|---|---|
| 1 | Getters y setters del formulario + mensajes | `PatientView` | — |
| 2 | `PatientDAOInMemory` (6 métodos) | `dao` | — |
| 3 | `PatientTableModel` (con `isCellEditable` en `false`) | `view` | — |
| 4 | Campos + constructor + helpers + listeners | `PatientController` | compila |
| 5 | Cambiar la firma del constructor | `AppNutritionist` | **sí, todo el CRUD** |
| 6 | Borrar el modelo de ejemplo del `JTable` | diseñador | sí |
| 7 | Etapa B (jars + PU + base + JPQL) | `META-INF`, `lib`, `dao` | con MySQL arriba |

Compilación manual (siempre `--release 25`):

```powershell
javac --release 25 -cp "lib/jakarta.persistence-2.2.3.jar;lib/flatlaf-3.5.4.jar" `
      -d build/classes (Get-ChildItem src -Recurse -Filter *.java).FullName
```

---

## 5. Glosario mínimo de este cableado

| Concepto | Qué es acá |
|---|---|
| **Modelo** | `Patient`, con datos, getters/setters y `calcularIMC()` |
| **Vista** | `PatientView`, que muestra y captura; no decide |
| **Controlador** | `PatientController`, que escucha, pide al DAO y actualiza la vista |
| **DAO** | Contrato (`PatientDAO`) + implementación (`PatientDAOInMemory` hoy, `PatientDAOJPA` después) |
| **TableModel** | `PatientTableModel`, adapta `List<Patient>` a `JTable` |

## 6. Dudas y vacíos

- [ ] ¿La cátedra prefiere `DefaultTableModel` (simple, pág. 4 a 7) o el `AbstractTableModel` (MVC, pág. 11 a 18) para la entrega? El segundo es mejor diseño, pero es más código.
- [x] Unidad de la altura: **metros**, ya rotulada como "Height (m)" y "Weight (kg)" en el diseñador.
- [ ] `dietComboBox` tiene 4 ítems de relleno ("Item 1"…"Item 4"): vaciarlo o dejarlo inerte hasta que exista `Diet` (Bloque A de [[hoja-de-ruta-7-a-9]]).
- [ ] ¿Se deshabilita el botón `Diets` o se deja clickeable sin acción?
- [ ] ¿Se renombra `DAOException` → `DaoException` y el paquete `excepciones` → `appNutritionist.dao`? Es de RomiM; hablarlo antes.

## Relacionado

- [[bug-jpql-dao]] — las 4 consultas JPQL rotas de los DAO (bloquea la Etapa B)
- [[Swing_MVC_Interfaces_Java]] — MVC en Swing (pág. 20 a 23) y eventos (pág. 14)
- [[JTable_en_Swing]] — `AbstractTableModel`, ordenamiento y selección
- [[ejemplo-de-excepciones]] — qué mensaje mostrar en cada `catch`
- [[convenciones]] — por qué getters y no campos públicos
- [[tutorial-dao-desde-cero]] — el patrón DAO completo
- [[hoja-de-ruta-7-a-9]] — bloques de la nota 7-9
- [[estado]] — estado real del código
- [[enunciado_rubrica]] — "CRUD en capas" (pág. 1) y "Manejo de Excepciones" (pág. 2)

---

**Navegación:** [[index]] · [[MAP]]
