---
fuente: síntesis basada en Swing_MVC_Interfaces_Java.pdf (pág. 14, 20 a 23 y 26 a 27), JTable_en_Swing.pdf (pág. 4 a 18), Listas_y_ComboBox_Swing.pdf (pág. 1 a 15), POO_Equals.pdf, JPA_Hibernate_DAO.pdf (pág. 9 a 12), enunciado_rubrica.pdf (pág. 1 y 2) y convenciones propias del proyecto
tags: [tarea, plan, mvc, swing, cableado, aliment]
estado: vigente
actualizado: 2026-11-10
aliases: [plan-cableado-aliment, plan-aliment]
---

# Plan de cableado: `Aliment` (Vista → Controlador → Modelo)

> [!info] Qué es esta nota
> El mismo recorrido que se hizo con `Patient` en [[plan-cableado-patient]], aplicado a `Aliment`: qué exponer en la vista, qué DAO crear, cómo armar el controlador y dónde están las trampas. Escrita para implementarla **sin depender del chat**.
>
> **Actualizada el 2026-11-10**, cuando `Patient` ya quedó cableado y funcionando contra `PatientDAOInMemory`. Todo el código usa los nombres reales del proyecto.

> [!warning] Procedencia del contenido
> - **Del profesor:** las reglas de MVC, el registro de eventos, la configuración de `JTable` y el uso de `JComboBox` (citas en cada sección).
> - **Criterio de equipo (NO del profesor):** `AlimentDAOInMemory`, `AlimentTableModel`, los `JOptionPane` en la vista y el orden de trabajo. Van marcados con `[!tip]` o `[!warning]`.

---

## 0. Estado de partida (verificado el 2026-11-10)

| Pieza | Estado |
|---|---|
| `Aliment.java` (modelo) | ✅ `id`, `name`, `category`, `caloriesPerServing`, constructor vacío y con parámetros, `toString()` → `"name (category)"` |
| `AlimentDAO` (interfaz) | ✅ extiende `DAO<Aliment, Integer>` y agrega `buscarPorCategoria(String)` |
| `AlimentDAOJPA` | ✅ hereda de `DAOJpa`; ⚠️ JPQL roto en la línea 19 (`Alimento`, `categoria`) |
| `AlimentView` | ✅ 4 campos, 4 botones CRUD, tabla y navegación; ❌ **no expone nada al controlador** (sólo `getButtonPatients()` y `getButtonDiets()`) |
| `AlimentController` | ⚠️ existe pero sólo navega: recibe `Aliment model` (sin uso) y `PatientDAO` |
| `AlimentTableModel` | ❌ no existe |
| `AlimentDAOInMemory` | ❌ no existe |
| `AppNutritionist` | Sólo crea el `PatientDAO` y arranca `PatientView` |

**Consecuencia:** no hay nada que "romper" en `Aliment`, pero tampoco nada que funcione. El trabajo es casi calcado al de `Patient` y ya se sabe cuánto cuesta.

---

## 1. Etapa A — Cablear sin base (contra un DAO en memoria)

### Paso A0 — El mapa de nombres

Esto es lo que hay en el `.form` y lo que hay que exponer. La columna "miembro" es el nombre real del campo en `AlimentView`.

| En la vista | Miembro | Getter/setter a crear |
|---|---|---|
| Campo ID | `idTextField` | `getIdText()` / `setIdText(String)` |
| Campo Name | `nameTextField` | `getNameText()` / `setNameText(String)` |
| Campo Category | `categoryTextField` | `getCategoryText()` / `setCategoryText(String)` |
| Campo Calories | `caloriesTextField` | `getCaloriesText()` / `setCaloriesText(String)` |
| Botón Insert | `insertButton` | `getButtonInsert()` |
| Botón Modify | `modifyButton` | `getButtonModify()` |
| Botón Delete | `deleteButton` | `getButtonDelete()` |
| Botón Search | `searchButton` | `getButtonSearch()` |
| Tabla | `alimentsTable` | `getTableAliments()` |
| Botones de navegación | `patientsButton`, `dietsButton` | ✅ ya existen |

> [!tip] `getTableAliments()` y no `getAlimentsTable()`
> En `PatientView` quedó `getTablePatients()`. Mantener el mismo orden evita tener que pensar dos veces al escribir el controlador.

> [!question] Decisión previa: ¿`Category` sigue siendo `JTextField` o pasa a `JComboBox`?
> `Listas_y_ComboBox_Swing.pdf` (pág. 1 a 15) muestra el uso de `JComboBox` para listas cerradas, y una categoría lo es (Frutas, Verduras, Carnes, Lácteos, Cereales).
> - **Si lo cambiás**, se hace **antes** de escribir el controlador: `getCategoryText()` desaparece y pasa a `getCategorySelected()` (`(String) categoryComboBox.getSelectedItem()`).
> - **Si lo dejás** como está, el plan sigue tal cual y se cambia después.
>
> No es un requisito de la rúbrica; es usabilidad. Elegilo una vez y no a mitad del cableado.

### Paso A1 — Getters, setters, mensajes y `clearForm()` en `AlimentView`

Todo va **después** de `// End of variables declaration//GEN-END:variables`, junto a los getters de navegación que ya existen. Son métodos escritos a mano: regenerar `initComponents()` no los borra.

```java
// Getters de los botones del CRUD
public javax.swing.JButton getButtonInsert() {return insertButton;}
public javax.swing.JButton getButtonModify() {return modifyButton;}
public javax.swing.JButton getButtonDelete() {return deleteButton;}
public javax.swing.JButton getButtonSearch() {return searchButton;}

// Getter de la tabla
public javax.swing.JTable getTableAliments() {return alimentsTable;}

// Getters de los campos
public String getIdText()       {return idTextField.getText();}
public String getNameText()     {return nameTextField.getText();}
public String getCategoryText() {return categoryTextField.getText();}
public String getCaloriesText() {return caloriesTextField.getText();}

// Setters para volcar un Aliment en el formulario
public void setIdText(String value)       {idTextField.setText(value);}
public void setNameText(String value)     {nameTextField.setText(value);}
public void setCategoryText(String value) {categoryTextField.setText(value);}
public void setCaloriesText(String value) {caloriesTextField.setText(value);}

// Limpia el formulario
public void clearForm() {
    setIdText("");
    setNameText("");
    setCategoryText("");
    setCaloriesText("");
}

// Mensajes al usuario
public void showError(String message) {javax.swing.JOptionPane.showMessageDialog(
        this, message, "Error", javax.swing.JOptionPane.ERROR_MESSAGE);}
public void showInfo(String message) {javax.swing.JOptionPane.showMessageDialog(
        this, message, "Información", javax.swing.JOptionPane.INFORMATION_MESSAGE);}

public boolean confirmInsert() {return javax.swing.JOptionPane.showConfirmDialog(
        this, "¿Desea dar de alta este alimento?", "Confirmar",
        javax.swing.JOptionPane.OK_CANCEL_OPTION) == javax.swing.JOptionPane.OK_OPTION;}
public boolean confirmModify() {return javax.swing.JOptionPane.showConfirmDialog(
        this, "¿Desea modificar este alimento?", "Confirmar",
        javax.swing.JOptionPane.OK_CANCEL_OPTION) == javax.swing.JOptionPane.OK_OPTION;}
public boolean confirmDelete() {return javax.swing.JOptionPane.showConfirmDialog(
        this, "¿Desea dar de baja este alimento?", "Confirmar",
        javax.swing.JOptionPane.OK_CANCEL_OPTION) == javax.swing.JOptionPane.OK_OPTION;}
```

> [!tip] Por qué los `JOptionPane` van en la vista
> Es criterio de equipo, no de la cátedra: el controlador decide **qué** mensaje mostrar y la vista decide **cómo**. Así el controlador se puede testear sin abrir ventanas.

> [!warning] `"Información"` con tilde
> En `PatientView` quedó `"Informacion"` sin tilde y está anotado como deuda. Acá conviene escribirlo bien desde el principio.

### Paso A2 — `AlimentDAOInMemory`

Archivo nuevo: `src/appNutritionist/dao/AlimentDAOInMemory.java`. Misma forma que [[plan-cableado-patient]] Paso A2, con los 5 métodos de `DAO` + `buscarPorCategoria`.

```java
package appNutritionist.dao;

import appNutritionist.model.Aliment;
import java.util.ArrayList;
import java.util.List;

/**
 * Implementación en memoria de {@link AlimentDAO}, para cablear la vista sin base de datos.
 */
public class AlimentDAOInMemory implements AlimentDAO {

    private final List<Aliment> aliments = new ArrayList<>();
    private int nextId = 1;

    @Override
    public Aliment insert(Aliment a) {
        a.setId(nextId++);
        aliments.add(a);
        return a;
    }

    @Override
    public Aliment findById(Integer id) {
        for (Aliment a : aliments) {
            if (a.getId() == id) {
                return a;
            }
        }
        return null;
    }

    @Override
    public List<Aliment> findAll() {
        return new ArrayList<>(aliments);
    }

    @Override
    public Aliment update(Aliment a) {
        for (int i = 0; i < aliments.size(); i++) {
            if (aliments.get(i).getId() == a.getId()) {
                aliments.set(i, a);
                return a;
            }
        }
        return a;
    }

    @Override
    public void delete(Integer id) {
        aliments.removeIf(a -> a.getId() == id);
    }

    @Override
    public List<Aliment> buscarPorCategoria(String category) {
        List<Aliment> encontrados = new ArrayList<>();
        for (Aliment a : aliments) {
            if (a.getCategory().toLowerCase().contains(category.toLowerCase())) {
                encontrados.add(a);
            }
        }
        return encontrados;
    }
}
```

> [!warning] Las firmas no son obvias
> `insert` y `update` **devuelven `Aliment`**, no `void`. Es lo que dice el `DAO<T, ID>` del equipo (commit `797b39c`) y lo que necesita el controlador para saber el id asignado y para recibir la instancia de `merge` en la Etapa B.

> [!note] `buscarPorCategoria` no se usa en este cableado
> El botón **Search** busca por ID, igual que en `Patient`. `buscarPorCategoria` queda implementado para cuando se agregue el filtro por categoría (candidato a "extra" del 10, ver [[hoja-de-ruta-7-a-9]] Bloque G).

### Paso A3 — `AlimentController`: firma, estado y helpers

El controlador actual (`AlimentController.java`) recibe `Aliment model` y sólo cablea la navegación a Patients. Hay que reemplazarlo.

```java
package appNutritionist.controller;

import appNutritionist.dao.AlimentDAO;
import appNutritionist.dao.PatientDAO;
import appNutritionist.model.Aliment;
import appNutritionist.view.AlimentTableModel;
import appNutritionist.view.AlimentView;
import appNutritionist.view.PatientView;
import excepciones.DAOException;

public class AlimentController {

    private Aliment selectedAliment = new Aliment();   // lo que está cargado en el formulario
    private final AlimentDAO alimentDao;
    private final PatientDAO patientDao;               // sólo para navegar de vuelta
    private final AlimentView view;
    private final AlimentTableModel tableModel = new AlimentTableModel();

    public AlimentController(AlimentDAO alimentDao, PatientDAO patientDao, AlimentView view) {
        this.alimentDao = alimentDao;
        this.patientDao = patientDao;
        this.view = view;

        view.getTableAliments().setModel(tableModel);
        refreshTable();
    }

    // --- helpers (privados, adentro del controlador) ---

    private Aliment readForm() {
        Aliment a = new Aliment();
        a.setName(view.getNameText().trim());
        a.setCategory(view.getCategoryText().trim());
        a.setCaloriesPerServing(Double.parseDouble(view.getCaloriesText().trim()));
        return a;
    }

    private void showAliment(Aliment a) {
        view.setIdText(String.valueOf(a.getId()));
        view.setNameText(a.getName());
        view.setCategoryText(a.getCategory());
        view.setCaloriesText(String.valueOf(a.getCaloriesPerServing()));
    }

    private void refreshTable() {
        try {
            tableModel.setAliments(alimentDao.findAll());
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
    }

    private void resetFields() {
        selectedAliment = new Aliment();
        view.clearForm();
        view.getTableAliments().clearSelection();
    }
}
```

> [!tip] Los helpers van adentro, no en una clase aparte
> `readForm`, `showAliment`, `refreshTable` y `resetFields` usan `view`, `alimentDao` y `selectedAliment`. Sacarlos obliga a pasar esos tres parámetros en cada llamada y no compra nada. Misma decisión que en `Patient`.

> [!warning] El bug del `model` que no se usa
> La firma actual es `(Aliment model, AlimentView view, PatientDAO patient)`. Ese `model` nunca se lee: el estado real es `selectedAliment`. La firma nueva lo reemplaza por el DAO, que es lo que el controlador necesita.

#### De dónde sale cada DAO: la instancia única

Acá aparece el mismo problema que en `Patient`: si cada navegación crea un DAO nuevo, se pierden los datos. La solución es que **`AppNutritionist` cree los dos DAO una sola vez** y los pase por constructor.

```java
// AppNutritionist.java
PatientDAO patientDao = new PatientDAOInMemory();
AlimentDAO alimentDao = new AlimentDAOInMemory();
// Etapa B: new PatientDAOJPA() y new AlimentDAOJPA()
new PatientController(patientDao, alimentDao, vista);
```

Eso obliga a dos cambios chicos en `PatientController` (que hoy recibe un solo DAO):

```java
private final PatientDAO patientDao;
private final AlimentDAO alimentDao;

public PatientController(PatientDAO patientDao, AlimentDAO alimentDao, PatientView view) { ... }

// y en la navegación (hoy línea 44):
new AlimentController(alimentDao, patientDao, vistaAliment);
```

> [!tip] ¿Y cuando llegue `Diet`?
> Van a ser tres DAO en el constructor. Ahí sí conviene un objeto `AppContext` que los agrupe (`context.patientDao`, `context.alimentDao`, `context.dietDao`). Ahora no: sería una clase para ahorrar un parámetro.

### Paso A4 — `AlimentTableModel`

> pág. 11 a 18 — JTable_en_Swing.pdf (ejemplo MVC con `AbstractTableModel`)
> pág. 8 y 9 — JTable_en_Swing.pdf (`setAutoCreateRowSorter`, `SINGLE_SELECTION`)

Archivo nuevo: `src/appNutritionist/view/AlimentTableModel.java`. Cuatro columnas, las mismas del `.form`.

```java
package appNutritionist.view;

import appNutritionist.model.Aliment;
import java.util.ArrayList;
import java.util.List;
import javax.swing.table.AbstractTableModel;

/**
 * Modelo de tabla para mostrar {@link Aliment} en {@link AlimentView}.
 */
public class AlimentTableModel extends AbstractTableModel {

    private final String[] columns = {"ID", "Name", "Category", "Calories"};
    private List<Aliment> aliments = new ArrayList<>();

    public void setAliments(List<Aliment> aliments) {
        this.aliments = aliments;
        fireTableDataChanged();
    }

    public Aliment getAliment(int row) {
        return aliments.get(row);
    }

    @Override public int getRowCount()    {return aliments.size();}
    @Override public int getColumnCount() {return columns.length;}
    @Override public String getColumnName(int column) {return columns[column];}

    // Sin esto, la tabla vuelve a ser editable al cambiar de modelo.
    @Override public boolean isCellEditable(int row, int column) {return false;}

    @Override
    public Object getValueAt(int row, int column) {
        Aliment a = aliments.get(row);
        return switch (column) {
            case 0 -> a.getId();
            case 1 -> a.getName();
            case 2 -> a.getCategory();
            case 3 -> Math.round(a.getCaloriesPerServing() * 100.0) / 100.0;
            default -> null;
        };
    }
}
```

> [!warning] Borrar el modelo de ejemplo del diseñador
> `AlimentView.form:236` dice `<Table columnCount="4" rowCount="4">`. Igual que en `Patient`, hay que dejarlo en **`rowCount="0"`** (Properties → `model` → `…`, o el `.form` con el archivo cerrado). No rompe nada en runtime —`setModel` lo pisa— pero regenera 4 filas de `null` cada vez que se guarda el formulario.

### Paso A5 — Los listeners

> pág. 14 — Swing_MVC_Interfaces_Java.pdf (registrar eventos con `addActionListener` y lambda)

En el constructor, después de `refreshTable()`:

```java
view.getButtonInsert().addActionListener(e -> insert());
view.getButtonModify().addActionListener(e -> modify());
view.getButtonDelete().addActionListener(e -> delete());
view.getButtonSearch().addActionListener(e -> search());

view.getTableAliments().addMouseListener(new java.awt.event.MouseAdapter() {
    @Override
    public void mouseClicked(java.awt.event.MouseEvent e) {
        selectRow();
    }
});
```

Y las cinco acciones:

```java
private void insert() {
    try {
        if (view.getNameText().trim().isEmpty() || view.getCategoryText().trim().isEmpty()) {
            view.showError("El nombre y la categoría son obligatorios.");
            return;
        }
        Aliment a = readForm();
        if (!view.confirmInsert()) return;
        alimentDao.insert(a);
        refreshTable();
        resetFields();
        view.showInfo("Alimento guardado con id " + a.getId() + ".");
    } catch (NumberFormatException ex) {
        view.showError("Las calorías deben ser un número (usá punto decimal).");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}

private void modify() {
    if (selectedAliment.getId() == 0) {
        view.showError("Seleccioná un alimento de la tabla primero.");
        return;
    }
    try {
        if (view.getNameText().trim().isEmpty() || view.getCategoryText().trim().isEmpty()) {
            view.showError("El nombre y la categoría son obligatorios.");
            return;
        }
        Aliment a = readForm();
        if (!view.confirmModify()) return;
        a.setId(selectedAliment.getId());
        selectedAliment = alimentDao.update(a);
        refreshTable();
        view.showInfo("Alimento actualizado.");
    } catch (NumberFormatException ex) {
        view.showError("Las calorías deben ser un número (usá punto decimal).");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}

private void delete() {
    if (selectedAliment.getId() == 0) {
        view.showError("Seleccioná un alimento de la tabla primero.");
        return;
    }
    if (!view.confirmDelete()) return;
    try {
        alimentDao.delete(selectedAliment.getId());
        refreshTable();
        resetFields();
        view.showInfo("Alimento eliminado.");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}

private void search() {
    try {
        int id = Integer.parseInt(view.getIdText().trim());
        Aliment a = alimentDao.findById(id);
        if (a == null) {
            view.showError("No existe un alimento con id " + id);
        } else {
            selectedAliment = a;
            showAliment(a);
        }
    } catch (NumberFormatException ex) {
        view.showError("El ID debe ser un número.");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}

private void selectRow() {
    int viewRow = view.getTableAliments().getSelectedRow();
    if (viewRow < 0) return;
    int modelRow = view.getTableAliments().convertRowIndexToModel(viewRow);
    selectedAliment = tableModel.getAliment(modelRow);
    showAliment(selectedAliment);
}
```

> [!warning] El bug del ordenamiento (`convertRowIndexToModel`)
> Con `autoCreateRowSorter(true)`, al ordenar por columna la fila **visual** deja de coincidir con la del modelo. Sin esa línea se edita el alimento equivocado, **sólo cuando la tabla está ordenada**. Es el mismo bug intermitente que ya se pagó en `Patient`.

### Paso A6 — Limpiar los handlers que dejó NetBeans

`AlimentView` registra sobre sí misma cuatro listeners vacíos:

- `nameTextFieldActionPerformed`
- `idTextFieldActionPerformed`
- `searchButtonActionPerformed`
- `modifyButtonActionPerformed`

**No poner lógica ahí**: los registra la vista, y eso es exactamente lo que MVC separa. Con el controlador agregando su propio `addActionListener`, si esos métodos tuvieran código se ejecutarían **dos** acciones por clic. Dejarlos vacíos o borrarlos desde el diseñador (Events → quitar el handler).

`insertButton` y `deleteButton` no tienen listener propio: perfecto, no hay nada que limpiar.

### Paso A7 — Navegación

- **Patients** (`patientsButton`): ya funciona. Sólo cambia la llamada al constructor nuevo:

```java
view.dispose();
javax.swing.SwingUtilities.invokeLater(() -> {
    PatientView vistaPatient = new PatientView();
    new PatientController(patientDao, alimentDao, vistaPatient);
    vistaPatient.setVisible(true);
});
```

- **Diets** (`dietsButton`): todavía no existe `DietView`. Dejarlo sin listener o deshabilitarlo (`view.getButtonDiets().setEnabled(false)`) para que no parezca roto. Se cablea en el Bloque B.4 de [[hoja-de-ruta-7-a-9]].

---

## 2. Etapa B — Cuando se quiera enchufar MySQL

Casi todo es configuración, y ya está detallada paso a paso en **C.6 de [[hoja-de-ruta-7-a-9]]**. Lo propio de `Aliment`:

- [ ] Agregar la entidad al `persistence.xml`: `<class>appNutritionist.model.Aliment</class>` (además de `Patient`).
- [ ] Arreglar el JPQL de `AlimentDAOJPA.java:19`:

```java
// hoy (roto: la entidad es Aliment y el campo es category)
"SELECT a FROM Alimento a WHERE a.categoria = :cat"
// corregido
"SELECT a FROM Aliment a WHERE LOWER(a.category) LIKE :cat"
```

- [ ] Cambiar una línea en `AppNutritionist`: `AlimentDAO alimentDao = new AlimentDAOJPA();`
- [ ] Probar el mismo circuito de 10 pruebas de abajo.

> [!note] El JPQL roto no frena el CRUD
> `buscarPorCategoria` no lo llama el controlador (Search es por ID). Pero explota el día que se use, y es la misma bomba de [[bug-jpql-dao]].

---

## 3. Trampas, todas juntas

| # | Trampa | Qué pasa si se ignora |
|---|---|---|
| 1 | Crear un `AlimentDAOInMemory` por navegación | al ir a Patients y volver, la tabla aparece vacía |
| 2 | `getSelectedRow()` sin `convertRowIndexToModel` | se edita el alimento equivocado, sólo con la tabla ordenada |
| 3 | Dejar `rowCount="4"` en el `.form` | 4 filas de `null` regeneradas en cada guardado del diseñador |
| 4 | `Double.parseDouble` con coma (`"120,5"`) | `NumberFormatException`; el mensaje tiene que decir "usá punto decimal" |
| 5 | Confirmar antes de validar | el usuario confirma y después le llega el error |
| 6 | No reiniciar `selectedAliment` al limpiar | `Modify` pisa al alimento que estaba seleccionado antes |
| 7 | Cambiar `Category` a `JComboBox` **después** de cablear | hay que tocar `readForm`, `showAliment` y los getters de nuevo |
| 8 | Poner lógica en los handlers de la vista | se ejecuta dos veces (vista + controlador) |

---

## 4. Orden de trabajo

| # | Paso | Dónde | Se puede probar |
|---|---|---|---|
| 1 | Getters, setters, mensajes y `clearForm()` | `AlimentView` | — |
| 2 | `AlimentDAOInMemory` (6 métodos) | `dao` | — |
| 3 | `AlimentTableModel` (4 columnas, read-only) | `view` | — |
| 4 | Reemplazar el controlador: firma, campos, helpers y listeners | `AlimentController` | compila |
| 5 | Pasar los dos DAO desde `AppNutritionist` y ajustar `PatientController` | `AppNutritionist`, `PatientController` | **sí, todo el CRUD** |
| 6 | `rowCount` de la tabla en 0 | diseñador | sí |
| 7 | Etapa B (entidad en el XML + JPQL + una línea) | `META-INF`, `dao` | con MySQL arriba |

Compilación manual (siempre `--release 25`):

```powershell
javac --release 25 -cp "lib/jakarta.persistence-2.2.3.jar;lib/flatlaf-3.5.4.jar" `
      -d build/classes (Get-ChildItem src -Recurse -Filter *.java).FullName
```

---

## 5. Verificación manual

| # | Prueba | Resultado esperado |
|---|---|---|
| 1 | Insert con `Manzana`, `Frutas`, `52` | fila nueva, mensaje "id 1", formulario limpio |
| 2 | Insert con calorías `abc` | "Las calorías deben ser un número", no se guarda |
| 3 | Insert sin nombre | "El nombre y la categoría son obligatorios" |
| 4 | Clic en una fila | los 4 campos se cargan |
| 5 | Ordenar por *Name* y hacer clic | carga el alimento correcto |
| 6 | Modify con datos válidos | fila actualizada |
| 7 | Search con id inexistente | "No existe un alimento con id X" |
| 8 | Delete → Cancelar | no borra |
| 9 | Delete → OK | fila desaparece, formulario limpio |
| 10 | Aliments → Patients → Aliments | los alimentos **siguen ahí** |

---

## 6. Glosario mínimo de este cableado

| Concepto | Qué es acá |
|---|---|
| **Modelo** | `Aliment`, con datos, getters/setters, `toString()` y sin lógica de negocio |
| **Vista** | `AlimentView`, que muestra y captura; no decide |
| **Controlador** | `AlimentController`, que escucha, pide al DAO y actualiza la vista |
| **DAO** | Contrato (`AlimentDAO`) + implementación (`AlimentDAOInMemory` hoy, `AlimentDAOJPA` después) |
| **TableModel** | `AlimentTableModel`, adapta `List<Aliment>` a `JTable` |

---

## Relacionado

- [[plan-cableado-patient]] — el mismo recorrido, ya implementado y probado
- [[hoja-de-ruta-7-a-9]] — Bloque B.3 y la receta C.6 para pasar a MySQL
- [[bug-jpql-dao]] — el JPQL roto de `AlimentDAOJPA:19`
- [[convenciones]] · [[tutorial-dao-desde-cero]] · [[estado]] · [[MAP]]
