---
fuente: síntesis basada en Swing_MVC_Interfaces_Java.pdf (pág. 14, 20 a 23), JTable_en_Swing.pdf (pág. 4 a 18), JPA_Hibernate_DAO.pdf (pág. 8 y 9), enunciado_rubrica.pdf (pág. 1 y 2) y convenciones propias del proyecto
tags: [tarea, plan, mvc, swing, cableado, patient]
estado: vigente
---

# Plan de cableado: `Patient` (Vista ↔ Controlador ↔ Modelo)

> [!info] Qué es esta nota
> El plan completo para conectar `PatientView` con `Patient` a través de `PatientController`, con el código de los métodos, los tips y las trampas. Escrita para poder implementarla **sin depender del chat**.
> **Estado del repo verificado el 2026-11-07** (ver sección 0). Si pasó mucho tiempo, revisá primero si tus compañeros movieron algo.

> [!warning] Procedencia del contenido
> - **Del profesor:** las reglas de MVC, el registro de eventos, la configuración de `JTable` y la unidad de persistencia (citas en cada sección).
> - **Criterio de equipo (NO del profesor):** el `PatientDAOInMemory`, el `PatientTableModel`, el `JOptionPane` en la vista y el orden de trabajo. Van marcados con `[!warning]` o `[!tip]`.

---

## 0. Chequeo del repo (antes de tocar nada)

Estado al 2026-11-07, con `NutritionistApp` en `GitHub francoemmanuelmartinez/NutritionistApp`:

```text
## master...origin/master          ← sin adelante/atrás
git log --oneline --left-right HEAD...origin/master   → vacío
git diff --stat HEAD origin/master                    → vacío
```

**Resultado: sincronizado.** No había commits nuevos de compañeros ni cambios locales sin commitear. Antes de arrancar, repetir:

```powershell
git fetch --all --prune
git status -sb
git log --oneline --left-right HEAD...origin/master
```

---

## 1. El hallazgo que condiciona todo el plan

La capa de persistencia **todavía no puede ejecutarse**:

| Pieza | Estado |
|---|---|
| `src/META-INF/persistence.xml` | **No existe** (tampoco la carpeta `META-INF`) |
| Proveedor JPA (Hibernate) | **No está** en `lib/` |
| Driver MySQL | **No está** en `lib/` |
| `lib/` actual | `flatlaf-3.5.4.jar`, `jakarta.persistence-2.2.3.jar` (sólo la API) |

`jakarta.persistence-2.2.3.jar` son **sólo las interfaces**. Sin proveedor, `Persistence.createEntityManagerFactory(...)` tira `"No Persistence provider for EntityManager named ..."` y `PatientDAOJPA` no se puede ni instanciar.

Por eso el cableado se parte en dos etapas que conviene **no mezclar**:

- **Etapa A — Vista ↔ Controlador ↔ Modelo.** Es lo que enseña `Swing_MVC_Interfaces_Java.pdf` (pág. 20 a 23) y no necesita base de datos. Se puede probar hoy.
- **Etapa B — Controlador ↔ DAO ↔ MySQL.** Necesita la unidad de persistencia (`JPA_Hibernate_DAO.pdf`, pág. 8 y 9). Es un trabajo aparte.

**Recomendación: hacer la Etapa A completa primero.** No se tira nada: el controlador queda igual y en la Etapa B sólo cambia qué implementación de `PatientDAO` se le inyecta.

---

## 2. Etapa A — Cableado Vista ↔ Controlador ↔ Modelo

> pág. 20 a 23 — Swing_MVC_Interfaces_Java.pdf
> Las tres reglas:
> - **Modelo:** lógica y datos. **No conoce Swing** (ni un `import javax.swing`).
> - **Vista:** componentes gráficos. *"Solo muestra y captura datos; expone getters y métodos de actualización."*
> - **Controlador:** *"Conecta la vista con el modelo. Escucha los eventos de la vista, invoca al modelo y actualiza la vista."*

`Patient` ya cumple la regla del modelo (no importa Swing). El trabajo está en la vista y en el controlador.

### Paso A1 — Completar el contrato de `PatientView`

Hoy la vista sólo expone `getBtnAliment()` y `getBtnDiet()`. Según la convención de [[convenciones]] (getters, nunca campos) y el ejemplo del profesor (`getNombre()`, `getBotonSaludar()`, `mostrarSaludo()`), faltan tres grupos.

**a) Lectura del formulario** (devuelven `String`, como `getNombre()` del ejemplo):

```java
public String getIdText()           { return jTextField8.getText(); }
public String getNameText()         { return jTextField1.getText(); }
public String getLastNameText()     { return jTextField2.getText(); }
public String getWeightText()       { return jTextField3.getText(); }
public String getHeightText()       { return jTextField4.getText(); }
public String getObservationsText() { return jTextField6.getText(); }
```

**b) Escritura en el formulario** (equivalentes a `mostrarSaludo`):

```java
public void setNameText(String s)         { jTextField1.setText(s); }
public void setLastNameText(String s)     { jTextField2.setText(s); }
public void setWeightText(String s)       { jTextField3.setText(s); }
public void setHeightText(String s)       { jTextField4.setText(s); }
public void setImcText(String s)          { jTextField5.setText(s); }
public void setObservationsText(String s) { jTextField6.setText(s); }

public void clearForm() {
    jTextField8.setText("");
    jTextField1.setText("");
    jTextField2.setText("");
    jTextField3.setText("");
    jTextField4.setText("");
    jTextField5.setText("");
    jTextField6.setText("");
}
```

**c) Botones, tabla y mensajes:**

```java
public javax.swing.JButton getBtnInsert() { return jButton3; }
public javax.swing.JButton getBtnModify() { return jButton4; }
public javax.swing.JButton getBtnDelete() { return jButton5; }
public javax.swing.JButton getBtnSearch() { return jButton6; }

public javax.swing.JTable getTablePatients() { return jTable1; }

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

### Paso A2 — Decidir a quién le habla el controlador

Hoy el controlador recibe un `Patient` suelto:

```java
private final Patient model;   // alcanza para navegar, no para un CRUD
```

Para un CRUD necesita **el paciente en edición** y **el repositorio de pacientes**. El repositorio debe ser la **interfaz** `PatientDAO` (que ya existe), no la implementación JPA:

```java
public class PatientController {

    private Patient actual;              // el paciente en edición
    private final PatientDAO dao;        // el contrato, no la implementación
    private final PatientView view;
}
```

Para poder probar la Etapa A **sin base de datos**, agregar una implementación en memoria (un *test double*, ~25 líneas):

```java
package appNutritionist.dao;

import appNutritionist.model.Patient;
import java.util.ArrayList;
import java.util.List;

public class PatientDAOInMemory implements PatientDAO {

    private final List<Patient> pacientes = new ArrayList<>();
    private int proximoId = 1;

    @Override
    public void insert(Patient p) {
        p.setId(proximoId++);
        pacientes.add(p);
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
    public void update(Patient p) {
        for (int i = 0; i < pacientes.size(); i++) {
            if (pacientes.get(i).getId() == p.getId()) {
                pacientes.set(i, p);
                return;
            }
        }
    }

    @Override
    public void delete(Integer id) {
        pacientes.removeIf(p -> p.getId() == id);
    }
}
```

En `AppNutritionist`:

```java
PatientView vista = new PatientView();
PatientDAO dao = new PatientDAOInMemory();      // Etapa A
// PatientDAO dao = new PatientDAOJPA(emf);     // Etapa B: una sola línea
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
    p.setName(view.getNameText());
    p.setLastName(view.getLastNameText());
    p.setWeight(Double.parseDouble(view.getWeightText()));
    p.setHeight(Double.parseDouble(view.getHeightText()));
    p.setObservations(view.getObservationsText());
    return p;
}

// 2) Del objeto al formulario
private void mostrarPaciente(Patient p) {
    view.setNameText(p.getName());
    view.setLastNameText(p.getLastName());
    view.setWeightText(String.valueOf(p.getWeight()));
    view.setHeightText(String.valueOf(p.getHeight()));
    view.setImcText(String.format("%.2f", p.getImc()));
    view.setObservationsText(p.getObservations());
}

// 3) Refrescar la tabla
private void refrescarTabla() {
    tableModel.setPacientes(dao.findAll());
}
```

`Double.parseDouble` es **el** punto donde el formulario explota si el usuario escribe "ochenta" o deja el campo vacío. Va con `try/catch (NumberFormatException e)` y `view.showError("El peso debe ser un número")`. Esto alimenta el criterio *Manejo de Excepciones* de la rúbrica (pág. 2), no sólo el DAO.

El IMC es regla de negocio: va en el **modelo**, no en la vista.

```java
// en Patient.java
public double getImc() {
    if (height <= 0) return 0;
    return weight / (height * height);
}
```

### Paso A4 — La tabla

> pág. 11 a 18 — JTable_en_Swing.pdf (ejemplo MVC con `AbstractTableModel`)
> pág. 8 y 9 — JTable_en_Swing.pdf (`setAutoCreateRowSorter`, `SINGLE_SELECTION`)

| | `DefaultTableModel` (pág. 4 a 7) | `AbstractTableModel` propio (pág. 11 a 18) |
|---|---|---|
| Código | 5 líneas | ~40 líneas, una clase aparte |
| Datos | `Object[][]` duplicado | Lee la `List<Patient>` directo |
| Ordenar por columna | funciona con `Object` | funciona si devolvés tipos reales en `getValueAt` |
| Criterio "bien estructurado" | aceptable | mejor |

**Recomendación: `AbstractTableModel`**, en `appNutritionist.view.PatientTableModel` (es una clase de Swing, va en `view`):

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
    @Override public String getColumnName(int c) { return columnas[c]; }

    @Override
    public Object getValueAt(int fila, int col) {
        Patient p = pacientes.get(fila);
        return switch (col) {
            case 0 -> p.getId();
            case 1 -> p.getName();
            case 2 -> p.getLastName();
            case 3 -> p.getWeight();
            case 4 -> p.getHeight();
            case 5 -> String.format("%.2f", p.getImc());
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
> Si no se quita el `DefaultTableModel` de prueba del `.form` (las filas de `null`), NetBeans lo vuelve a generar en cada edición del diseño y pisa el modelo del controlador.

### Paso A5 — Los listeners (el corazón del cableado)

> pág. 14 — Swing_MVC_Interfaces_Java.pdf (registrar eventos con `addActionListener` y lambda)

```java
public PatientController(PatientDAO dao, PatientView view) {
    this.dao = dao;
    this.view = view;
    this.actual = new Patient();

    view.getTablePatients().setModel(tableModel);
    refrescarTabla();                                    // arranca con datos

    view.getBtnInsert().addActionListener(e -> insertar());
    view.getBtnModify().addActionListener(e -> modificar());
    view.getBtnDelete().addActionListener(e -> eliminar());
    view.getBtnSearch().addActionListener(e -> buscar());

    view.getTablePatients().addMouseListener(new java.awt.event.MouseAdapter() {
        @Override
        public void mouseClicked(java.awt.event.MouseEvent e) {
            seleccionarFila();
        }
    });

    view.getBtnAliment().addActionListener(e -> irAAliments());   // lo que ya funcionaba
}
```

Cada acción:

```java
private void insertar() {
    try {
        Patient p = leerFormulario();
        dao.insert(p);              // Etapa B: acá aparece "No se pudo insertar el paciente"
        refrescarTabla();
        view.clearForm();
        view.showInfo("Paciente guardado.");
    } catch (NumberFormatException ex) {
        view.showError("Peso y altura deben ser números.");
    } catch (DaoException ex) {     // sólo compila cuando exista DaoException
        view.showError(ex.getMessage());
    }
}

private void buscar() {
    try {
        int id = Integer.parseInt(view.getIdText());
        Patient p = dao.findById(id);
        if (p == null) {
            view.showError("No existe un paciente con id " + id);   // "no existe" ≠ "falló"
        } else {
            mostrarPaciente(p);
        }
    } catch (NumberFormatException ex) {
        view.showError("El ID debe ser un número.");
    } catch (DaoException ex) {
        view.showError(ex.getMessage());
    }
}
```

> [!warning] El bug del ordenamiento (`convertRowIndexToModel`)
> Con `autoCreateRowSorter(true)`, al ordenar por columna la fila **visual** deja de coincidir con la del modelo:
> ```java
> int filaVista  = view.getTablePatients().getSelectedRow();
> int filaModelo = view.getTablePatients().convertRowIndexToModel(filaVista);
> actual = tableModel.getPaciente(filaModelo);
> ```
> Sin ese `convertRowIndexToModel` se edita el paciente equivocado, **sólo cuando la tabla está ordenada**. Es un bug intermitente y difícil de reproducir.

### Paso A6 — Limpiar los handlers que dejó NetBeans

`PatientView` tiene `jTextField1ActionPerformed`, `jComboBox1ActionPerformed`, etc., todos vacíos. **No poner lógica ahí**: esos listeners los registra la vista sobre sí misma, que es justo lo que MVC separa. Dejarlos vacíos o borrarlos desde el diseñador; la lógica va en el controlador.

### Paso A7 — Navegación

`getBtnAliment()` ya funciona. `getBtnDiet()` todavía no tiene destino (no existe `DietView`): dejarlo sin listener hasta el Bloque B/C de [[hoja-de-ruta-7-a-9]], o deshabilitarlo con `getBtnDiet().setEnabled(false)` para que no parezca roto.

---

## 3. Etapa B — Cuando se quiera enchufar MySQL

- [ ] `src/META-INF/persistence.xml` con la unidad de persistencia (pág. 8 — `JPA_Hibernate_DAO.pdf`): url `jdbc:mysql://localhost:3306/...`, usuario, clave, `hibernate.hbm2ddl.auto=update`.
- [ ] Agregar a `lib/` el **proveedor** (Hibernate) y el **driver** (MySQL). Con Maven viene todo junto (pág. 5); con Ant hay que bajar los jars uno por uno y agregarlos a `javac.classpath` en `nbproject/project.properties`.
- [ ] `Persistence.createEntityManagerFactory("nutritionistPU")` **una sola vez** en `AppNutritionist` (pág. 9), y pasar ese `emf` al DAO.
- [ ] `DaoException` (todavía no existe) — ver [[ejemplo-de-excepciones]].
- [ ] Cambiar una línea en `AppNutritionist`: `new PatientDAOJPA(emf)`.

> [!question] Decisión pendiente para la Etapa B
> Con Ant (sin Maven), Hibernate arrastra ~8 jars (`hibernate-core`, `jboss-logging`, `byte-buddy`, `antlr4-runtime`, …). La alternativa es **JDBC** (`mysql-connector-j.jar`, un solo jar), que es el otro PDF de la cátedra ([[JDBC_DAO_MySQL]]). La Etapa A es independiente de esa decisión.

---

## 4. Orden de trabajo

| # | Paso | Dónde | Se puede probar |
|---|---|---|---|
| 1 | Getters y métodos de actualización | `PatientView` | — |
| 2 | `getImc()` | `Patient` | — |
| 3 | `PatientDAOInMemory` | `dao` | — |
| 4 | `PatientTableModel` | `view` | — |
| 5 | Campos + constructor + listeners | `PatientController` | compila |
| 6 | Inyectar el DAO en memoria | `AppNutritionist` | **sí, todo el CRUD** |
| 7 | Borrar el modelo de ejemplo del `JTable` | diseñador | sí |
| 8 | Etapa B (persistencia) | `META-INF`, `lib`, DAO | con MySQL arriba |

Compilación manual (siempre `--release 25`):

```powershell
javac --release 25 -cp "lib/jakarta.persistence-2.2.3.jar;lib/flatlaf-3.5.4.jar" `
      -d build/classes (Get-ChildItem src -Recurse -Filter *.java).FullName
```

---

## 5. Glosario mínimo de este cableado

| Concepto | Qué es acá |
|---|---|
| **Modelo** | `Patient`, con datos, getters/setters y `getImc()` |
| **Vista** | `PatientView`, que muestra y captura; no decide |
| **Controlador** | `PatientController`, que escucha, pide al DAO y actualiza la vista |
| **DAO** | Contrato (`PatientDAO`) + implementación (`PatientDAOInMemory` hoy, `PatientDAOJPA` después) |
| **TableModel** | `PatientTableModel`, adapta `List<Patient>` a `JTable` |

## 6. Dudas y vacíos

- [ ] ¿La cátedra prefiere `DefaultTableModel` (simple, pág. 4 a 7) o el `AbstractTableModel` (MVC, pág. 11 a 18) para la entrega? El segundo es mejor diseño, pero es más código.
- [ ] El combo de `Diet` (`jComboBox1`) queda inerte: depende del Bloque A de [[hoja-de-ruta-7-a-9]].
- [ ] ¿Se deshabilita `Diets` o se deja clickeable sin acción?

## Relacionado

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
