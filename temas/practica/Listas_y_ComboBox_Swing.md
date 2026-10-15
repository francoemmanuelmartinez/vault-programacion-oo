---
fuente: Listas_y_ComboBox_Swing.pdf
tags: [tema, practica, swing, jlist, jcombobox, mvc]
estado: completo
---

# Listas y ComboBox en Swing (JList, JComboBox y Modelos)

> [!info] Fuente
> `Listas_y_ComboBox_Swing.pdf` — pág. 1 a 21
> **Profesor:** Esteban Santini — Islft 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

> pág. 1 — Listas_y_ComboBox_Swing.pdf

`JList` y `JComboBox` son los componentes estándar de Swing para presentar colecciones de opciones al usuario. Al igual que `JTable`, separan la visualización de los datos mediante modelos dedicados (`ListModel` / `DefaultListModel` y `ComboBoxModel` / `DefaultComboBoxModel`), facilitando la aplicación del patrón MVC.

---

## Lo que dice el profesor

### 1. `JList`: Listas de Selección

> pág. 3 a 5 — Listas_y_ComboBox_Swing.pdf

- **Definición:** Muestra una colección vertical de elementos de los que el usuario puede elegir uno o varios.
- **Parametrización:** Es un componente genérico (`JList<String>`, `JList<Alumno>`), lo que asegura el tipado en tiempo de compilación.
- **Scroll obligatorio:** Siempre se usa envuelto en un `JScrollPane`.
- **Modelos de datos:**
  - **Datos fijos (Array):** `new JList<>(array)` crea un modelo de solo lectura. Ideal para catálogos estáticos.
  - **Datos dinámicos (`DefaultListModel`):** Modelo mutable (`addElement()`, `removeElement()`, `clear()`). Cualquier cambio en el modelo repinta la lista automáticamente sin tocar el `JList`.

- **Código básico:**
  ```java
  String[] frutas = {"Manzana", "Banana", "Naranja"};
  JList<String> lista = new JList<>(frutas);
  lista.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
  lista.setVisibleRowCount(5);
  ventana.add(new JScrollPane(lista));
  String elegido = lista.getSelectedValue();
  ```

### 2. `JComboBox`: Listas Desplegables

> pág. 6 a 8 — Listas_y_ComboBox_Swing.pdf

- **Definición:** Combina un campo visible con una lista desplegable (una opción visible a la vez).
- **Parametrización:** `JComboBox<String>`. Puede ser de solo selección o editable (`setEditable(true)`).
- **Modelo mutable:** `DefaultComboBoxModel<String>` permite manipular opciones en tiempo de ejecución.
- **Código básico:**
  ```java
  JComboBox<String> combo = new JComboBox<>();
  combo.addItem("Frutas");
  combo.addItem("Bebidas");
  combo.setSelectedIndex(0);
  ventana.add(combo);
  String elegido = (String) combo.getSelectedItem();
  ```

### 3. Eventos: `JList` vs. `JComboBox`

> pág. 9 — Listas_y_ComboBox_Swing.pdf

- **`JList`:** Se escucha con `addListSelectionListener(ListSelectionEvent e)`. El método a implementar es `valueChanged()`. Conviene chequear `!e.getValueIsAdjusting()` para ignorar eventos intermedios mientras el usuario arrastra la selección.
- **`JComboBox`:** Se escucha con `addActionListener(ActionEvent e)` (igual que un `JButton`). Se dispara al confirmar una opción.

---

### 4. Ejemplo MVC Completo: Catálogo por Categoría

> pág. 11 a 17 — Listas_y_ComboBox_Swing.pdf

Un `JComboBox` de categorías filtra dinámicamente los productos que muestra un `JList`.

- **1) El Modelo (`CatalogoModelo`):**
  ```java
  public class CatalogoModelo {
      private final Map<String, List<String>> datos = new LinkedHashMap<>();

      public CatalogoModelo() {
          datos.put("Frutas", List.of("Manzana", "Banana", "Naranja", "Pera"));
          datos.put("Bebidas", List.of("Agua", "Jugo", "Gaseosa"));
          datos.put("Lácteos", List.of("Leche", "Yogur", "Queso"));
      }

      public List<String> obtenerCategorias() {
          return new ArrayList<>(datos.keySet());
      }

      public List<String> obtenerProductos(String cat) {
          return datos.getOrDefault(cat, List.of());
      }
  }
  ```

- **2) La Vista (`CatalogoVista`):**
  ```java
  public class CatalogoVista extends JFrame {
      private final JComboBox<String> comboCategorias = new JComboBox<>();
      private final DefaultListModel<String> modeloLista = new DefaultListModel<>();
      private final JList<String> listaProductos = new JList<>(modeloLista);

      public CatalogoVista() {
          super("Catálogo por categoría");
          setLayout(new BorderLayout(10, 10));
          add(comboCategorias, BorderLayout.NORTH);
          add(new JScrollPane(listaProductos), BorderLayout.CENTER);
          setSize(320, 280);
          setLocationRelativeTo(null);
          setDefaultCloseOperation(EXIT_ON_CLOSE);
      }

      public JComboBox<String> getComboCategorias() { return comboCategorias; }
      public DefaultListModel<String> getModeloLista() { return modeloLista; }
  }
  ```

- **3) El Controlador (`CatalogoControlador`):**
  ```java
  public class CatalogoControlador {
      private final CatalogoModelo modelo;
      private final CatalogoVista vista;

      public CatalogoControlador(CatalogoModelo modelo, CatalogoVista vista) {
          this.modelo = modelo;
          this.vista = vista;
          
          // Carga inicial del combo
          modelo.obtenerCategorias().forEach(vista.getComboCategorias()::addItem);
          
          // Escucha cambios en el combo
          vista.getComboCategorias().addActionListener(e -> actualizarLista());
          actualizarLista();
      }

      private void actualizarLista() {
          String cat = (String) vista.getComboCategorias().getSelectedItem();
          DefaultListModel<String> modeloLista = vista.getModeloLista();
          modeloLista.clear(); // Limpia la lista anterior
          modelo.obtenerProductos(cat).forEach(modeloLista::addElement); // Repuebla
      }
  }
  ```

### 5. Selección Múltiple en `JList`

> pág. 19 — Listas_y_ComboBox_Swing.pdf

- Habilitar selección múltiple libre (Ctrl/Shift + clic):
  ```java
  lista.setSelectionMode(ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);
  List<String> elegidos = lista.getSelectedValuesList();
  ```

---

## Ampliación de la IA

> [!warning] Buena práctica con modelos mutables
> Al actualizar un `DefaultListModel`, nunca reasignes el `JList` completo; utilizá `clear()` y agregá los nuevos elementos uno a uno con `addElement()`, o reemplazá el modelo con `setModel()`. Esto evita que los listeners del componente queden huérfanos.

---

## Dudas y vacíos

- [ ] ¿Cómo renderizar objetos complejos en un `JList` (por ejemplo, mostrar el nombre del paciente pero conservar el objeto `Patient` completo) utilizando un `ListCellRenderer` personalizado?

---

## Relacionado

- [[JTable_en_Swing]] — Tablas y modelos tabulares
- [[Swing_MVC_Interfaces_Java]] — Arquitectura en capas y Event Dispatch Thread
- [[enunciado_rubrica]] — Requisitos de la interfaz gráfica
- [[00-estado]] — Estado actual de las vistas en `NutritionistApp`
- [[MAP]] — Mapa general del vault

---

**Navegación:** [[index]] · [[MAP]]
