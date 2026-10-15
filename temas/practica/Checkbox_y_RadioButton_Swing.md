---
fuente: Checkbox_y_RadioButton_Swing.pdf
tags: [tema, practica, swing, jcheckbox, jradiobutton, mvc]
estado: completo
---

# Checkbox y RadioButton en Swing

> [!info] Fuente
> `Checkbox_y_RadioButton_Swing.pdf` — pág. 1 a 24
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

> pág. 1 — Checkbox_y_RadioButton_Swing.pdf

`JCheckBox` permite seleccionar múltiples opciones independientes de tipo sí/no, mientras que `JRadioButton` (agrupados mediante `ButtonGroup`) modela elecciones mutuamente excluyentes (una sola opción entre varias). Ninguno de los dos utiliza un modelo de datos compartido como `ListModel`, sino que cada componente administra su propio estado booleano mediante `isSelected()`.

---

## Lo que dice el profesor

### 1. `JCheckBox`: Selección Múltiple

> pág. 3 a 5 — Checkbox_y_RadioButton_Swing.pdf

- **Definición:** Opción de tipo sí/no que el usuario puede marcar o desmarcar sin afectar a las demás (opciones no excluyentes).
- **Sin modelo compartido:** Cada `JCheckBox` administra su propio estado con `isSelected()`.
- **Eventos (`ItemListener`):** Se escuchan con `addItemListener(ItemEvent e)`. El método `e.getStateChange()` indica si se marcó (`ItemEvent.SELECTED`) o se desmarcó (`ItemEvent.DESELECTED`).
- **Código básico:**
  ```java
  JCheckBox cbJamon = new JCheckBox("Jamón");
  cbJamon.addItemListener(e -> {
      boolean marcado = e.getStateChange() == ItemEvent.SELECTED;
      System.out.println("Jamón: " + marcado);
  });
  panel.add(cbJamon);
  ```

### 2. `JRadioButton`: Selección Única y `ButtonGroup`

> pág. 6 a 8 — Checkbox_y_RadioButton_Swing.pdf

- **Definición:** Círculo de selección única (mutuamente excluyentes).
- **El rol de `ButtonGroup`:** Por sí solo, un `JRadioButton` se comporta como un checkbox. La exclusividad la aporta un objeto `ButtonGroup` que los agrupa lógicamente.
- **Importante:** `ButtonGroup` **no es un componente visual** (no se agrega al `JPanel`); solo administra el estado de exclusividad. La disposición visual sigue a cargo del `JPanel` con su *Layout Manager*.
- **Código básico:**
  ```java
  JRadioButton rbEfectivo = new JRadioButton("Efectivo");
  JRadioButton rbTarjeta = new JRadioButton("Tarjeta");
  ButtonGroup grupo = new ButtonGroup();
  grupo.add(rbEfectivo);
  grupo.add(rbTarjeta);
  
  rbEfectivo.setActionCommand("EFECTIVO");
  rbTarjeta.setActionCommand("TARJETA");
  
  // Leer la selección vigente
  String elegido = grupo.getSelection().getActionCommand();
  ```

---

### 3. Ejemplo MVC 1: Ingredientes de un pedido (`JCheckBox`)

> pág. 10 a 15 — Checkbox_y_RadioButton_Swing.pdf

Varios `JCheckBox` independientes calculan dinámicamente el precio total de extras.

- **1) El Modelo (`Ingrediente` y `PedidoModelo`):**
  ```java
  public class Ingrediente {
      private final String nombre;
      private final double precio;
      public Ingrediente(String nombre, double precio) {
          this.nombre = nombre; this.precio = precio;
      }
      public String getNombre() { return nombre; }
      public double getPrecio() { return precio; }
  }

  public class PedidoModelo {
      private final List<Ingrediente> disponibles = List.of(
          new Ingrediente("Muzzarella extra", 400),
          new Ingrediente("Jamón", 500),
          new Ingrediente("Panceta", 600)
      );
      private final Set<String> seleccionados = new HashSet<>();

      public List<Ingrediente> obtenerDisponibles() { return disponibles; }
      
      public void alternar(String nombre, boolean marcado) {
          if (marcado) seleccionados.add(nombre);
          else seleccionados.remove(nombre);
      }

      public double calcularTotalExtras() {
          return disponibles.stream()
                  .filter(i -> seleccionados.contains(i.getNombre()))
                  .mapToDouble(Ingrediente::getPrecio)
                  .sum();
      }
  }
  ```

- **2) La Vista (`PedidoVista`):**
  ```java
  public class PedidoVista extends JFrame {
      private final JPanel panel = new JPanel(new GridLayout(0, 1));
      private final JLabel labelTotal = new JLabel("Extras: $0");
      private final List<JCheckBox> checks = new ArrayList<>();

      public PedidoVista(List<Ingrediente> disponibles) {
          super("Ingredientes del pedido");
          for (Ingrediente ing : disponibles) {
              JCheckBox cb = new JCheckBox(ing.getNombre() + " (+$" + (int) ing.getPrecio() + ")");
              cb.setActionCommand(ing.getNombre());
              checks.add(cb);
              panel.add(cb);
          }
          setLayout(new BorderLayout(10, 10));
          add(panel, BorderLayout.CENTER);
          add(labelTotal, BorderLayout.SOUTH);
          setSize(340, 260);
          setLocationRelativeTo(null);
          setDefaultCloseOperation(EXIT_ON_CLOSE);
      }
      public List<JCheckBox> getChecks() { return checks; }
      public void mostrarTotal(double total) { labelTotal.setText("Extras: $" + (int) total); }
  }
  ```

- **3) El Controlador (`PedidoControlador`):**
  ```java
  public class PedidoControlador {
      private final PedidoModelo modelo;
      private final PedidoVista vista;

      public PedidoControlador(PedidoModelo modelo, PedidoVista vista) {
          this.modelo = modelo;
          this.vista = vista;
          for (JCheckBox cb : vista.getChecks()) {
              cb.addItemListener(e -> actualizarTotal(cb, cb.isSelected()));
          }
      }

      private void actualizarTotal(JCheckBox cb, boolean marcado) {
          modelo.alternar(cb.getActionCommand(), marcado);
          vista.mostrarTotal(modelo.calcularTotalExtras());
      }
  }
  ```

---

### 4. Ejemplo MVC 2: Método de pago (`JRadioButton`)

> pág. 16 a 21 — Checkbox_y_RadioButton_Swing.pdf

`JRadioButton` agrupados con `ButtonGroup` calculan recargos porcentuales sobre un monto base.

- **1) El Modelo (`MetodoPago` y `PagoModelo`):**
  ```java
  public class MetodoPago {
      private final String nombre;
      private final double recargoPorcentaje;
      public MetodoPago(String nombre, double recargoPorcentaje) {
          this.nombre = nombre; this.recargoPorcentaje = recargoPorcentaje;
      }
      public String getNombre() { return nombre; }
      public double getRecargoPorcentaje() { return recargoPorcentaje; }
  }

  public class PagoModelo {
      private final List<MetodoPago> metodos = List.of(
          new MetodoPago("Efectivo", 0),
          new MetodoPago("Crédito", 10),
          new MetodoPago("Transferencia", 0)
      );
      private final double montoBase = 10000;
      private MetodoPago seleccionado = metodos.get(0);

      public List<MetodoPago> obtenerMetodos() { return metodos; }
      
      public void seleccionar(String nombre) {
          metodos.stream().filter(m -> m.getNombre().equals(nombre)).findFirst().ifPresent(m -> seleccionado = m);
      }

      public double calcularTotal() {
          double recargo = seleccionado.getRecargoPorcentaje();
          return montoBase * (1 + recargo / 100);
      }
  }
  ```

- **2) La Vista (`PagoVista`):**
  ```java
  public class PagoVista extends JFrame {
      private final ButtonGroup grupo = new ButtonGroup();
      private final JPanel panel = new JPanel(new GridLayout(0, 1));
      private final List<JRadioButton> botones = new ArrayList<>();
      private final JLabel labelTotal = new JLabel();

      public PagoVista(List<MetodoPago> metodos) {
          super("Método de pago");
          for (MetodoPago m : metodos) {
              JRadioButton rb = new JRadioButton(m.getNombre() + " (+" + (int) m.getRecargoPorcentaje() + "%)");
              rb.setActionCommand(m.getNombre());
              grupo.add(rb);
              botones.add(rb);
              panel.add(rb);
          }
          botones.get(0).setSelected(true);
          setLayout(new BorderLayout(10, 10));
          add(panel, BorderLayout.CENTER);
          add(labelTotal, BorderLayout.SOUTH);
          setSize(340, 260);
          setLocationRelativeTo(null);
          setDefaultCloseOperation(EXIT_ON_CLOSE);
      }
      public ButtonGroup getGrupo() { return grupo; }
      public List<JRadioButton> getBotones() { return botones; }
      public void mostrarTotal(double total) { labelTotal.setText("Total a pagar: $" + (int) total); }
  }
  ```

- **3) El Controlador (`PagoControlador`):**
  ```java
  public class PagoControlador {
      private final PagoModelo modelo;
      private final PagoVista vista;

      public PagoControlador(PagoModelo modelo, PagoVista vista) {
          this.modelo = modelo;
          this.vista = vista;
          for (JRadioButton rb : vista.getBotones()) {
              rb.addActionListener(e -> actualizarTotal());
          }
          actualizarTotal();
      }

      private void actualizarTotal() {
          String elegido = vista.getGrupo().getSelection().getActionCommand();
          modelo.seleccionar(elegido);
          vista.mostrarTotal(modelo.calcularTotal());
      }
  }
  ```

---

## Ampliación de la IA

> [!warning] Buena práctica con ActionCommand
> Utilizá siempre `setActionCommand()` en checkboxes y radiobuttons para asignarles una clave estable (como el nombre del ítem). Si leés el texto visible con `getText()`, cualquier cambio estético en la etiqueta (ej. agregarle un precio o un porcentaje entre paréntesis) rompe la lógica del controlador.

---

## Dudas y vacíos

- [ ] ¿Cómo implementar validaciones complejas cuando se combinan múltiples checkboxes condicionales con un radio button?

---

## Relacionado

- [[Listas_y_ComboBox_Swing]] — Selección de opciones basadas en listas y modelos
- [[Swing_MVC_Interfaces_Java]] — Arquitectura en capas y manejo de eventos
- [[enunciado_rubrica]] — Criterios de evaluación de la interfaz gráfica
- [[00-estado]] — Estado actual de las vistas en `NutritionistApp`
- [[MAP]] — Mapa general del vault

---

**Navegación:** [[index]] · [[MAP]]
