---
fuente: auditoría del código real de `NutritionistApp` en `origin/master` (`1f785b6`) + Swing_MVC_Interfaces_Java.pdf (pág. 14, 20 a 23 y 26 a 27), JTable_en_Swing.pdf (pág. 4 a 18), Listas_y_ComboBox_Swing.pdf (pág. 1 a 15), JPA_Hibernate_DAO.pdf (pág. 9 a 16) y enunciado_rubrica.pdf (pág. 1 y 2)
tags: [tarea, plan, mvc, swing, cableado, diet, crud]
estado: propuesto
actualizado: 2026-11-11
aliases: [plan-cableado-diet, plan-diet, plan-crud-diet, plan-dietas]
---

# Plan de cableado: `Diet` (Vista → Controlador → Modelo)

> [!info] Qué es esta nota
> El tercer cableado del proyecto, después de `Patient` y `Aliment`. Es el que falta para cerrar la entrega: **no existe `DietView`** y `DietController` está vacío. Escrito para que se pueda implementar **sin depender del chat**, con el código exacto y las trampas que ya pagaron los otros dos.
>
> **Punto de partida:** `origin/master` = `1f785b6` (RomiM, 2026-11-11 12:16). Antes de empezar hay que hacer el `pull`.

> [!warning] Procedencia del contenido
> - **Del profesor:** la separación Vista → Controlador → Modelo y el registro de eventos (pág. 14, 20 a 23 y 26 a 27 — `Swing_MVC_Interfaces_Java.pdf`), la configuración de `JTable` y `AbstractTableModel` (pág. 4 a 18 — `JTable_en_Swing.pdf`), y que JPQL se escribe sobre entidades (pág. 9 a 16 — `JPA_Hibernate_DAO.pdf`).
> - **Criterio de equipo / IA (NO del profesor):** los nombres de los métodos, los textos de los mensajes, el orden de las fases y la tabla de troubleshooting. Van marcados con `[!tip]` o `[!warning]`.

---

## 0. Diagnóstico: qué hay y qué falta

| Pieza | Estado en `1f785b6` | Comentario |
|---|---|---|
| `model/Diet.java` | ✅ **lista** | `id`, `name`, `description`; constructor vacío y `(String name, String description)`; `toString()` → `name` |
| `dao/DietDAO.java` | ✅ **lista** | `extends DAO<Diet, Integer>` + `buscarPorNombre(String)` |
| `dao/DietDAOJPA.java` | ⚠️ **JPQL roto** | `:19` → `FROM diets` (nombre de tabla, no de entidad) |
| `controller/DietController.java` | ❌ **vacío** | sólo el header de licencia y la clase sin cuerpo |
| `view/DietView.java` + `.form` | ❌ **no existe** | hay que crearla en el diseñador |
| `view/DietTableModel.java` | ❌ **no existe** | |
| Botón **Diets** en las vistas | ❌ **sin listener** | `getButtonDiets()` existe (`PatientView:469`, `AlimentView:364`) pero nadie lo escucha |
| `AppNutritionist` | ✅ **ya cableado** | crea `new DietDAOJPA()` y ya se lo pasa a `PatientController` y `AlimentController` |
| Combo de dietas en `PatientView` | ✅ **funciona** | `PatientController.loadDiets()` usa `dietDao.findAll()` |
| `dao/DietDetailDAOJPA.buscarPorDieta` | ❌ `throw` | no afecta este CRUD (ver sección 12) |

> [!important] La buena noticia
> **El modelo, el DAO y la conexión ya existen.** No hay que tocar `Diet.java`, ni `DietDAO.java`, ni `AppNutritionist`, ni `persistence.xml`. Lo que falta es la vista, el modelo de tabla, el controlador y una línea de JPQL. Todo eso es trabajo de una sesión.

> [!tip] Lo que ya funciona y conviene probar antes de empezar
> Abrí la app, andá a **Patients** y mirá el combo de dietas: tiene que mostrar `-- Sin dieta --`, `Hipocalórica` y `Vegetariana`. Si eso anda, `DietDAO.findAll()` y la conexión están bien, y lo que sigue es sólo interfaz.

---

## 1. Alcance: qué entra y qué no

### Entra

- Una pantalla nueva `DietView` con tabla, formulario y los cuatro botones del CRUD.
- `DietController` completo: insert, modify, delete, search por nombre y selección de fila.
- `DietTableModel` de tres columnas.
- **Los listeners del botón Diets** en `PatientView` y `AlimentView`, para que se pueda llegar a la pantalla nueva desde las otras dos.
- El JPQL de `DietDAOJPA`.
- El manejo del error de clave foránea al borrar una dieta en uso.

### No entra (y conviene decirlo en voz alta)

- **Los alimentos de cada dieta** (`DietDetail`). Eso es una segunda pantalla: la tabla pivot existe, pero no hay forma de decir *"200 g de arroz"* porque `DietDetail` sólo tiene `(Diet, Aliment)` y una restricción de unicidad que impide repetir el alimento. Lo dejo como fase opcional al final (sección 12).
- **Excepciones propias para las claves foráneas** (`DietConPacientesException` o similar). Se puede hacer después; acá alcanza con un mensaje claro. RomiM quedó en ver ese tema.

> [!warning] Hablar con RomiM antes de arrancar
> En el reparto original del equipo, **la vista de dietas era de RomiM** — lo dijo ella por WhatsApp: *"Quedaría por hacer la vista de las dietas"*. Si se reasigna, avisarle **antes** de empezar, no después de pushear. Dos personas creando `DietView.java` al mismo tiempo es el peor merge posible: el archivo no existe en `master`, así que git lo va a tratar como "agregado por los dos" y no hay forma automática de resolverlo.

---

## 2. Fase 1 — Crear `DietView` en el diseñador

### 2.1 Los componentes

En NetBeans: **File → New File → Swing GUI Forms → JFrame Form**, nombre `DietView`, paquete `appNutritionist.view`.

> pág. 20 a 23 — Swing_MVC_Interfaces_Java.pdf
> La vista arma la interfaz y **expone** sus componentes con getters; no decide nada.

| Componente | Variable (Change Variable Name) | Propiedades |
|---|---|---|
| `JTextField` | `idTextField` | `editable` **desmarcado**, `toolTipText` → `Se asigna automaticamente` |
| `JTextField` | `nameTextField` | `toolTipText` → `Nombre de la dieta` |
| `JTextField` | `descriptionTextField` | `toolTipText` → `Descripcion breve (opcional)` |
| `JButton` | `insertButton` | texto `Insert` |
| `JButton` | `modifyButton` | texto `Modify` |
| `JButton` | `deleteButton` | texto `Delete` |
| `JButton` | `searchButton` | texto `Search`, tooltip → `Input a name and click this button to search` |
| `JButton` | `patientsButton` | texto `Patients` |
| `JButton` | `alimentsButton` | texto `Aliments` |
| `JTable` scrollable | `dietsTable` | ver 2.2 |
| `JLabel` ×3 | `idLabel`, `nameLabel`, `descriptionLabel` | textos `ID`, `Name`, `Description` |

**No hace falta un `JComboBox` acá.** El combo de dietas vive en `PatientView` y ya está cableado; `DietView` no elige dietas, las administra.

### 2.2 La tabla

En las propiedades de `dietsTable`:

| Propiedad | Valor | Por qué |
|---|---|---|
| `model` | `columnCount = 3`, `rowCount = **0**` | si queda en 4, el diseñador regenera filas de `null` en cada guardado (es la trampa 3 de [[plan-cableado-aliment]]) |
| `autoCreateRowSorter` | `true` | ordenar por columna al hacer clic en el encabezado |
| `selectionMode` | `SINGLE_SELECTION` | el formulario edita una sola dieta |
| `fillsViewportHeight` | `true` | cosmético, para que no quede una franja gris |

> pág. 8 y 9 — JTable_en_Swing.pdf
> `setAutoCreateRowSorter` y `SINGLE_SELECTION`. **Ojo:** con el sorter activo, la fila que ves no es la fila del modelo — de eso se ocupa `convertRowIndexToModel` en el paso 4.

### 2.3 Getters, setters, mensajes y `clearForm()`

Todo va **después** de `// End of variables declaration//GEN-END:variables`. Son métodos escritos a mano: regenerar `initComponents()` no los borra.

```java
// --- Botones del CRUD ---
public javax.swing.JButton getButtonInsert() {return insertButton;}
public javax.swing.JButton getButtonModify() {return modifyButton;}
public javax.swing.JButton getButtonDelete() {return deleteButton;}
public javax.swing.JButton getButtonSearch() {return searchButton;}

// --- Navegación ---
public javax.swing.JButton getButtonPatients() {return patientsButton;}
public javax.swing.JButton getButtonAliments() {return alimentsButton;}

// --- Tabla ---
public javax.swing.JTable getTableDiets() {return dietsTable;}

// --- Campos ---
public String getIdText()          {return idTextField.getText();}
public String getNameText()        {return nameTextField.getText();}
public String getDescriptionText() {return descriptionTextField.getText();}

public void setIdText(String value)          {idTextField.setText(value);}
public void setNameText(String value)        {nameTextField.setText(value);}
public void setDescriptionText(String value) {descriptionTextField.setText(value);}

// --- Limpiar el formulario ---
public void clearForm() {
    setIdText("");
    setNameText("");
    setDescriptionText("");
}

// --- Mensajes ---
public void showError(String message) {javax.swing.JOptionPane.showMessageDialog(
        this, message, "Error", javax.swing.JOptionPane.ERROR_MESSAGE);}
public void showInfo(String message) {javax.swing.JOptionPane.showMessageDialog(
        this, message, "Información", javax.swing.JOptionPane.INFORMATION_MESSAGE);}

public boolean confirmInsert() {return javax.swing.JOptionPane.showConfirmDialog(
        this, "¿Desea dar de alta esta dieta?", "Confirmar",
        javax.swing.JOptionPane.OK_CANCEL_OPTION) == javax.swing.JOptionPane.OK_OPTION;}
public boolean confirmModify() {return javax.swing.JOptionPane.showConfirmDialog(
        this, "¿Desea modificar esta dieta?", "Confirmar",
        javax.swing.JOptionPane.OK_CANCEL_OPTION) == javax.swing.JOptionPane.OK_OPTION;}
public boolean confirmDelete() {return javax.swing.JOptionPane.showConfirmDialog(
        this, "¿Desea dar de baja esta dieta?", "Confirmar",
        javax.swing.JOptionPane.OK_CANCEL_OPTION) == javax.swing.JOptionPane.OK_OPTION;}
```

> [!tip] Por qué los `JOptionPane` van en la vista
> Es criterio de equipo, no de la cátedra: el controlador decide **qué** mensaje mostrar y la vista decide **cómo**. Así el controlador se puede probar sin abrir ventanas.

> [!warning] `"Información"` lleva tilde
> En `AlimentView` quedó `"Informacion"` sin tilde y está anotado como deuda. En esta vista escribilo bien desde el principio.

### 2.4 Dos límites que impone la base

`Sql/nutricion.sql:9-10` declara:

```sql
name        VARCHAR(50) NOT NULL,
description VARCHAR(100)
```

O sea: el nombre **no puede ser vacío** (es `NOT NULL`) y **ninguno de los dos puede pasar** de 50 / 100 caracteres. Si no se valida en el controlador, MySQL responde con *"Data too long for column 'description'"* y el usuario ve un mensaje técnico en inglés. Se valida en el paso 4.3.

---

## 3. Fase 2 — `DietTableModel`

Archivo nuevo: `src/appNutritionist/view/DietTableModel.java`.

> pág. 11 a 18 — JTable_en_Swing.pdf (ejemplo con `AbstractTableModel`)
> La tabla no guarda datos: los pide al modelo, que sabe cuántas filas y columnas hay y qué devolver en cada celda.

```java
package appNutritionist.view;

import appNutritionist.model.Diet;
import java.util.ArrayList;
import java.util.List;
import javax.swing.table.AbstractTableModel;

/**
 * Modelo de tabla para mostrar {@link Diet} en {@link DietView}.
 */
public class DietTableModel extends AbstractTableModel {

    private final String[] columns = {"ID", "Name", "Description"};
    private List<Diet> diets = new ArrayList<>();

    public void setDiets(List<Diet> diets) {
        this.diets = diets;
        fireTableDataChanged();
    }

    public Diet getDiet(int row) {
        return diets.get(row);
    }

    @Override public int getRowCount()    {return diets.size();}
    @Override public int getColumnCount() {return columns.length;}
    @Override public String getColumnName(int column) {return columns[column];}

    // Sin esto, la tabla vuelve a ser editable al cambiar de modelo.
    @Override public boolean isCellEditable(int row, int column) {return false;}

    @Override
    public Object getValueAt(int row, int column) {
        Diet d = diets.get(row);
        return switch (column) {
            case 0 -> d.getId();
            case 1 -> d.getName();
            case 2 -> d.getDescription() == null ? "" : d.getDescription();
            default -> null;
        };
    }
}
```

> [!important] El `null` de `description`
> La columna es `VARCHAR(100)` **sin** `NOT NULL`. Una dieta cargada por SQL puede tener `description = NULL`, y una celda con `null` sale vacía (está bien) pero `showDiet` sí se rompe si llamás `.setText(null)` sin cuidado. Por eso el `== null ? "" :` está en los dos lados.

---

## 4. Fase 3 — `DietController`

Se **reemplaza entero** el archivo actual, que sólo tiene el header de licencia.

> pág. 14 — Swing_MVC_Interfaces_Java.pdf
> Registrar los eventos con `addActionListener` y una lambda que delegue en un método privado del controlador.

### 4.1 Firma, estado y listeners

```java
package appNutritionist.controller;

import appNutritionist.dao.AlimentDAO;
import appNutritionist.dao.DietDAO;
import appNutritionist.dao.PatientDAO;
import appNutritionist.model.Diet;
import appNutritionist.view.AlimentView;
import appNutritionist.view.DietTableModel;
import appNutritionist.view.DietView;
import appNutritionist.view.PatientView;
import excepciones.DAOException;
import java.util.List;

public class DietController {

    private Diet selectedDiet = new Diet();   // la dieta cargada en el formulario
    private final DietDAO dietDao;
    private final PatientDAO patientDao;      // sólo para navegar
    private final AlimentDAO alimentDao;      // sólo para navegar
    private final DietView view;
    private final DietTableModel tableModel = new DietTableModel();

    public DietController(DietDAO dietDao, PatientDAO patientDao,
                          AlimentDAO alimentDao, DietView view) {
        this.dietDao = dietDao;
        this.patientDao = patientDao;
        this.alimentDao = alimentDao;
        this.view = view;

        view.getTableDiets().setModel(tableModel);
        refreshTable();

        view.getButtonInsert().addActionListener(e -> insert());
        view.getButtonModify().addActionListener(e -> modify());
        view.getButtonDelete().addActionListener(e -> delete());
        view.getButtonSearch().addActionListener(e -> search());

        // Al seleccionar una fila se cargan sus datos en el formulario
        view.getTableDiets().getSelectionModel().addListSelectionListener(e -> {
            if (!e.getValueIsAdjusting()) {
                selectRow();
            }
        });

        // Navegación: vuelve a Patients. Se recrea la vista y el controlador.
        view.getButtonPatients().addActionListener(e -> {
            view.dispose();
            javax.swing.SwingUtilities.invokeLater(() -> {
                PatientView vistaPatient = new PatientView();
                new PatientController(patientDao, dietDao, alimentDao, vistaPatient);
                vistaPatient.setVisible(true);
            });
        });

        // Navegación: va a Aliments.
        view.getButtonAliments().addActionListener(e -> {
            view.dispose();
            javax.swing.SwingUtilities.invokeLater(() -> {
                AlimentView vistaAliment = new AlimentView();
                new AlimentController(alimentDao, patientDao, dietDao, vistaAliment);
                vistaAliment.setVisible(true);
            });
        });
    }
```

> [!tip] La firma calca la de los otros dos controladores
> `(DAO principal, los otros dos DAO, vista)`, en el mismo orden que `PatientController(patientDao, dietDao, alimentDao, view)` y `AlimentController(alimentDao, patientDao, dietDao, view)`. Mantener el orden evita tener que pensar dos veces al navegar.

> [!note] `patientDao` y `alimentDao` no se usan para nada más
> Están sólo para poder construir la vista de destino cuando se navega. Es la misma decisión que ya tomaron los otros controladores: los DAO se crean **una sola vez** en `AppNutritionist` y viajan por constructor.

### 4.2 Helpers

```java
    // --- helpers (privados, adentro del controlador) ---

    private Diet readForm() {
        Diet d = new Diet();
        d.setName(view.getNameText().trim());
        d.setDescription(view.getDescriptionText().trim());
        return d;
    }

    private void showDiet(Diet d) {
        view.setIdText(String.valueOf(d.getId()));
        view.setNameText(d.getName());
        view.setDescriptionText(d.getDescription() == null ? "" : d.getDescription());
    }

    private void refreshTable() {
        try {
            tableModel.setDiets(dietDao.findAll());
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
    }

    private void resetFields() {
        selectedDiet = new Diet();
        view.clearForm();
        view.getTableDiets().clearSelection();
    }

    // Valida lo que la base no perdona: nombre vacio y largos maximos.
    private boolean validar(Diet d) {
        if (d.getName().isEmpty()) {
            view.showError("El nombre de la dieta es obligatorio.");
            return false;
        }
        if (d.getName().length() > 50) {
            view.showError("El nombre no puede superar los 50 caracteres.");
            return false;
        }
        if (d.getDescription().length() > 100) {
            view.showError("La descripcion no puede superar los 100 caracteres.");
            return false;
        }
        return true;
    }
```

### 4.3 Las cuatro acciones

```java
    private void insert() {
        try {
            Diet d = readForm();
            if (!validar(d)) return;
            if (!view.confirmInsert()) return;
            dietDao.insert(d);
            refreshTable();
            resetFields();
            view.showInfo("Dieta guardada con id " + d.getId() + ".");
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
    }

    private void modify() {
        if (selectedDiet.getId() == 0) {
            view.showError("Selecciona una dieta de la tabla primero.");
            return;
        }
        try {
            Diet d = readForm();
            if (!validar(d)) return;
            if (!view.confirmModify()) return;
            d.setId(selectedDiet.getId());          // el id no se edita a mano
            selectedDiet = dietDao.update(d);
            refreshTable();
            view.showInfo("Dieta actualizada.");
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
    }

    private void delete() {
        if (selectedDiet.getId() == 0) {
            view.showError("Selecciona una dieta de la tabla primero.");
            return;
        }
        if (!view.confirmDelete()) return;
        try {
            dietDao.delete(selectedDiet.getId());
            refreshTable();
            resetFields();
            view.showInfo("Dieta eliminada.");
        } catch (DAOException ex) {
            // Lo mas probable: la dieta tiene pacientes asignados o alimentos cargados.
            view.showError("No se pudo eliminar la dieta. Puede tener pacientes asignados "
                    + "o alimentos cargados.\n" + ex.getMessage());
        }
    }

    private void search() {
        String nombre = view.getNameText().trim();

        // Campo vacio = quitar el filtro y mostrar todas otra vez
        if (nombre.isEmpty()) {
            refreshTable();
            resetFields();
            return;
        }
        try {
            List<Diet> encontradas = dietDao.buscarPorNombre(nombre);
            if (encontradas.isEmpty()) {
                view.showError("No hay dietas que contengan \"" + nombre + "\".");
                return;
            }
            tableModel.setDiets(encontradas);
            selectedDiet = new Diet();
            view.getTableDiets().clearSelection();
            if (encontradas.size() == 1) {          // una sola: la deja lista en el formulario
                selectedDiet = encontradas.get(0);
                showDiet(selectedDiet);
                view.getTableDiets().setRowSelectionInterval(0, 0);
            }
            view.showInfo(encontradas.size() + " dieta(s) encontrada(s).");
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
    }

    //convertRowIndexToModel para que la dieta seleccionada sea correcta aunque se ordene la tabla
    private void selectRow() {
        int viewRow = view.getTableDiets().getSelectedRow();
        if (viewRow < 0) return;
        int modelRow = view.getTableDiets().convertRowIndexToModel(viewRow);
        selectedDiet = tableModel.getDiet(modelRow);
        showDiet(selectedDiet);
    }
}
```

> [!warning] El bug del ordenamiento
> Sin `convertRowIndexToModel`, al ordenar la tabla por *Name* y hacer clic, se carga **otra** dieta en el formulario. Es intermitente —funciona mientras la tabla esté en orden natural— y ya se pagó una vez en `Patient` y otra en `Aliment`.

> [!warning] `search()` depende de que la Fase 5 esté hecha
> Si todavía no cambiaste el JPQL de `DietDAOJPA`, este método explota con *"The abstract schema type 'diets' is unknown"*. Hacé la Fase 5 **antes** de probar el botón Search, o comentá el listener hasta entonces.

---

## 5. Fase 4 — Conectar la navegación

### 5.1 `AppNutritionist` — no se toca

Ya está:

```java
PatientDAO patientDao = new PatientDAOJPA();
DietDAO dietDao = new DietDAOJPA();
AlimentDAO alimentDao = new AlimentDAOJPA();
```

Y ya se los pasa a los dos controladores existentes. **No hay que cambiar nada acá.** La primera pantalla sigue siendo `PatientView`.

### 5.2 El botón Diets en `PatientController` y `AlimentController`

Hoy `getButtonDiets()` existe en las dos vistas y **nadie lo escucha**. Hay que agregar un bloque en el constructor de cada controlador, al lado del bloque de navegación que ya está.

En `PatientController`, después del `view.getButtonAliments().addActionListener(...)`:

```java
view.getButtonDiets().addActionListener(e -> {
    view.dispose();
    javax.swing.SwingUtilities.invokeLater(() -> {
        DietView vistaDiet = new DietView();
        new DietController(dietDao, patientDao, alimentDao, vistaDiet);
        vistaDiet.setVisible(true);
    });
});
```

En `AlimentController`, después del `view.getButtonPatients().addActionListener(...)`, el mismo bloque (los nombres de los campos son iguales en los dos controladores).

Y agregar el import en los dos:

```java
import appNutritionist.view.DietView;
```

`DietController` está en el mismo paquete (`appNutritionist.controller`), así que **no** necesita import.

> [!danger] Acá está el único conflicto real del proyecto
> `PatientController` y `AlimentController` son **los mismos dos archivos** donde el otro integrante del equipo va a cambiar `search()` para las búsquedas por apellido y por categoría (Fase 5 de [[plan-crud-bd]]).
>
> Las dos ediciones son en zonas distintas del archivo (el constructor contra el método `search()`), así que git debería resolverlo solo. Pero si las dos personas tocan el mismo bloque en la misma rama, hay conflicto.
>
> **Recomendación:** que los listeners del botón Diets vayan en un **commit propio y chico**, pusheado apenas compile, antes de que el otro empiece con las búsquedas. Así el que llega segundo hace un `pull` y el conflicto no existe.

### 5.3 Los botones de `DietView`

Los dos bloques están en el constructor del controlador (sección 4.1). No hay nada más que cablear: `DietView` es la única vista que navega hacia las otras dos, y las otras dos ya saben volver a `Patients`.

---

## 6. Fase 5 — El JPQL de `DietDAOJPA`

`src/appNutritionist/dao/DietDAOJPA.java:19`. **Una sola palabra.**

```java
// antes  (roto: "diets" es el nombre de la tabla, no de la entidad)
"SELECT d FROM diets d WHERE LOWER(d.name) LIKE :nom"

// después
"SELECT d FROM Diet d WHERE LOWER(d.name) LIKE :nom"
```

Con el resto intacto: `consultar("...", "nom", "%" + nombre.toLowerCase() + "%")`.

> pág. 13 a 16 — JPA_Hibernate_DAO.pdf
> «JPQL trabaja sobre entidades y atributos en lugar de tablas y columnas.»

Después de `FROM` va **la clase**, no el `@Table`.

> [!tip] Esto es más fácil de lo que parece
> El `d.name` de esta consulta **ya está bien**: `1f785b6` renombró `Diet.nombre` a `Diet.name`. Lo único que falta es la entidad. Es la única de las cinco consultas rotas que quedó a mitad de camino, y es la más barata de las que están en [[bug-jpql-dao]].

> [!warning] Coordinar quién arregla este JPQL
> [[plan-crud-bd]] asigna los cinco JPQL a la otra persona del equipo. Si los dos tocan `DietDAOJPA.java` al mismo tiempo, es un conflicto de una línea que igual hay que resolver a mano. Acordar antes: **o lo hace quien hace la vista de dietas, o lo hace el otro y esta fase se saltea**.

Verificación:

```powershell
Select-String -Path "src\appNutritionist\dao\DietDAOJPA.java" -Pattern 'FROM diets\b'
```

No tiene que devolver nada.

---

## 7. Fase 6 — Probar todo contra la base

> [!important] La prueba que vale
> **Cerrá y volvé a abrir la app después de cada alta, modificación o baja.** Si el dato sigue ahí, está en MySQL. Si desaparece, el DAO que se está usando es el equivocado.

| # | Prueba | Resultado esperado |
|---|---|---|
| 1 | Abrir la app → botón **Diets** | abre `DietView` con las 2 dietas del script: `Hipocalórica` y `Vegetariana` |
| 2 | La columna Description muestra los textos del script | textos completos, sin cortar |
| 3 | Insert con nombre y descripción | mensaje con el id asignado, fila nueva, formulario limpio |
| 4 | **Cerrar y reabrir la app** | la dieta nueva **sigue** ahí |
| 5 | Insert sin nombre | "El nombre de la dieta es obligatorio", no guarda |
| 6 | Insert con nombre de 51 caracteres | "El nombre no puede superar los 50 caracteres", no guarda |
| 7 | Insert con descripción de 101 caracteres | "La descripcion no puede superar los 100 caracteres", no guarda |
| 8 | Insert sin descripción (vacía) | guarda igual, la columna Description queda vacía |
| 9 | Clic en una fila | se cargan los 3 campos |
| 10 | Ordenar por *Name* y hacer clic | carga la dieta correcta (prueba `convertRowIndexToModel`) |
| 11 | Modify del nombre a `Hipocalorica estricta` | mensaje de actualizado; al reabrir, el nombre nuevo |
| 12 | Modify de la descripción | persiste |
| 13 | Delete → Cancelar | no borra |
| 14 | Delete de `Vegetariana` (id 2, **tiene pacientes y alimentos**) | mensaje "No se pudo eliminar la dieta…", **sin crash**, la fila sigue |
| 15 | Insert de una dieta nueva sin asignar → Delete | borra sin problema |
| 16 | Diets → Patients | el combo de dietas muestra la dieta nueva y ya no la borrada |
| 17 | Asignarle la dieta nueva a un paciente con **Modify** | persiste al reabrir |
| 18 | Patients → Diets → Patients → Diets | los datos siguen (los DAO se comparten, no se recrean) |
| 19 | Search con `hipo` | queda sólo `Hipocalórica` |
| 20 | Search con `zzz` (inexistente) | "No hay dietas que contengan…" |
| 21 | Search con el campo vacío | vuelve a mostrar todas |

> [!note] La prueba 14 es la importante
> Es la única que ejercita la clave foránea. `Hipocalórica` (id 1) y `Vegetariana` (id 2) están referenciadas desde `Patients.id_diet` y desde `DietsDetails.id_diet`, así que MySQL **rechaza** el `DELETE`. Lo que tiene que pasar es que la app muestre un mensaje y siga viva, no que se cierre con un stack trace.

---

## 8. Trampas, todas juntas

| # | Trampa | Qué pasa si se ignora |
|---|---|---|
| 1 | Dejar `rowCount="4"` en el `.form` | el diseñador regenera 4 filas de `null` en cada guardado |
| 2 | `getSelectedRow()` sin `convertRowIndexToModel` | se edita la dieta equivocada, **sólo** con la tabla ordenada |
| 3 | `description` es `NULL` en la base | `showDiet` revienta si no se protege con `== null ? "" :` |
| 4 | No validar el largo | `Data too long for column 'name'` (o `'description'`), un mensaje de MySQL en inglés |
| 5 | Confirmar antes de validar | el usuario confirma y **después** le llega el error |
| 6 | No reiniciar `selectedDiet` al limpiar | `Modify` pisa la dieta que estaba seleccionada antes |
| 7 | Usar `new Diet("Vegetariana")` | no compila: el constructor pide **dos** argumentos, `(name, description)` |
| 8 | Escribir `diet.getNombre()` / `setNombre()` | no existen. `Diet` usa `getName()` / `setName()` desde `1f785b6` |
| 9 | Copiar `{@link Dieta}` o `{@link DietaDAO}` en el Javadoc | NetBeans lo pinta en rojo; no rompe la compilación pero sí `Generate Javadoc` |
| 10 | Poner lógica en los `...ActionPerformed` de la vista | se ejecuta dos veces (vista + controlador) |
| 11 | Crear `DietView` sin avisarle a RomiM | dos ramas con el mismo archivo nuevo: conflicto sin resolución automática |

> [!tip] La trampa 11 es la única que puede costar horas
> Las otras diez se arreglan leyendo el error. Que dos personas creen el mismo `DietView.java` desde cero, no.

---

## 9. Si algo falla

| Mensaje | Causa | Solución |
|---|---|---|
| `The abstract schema type 'diets' is unknown` | JPQL sin arreglar | Fase 5 |
| `Cannot delete or update a parent row: a foreign key constraint fails` | la dieta tiene pacientes o alimentos | es **esperado**: el controlador lo informa (prueba 14) |
| `Data too long for column 'name'` | falta la validación de 50 | Fase 4.2, `validar()` |
| `Field 'name' doesn't have a default value` | se intentó insertar con el nombre vacío | ídem |
| `Unknown column 't1.nombre' in 'field list'` | la base se creó **antes** de `1f785b6` | recrear con el script actual (ver [[bug-jpql-dao]]) |
| `No Persistence provider for EntityManager named nutricionPU` | faltan EclipseLink y el driver en `lib/` | Fase 1 de [[plan-crud-bd]] |
| `Access denied for user 'root'@'localhost'` | contraseña distinta de `2711` | no commitear la propia |
| La tabla se llena pero al reabrir no está el dato | se está usando un DAO en memoria | `AppNutritionist` tiene que decir `new DietDAOJPA()` |
| Stack trace al borrar con la base caída | `DAOJpa.delete()` crea el `EntityManager` fuera del `try` | ver la nota al pie de la sección 12 |

---

## 10. Commit y push

### 10.1 Qué entra

| Archivo | Por qué |
|---|---|
| `view/DietView.java` + `view/DietView.form` | la pantalla nueva |
| `view/DietTableModel.java` | el modelo de tabla |
| `controller/DietController.java` | el CRUD |
| `controller/PatientController.java`, `controller/AlimentController.java` | los listeners del botón Diets |
| `dao/DietDAOJPA.java` | el JPQL (si lo hace quien hace la vista) |

### 10.2 Dos commits, no uno

```powershell
# 1) la pantalla y el CRUD
git add src/appNutritionist/view/DietView.java src/appNutritionist/view/DietView.form `
        src/appNutritionist/view/DietTableModel.java src/appNutritionist/controller/DietController.java
git commit -m "Agregue la vista de dietas con su tabla, su controlador y el CRUD completo"

# 2) la navegacion y el JPQL
git add src/appNutritionist/controller/PatientController.java `
        src/appNutritionist/controller/AlimentController.java `
        src/appNutritionist/dao/DietDAOJPA.java
git commit -m "Cablee el boton Diets de las vistas y arregle el JPQL de DietDAOJPA"
```

> [!tip] Por qué separados
> Si el segundo commit choca con el de las búsquedas, se resuelve **ese** y el primero ya está a salvo. Un solo commit grande obliga a rehacer todo.

### 10.3 Antes de commitear

```powershell
git status -s
```

No tienen que aparecer `dist/`, `build/`, `nbproject/private/` ni una `persistence.xml` con una contraseña distinta de la del equipo.

### 10.4 Lo que **no** hacer

- No `amend` ni reescribir commits de RomiM.
- No renombrar columnas de `Sql/nutricion.sql`: el esquema ya está alineado ([[bug-jpql-dao]]).
- No tocar `Diet.java` ni `DietDAO.java`: están bien.
- No commitear la contraseña propia de MySQL.

---

## 11. Lo que dice el profesor

> pág. 14 — Swing_MVC_Interfaces_Java.pdf
> Los eventos se registran con `addActionListener`, y desde Java 8 se puede usar una **expresión lambda** que llame a un método del controlador.

> pág. 20 a 23 — Swing_MVC_Interfaces_Java.pdf
> La **vista** arma la interfaz y expone sus componentes; el **controlador** escucha, decide y actualiza; el **modelo** guarda los datos. La vista no habla con el DAO.

> pág. 11 a 18 — JTable_en_Swing.pdf
> El `AbstractTableModel` es el que le dice a la `JTable` cuántas filas hay y qué mostrar en cada celda. `setAutoCreateRowSorter(true)` habilita el ordenamiento por columna.

> pág. 13 a 16 — JPA_Hibernate_DAO.pdf
> «JPQL trabaja sobre entidades y atributos en lugar de tablas y columnas.»

Todo el plan está construido sobre esas cuatro ideas: la vista expone, el controlador decide, el modelo de tabla traduce, y el JPQL habla en entidades.

---

## 12. Fase 7 (opcional) — los alimentos de la dieta

**No es parte del CRUD de dietas.** Si sobra tiempo, es el "extra" que puede levantar nota, pero tiene un problema de diseño que hay que resolver primero.

### Lo que hay

`model/DietDetail.java` es una tabla pivot:

```java
@Table(name = "DietsDetails",
    uniqueConstraints = @UniqueConstraint(columnNames = {"id_diet", "id_aliment"}))

@ManyToOne(optional = false) @JoinColumn(name = "id_diet",    nullable = false)   private Diet diet;
@ManyToOne(optional = false) @JoinColumn(name = "id_aliment", nullable = false)   private Aliment aliment;
```

### El problema

La restricción `UNIQUE (id_diet, id_aliment)` dice que **un alimento no puede repetirse en la misma dieta**, y no existe ningún campo `cantidad`. O sea:

- Podés decir *"la dieta 1 incluye manzana, pollo y lechuga"*.
- **No** podés decir *"200 g de arroz"*, ni *"arroz al mediodía y a la noche"*.

Para una dieta eso es una limitación real. Las salidas son dos, y hay que elegirlas **antes** de programar:

| Opción | Costo | Consecuencia |
|---|---|---|
| Dejarlo como pivot puro | 0 | el detalle dice *qué* alimentos, no *cuánto* |
| Agregar `cantidad DOUBLE` (o `gramos INT`) a `DietDetail` | cambiar entidad + script + recrear la base | se puede expresar "200 g"; hay que avisarle a RomiM porque toca su esquema |

> [!warning] Hablarlo con el equipo antes, no después
> Si se agrega `cantidad` **después** de que los otros dos cablearon sus pantallas, hay que recrear la base y volver a probar todo. Decidirlo antes de la primera línea de código del detalle.

### Lo que haría falta para cablearlo

1. `DietDetailDAOJPA.buscarPorDieta` — hoy es un `throw` (ver [[bug-jpql-dao]]):

   ```java
   @Override
   public List<DietDetail> buscarPorDieta(Integer idDiet) throws DAOException {
       return consultar("SELECT dd FROM DietDetail dd WHERE dd.diet.id = :idDiet",
                        "idDiet", idDiet);
   }
   ```

2. Una pantalla con **dos tablas**: la de la izquierda con los alimentos de la dieta, la de la derecha con todos los alimentos disponibles, y dos botones para pasar de una a otra.
3. Modelos de tabla para las dos (el de la derecha puede reutilizar `AlimentTableModel`).
4. Manejo del `UNIQUE`: agregar dos veces el mismo alimento tira excepción, hay que informarlo.

Es media sesión más, con suerte. **El CRUD de dietas se entrega sin esto.**

> [!note] Un bug latente que va a aparecer en el DELETE
> `DAOJpa.delete()` (`dao/DAOJpa.java:99`) crea el `EntityManager` **fuera** del `try`:
>
> ```java
> public void delete(ID id) throws DAOException {
>     EntityManager em = JPAUtil.crearEntityManager();   // ← acá, fuera
>     EntityTransaction tx = em.getTransaction();
>     try { ... }
> ```
>
> Si la base está caída, esa línea tira una `RuntimeException` que **no** es `DAOException`, así que el `catch (DAOException ex)` del controlador no la atrapa y sale un stack trace en consola sin mensaje al usuario. Los otros cuatro métodos (`insert`, `findById`, `findAll`, `update`) ya están protegidos; `delete` y `consultar` no.
>
> Arreglo de tres líneas, y es deuda compartida, no de esta pantalla: `EntityManager em = null;` arriba y `em = JPAUtil.crearEntityManager();` adentro del `try`, con `if (em != null)` en el `finally` — igual que en `insert`.

---

## Relacionado

- [[plan-cableado-patient]] · [[plan-cableado-aliment]] — los dos cableados anteriores, con el mismo patrón
- [[plan-crud-bd]] — el plan general de la entrega; la Fase 5 (búsquedas) es donde está el conflicto de archivos
- [[bug-jpql-dao]] — el detalle de los cinco JPQL rotos y de los ocho `{@link}`
- [[hoja-de-ruta-7-a-9]] — el checklist general del trabajo
- [[convenciones]] · [[ejemplo-de-excepciones]] · [[tutorial-dao-desde-cero]]
- [[enunciado_rubrica]] — qué pide la entrega
