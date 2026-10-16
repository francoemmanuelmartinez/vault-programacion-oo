---
tags: [tarea, swing, contingency, codigo]
estado: verificado-compila
aliases: [03-vistas-manuales]
---

# Vistas en Swing manual (contingencia)

> [!warning] Esto es una **contingencia**, no el estado actual
> Las vistas que hoy tiene el proyecto (`viewPatient.java`, `viewAliment.java`)
> son **código generado por el NetBeans GUI Builder**. Este note tiene las
> versiones **escritas a mano**, listas para reemplazar a las generadas, por si
> la rúbrica resulta que el designer no se permite.
>
> Decisión del Estudiante: probablemente el profesor lo permite, pero por las
> dudas dejamos el replacement preparado.
>
> **Verificar contra `enunciado_rubrica.pdf` antes de tocar nada.**

## Estado de verificación ✅

Ambos archivos compilan limpios con el mismo target del proyecto:

```text
javac --release 25 -Xlint:all -Werror   →   exit code 0
ViewPatient.class
ViewAliment.class
```

## Qué cambia respecto a las generadas

| | Generadas (NetBeans) | Manuales (estas) |
|---|---|---|
| Layout | `GroupLayout` | `GridBagLayout` + `BorderLayout` |
| Nombres de componente | `jTextField1`, `jLabel4` | `txtName`, `btnInsert` |
| Nombres de clase | `viewPatient` | `ViewPatient` |
| Tamaño de ventana | `pack()` sobre GroupLayout | `pack()` + `setLocationRelativeTo(null)` |
| Código muerto | `main()` con Nimbus + `logger` | `main()` mínimo |
| Variables | `public` sueltas al final | `private final` agrupadas arriba |
| IMC | campo editable como cualquier otro | `setEditable(false)`, se calcula |

## ⚠️ Un bug que el compilador encontró

La primera versión de `ViewPatient` usaba `getHeight()` como getter del campo
"Height". **No compila**:

```text
error: getHeight() in ViewPatient cannot override getHeight() in Component
       return type double is not compatible with int
```

`java.awt.Component` ya define `getHeight()`, que devuelve `int` (píxeles).
Cualquier getter con ese nombre choca. Por eso quedó `getHeightMeters()`, y de
paso `getWeightKg()` para que las unidades sean explícitas.

Buen recordatorio de por qué compilar antes de entregar.

## Contingencia: `setVisible(false)` en vez de `dispose()`

> [!info] Estado
> **No es lo que hay implementado.** Es la alternativa, dejada documentada por
> si hay que cambiar de estrategia. Lo que hay hoy es `dispose()`.

La transición de ventanas se resolvió con `dispose()`: se cierra la vista actual
y se crea una instancia nueva del destino.

```java
view.getBtnAliment().addActionListener(e -> {
    // 1. Cierra y libera los recursos de la vista actual
    view.dispose();

    // 2. Inicializa la vista y el controlador de destino en el Event Dispatch Thread
    javax.swing.SwingUtilities.invokeLater(() -> {
        viewAliment vistaAliment = new viewAliment();
        new controllerAliment(new modelAliment(), vistaAliment);
        vistaAliment.setVisible(true);
    });
});
```

### Cuándo conviene la otra

Si el Estudiante necesita que el formulario conserve lo que el usuario escribió
al cambiar de ventana, `dispose()` no sirve: destruye los componentes y el texto
se pierde. Ahí la alternativa es **ocultar y reutilizar** las mismas instancias,
guardando las referencias en algún lado:

```java
// En el punto donde se cablea la aplicación, no en cada click:
private viewPatient vistaPatient;
private viewAliment vistaAliment;
private controllerPatient controlPatient;
private controllerAliment controlAliment;

// Ir a la otra ventana:
private void irAAliment() {
    vistaPatient.setVisible(false);
    if (vistaAliment == null) {
        vistaAliment = new viewAliment();
        controlAliment = new controllerAliment(new modelAliment(), vistaAliment);
    }
    vistaAliment.setVisible(true);
}
```

### Diferencias

| | `dispose()` (lo actual) | `setVisible(false)` |
|---|---|---|
| Estado del formulario | Se pierde | Se conserva |
| Memoria | Libera los componentes | Las dos vistas quedan vivas |
| Ciclo de vida | Una instancia por cada visita | Una sola instancia por vista |
| Quién guarda las referencias | Nadie, se crean y se descartan | Alguien tiene que guardarlas |

El punto de partida es la segunda columna de la tabla de [[estado]]: el
proyecto tiene **cero** lógica de negocio, así que todavía no hay nada que
preserve. `dispose()` es la opción simple y no obliga a mantener estado global
de ventanas. Recién cuando los modelos tengan datos va a importar.

> [!warning] Ampliación de la IA, no del profesor
> Esta sección compara dos estrategias de ciclo de vida de Swing. No está en los
> PDFs de la cátedra: sale del comportamiento documentado de `Window`.

## Cómo aplicar la contingencia

1. Leer la rúbrica y confirmar si el designer está permitido.
2. Si no está permitido: crear `Views/ViewPatient.java` y `Views/ViewAliment.java`
   con el código de abajo.
3. Renombrar los imports en `AppNutritionist.java`, `Controller/controllerPatient.java`
   y `Controller/controllerAliment.java`.
4. Borrar los archivos generados.
5. `git commit` antes de empezar, para poder volver atrás.

> Ojo: adoptar `ViewPatient` en PascalCase implica **renombrar también**
> `viewPatient`, `viewAliment`, `modelPatient`, `modelAliment`,
> `controllerPatient` y `controllerAliment`, más todos los imports. Es el mismo
> problema del riesgo 1 de [[estado]], así que conviene resolver los dos juntos.

---

## `ViewPatient.java`

```java
package appNutritionist.Views;

import java.awt.BorderLayout;
import java.awt.GridBagConstraints;
import java.awt.GridBagLayout;
import java.awt.Insets;
import javax.swing.BorderFactory;
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;
import javax.swing.JTextField;

/**
 * Vista de Patient armada a mano, sin NetBeans GUI Builder.
 * Equivalente funcional de la ventana "Patients" del proyecto.
 *
 * Nota: los getters no pueden llamarse getHeight() porque colisionan con
 * Component.getHeight(), que devuelve int (pixeles). Por eso se usan
 * getWeightKg() y getHeightMeters().
 */
public final class ViewPatient extends JFrame {

    private static final long serialVersionUID = 1L;

    private final JTextField txtId = new JTextField(10);
    private final JTextField txtName = new JTextField(10);
    private final JTextField txtLastName = new JTextField(10);
    private final JTextField txtWeight = new JTextField(10);
    private final JTextField txtHeight = new JTextField(10);
    private final JTextField txtImc = new JTextField(10);
    private final JTextField txtObservations = new JTextField(20);

    private final JButton btnSearch = new JButton("Search");
    private final JButton btnInsert = new JButton("Insert");
    private final JButton btnModify = new JButton("Modify");
    private final JButton btnDelete = new JButton("Delete");
    private final JButton btnAliment = new JButton("Aliment");
    private final JButton btnDiets = new JButton("Diets");

    public ViewPatient() {
        super("Patients");
        initComponents();
    }

    private void initComponents() {
        // El IMC es calculado por el controller: el usuario no lo escribe.
        txtImc.setEditable(false);

        JPanel header = new JPanel(new BorderLayout());
        header.add(new JLabel("Patients"), BorderLayout.WEST);
        JPanel nav = new JPanel();
        nav.add(btnAliment);
        nav.add(btnDiets);
        header.add(nav, BorderLayout.EAST);

        JPanel form = new JPanel(new GridBagLayout());
        addRow(form, 0, "Patient's Id", txtId, btnSearch);
        addRow(form, 1, "Name", txtName, btnInsert);
        addRow(form, 2, "Last Name", txtLastName, btnModify);
        addRow(form, 3, "Weight (kg)", txtWeight, btnDelete);
        addRow(form, 4, "Height (m)", txtHeight, null);
        addRow(form, 5, "IMC", txtImc, null);
        addRow(form, 6, "Observations", txtObservations, null);

        JPanel root = new JPanel(new BorderLayout(10, 10));
        root.setBorder(BorderFactory.createEmptyBorder(15, 15, 15, 15));
        root.add(header, BorderLayout.NORTH);
        root.add(form, BorderLayout.CENTER);

        setContentPane(root);
        pack();
        setLocationRelativeTo(null);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }

    private void addRow(JPanel panel, int row, String label, JTextField field, JButton button) {
        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(5, 5, 5, 5);
        c.anchor = GridBagConstraints.WEST;

        c.gridx = 0;
        c.gridy = row;
        panel.add(new JLabel(label), c);

        c.gridx = 1;
        c.fill = GridBagConstraints.HORIZONTAL;
        c.weightx = 1.0;
        panel.add(field, c);

        c.gridx = 2;
        c.fill = GridBagConstraints.NONE;
        c.weightx = 0.0;
        if (button != null) {
            panel.add(button, c);
        }
    }

    // --- Getters que usa el controller ---

    public String getId() {
        return txtId.getText();
    }

    public String getName() {
        return txtName.getText();
    }

    public String getLastName() {
        return txtLastName.getText();
    }

    public double getWeightKg() {
        return Double.parseDouble(txtWeight.getText().trim());
    }

    public double getHeightMeters() {
        return Double.parseDouble(txtHeight.getText().trim());
    }

    public String getObservations() {
        return txtObservations.getText();
    }

    public void setImc(double imc) {
        txtImc.setText(String.valueOf(imc));
    }

    public void setId(String id) {
        txtId.setText(id);
    }

    public void clearForm() {
        txtId.setText("");
        txtName.setText("");
        txtLastName.setText("");
        txtWeight.setText("");
        txtHeight.setText("");
        txtImc.setText("");
        txtObservations.setText("");
    }

    /** Calcula el IMC. Peso en kg, altura en m. */
    public void calculateImc() {
        double weight = getWeightKg();
        double height = getHeightMeters();
        setImc(Math.round(weight / Math.pow(height, 2)));
    }

    public JButton getBtnInsert() {
        return btnInsert;
    }

    public JButton getBtnModify() {
        return btnModify;
    }

    public JButton getBtnDelete() {
        return btnDelete;
    }

    public JButton getBtnSearch() {
        return btnSearch;
    }

    public JButton getBtnAliment() {
        return btnAliment;
    }

    public JButton getBtnDiets() {
        return btnDiets;
    }

    public static void main(String[] args) {
        javax.swing.SwingUtilities.invokeLater(() -> new ViewPatient().setVisible(true));
    }
}
```

---

## `ViewAliment.java`

```java
package appNutritionist.Views;

import java.awt.BorderLayout;
import java.awt.GridBagConstraints;
import java.awt.GridBagLayout;
import java.awt.Insets;
import javax.swing.BorderFactory;
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JPanel;
import javax.swing.JTextField;

/**
 * Vista de Aliment armada a mano, sin NetBeans GUI Builder.
 * Equivalente funcional de la ventana "Aliment" del proyecto.
 */
public final class ViewAliment extends JFrame {

    private static final long serialVersionUID = 1L;

    private final JTextField txtName = new JTextField(15);
    private final JTextField txtCategory = new JTextField(15);
    private final JTextField txtCalories = new JTextField(15);

    private final JButton btnSearch = new JButton("Search");
    private final JButton btnInsert = new JButton("Insert");
    private final JButton btnModify = new JButton("Modify");
    private final JButton btnDelete = new JButton("Delete");
    private final JButton btnPatient = new JButton("Patient");
    private final JButton btnDiets = new JButton("Diets");

    public ViewAliment() {
        super("Aliment");
        initComponents();
    }

    private void initComponents() {
        JPanel header = new JPanel(new BorderLayout());
        header.add(new JLabel("Aliment"), BorderLayout.WEST);
        JPanel nav = new JPanel();
        nav.add(btnPatient);
        nav.add(btnDiets);
        header.add(nav, BorderLayout.EAST);

        JPanel form = new JPanel(new GridBagLayout());
        addRow(form, 0, "Aliment", txtName, btnSearch);
        addRow(form, 1, "Category", txtCategory, btnInsert);
        addRow(form, 2, "Calories per serving", txtCalories, btnModify);

        JPanel actions = new JPanel(new BorderLayout());
        actions.add(btnDelete, BorderLayout.WEST);

        JPanel root = new JPanel(new BorderLayout(10, 10));
        root.setBorder(BorderFactory.createEmptyBorder(15, 15, 15, 15));
        root.add(header, BorderLayout.NORTH);
        root.add(form, BorderLayout.CENTER);
        root.add(actions, BorderLayout.SOUTH);

        setContentPane(root);
        pack();
        setLocationRelativeTo(null);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }

    private void addRow(JPanel panel, int row, String label, JTextField field, JButton button) {
        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(5, 5, 5, 5);
        c.anchor = GridBagConstraints.WEST;

        c.gridx = 0;
        c.gridy = row;
        panel.add(new JLabel(label), c);

        c.gridx = 1;
        c.fill = GridBagConstraints.HORIZONTAL;
        c.weightx = 1.0;
        panel.add(field, c);

        c.gridx = 2;
        c.fill = GridBagConstraints.NONE;
        c.weightx = 0.0;
        if (button != null) {
            panel.add(button, c);
        }
    }

    // --- Getters que usa el controller ---

    public String getName() {
        return txtName.getText();
    }

    public String getCategory() {
        return txtCategory.getText();
    }

    public int getCalories() {
        return Integer.parseInt(txtCalories.getText().trim());
    }

    public void clearForm() {
        txtName.setText("");
        txtCategory.setText("");
        txtCalories.setText("");
    }

    public JButton getBtnInsert() {
        return btnInsert;
    }

    public JButton getBtnModify() {
        return btnModify;
    }

    public JButton getBtnDelete() {
        return btnDelete;
    }

    public JButton getBtnSearch() {
        return btnSearch;
    }

    public JButton getBtnPatient() {
        return btnPatient;
    }

    public JButton getBtnDiets() {
        return btnDiets;
    }

    public static void main(String[] args) {
        javax.swing.SwingUtilities.invokeLater(() -> new ViewAliment().setVisible(true));
    }
}
```

---

## Lo que estos getters le dejan al controller

`controllerPatient` queda así de corto, que es justo lo que busca
`Codigo_Limpio_Nombres_y_Estructura.pdf`: la vista no hace nada, solo expone
datos y eventos.

```java
view.getBtnInsert().addActionListener(e -> model.insert(leer()));
view.getBtnModify().addActionListener(e -> model.update(leer()));
view.getBtnDelete().addActionListener(e -> model.delete(view.getId()));
```

Los getters de texto (`getWeightKg()`, `getHeightMeters()`, `getCalories()`)
usan `Double.parseDouble` / `Integer.parseInt`, que **tuercen** si el campo está
vacío o tiene letras. Eso hay que envolverlo en `try/catch` y mostrar un
`JOptionPane.showMessageDialog`. Ver `Errores_y_Excepciones_Java.pdf`.

## Pendiente

- [ ] Confirmar con la rúbrica si el GUI Builder está permitido.
- [x] Navegación `Patient` ↔ `Aliment` con las vistas generadas. No depende de
      esta contingencia: funciona con las que hay hoy.
- [x] Los botones de navegación están **encapsulados**: los campos son privados y
      las vistas los exponen con `getBtnAliment()` y `getBtnPatient()`. El getter
      va fuera de las regiones `GEN`, así que NetBeans lo conserva al regenerar.
      Los getters van en el `.java` a mano; no hay forma de declararlos en el
      `.form` porque el designer solo controla los campos, no los métodos.
- [ ] Averiguar qué es "Diets": hay botón en las dos vistas pero **no existe**
      `viewDiet`, ni `modelDiet`, ni `controllerDiet`. Si la tarea lo pide, falta
      una entidad entera. Los botones quedaron **sin listener** a propósito.
- [ ] No hay ningún `JTable` en el proyecto, pero existe `JTable_en_Swing.pdf`.
      Casi seguro la tarea pide listar pacientes y alimentos en una tabla.
- [ ] Si se aplica esta contingencia (vistas a mano), `getBtnAliment()` y
      `getBtnPatient()` pasan a devolver los botones con nombre `btnAliment` y
      `btnPatient` en vez de `alimentBt` y `jButton1`. Los controladores no
      cambian.

## Relacionado

[[estado]] · [[enunciado_rubrica]]