---
fuente: auditoría del código real de `NutritionistApp` en `origin/master` (`1f785b6` + `751bb35`) + Swing_MVC_Interfaces_Java.pdf (pág. 14, 20 a 23 y 26 a 27), JTable_en_Swing.pdf (pág. 4 a 18), Listas_y_ComboBox_Swing.pdf (pág. 1 a 15), JPA_Hibernate_DAO.pdf (pág. 9 a 16) y enunciado_rubrica.pdf (pág. 1 y 2)
tags: [tarea, plan, mvc, swing, cableado, diet, crud]
estado: en curso
actualizado: 2026-11-12
aliases: [plan-cableado-diet, plan-diet, plan-crud-diet, plan-dietas]
---

# Plan de cableado: `Diet` (Vista → Controlador → Modelo)

> [!info] Qué es esta nota
> El tercer cableado del proyecto, después de `Patient` y `Aliment`. Es el que falta para cerrar la entrega: `DietController` está vacío y `DietView` existe pero **todavía no expone nada**. Escrito para que se pueda implementar **sin depender del chat**, con el código exacto y las trampas que ya pagaron los otros dos.
>
> **Punto de partida:** `origin/master` = `751bb35` (Caro, 2026-11-11), que es `1f785b6` + `DietView` + su corrección. Antes de empezar hay que hacer el `pull`.
>
> **Novedad 2026-11-12:** Caro creó la vista, así que **§2 bis** reemplaza a §2.1 y §2.2: en vez de crear `DietView` desde cero, se ajusta la que ya está. Incluye los 12 pasos del diseñador y los ~30 métodos que tiene que exponer.
>
> **Y el panel de alimentos** —la segunda tabla, el combo y Add/Remove— está completo en **§4 bis**: constructor sobrecargado, las dos listas en paralelo, la cascada manual del `delete()` y las 16 pruebas. La nota ya no tiene pendientes.

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
| `view/DietView.java` + `.form` | ⚠️ **existe, sin ajustar** | la creó Caro (`6f6a449`, corregida en `751bb35`); le falta todo lo de §2 bis: 2 columnas, getters, rótulos y estado del panel |
| `view/DietTableModel.java` | ❌ **no existe** | |
| Botón **Diets** en las vistas | ❌ **sin listener** | `getButtonDiets()` existe (`PatientView:469`, `AlimentView:364`) pero nadie lo escucha |
| `AppNutritionist` | ✅ **ya cableado** | crea `new DietDAOJPA()` y ya se lo pasa a `PatientController` y `AlimentController` |
| Combo de dietas en `PatientView` | ✅ **funciona** | `PatientController.loadDiets()` usa `dietDao.findAll()` |
| `dao/DietDetailDAOJPA.buscarPorDieta` | ❌ `throw` | ya **no** es opcional: sin esto no se puede llenar la tabla de alimentos de la dieta (§4 bis.8) |

> [!important] La buena noticia
> **El modelo, el DAO y la conexión ya existen.** No hay que tocar `Diet.java`, ni `DietDAO.java`, ni `AppNutritionist`, ni `persistence.xml`. Lo que falta es **ajustar** la vista (§2 bis), el modelo de tabla, el controlador —con el panel de alimentos— y una línea de JPQL. Todo eso es trabajo de una sesión.

> [!tip] Lo que ya funciona y conviene probar antes de empezar
> Abrí la app, andá a **Patients** y mirá el combo de dietas: tiene que mostrar `-- Sin dieta --`, `Hipocalórica` y `Vegetariana`. Si eso anda, `DietDAO.findAll()` y la conexión están bien, y lo que sigue es sólo interfaz.

---

## 1. Alcance: qué entra y qué no

### Entra

- **Ajustar** `DietView` (ya existe) para que sea intuitiva: 2 columnas en la tabla de dietas, descripción en un área de texto corta, rótulos, tooltips y los getters que hoy no tiene (§2 bis).
- `DietController` completo: insert, modify, delete, search por nombre y selección de fila.
- `DietTableModel` de dos columnas.
- **Los listeners del botón Diets** en `PatientView` y `AlimentView` para llegar a la pantalla nueva desde las otras dos.
- El JPQL de `DietDAOJPA`.
- El manejo del error de clave foránea al borrar una dieta en uso.
- **El panel de alimentos de la dieta**: agregar y quitar filas de `DietDetail` desde la misma pantalla (segunda tabla + combo + Add/Remove). Requiere implementar `DietDetailDAOJPA.buscarPorDieta`.

### No entra (y conviene decirlo en voz alta)

- **La cantidad de cada alimento** (*"200 g de arroz"*). `DietDetail` sólo tiene `(Diet, Aliment)` y un `UNIQUE (id_diet, id_aliment)`. La tabla dice **qué** alimentos incluye la dieta, no **cuánto**. Agregar `cantidad` implica cambiar la entidad, el script y recrear la base: es una decisión de equipo, no de esta pantalla.
- **Excepciones propias para las claves foráneas** (`DietConPacientesException` o similar). Se puede hacer después; acá alcanza con un mensaje claro. RomiM quedó en ver ese tema.

> [!warning] Hablar con RomiM antes de arrancar
> En el reparto original del equipo, **la vista de dietas era de RomiM** — lo dijo ella por WhatsApp: *"Quedaría por hacer la vista de las dietas"*. Caro la terminó, así que ese riesgo ya pasó; lo que queda es que **RomiM no vuelva a escribirla** y que las excepciones de FK sigan siendo suyas.

---

## 2. Fase 1 — Crear `DietView` en el diseñador

> [!warning] Esta fase ya está hecha
> Caro creó la vista en `6f6a449` y la corrigió en `751bb35`. Lo que sigue queda como **referencia del diseño original** (qué componentes lleva, qué propiedades, qué getters), pero el resultado real difiere en tres puntos: la tabla de dietas tiene 4 columnas, la descripción es un `JTextArea` y no hay un solo método público.
>
> **Para trabajar, usá §2 bis.** Ahí está el paso a paso sobre la vista que ya existe. De esta sección sólo sigue válido §2.4 (los límites de 50/100 caracteres).
>
> ⚠️ Los nombres de esta sección (por ejemplo `descriptionTextField` en §2.1 y §2.3) **no son los definitivos**: la descripción se queda como `descriptionTextArea` (decisión del 2026-11-12, §2 bis.2). Los getters y setters que valen son los de §2 bis.9.

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

## 2 bis. Ajustes a la `DietView` que ya existe

> [!important] Por qué esta sección existe
> `DietView` **ya no hay que crearla**: Caro la pusheó en `6f6a449` (*"Creacion de DietView.java"*) y la corrigió en `751bb35` (*"Se corrige error en DietView.java en dietAlimentsTable"*). Lo que sigue son los ajustes para que la pantalla haga lo que tiene que hacer —administrar dietas **y** los alimentos de cada dieta— sin que haya que explicarla. **Reemplaza a §2.1 y §2.2** como fuente de verdad.

### 2 bis.0 Antes de tocar nada

1. `git pull` y confirmá que estás parado en `751bb35`.
2. **Avisale a Caro:** *"estoy retocando `DietView`, no la toques hasta que pushee"*. El `.form` es XML generado por el diseñador y un merge entre dos versiones del mismo form **no se resuelve solo**: git ve dos archivos de texto y elige mal.
3. Commiteá lo que ya esté andando **antes** de empezar, aunque la vista todavía no haga nada. Un commit chico hoy es un conflicto chico mañana.
4. **Todos los cambios se hacen en el diseñador**, nunca a mano dentro de `initComponents()`: NetBeans regenera ese método al guardar y te borra la edición.
5. Al terminar cada paso, `Ctrl+S` y compilar. Si un paso rompe el layout, `Ctrl+Z` y volver.

### 2 bis.1 `dietsTable`: de 4 columnas a 2

**Qué:** hoy declara `ID`, `Name`, `Aliment 1`, `Aliment 2`. Hay dos cosas mal. La primera: una dieta tiene **N** alimentos, no dos —`Vegetariana` del script ya tiene tres (Banana, Arroz integral, Palta)— y esos alimentos ya se muestran abajo, en `dietAlimentsTable`; con las dos cosas a la vez la pantalla dice lo mismo dos veces y se puede desincronizar. La segunda: el `Description` no va en la grilla, porque es el único texto largo de la tabla y la celda lo recorta.

**Cómo:**

1. Clic derecho sobre `dietsTable` → **Table Contents…**
2. Pestaña **Columns**: seleccionar `Aliment 2` → **Delete**; `Aliment 1` → **Delete**; `Description` → **Delete**.
3. Verificar que quedan **`ID`, `Name`** (doble clic para renombrar).
4. Pestaña **Rows**: tiene que decir **0**. Si hay filas, borralas — si no, el diseñador hornea filas de `null` en cada guardado (trampa 1 de §8).
5. `Ctrl+S` y compilar.

> [!important] Por qué la descripción no va en la tabla
> Es el mismo criterio que ya usa `PatientTableModel`: **`observations`, que es `VARCHAR(500)`, no está en la grilla**. La tabla muestra lo que sirve para **identificar y comparar** registros (`ID`, `Name`); el texto largo se lee completo en el formulario, que se llena al seleccionar la fila.
>
> Consecuencia práctica: no hay que pelear con celdas recortadas, ni con `\n` adentro de una celda (que el renderer por defecto dibuja como un cuadradito), ni con alto de fila variable. Y la descripción **no se pierde**: sigue en la base y aparece completa en `descriptionTextArea` al hacer clic.
>
> Para la defensa: *"la descripción no va en la tabla porque es texto largo y la celda lo recorta; se muestra completa en el formulario, igual que las observaciones en pacientes"*.

**Resultado:** 2 columnas, que es exactamente lo que espera `DietTableModel` en §3.

### 2 bis.2 `descriptionTextArea`: se queda, pero con `lineWrap`

**Qué:** hoy es un `JTextArea` de 20 columnas por 5 filas, adentro de `jScrollPane1`. La columna es `description VARCHAR(100)` (`Sql/nutricion.sql:10`): un texto corto.

**Decisión (2026-11-12):** se **mantiene la textarea**. El argumento original para cambiarla era que el `\n` salía como cuadradito en la celda de `dietsTable`, y eso desapareció cuando la descripción salió de la tabla (§2 bis.1). Ahora el único lugar donde se muestra es este formulario, y una textarea dibuja los saltos de línea bien.

Pero hay que configurarla, porque hoy **no tiene `lineWrap`**: al escribir, el texto se corre horizontalmente en vez de bajar de renglón.

**Cómo:**

1. Seleccionar `descriptionTextArea` en el **Navigator**.
2. En **Properties**, tildar **`lineWrap`** y **`wrapStyleWord`** (corta por palabra, no por letra).
3. Bajar `rows` de 5 a **3**: con 100 caracteres alcanza, y le devuelve ~40 px al formulario.
4. Opcional: `toolTipText` → `Descripcion breve (opcional)`.
5. `Ctrl+S` y compilar.

**Resultado:** el texto baja de renglón al escribir y el componente sigue llamándose `descriptionTextArea`, así que los getters son los de §2 bis.9.

> [!important] El `Enter` se neutraliza al guardar, no en la vista
> La textarea sigue aceptando `Enter`, y un salto de línea adentro de un `VARCHAR(100)` no aporta nada. En vez de bloquear la tecla (un `KeyListener` en la vista sería lógica en el lugar equivocado), el controlador lo normaliza en `readForm()` con `replaceAll("\\s+", " ").trim()` (§4.3). Así el usuario puede apretar `Enter` sin miedo y la base guarda una sola línea.

> [!warning] Si no tildás `lineWrap`
> Queda exactamente el problema de `observations`: el texto se escribe y se va escondiendo a la izquierda. Es una propiedad, no código: no hay forma de arreglarlo desde el controlador.

### 2 bis.3 El campo ID no se edita

**Qué:** `idTextField` es editable en las tres vistas del proyecto. El id lo asigna MySQL (`AUTO_INCREMENT`): si alguien lo toca, el `INSERT`/`UPDATE` va a otro lado o falla.

**Cómo:** seleccionar `idTextField` → ventana **Properties** → pestaña **Properties** → desmarcar **editable**. Y en `toolTipText` escribir `Se asigna automáticamente`.

> [!tip] Aprovechá y arreglá las otras dos
> `AlimentView` y `PatientView` tienen el mismo problema. Son dos clics cada una y suman al "intuitiva" del enunciado.

### 2 bis.4 Los rótulos: el cambio que más rinde

**Qué:** hoy `alimentsLabel` dice `"Aliments of the diet :"` y está al lado del **combo**, que lista los alimentos *disponibles*, no los de la dieta. Y la tabla que sí muestra "los de la dieta" no tiene ningún título. La pantalla miente en el único punto donde el profe va a mirar.

**Cómo:**

1. Seleccionar `alimentsLabel` → **Change Variable Name…** → `alimentLabel`. Cambiar el texto a `Aliment :`.
2. Arrastrar un **Label** desde la paleta y soltarlo **arriba de `dietAlimentsScrollPane`**.
3. **Change Variable Name…** → `dietAlimentsLabel`. Texto: `Aliments of the diet`.
4. Opcional: otro label arriba de `dietsTable`, variable `dietsLabel`, texto `Registered diets`.

`dietAlimentsLabel` va a mostrar el texto dinámico que le mande el controlador:

```
Aliments of the diet : Hipocalórica (3)
```

Eso es lo que convierte "una tabla abajo con datos" en *"los alimentos de la dieta que tengo seleccionada"*. Es la diferencia entre que haya que explicarlo y que se entienda solo.

### 2 bis.5 Tooltips

`DietView` hoy tiene **cero** tooltips; `AlimentView` ya usa tres. Son gratis y contestan las dudas antes de que aparezcan. Seleccionar cada componente → **Properties** → `toolTipText`:

| Componente | toolTipText |
|---|---|
| `alimentComboBox` | `Alimentos que todavía no están en esta dieta` |
| `addAlimentButton` | `Agrega el alimento elegido a la dieta` |
| `removeAlimentButton` | `Quita el alimento de la dieta (no lo borra del sistema)` |
| `searchButton` | `Escribí parte del nombre y presioná para buscar` |
| `idTextField` | `Se asigna automáticamente` |

El de **Remove** es el más importante: contesta *"¿se borra la manzana?"* con pasar el mouse, y es justo la duda conceptual de la tabla pivot.

### 2 bis.6 El combo arranca vacío

**Qué:** `alimentComboBox` viene con `"Item 1"`, `"Item 2"`, `"Item 3"`, `"Item 4"`. Es relleno del diseñador; el controlador lo pisa. Pero un combo con basura se ve mal en la defensa y confunde mientras no hay controlador.

**Cómo:** clic derecho sobre el combo → **Edit** (o la pestaña del modelo en Properties) → seleccionar los cuatro ítems → **Delete**. Tiene que quedar con 0 ítems.

Es el mismo relleno que tiene `dietComboBox` en `PatientView`; ahí no hace falta tocarlo porque el controlador ya lo llena en cada arranque.

### 2 bis.7 El panel de alimentos arranca deshabilitado

**Qué:** si no hay dieta elegida, **Add** no tiene a qué agregar y **Remove** no tiene qué quitar. Abrir la pantalla con todo habilitado es la primera confusión del recorrido.

**Cómo:** en el diseñador, dejar `alimentComboBox`, `addAlimentButton`, `removeAlimentButton` y `dietAlimentsTable` con **enabled** desmarcado. El controlador los habilita cuando hay una dieta con id (método `setAlimentsPanelEnabled` de 2 bis.9).

> [!warning] Deshabilitar el panel **no** deshabilita a los hijos
> Si le sacás `enabled` al `JPanel` con borde "Aliments", el combo y los botones siguen activos. Hay que hacerlo componente por componente, y por eso el método recibe los cuatro.

### 2 bis.8 (opcional) Agrupar el panel de alimentos

Hoy el combo y los botones están en un panel con borde *Aliments* y la tabla que muestran está **afuera**, abajo. Mover la tabla adentro del panel hace que el bloque entero se lea como una unidad: **selector → Add/Remove → lo que tiene la dieta**.

**Cómo:** en el **Navigator**, arrastrar `dietAlimentsScrollPane` y soltarlo dentro de `alimentsPanel`. Después reacomodar el `GroupLayout` del panel para que la tabla ocupe una fila propia, debajo del combo.

Si el layout queda feo y no hay tiempo, `Ctrl+Z` y listo: **el 80% del beneficio está en el rótulo de 2 bis.4**, no en el agrupamiento.

### 2 bis.9 Los métodos que expone la vista

Todo va **después** de `// End of variables declaration//GEN-END:variables`. Son métodos escritos a mano: regenerar `initComponents()` no los borra.

```java
// --- Navegación ---
public javax.swing.JButton getButtonPatients() {return patientsButton;}
public javax.swing.JButton getButtonAliments() {return alimentsButton;}

// --- Botones del CRUD ---
public javax.swing.JButton getButtonInsert() {return insertButton;}
public javax.swing.JButton getButtonModify() {return modifyButton;}
public javax.swing.JButton getButtonDelete() {return deleteButton;}
public javax.swing.JButton getButtonSearch() {return searchButton;}

// --- Panel de alimentos de la dieta ---
public javax.swing.JButton getButtonAddAliment()    {return addAlimentButton;}
public javax.swing.JButton getButtonRemoveAliment() {return removeAlimentButton;}

// --- Tablas ---
public javax.swing.JTable getTableDiets()        {return dietsTable;}
public javax.swing.JTable getTableDietAliments() {return dietAlimentsTable;}

// --- Campos del formulario ---
public String getIdText()          {return idTextField.getText();}
public String getNameText()        {return nameTextField.getText();}
public String getDescriptionText() {return descriptionTextArea.getText();}

public void setIdText(String value)          {idTextField.setText(value);}
public void setNameText(String value)        {nameTextField.setText(value);}
public void setDescriptionText(String value) {descriptionTextArea.setText(value);}

public void clearForm() {
    setIdText("");
    setNameText("");
    setDescriptionText("");
}

// --- Panel de alimentos de la dieta ---

// Llena el combo con los alimentos que TODAVIA no estan en la dieta.
public void setAlimentOptions(java.util.List<String> nombres) {
    javax.swing.DefaultComboBoxModel<String> model = new javax.swing.DefaultComboBoxModel<>();
    for (String nombre : nombres) {
        model.addElement(nombre);
    }
    alimentComboBox.setModel(model);
}

// Devuelve la posicion elegida en el combo, o -1 si no hay ninguna.
public int getAlimentIndex() {return alimentComboBox.getSelectedIndex();}

// Habilita o deshabilita el panel completo, componente por componente.
public void setAlimentsPanelEnabled(boolean activo) {
    alimentComboBox.setEnabled(activo);
    addAlimentButton.setEnabled(activo);
    removeAlimentButton.setEnabled(activo);
    dietAlimentsTable.setEnabled(activo);
}

// "Aliments of the diet : Hipocalorica (3)"
public void setDietAlimentsTitle(String titulo) {dietAlimentsLabel.setText(titulo);}

public void clearDietAlimentsSelection() {dietAlimentsTable.clearSelection();}

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

// Confirmacion propia del panel: dice el nombre del alimento y aclara el alcance.
public boolean confirmRemoveAliment(appNutritionist.model.Aliment aliment) {
    return javax.swing.JOptionPane.showConfirmDialog(this,
            "¿Quitar " + aliment.getAliment() + " de esta dieta?\n"
            + "El alimento sigue existiendo en el sistema.",
            "Confirmar", javax.swing.JOptionPane.OK_CANCEL_OPTION)
            == javax.swing.JOptionPane.OK_OPTION;
}
```

> [!note] Por qué `getAlimentIndex()` y no un `getAliment()`
> La vista no puede devolver el `Aliment` porque no lo tiene: sólo conoce los textos que le pasaron. El controlador guarda la lista real en paralelo y traduce el índice — el mismo patrón que `PatientController` usa con `List<Diet> diets`.

### 2 bis.10 Verificación visual (sin controlador todavía)

Todavía no hay `DietController`, así que la pantalla va a estar vacía. Lo que se verifica es la **forma**:

| # | Qué mirar | Esperado |
|---|---|---|
| 1 | La tabla de dietas | 2 columnas: `ID`, `Name`. Sin `Aliment 1/2` ni `Description` |
| 2 | Descripción | recuadro de ~3 renglones que **baja de renglón** al escribir (tiene `lineWrap`) |
| 3 | El campo ID | gris, no se puede escribir |
| 4 | El combo de alimentos | vacío, sin `Item 1..4` |
| 5 | Add / Remove / combo | grises al abrir |
| 6 | Rótulo sobre la tabla de abajo | `Aliments of the diet` |
| 7 | Rótulo junto al combo | `Aliment :` |
| 8 | Tooltips | al pasar el mouse por Remove: *"Quita el alimento de la dieta (no lo borra del sistema)"* |
| 9 | `Ctrl+F` en `DietView.java` | aparecen `descriptionTextArea` y `jScrollPane1`; **no** queda ningún `descriptionTextField` |
| 10 | Compilar | `ExitCode: 0` |

> [!tip] La prueba que vale
> Cerrá el diseñador, reabrí `DietView.form` y mirá que todo siga igual. Si el form se "rearma" solo cada vez que lo abrís, quedó algo mal guardado.

### 2 bis.11 Trampas del diseñador

| # | Trampa | Qué pasa |
|---|---|---|
| 1 | Editar `initComponents()` a mano | el próximo guardado del form te borra los cambios, sin aviso |
| 2 | Borrar una columna de la tabla y dejar `rowCount > 0` | el diseñador regenera filas de `null` en cada guardado |
| 3 | Renombrar una variable después de escribir métodos a mano | NetBeans actualiza `initComponents()` y los campos, **no** tus métodos: quedan apuntando a un nombre que ya no existe y no compila |
| 4 | Commitear `DietView.java` sin `DietView.form` (o al revés) | el par queda desincronizado y el diseñador reescribe el `.java` en la próxima apertura, pisando tu código |
| 5 | Sacarle `enabled` al `JPanel` | no deshabilita a los hijos |
| 6 | Reacomodar el `GroupLayout` "a mano" en el código | NetBeans lo regenera; usá el diseñador o `Set Layout` de la paleta |
| 7 | Trabajar sin avisarle a Caro | merge de dos `.form`: conflicto XML sin resolución automática |

> [!danger] La trampa 4 es la que más se paga
> `DietView.java` y `DietView.form` son **una sola unidad**. En el commit van siempre los dos, en el mismo `git add`.

### 2 bis.12 Lo que sigue después de esto

Con la vista así, el controlador puede hacer las cuatro operaciones del CRUD **más** agregar y quitar alimentos de la dieta. Las dos cosas están escritas:

- **§4 (Fase 3)** — el CRUD de dietas.
- **§4 bis** — el panel de alimentos: constructor sobrecargado, las dos listas en paralelo, `addAliment`/`removeAliment`, `delete()` con cascada manual, `buscarPorDieta` y las 16 pruebas.

Las cuatro cosas que había que resolver quedaron así:

| Qué | Dónde |
|---|---|
| Implementar `DietDetailDAOJPA.buscarPorDieta` (hoy es un `throw`) | §4 bis.8 |
| `List<DietDetail>` en paralelo con la tabla de abajo | §4 bis.1 |
| Combo filtrado para no chocar con `UNIQUE (id_diet, id_aliment)` | §4 bis.3 |
| Delete de una dieta con alimentos (`fk_detail_diet` sin `ON DELETE CASCADE`) | §4 bis.6 |

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

    private final String[] columns = {"ID", "Name"};
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
            default -> null;
        };
    }
}
```

> [!important] El `null` de `description`
> La columna es `VARCHAR(100)` **sin** `NOT NULL`. Una dieta cargada por SQL puede tener `description = NULL`, y `showDiet` se rompe si llamás `.setText(null)` sin cuidado. Por eso `view.setDescriptionText(...)` va con `== null ? "" :` (§4.3). Ya no hay celda que proteger: la descripción no se muestra en la tabla.

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
        // La descripcion viene de un JTextArea: cualquier Enter se convierte en un
        // espacio para que en el VARCHAR(100) no queden saltos de linea.
        d.setDescription(view.getDescriptionText().replaceAll("\\s+", " ").trim());
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

## 4 bis. Fase 3 bis — el panel de alimentos en `DietController`

> [!important] Qué agrega esta sección
> §4 escribió el controlador del **CRUD de dietas**. Esto es lo que falta para que la pantalla también administre **qué alimentos tiene cada dieta**: cargar la segunda tabla, agregar, quitar y manejar el borrado de una dieta que tiene alimentos.
>
> Se puede hacer **después** de §4 y sin reescribirlo: son bloques nuevos que se suman al mismo archivo. Lo único que cambia del código anterior está marcado como **[reemplaza]**.

### 4 bis.0 Lo que hay antes de empezar

| Requisito | Dónde |
|---|---|
| `DietView` con los métodos de §2 bis.9 (incluye los getters de Add/Remove) | §2 bis |
| `DietTableModel` | §3 |
| `AlimentTableModel` — se **reutiliza**, no se escribe uno nuevo | `view/AlimentTableModel.java` |
| `DietDetailDAOJPA.buscarPorDieta` implementado | §4 bis.8 |
| La base con datos de ejemplo cargados | §7 |

> [!tip] La tabla de abajo no necesita modelo propio
> `AlimentTableModel` ya tiene las 4 columnas exactas (`ID`, `Name`, `Category`, `Calories`) y un `setAliments(List<Aliment>)`. Es literalmente el mismo modelo que usa `AlimentView`. Escribir un `DietAlimentTableModel` sería duplicar código para nada.

### 4 bis.1 Estado nuevo del controlador

Se suman **cuatro** campos a la clase — **[reemplaza]** el bloque de campos de §4.1:

```java
    private final DietDetailDAO dietDetailDao;                 // la tabla pivot
    private final AlimentTableModel dietAlimentsModel = new AlimentTableModel();
    private final List<DietDetail> dietDetails = new ArrayList<>();   // mismo indice que la tabla de abajo
    private final List<Aliment> availableAliments = new ArrayList<>(); // los que faltan agregar
```

> [!important] Por qué dos listas
> `dietAlimentsTable` muestra **alimentos**, pero para borrar hay que pasarle al DAO el **id del `DietDetail`**. Las dos listas van en el mismo orden que las filas de la tabla: el índice es el puente. Es la misma idea que `List<Diet> diets` en `PatientController`, pero con dos niveles.

### 4 bis.2 El constructor — **[reemplaza]** el de §4.1

Para no tocar `PatientController` ni `AlimentController`, va un constructor sobrecargado: el de 4 argumentos que ya usan las otras pantallas sigue funcionando y delega en el nuevo creando el DAO que falta.

```java
    // Compatibilidad: PatientController y AlimentController siguen llamando a este.
    public DietController(DietDAO dietDao, PatientDAO patientDao,
                          AlimentDAO alimentDao, DietView view) {
        this(dietDao, new DietDetailDAOJPA(), patientDao, alimentDao, view);
    }

    public DietController(DietDAO dietDao, DietDetailDAO dietDetailDao,
                          PatientDAO patientDao, AlimentDAO alimentDao, DietView view) {
        this.dietDao = dietDao;
        this.dietDetailDao = dietDetailDao;
        this.patientDao = patientDao;
        this.alimentDao = alimentDao;
        this.view = view;

        view.getTableDiets().setModel(tableModel);
        view.getTableDietAliments().setModel(dietAlimentsModel);   // reusa AlimentTableModel
        refreshTable();
        refreshDietAliments();      // deja el panel de alimentos vacio y deshabilitado

        view.getButtonInsert().addActionListener(e -> insert());
        view.getButtonModify().addActionListener(e -> modify());
        view.getButtonDelete().addActionListener(e -> delete());
        view.getButtonSearch().addActionListener(e -> search());
        view.getButtonAddAliment().addActionListener(e -> addAliment());
        view.getButtonRemoveAliment().addActionListener(e -> removeAliment());

        // Dieta seleccionada -> se cargan sus datos y sus alimentos
        view.getTableDiets().getSelectionModel().addListSelectionListener(e -> {
            if (!e.getValueIsAdjusting()) {
                selectRow();
            }
        });

        // Las dos navegaciones de §4.1 quedan igual, pero creando el controlador con el DAO de detalle
        view.getButtonPatients().addActionListener(e -> {
            view.dispose();
            javax.swing.SwingUtilities.invokeLater(() -> {
                PatientView vistaPatient = new PatientView();
                new PatientController(patientDao, dietDao, alimentDao, vistaPatient);
                vistaPatient.setVisible(true);
            });
        });

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

Imports que agrega — **[reemplaza]** el bloque de §4.1:

```java
import appNutritionist.dao.DietDetailDAO;          // <-- nuevo (interfaz)
import appNutritionist.dao.DietDetailDAOJPA;       // <-- nuevo (implementacion)
import appNutritionist.model.Aliment;              // <-- nuevo
import appNutritionist.model.DietDetail;           // <-- nuevo
import appNutritionist.model.Patient;              // <-- nuevo (contarPacientesConDieta)
import java.util.ArrayList;                        // <-- nuevo
```

> [!tip] Por qué el constructor sobrecargado y no cambiar los otros dos controladores
> La alternativa "pura" es agregar `DietDetailDAO` a los constructores de `PatientController` y `AlimentController`, que lo único que harían es guardarlo para pasarlo al de dietas. Eso son **7 puntos a tocar** en los mismos dos archivos que ya son el punto de conflicto con las búsquedas de la Fase 5. Con el constructor sobrecargado, esos dos archivos no se tocan: sólo se les agrega el listener del botón Diets (§5.2).
>
> Costo: `DietController` crea su propio `DietDetailDAOJPA()` en ese camino. Es la única excepción a "los DAO se crean una vez en `AppNutritionist`", y está aislada en una línea con comentario.

### 4 bis.3 Cargar los alimentos de la dieta

```java
    // Recarga la tabla de abajo, el combo y el estado del panel.
    private void refreshDietAliments() {
        List<Aliment> delDieta = new ArrayList<>();
        dietDetails.clear();

        boolean hayDietaGuardada = selectedDiet.getId() != 0;
        if (hayDietaGuardada) {
            try {
                dietDetails.addAll(dietDetailDao.buscarPorDieta(selectedDiet.getId()));
                for (DietDetail dd : dietDetails) {
                    delDieta.add(dd.getAlimento());     // ojo: el getter es getAlimento()
                }
            } catch (DAOException ex) {
                view.showError(ex.getMessage());
            }
        }

        dietAlimentsModel.setAliments(delDieta);
        view.setAlimentsPanelEnabled(hayDietaGuardada);
        view.setDietAlimentsTitle(hayDietaGuardada
                ? "Aliments of the diet : " + selectedDiet.getName() + " (" + dietDetails.size() + ")"
                : "Aliments of the diet");
        loadAvailableAliments();
    }

    // El combo solo ofrece los alimentos que TODAVIA no estan en esta dieta.
    private void loadAvailableAliments() {
        availableAliments.clear();
        List<String> nombres = new ArrayList<>();

        if (selectedDiet.getId() != 0) {
            try {
                for (Aliment a : alimentDao.findAll()) {
                    if (!contieneAlimento(a)) {
                        availableAliments.add(a);
                        nombres.add(String.valueOf(a));     // "Manzana (Frutas)"
                    }
                }
            } catch (DAOException ex) {
                view.showError(ex.getMessage());
            }
        }
        view.setAlimentOptions(nombres);
    }

    private boolean contieneAlimento(Aliment a) {
        for (DietDetail dd : dietDetails) {
            if (dd.getAlimento().getId() == a.getId()) {
                return true;
            }
        }
        return false;
    }
```

> [!warning] Se compara por `id`, no por nombre
> Dos alimentos distintos pueden llamarse parecido, y `Aliment` no tiene `equals`/`hashCode`, así que `contains()` compara por identidad de objeto (y falla siempre, porque vienen de otro `EntityManager`). Comparar `getId()` es lo único que funciona.

### 4 bis.4 Agregar y quitar

```java
    private void addAliment() {
        if (selectedDiet.getId() == 0) {
            view.showError("Selecciona una dieta guardada antes de agregarle alimentos.");
            return;
        }
        int index = view.getAlimentIndex();
        if (index < 0 || index >= availableAliments.size()) {
            view.showError("No quedan alimentos para agregar a esta dieta.");
            return;
        }

        Aliment a = availableAliments.get(index);
        try {
            // El pivot nuevo: (dieta, alimento)
            dietDetailDao.insert(new DietDetail(selectedDiet, a));
            refreshDietAliments();
            view.showInfo(a.getAliment() + " agregado a " + selectedDiet.getName() + ".");
        } catch (DAOException ex) {
            view.showError("No se pudo agregar el alimento.\n" + ex.getMessage());
            refreshDietAliments();      // deja la pantalla coherente aunque haya fallado
        }
    }

    private void removeAliment() {
        if (selectedDiet.getId() == 0) {
            view.showError("Selecciona una dieta primero.");
            return;
        }
        int viewRow = view.getTableDietAliments().getSelectedRow();
        if (viewRow < 0) {
            view.showError("Selecciona un alimento de la tabla para quitarlo de la dieta.");
            return;
        }

        // La tabla muestra alimentos, pero el DAO borra DietDetail: convertimos el indice.
        int modelRow = view.getTableDietAliments().convertRowIndexToModel(viewRow);
        if (modelRow < 0 || modelRow >= dietDetails.size()) {
            return;
        }
        DietDetail dd = dietDetails.get(modelRow);

        if (!view.confirmRemoveAliment(dd.getAlimento())) return;

        try {
            dietDetailDao.delete(dd.getId());       // borra la RELACION, no el alimento
            refreshDietAliments();
            view.showInfo(dd.getAlimento().getAliment() + " ya no forma parte de la dieta.");
        } catch (DAOException ex) {
            view.showError("No se pudo quitar el alimento.\n" + ex.getMessage());
            refreshDietAliments();
        }
    }
```

> [!note] `new DietDetail(selectedDiet, a)` con entidades "detachadas"
> `selectedDiet` salió de un `EntityManager` que ya se cerró, y `a` también. No es problema: `DietDetail` no tiene `cascade` y EclipseLink sólo necesita los `id` para escribir `id_diet` e `id_aliment`. Si algún día ves *"Object is not a valid entity"*, la salida es `dietDao.findById(selectedDiet.getId())` antes de armar el pivot.

### 4 bis.5 `selectRow`, `insert`, `modify` y `resetFields` — **[reemplaza]** los de §4.2 y §4.3

```java
    private void selectRow() {
        int viewRow = view.getTableDiets().getSelectedRow();
        if (viewRow < 0) return;
        int modelRow = view.getTableDiets().convertRowIndexToModel(viewRow);
        selectedDiet = tableModel.getDiet(modelRow);
        showDiet(selectedDiet);
        refreshDietAliments();          // <-- lo unico que se agrega al de §4.3
    }

    private void resetFields() {
        selectedDiet = new Diet();
        view.clearForm();
        view.getTableDiets().clearSelection();
        refreshDietAliments();          // deja la tabla de abajo vacia y el panel gris
    }

    // Deja seleccionada la dieta con ese id (despues de un alta o de una modificacion).
    private void selectDietEnTabla(int id) {
        for (int i = 0; i < tableModel.getRowCount(); i++) {
            if (tableModel.getDiet(i).getId() == id) {
                int viewRow = view.getTableDiets().convertRowIndexToView(i);
                view.getTableDiets().setRowSelectionInterval(viewRow, viewRow);
                return;
            }
        }
    }
```

Y las dos altas/bajas cambian sólo el final:

```java
    // insert(): despues de dietDao.insert(d)
    dietDao.insert(d);
    refreshTable();
    selectDietEnTabla(d.getId());       // la deja lista para agregarle alimentos
    view.showInfo("Dieta guardada con id " + d.getId() + ". Ya podes agregarle alimentos.");

    // modify(): despues de dietDao.update(d)
    selectedDiet = dietDao.update(d);
    refreshTable();
    selectDietEnTabla(selectedDiet.getId());
    view.showInfo("Dieta actualizada.");
```

> [!tip] Seleccionar la dieta nueva es lo que hace que el flujo cierre
> Sin esto, después de guardar la dieta el panel queda gris y el usuario tiene que adivinar que primero hay que hacer clic en la tabla. Con esto: **Insert → la dieta queda seleccionada → el panel se habilita → agregás alimentos**. Un camino, sin instrucciones.

### 4 bis.6 `delete()` con los alimentos asociados — **[reemplaza]** el de §4.3

`fk_detail_diet` **no** tiene `ON DELETE CASCADE`: si la dieta tiene filas en `DietsDetails`, MySQL rechaza el `DELETE` de la dieta. Hay dos salidas; la buena es borrar primero las relaciones, con una confirmación que lo diga.

```java
    private void delete() {
        if (selectedDiet.getId() == 0) {
            view.showError("Selecciona una dieta de la tabla primero.");
            return;
        }

        int id = selectedDiet.getId();

        // 1) Guarda previa: si hay pacientes asignados no se intenta nada.
        //    Sin esto, la cascada borraria los DietDetail y despues fallaria la dieta:
        //    la dieta quedaria viva pero sin alimentos.
        int pacientes = contarPacientesConDieta(id);
        if (pacientes > 0) {
            view.showError("No se puede eliminar \"" + selectedDiet.getName() + "\":\n"
                    + pacientes + " paciente(s) la tienen asignada.\n"
                    + "Cambiales la dieta primero.");
            return;
        }

        int cantidad = dietDetails.size();          // los que se cargaron al seleccionarla
        if (!view.confirmDeleteWithAliments(selectedDiet.getName(), cantidad)) return;

        try {
            // 2) primero las relaciones (hijos), si no la FK bloquea el borrado de la dieta
            for (DietDetail dd : new ArrayList<>(dietDetails)) {
                dietDetailDao.delete(dd.getId());
            }
            // 3) despues la dieta
            dietDao.delete(id);
            refreshTable();
            resetFields();
            view.showInfo(cantidad == 0
                    ? "Dieta eliminada."
                    : "Dieta eliminada junto con sus " + cantidad + " alimentos asociados.");
        } catch (DAOException ex) {
            view.showError("No se pudo eliminar la dieta.\n" + ex.getMessage());
            refreshTable();
            refreshDietAliments();
        }
    }

    // Cuenta los pacientes que tienen asignada esa dieta (pueden ser 0).
    private int contarPacientesConDieta(int idDieta) {
        int total = 0;
        try {
            for (Patient p : patientDao.findAll()) {
                if (p.getDieta() != null && p.getDieta().getId() == idDieta) {
                    total++;
                }
            }
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
        return total;
    }
```

> [!important] Por qué se pregunta por los pacientes **antes** de borrar
> `fk_paciente_dieta` tampoco tiene `ON DELETE CASCADE`. Si se borraran primero los detalles y después se intentara la dieta, MySQL rechazaría el `DELETE` por los pacientes y la dieta quedaría **viva pero sin alimentos**, sin que nadie lo haya pedido. La guarda convierte ese desastre silencioso en un mensaje claro, y de paso ahorra el viaje a la base.

Y el método que va en la **vista** (`DietView`, después de `confirmDelete()`):

```java
public boolean confirmDeleteWithAliments(String nombreDieta, int cantidad) {
    String mensaje = cantidad == 0
            ? "¿Desea dar de baja la dieta " + nombreDieta + "?"
            : "La dieta " + nombreDieta + " tiene " + cantidad + " alimento(s) asociado(s).\n"
              + "Se van a quitar de la dieta. Los alimentos siguen existiendo en el sistema.\n\n"
              + "¿Desea continuar?";
    return javax.swing.JOptionPane.showConfirmDialog(this, mensaje, "Confirmar",
            javax.swing.JOptionPane.OK_CANCEL_OPTION,
            javax.swing.JOptionPane.WARNING_MESSAGE) == javax.swing.JOptionPane.OK_OPTION;
}
```

> [!warning] La cascada no es una transacción
> Cada `delete()` abre y cierra su propio `EntityManager` (§DAOJpa). La guarda de pacientes cubre el caso real (dieta en uso), pero si la base se cae **en el medio** de la cascada —borró dos detalles de cinco y falla el tercero— la dieta queda con dos alimentos menos y **no** hay rollback. El `catch` refresca la pantalla para que se vea el estado real. Una transacción de verdad requiere un método en el DAO que reciba la dieta y borre todo con un solo `EntityManager`.

### 4 bis.7 Los dos getters del panel

Ya están en el bloque de §2 bis.9. Si en tu copia de `DietView` faltan, son estos:

```java
public javax.swing.JButton getButtonAddAliment()    {return addAlimentButton;}
public javax.swing.JButton getButtonRemoveAliment() {return removeAlimentButton;}
```

### 4 bis.8 `buscarPorDieta` en el DAO

`src/appNutritionist/dao/DietDetailDAOJPA.java:17` — hoy tira `UnsupportedOperationException`. Es la única consulta que falta en todo el proyecto:

```java
    @Override
    public List<DietDetail> buscarPorDieta(Integer idDiet) throws DAOException {
        return consultar("SELECT dd FROM DietDetail dd WHERE dd.diet.id = :idDiet",
                         "idDiet", idDiet);
    }
```

> [!warning] `dd.diet.id`, no `dd.dieta.id`
> JPQL usa **nombres de campo**. El campo es `private Diet diet;` aunque el getter se llame `getDieta()`. Si escribís `dd.dieta.id`, EclipseLink responde *"The state field path 'dd.dieta.id' cannot be resolved to a valid type."*

Verificación rápida (con la base cargada):

```powershell
Select-String -Path "src\appNutritionist\dao\DietDetailDAOJPA.java" -Pattern 'UnsupportedOperationException'
```

No tiene que devolver nada.

### 4 bis.9 Pruebas del panel

Se corren después de las 21 de §7, con la base y los datos de ejemplo cargados.

| # | Acción | Resultado esperado |
|---|---|---|
| 1 | Abrir **Diets** sin seleccionar nada | la tabla de abajo vacía, el combo vacío, Add/Remove grises |
| 2 | Clic en `Hipocalórica` | título `Aliments of the diet : Hipocalórica (3)` y **Manzana, Pechuga de pollo, Lechuga** |
| 3 | Clic en `Vegetariana` | título con `(3)` y **Banana, Arroz integral, Palta** |
| 4 | Mirar el combo con `Hipocalórica` | **4** alimentos: los que no están (Banana, Arroz integral, Yogur, Palta) |
| 5 | Agregar `Banana` → Add | aparece en la tabla, el título pasa a `(4)` y Banana **desaparece del combo** |
| 6 | Volver a `Vegetariana` y a `Hipocalórica` | la tabla sigue con 4: se guardó |
| 7 | **Cerrar y reabrir la app** | sigue con 4 (está en MySQL) |
| 8 | Seleccionar `Lechuga` → Remove → Cancelar | no borra nada |
| 9 | Seleccionar `Lechuga` → Remove → Aceptar | el mensaje dice *"ya no forma parte de la dieta"*, la fila desaparece y Lechuga **vuelve al combo** |
| 10 | Remove sin fila seleccionada | mensaje, sin excepción |
| 11 | Ordenar la tabla de abajo por *Name* y quitar la primera fila | se quita **esa**, no otra (`convertRowIndexToModel`) |
| 12 | Insert de una dieta nueva | queda **seleccionada**, el panel se habilita y el título dice `(0)` |
| 13 | Agregar 2 alimentos a la dieta nueva | funciona sin haber tocado nada más |
| 14 | Delete de la dieta nueva | el diálogo avisa *"tiene 2 alimento(s) asociado(s)… los alimentos siguen existiendo"*; borra la dieta y sus dos relaciones |
| 15 | Delete de `Hipocalórica` (la tiene Martín) | mensaje *"No se puede eliminar… 1 paciente(s) la tienen asignada… Cambiales la dieta primero"*, **sin crash**, la dieta sigue con sus **3** alimentos |
| 16 | Workbench: `SELECT * FROM DietsDetails;` | coincide exactamente con lo que muestra la pantalla |

> [!important] Las pruebas 7 y 16 son las que valen
> La 7 demuestra que el pivot se persistió, no que se pintó en una tabla. La 16 demuestra que la app y MySQL cuentan la misma historia — que es exactamente lo que va a mirar el profesor.

### 4 bis.10 Si algo falla

| Mensaje | Causa | Solución |
|---|---|---|
| `UnsupportedOperationException` al seleccionar una dieta | `buscarPorDieta` sin implementar | §4 bis.8 |
| `Duplicate entry ... for key 'UNQ_DietsDetails_0'` | el alimento ya está en esa dieta | el combo filtrado lo evita; si pasa, es que no se recargó (`refreshDietAliments`) |
| `Cannot add or update a child row: a foreign key constraint fails` | `id_diet` o `id_aliment` inexistente (dieta sin guardar) | §4 bis.4, el guard de `getId() == 0` |
| `The state field path 'dd.dieta.id' cannot be resolved` | JPQL con el nombre del getter | `dd.diet.id` |
| `Cannot delete or update a parent row` al borrar la dieta | quedaron detalles o hay pacientes asignados | §4 bis.6: primero los detalles; pacientes es un mensaje |
| La tabla de abajo no se llena | `getTableDietAliments().setModel(...)` olvidado | §4 bis.2 |
| El alimento agregado sigue en el combo | falta `loadAvailableAliments()` después del insert | está dentro de `refreshDietAliments()` |
| Se quita el alimento equivocado, sólo con la tabla ordenada | falta `convertRowIndexToModel` | §4 bis.4 |
| El panel queda habilitado después de limpiar | `resetFields()` no llama a `refreshDietAliments()` | §4 bis.5 |
| `Stack trace` al borrar con la base caída | `DAOJpa.delete()` crea el `EntityManager` fuera del `try` | deuda compartida, ver §12 |

### 4 bis.11 Trampas del panel

| # | Trampa | Qué pasa |
|---|---|---|
| 1 | Usar el índice de la **vista** para indexar `dietDetails` | con la tabla ordenada se borra otro alimento |
| 2 | Comparar alimentos con `equals`/`contains` | `Aliment` no define `equals`: compara identidad de objeto y nunca coincide |
| 3 | No volver a llenar el combo después de agregar | el alimento se puede agregar dos veces y choca con el `UNIQUE` |
| 4 | Borrar la dieta antes que sus `DietDetail` | `fk_detail_diet` lo rechaza |
| 5 | Creer que la cascada es atómica | no hay transacción entre DAOs: puede quedar a medias |
| 6 | Habilitar el panel con una dieta sin guardar | `id_diet` no existe: falla el `INSERT` del pivot |
| 7 | Poner el `List<DietDetail>` como `static` | se comparte entre ventanas y explota al navegar |
| 8 | Cargar los alimentos con `dietDao.findAll()` | las dietas no traen sus detalles: hay que preguntar por dieta |

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
| 2 | Clic en una fila | el campo Description se llena con el texto completo del script (aunque no esté en la tabla) |
| 3 | Insert con nombre y descripción | mensaje con el id asignado, fila nueva, formulario limpio |
| 4 | **Cerrar y reabrir la app** | la dieta nueva **sigue** ahí |
| 5 | Insert sin nombre | "El nombre de la dieta es obligatorio", no guarda |
| 6 | Insert con nombre de 51 caracteres | "El nombre no puede superar los 50 caracteres", no guarda |
| 7 | Insert con descripción de 101 caracteres | "La descripcion no puede superar los 100 caracteres", no guarda |
| 8 | Insert sin descripción (vacía) | guarda igual, el campo Description queda vacío |
| 9 | Clic en una fila | se cargan los 3 campos |
| 10 | Ordenar por *Name* y hacer clic | carga la dieta correcta (prueba `convertRowIndexToModel`) |
| 11 | Modify del nombre a `Hipocalorica estricta` | mensaje de actualizado; al reabrir, el nombre nuevo |
| 12 | Modify de la descripción | persiste |
| 13 | Delete → Cancelar | no borra |
| 14 | Delete de `Vegetariana` (id 2, **la tiene Lucía y tiene 3 alimentos**) | mensaje *"No se puede eliminar… 1 paciente(s) la tienen asignada"*, **sin crash**, la fila sigue **con sus 3 alimentos** |
| 15 | Insert de una dieta nueva sin asignar → Delete | borra sin problema |
| 16 | Diets → Patients | el combo de dietas muestra la dieta nueva y ya no la borrada |
| 17 | Asignarle la dieta nueva a un paciente con **Modify** | persiste al reabrir |
| 18 | Patients → Diets → Patients → Diets | los datos siguen (los DAO se comparten, no se recrean) |
| 19 | Search con `hipo` | queda sólo `Hipocalórica` |
| 20 | Search con `zzz` (inexistente) | "No hay dietas que contengan…" |
| 21 | Search con el campo vacío | vuelve a mostrar todas |

> [!note] La prueba 14 es la importante
> Es la única que ejercita las dos claves foráneas a la vez. `Vegetariana` (id 2) está referenciada desde `Patients.id_diet` (Lucía) y desde `DietsDetails.id_diet` (3 filas). La app tiene que **detectarlo antes de borrar** y avisar; lo que no puede pasar es que borre los detalles, que MySQL rechace la dieta y quede una dieta sin alimentos y sin mensaje. Si el mensaje no aparece y en su lugar sale un stack trace, la guarda `contarPacientesConDieta` (§4 bis.6) no está puesta.

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
| 11 | Editar `initComponents()` a mano | el próximo guardado del form en el diseñador te borra el cambio, sin aviso |
| 12 | Commitear `DietView.java` sin `DietView.form` (o al revés) | el par queda desincronizado y el diseñador reescribe el `.java` al abrirlo |
| 13 | Dos personas tocando `DietView.form` a la vez | conflicto de XML generado: no se resuelve solo |
| 14 | Dejar `dietsTable` con las columnas `Aliment 1` / `Aliment 2` | no puede representar N alimentos y duplica lo que ya muestra `dietAlimentsTable` |
| 15 | Dejar `descriptionTextArea` **sin `lineWrap`** | el texto no baja de renglón: se corre y se esconde a la izquierda, como pasaba con `observations`. Es una propiedad del diseñador, no código |
| 16 | Escribir `dd.dieta.id` en el JPQL del detalle | JPQL usa **campos**, no getters: es `dd.diet.id`. Si no, *"The state field path cannot be resolved"* |
| 17 | Borrar los `DietDetail` sin mirar si hay pacientes | la cascada funciona, la dieta no se puede borrar, y queda **sin alimentos**: pérdida silenciosa. La guarda de §4 bis.6 lo evita |
| 18 | Habilitar Add/Remove con una dieta sin `id` | el `INSERT` del pivot manda `id_diet` nulo: *"foreign key constraint fails"* |
| 19 | Comparar alimentos con `contains()` o `equals()` | `Aliment` no define `equals`: compara identidad y nunca coincide, el combo ofrece alimentos que ya están |
| 20 | Poner el panel de alimentos con `enabled` en el `JPanel` | el `JPanel` deshabilitado no deshabilita a sus hijos (§2 bis.7) |
| 21 | Dejar `Description` como columna de `dietsTable` | la celda es de una línea: el texto se recorta, y si tiene `\n` sale un cuadradito. El texto largo va en el formulario (§2 bis.1) |
| 22 | Guardar el texto de la textarea sin normalizar | un `Enter` entra como `\n` en el `VARCHAR(100)`. `readForm()` lo convierte en espacio con `replaceAll("\\s+", " ")` (§4.3) |

> [!tip] Las trampas 11 a 13 son las únicas que pueden costar horas
> Las otras se arreglan leyendo el error. Un `initComponents()` pisado o un `.form` mergeado a mano, no: se pierde trabajo sin que nadie se entere.
>
> La **17** es la más peligrosa de las nuevas porque no muestra error: la app dice *"No se pudo eliminar la dieta"* y parece que no pasó nada, pero los alimentos ya no están en la dieta.

---

## 9. Si algo falla

| Mensaje | Causa | Solución |
|---|---|---|
| `The abstract schema type 'diets' is unknown` | JPQL sin arreglar | Fase 5 |
| `Cannot delete or update a parent row: a foreign key constraint fails` | la dieta tiene pacientes o alimentos | con la guarda de §4 bis.6 no debería llegar acá; si llega, hay un detalle sin recargar |
| `Data too long for column 'name'` | falta la validación de 50 | Fase 4.2, `validar()` |
| `Field 'name' doesn't have a default value` | se intentó insertar con el nombre vacío | ídem |
| `Duplicate entry ... for key 'UNQ_DietsDetails_0'` | el alimento ya está en la dieta | el combo filtrado lo evita; si pasa, falta `refreshDietAliments()` |
| `UnsupportedOperationException` al seleccionar una dieta | `buscarPorDieta` sin implementar | §4 bis.8 |
| La tabla de abajo no se llena | falta `setModel` en el constructor | §4 bis.2 |
| Se quita un alimento distinto, sólo con la tabla ordenada | falta `convertRowIndexToModel` | §4 bis.4 |
| Se borró la dieta y quedaron sus alimentos | se borró al revés | §4 bis.6: primero los detalles |
| El panel sigue habilitado después de limpiar | `resetFields()` no llama a `refreshDietAliments()` | §4 bis.5 |
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
| `view/DietView.java` + `view/DietView.form` | la vista ajustada (§2 bis) |
| `view/DietTableModel.java` | el modelo de tabla |
| `controller/DietController.java` | el CRUD **y** el panel de alimentos (§4 y §4 bis) |
| `controller/PatientController.java`, `controller/AlimentController.java` | los listeners del botón Diets |
| `dao/DietDAOJPA.java` | el JPQL de `Diet` |
| `dao/DietDetailDAOJPA.java` | `buscarPorDieta` (§4 bis.8) |

### 10.2 Dos commits, no uno

```powershell
# 1) la pantalla y el CRUD, con el panel de alimentos
git add src/appNutritionist/view/DietView.java src/appNutritionist/view/DietView.form `
        src/appNutritionist/view/DietTableModel.java `
        src/appNutritionist/controller/DietController.java `
        src/appNutritionist/dao/DietDetailDAOJPA.java
git commit -m "Agregue la vista de dietas con su tabla, su controlador, el CRUD y los alimentos de cada dieta"

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

## 12. Fase 7 — los alimentos de la dieta

> [!important] Ya no es opcional (2026-11-12)
> Cuando se escribió esta sección, el detalle de la dieta era "el extra que levanta nota". **Cambió el alcance:** Caro diseñó `DietView` con una segunda tabla, un combo y los botones Add/Remove, así que el panel de alimentos entra en el trabajo. Lo que sigue sirve como referencia del modelo, del problema de diseño y de lo que hay que implementar del lado del DAO. **La vista está resuelta en §2 bis y el controlador en §4 bis**, incluida la implementación de `buscarPorDieta` (§4 bis.8).
>
> Sigue siendo opcional lo único que de verdad no se puede hacer: **la cantidad** (*"200 g de arroz"*), porque no existe la columna.

**Qué es:** la tabla pivot que relaciona dietas con alimentos. No es un CRUD aparte: se administra desde la misma pantalla de dietas.

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

### Cómo quedó cableado

El diseño cambió respecto de lo que se pensaba acá: **no** son dos tablas lado a lado, es **una tabla + un combo filtrado** en la misma pantalla de dietas.

| Pieza | Dónde está |
|---|---|
| Segunda tabla con los alimentos de la dieta (reutiliza `AlimentTableModel`) | §2 bis.1 y §4 bis.2 |
| Combo con los alimentos que todavía no están + Add/Remove | §2 bis.9 y §4 bis.3, §4 bis.4 |
| Título dinámico `Aliments of the diet : Hipocalórica (3)` | §2 bis.4 |
| `List<DietDetail>` en paralelo para poder borrar el pivot | §4 bis.1 |
| Manejo del `UNIQUE`: el combo filtrado lo evita por diseño | §4 bis.3 |
| Cascada manual al borrar la dieta | §4 bis.6 |
| `buscarPorDieta` implementado | §4 bis.8 |
| Las 16 pruebas | §4 bis.9 |

La consulta que faltaba era esta:

```java
@Override
public List<DietDetail> buscarPorDieta(Integer idDiet) throws DAOException {
    return consultar("SELECT dd FROM DietDetail dd WHERE dd.diet.id = :idDiet",
                     "idDiet", idDiet);
}
```

Lo único que sigue afuera es **la cantidad**: mientras no exista la columna, el detalle dice *qué* alimentos, no *cuánto*.

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
