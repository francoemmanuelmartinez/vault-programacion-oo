---
fuente: JTable_en_Swing.pdf
tags: [tema, practica, swing, jtable, tablemodel, mvc]
estado: completo
---

# Tablas de Datos con JTable en Swing

> [!info] Fuente
> `JTable_en_Swing.pdf` — pág. 1 a 21
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

> pág. 1 — JTable_en_Swing.pdf

`JTable` es el componente de Swing para mostrar colecciones de datos organizados en filas y columnas, similar a una grilla de hoja de cálculo. Al igual que `JList` y `JComboBox`, no almacena los datos directamente, sino que delega el modelo de datos en un `TableModel`.

---

## Lo que dice el profesor

### 1. JTable: Filas y Columnas

> pág. 3 — JTable_en_Swing.pdf

- **Definición:** Muestra datos tabulares.
- **Delegación de datos:** No guarda los datos él mismo; trabaja siempre junto a un `TableModel` que administra el contenido.
- **Envoltorio obligatorio:** Casi siempre se usa dentro de un `JScrollPane` para permitir el desplazamiento vertical cuando hay más filas que espacio visible.
- **Componentes relacionados:**
  - `TableModel` (modelo de datos).
  - `JScrollPane` (scroll).
  - `TableCellRenderer` (cómo se dibuja cada celda).

### 2. Tres Formas de Cargar Datos en un JTable

> pág. 4 a 7 — JTable_en_Swing.pdf

#### Forma 1: Con un Array Estático (La más simple)
- Útil para prototipos o demos rápidas. Los datos quedan sueltos en un array bidimensional, sin conexión con el dominio.
  ```java
  Object[][] datos = {
      {"Ana García", 8, 7, 9},
      {"Bruno Pérez", 5, 4, 6}
  };
  Object[] columnas = {"Alumno", "Parcial 1", "Parcial 2", "Parcial 3"};
  JTable tabla = new JTable(datos, columnas);
  ventana.add(new JScrollPane(tabla));
  ```

#### Forma 2: Con `DefaultTableModel` (Dinámico básico)
- Modelo mutable que permite agregar o quitar filas en tiempo de ejecución (`addRow`, `removeRow`, `setValueAt`). Sigue usando `Object` sueltos en cada celda.
  ```java
  DefaultTableModel modelo = new DefaultTableModel(new Object[]{"Alumno", "Promedio"}, 0);
  modelo.addRow(new Object[]{"Ana García", 8.0});
  modelo.addRow(new Object[]{"Bruno Pérez", 5.0});
  JTable tabla = new JTable(modelo);
  modelo.removeRow(1);
  ```

#### Forma 3: Con `AbstractTableModel` Propio (Máximo control y MVC)
- Permite que la tabla lea directamente objetos del dominio (ej. listas de `Alumno`) sin duplicar datos en arrays genéricos. Solo exige implementar tres métodos:
  ```java
  public class MiTableModel extends AbstractTableModel {
      private List<Alumno> alumnos = new ArrayList<>();

      @Override
      public int getRowCount() { return alumnos.size(); }
      
      @Override
      public int getColumnCount() { return 3; }
      
      @Override
      public Object getValueAt(int fila, int columna) {
          Alumno a = alumnos.get(fila);
          return switch (columna) {
              case 0 -> a.getNombre();
              case 1 -> a.getNota1();
              case 2 -> a.getPromedio();
              default -> null;
          };
      }
  }
  ```

### 3. Configuración del JTable

> pág. 8 y 9 — JTable_en_Swing.pdf

- **Ancho de columnas:** `tabla.getColumnModel().getColumn(0).setPreferredWidth(160);`
- **Auto-redimensionamiento:** `tabla.setAutoResizeMode(JTable.AUTO_RESIZE_ALL_COLUMNS);`
- **Ordenamiento automático:** `tabla.setAutoCreateRowSorter(true);` (permite ordenar haciendo clic en los encabezados sin escribir comparadores).
- **Modo de selección:** `tabla.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);`
- **Edición de celdas:** El método `isCellEditable(fila, columna)` decide si una celda se edita. Por defecto en `AbstractTableModel` devuelve `false` (solo lectura).

---

### 4. Ejemplo MVC Completo: Notas de una Materia

> pág. 11 a 18 — JTable_en_Swing.pdf

Un `JComboBox` filtra alumnos (Todos, Aprobados, Desaprobados) y un `JTable` muestra los resultados calculados al instante.

- **1) El Modelo (`Alumno` y `CalificacionesModelo`):**
  ```java
  public class Alumno {
      private final String nombre;
      private final int nota1, nota2, nota3;

      public Alumno(String nombre, int n1, int n2, int n3) {
          this.nombre = nombre;
          this.nota1 = n1; this.nota2 = n2; this.nota3 = n3;
      }
      public String getNombre() { return nombre; }
      public int getNota1() { return nota1; }
      public int getNota2() { return nota2; }
      public int getNota3() { return nota3; }
      public double getPromedio() { return (nota1 + nota2 + nota3) / 3.0; }
      public boolean estaAprobado() { return getPromedio() >= 6.0; }
  }
  ```

- **2) El TableModel (`AlumnosTableModel`):**
  ```java
  public class AlumnosTableModel extends AbstractTableModel {
      private static final String[] COLUMNAS = {"Alumno", "Parcial 1", "Parcial 2", "Parcial 3", "Promedio", "Estado"};
      private List<Alumno> alumnos = new ArrayList<>();

      public void actualizar(List<Alumno> nueva) {
          this.alumnos = nueva;
          fireTableDataChanged(); // ¡Avisa a la tabla que se repinte!
      }

      @Override public int getRowCount() { return alumnos.size(); }
      @Override public int getColumnCount() { return COLUMNAS.length; }
      @Override public String getColumnName(int col) { return COLUMNAS[col]; }

      @Override
      public Object getValueAt(int fila, int col) {
          Alumno a = alumnos.get(fila);
          return switch (col) {
              case 0 -> a.getNombre();
              case 1 -> a.getNota1();
              case 2 -> a.getNota2();
              case 3 -> a.getNota3();
              case 4 -> String.format("%.2f", a.getPromedio());
              case 5 -> a.estaAprobado() ? "Aprobado" : "Desaprobado";
              default -> null;
          };
      }
  }
  ```

- **3) La Vista (`VistaAlumnos`):**
  ```java
  public class VistaAlumnos extends JFrame {
      private final JComboBox<String> comboFiltro = new JComboBox<>(new String[]{"Todos", "Aprobados", "Desaprobados"});
      private final AlumnosTableModel modeloTabla = new AlumnosTableModel();
      private final JTable tabla = new JTable(modeloTabla);

      public VistaAlumnos() {
          super("Calificaciones");
          setLayout(new BorderLayout(10, 10));
          add(comboFiltro, BorderLayout.NORTH);
          add(new JScrollPane(tabla), BorderLayout.CENTER);
          setSize(480, 280);
          setLocationRelativeTo(null);
          setDefaultCloseOperation(EXIT_ON_CLOSE);
      }
      public JComboBox<String> getComboFiltro() { return comboFiltro; }
      public AlumnosTableModel getModeloTabla() { return modeloTabla; }
  }
  ```

- **4) El Controlador (`CalificacionesControlador`):**
  ```java
  public class CalificacionesControlador {
      private final CalificacionesModelo modelo;
      private final VistaAlumnos vista;

      public CalificacionesControlador(CalificacionesModelo modelo, VistaAlumnos vista) {
          this.modelo = modelo;
          this.vista = vista;
          vista.getComboFiltro().addActionListener(e -> actualizarTabla());
          actualizarTabla();
      }

      private void actualizarTabla() {
          String filtro = (String) vista.getComboFiltro().getSelectedItem();
          vista.getModeloTabla().actualizar(modelo.obtenerAlumnos(filtro));
      }
  }
  ```

---

## Ampliación de la IA

> [!warning] Buenas prácticas con JTable
> - Nunca recrees un `JTable` nuevo al actualizar datos; actualizá el `TableModel` existente y llamá a `fireTableDataChanged()`.
> - Los valores derivados (como el promedio o el estado) jamás se guardan duplicados como atributos mutables en la entidad si se pueden calcular al vuelo en `getValueAt()`.

---

## Dudas y vacíos

- [ ] ¿Cómo implementar un renderer personalizado para pintar las celdas de rojo cuando el alumno está desaprobado? (Ver `TableCellRenderer`).

---

## Relacionado

- [[Listas_y_ComboBox_Swing]] — El mismo patrón con listas y combos
- [[Swing_MVC_Interfaces_Java]] — Fundamentos de Swing y arquitectura en capas
- [[enunciado_rubrica]] — Criterios de evaluación de la interfaz gráfica
- [[00-estado]] — Estado de las vistas en `NutritionistApp`
- [[MAP]] — Mapa general del vault

---

**Navegación:** [[index]] · [[MAP]]
