---
fuente: Swing_MVC_Interfaces_Java.pdf
tags: [tema, practica, swing, mvc, gui, interfaces]
estado: completo
---

# Interfaces Gráficas de Escritorio en Java (Swing y MVC)

> [!info] Fuente
> `Swing_MVC_Interfaces_Java.pdf` — pág. 1 a 29
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

> pág. 1 — Swing_MVC_Interfaces_Java.pdf

Swing es la biblioteca gráfica estándar de Java para construir interfaces de escritorio mediante componentes livianos (lightweight). Combinada con la arquitectura Modelo-Vista-Controlador (MVC), permite separar la interfaz de usuario de la lógica de negocio, logrando un código mantenible, testeable y organizado en capas.

---

## Lo que dice el profesor

### 1. Breve historia de las GUI en Java

> pág. 3 a 7 — Swing_MVC_Interfaces_Java.pdf

- **1995 (Applets y AWT):** Java 1.0 introduce applets para navegadores y **AWT** (Abstract Window Toolkit). AWT delega el dibujado en componentes nativos del sistema operativo (*peers*), por lo que se dice que son componentes "pesados" (*heavyweight*). Solo ofrece el "mínimo común denominador" entre plataformas.
- **1998 (Swing / JFC):** Parte de las *Java Foundation Classes*. Introduce componentes **livianos** (*lightweight*), dibujados píxel a píxel por Java. Ofrece un catálogo mucho más rico (`JTable`, `JTree`, `JTabbedPane`) y *Look & Feel* intercambiable.
- **2008 (JavaFX):** Plataforma moderna con estilos CSS y FXML (hoy OpenJFX).
- **Convención de nomenclatura:** Todos los componentes Swing empiezan con la letra **`J`** (`JButton`, `JLabel`, `JFrame`, `JTextField`), diferenciándose de AWT (`Button`, `Label`).

### 2. Jerarquía de Clases de Swing

> pág. 8 — Swing_MVC_Interfaces_Java.pdf

Todo componente Swing hereda de `java.awt.Component` y `java.awt.Container`, conservando el sistema de eventos y gestores de diseño de AWT:
- `Object` → `Component` → `Container`
  - `Window` → `Frame` → `JFrame` (ventanas principales)
  - `JComponent` (clase base de componentes livianos):
    - `JPanel` (contenedores intermedios)
    - `JLabel` (etiquetas de texto/imagen)
    - `AbstractButton` → `JButton` (botones de acción)
    - `JTextComponent` → `JTextField` (campos de texto de una línea)

### 3. Los Cuatro Pasos para Trabajar con Swing

> pág. 9 a 13 — Swing_MVC_Interfaces_Java.pdf

La interfaz se arma siempre por **composición** (contenedores que contienen componentes) siguiendo cuatro pasos:
1. **Instanciar:** Crear los objetos con `new` (`JFrame`, `JLabel`, `JTextField`, `JButton`).
2. **Configurar:** Ajustar atributos con métodos `set` (título, tamaño, texto, color).
3. **Componer:** Agregar los componentes al contenedor con `add()`, organizados mediante un *Layout Manager*.
4. **Escuchar:** Registrar eventos con `addActionListener()`.

- **`JFrame` (Ventana principal):**
  ```java
  JFrame ventana = new JFrame("Mi ventana");
  ventana.setSize(400, 300);
  ventana.setLocationRelativeTo(null); // Centra en pantalla
  ventana.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
  ventana.setLayout(new FlowLayout());
  ventana.setVisible(true); // Siempre al final
  ```
- **`JLabel`:** Muestra texto o imágenes no editables. Soporta HTML sencillo (`new JLabel("<html><b>Hola</b></html>")`).
- **`JTextField`:** Campo de una sola línea (`new JTextField(20)` de ancho). Se lee con `getText()` y se escribe con `setText()`.
- **`JButton`:** Botón de acción. Permite `setEnabled(false)` y `setToolTipText()`.

### 4. Manejo de Eventos (Patrón Observer)

> pág. 14 — Swing_MVC_Interfaces_Java.pdf

Para reaccionar a un clic de botón, se registra un `ActionListener` y se implementa `actionPerformed()`:
- **Opción 1: Clase anónima** (clásica).
- **Opción 2: Expresión Lambda** (Java 8+, recomendada por simplicidad):
  ```java
  boton.addActionListener(e -> etiqueta.setText("¡Hola!"));
  ```
  *(Requiere importar `java.awt.event.ActionListener` y `java.awt.event.ActionEvent`).*

### 5. Layout Managers Básicos

> pág. 15 — Swing_MVC_Interfaces_Java.pdf

En Swing no se usan coordenadas fijas; los *Layout Managers* organizan y redimensionan los componentes automáticamente:
- **`FlowLayout`:** Ubica los componentes en fila de izquierda a derecha y salta de línea (layout por defecto de `JPanel`).
- **`BorderLayout`:** Divide el contenedor en cinco zonas: `NORTH`, `SOUTH`, `EAST`, `WEST` y `CENTER` (layout por defecto de `JFrame`).
- **`GridLayout`:** Grilla de filas × columnas donde todas las celdas miden exactamente lo mismo (`new GridLayout(2, 3)`).

### 6. Estilos de Construcción: Del `main` a Clases Propias y MVC

> pág. 16 a 20 — Swing_MVC_Interfaces_Java.pdf

- **Estilo 1 (Composición en el `main`):** Todo en un solo método; útil para prototipos pero inmanejable a medida que crece.
- **Estilo 2 (Herencia de `JFrame`):** Cada ventana es una clase propia; los componentes son atributos accesibles desde toda la clase.
- **Estilo 3 (Paneles reutilizables extendiendo `JPanel`):** Permite armar "piezas" de interfaz modulares que exponen getters para que la ventana principal los combine.

### 7. Arquitectura en Capas: MVC en Swing

> pág. 20 a 23 — Swing_MVC_Interfaces_Java.pdf

La regla de oro de la arquitectura en capas:
- **Modelo:** Lógica de negocio y datos. **No conoce a Swing** (ni un solo `import javax.swing`).
- **Vista:** Componentes gráficos (`JFrame`, `JPanel`, botones). Solo muestra y captura datos; expone getters y métodos de actualización.
- **Controlador:** Conecta la vista con el modelo. Escucha los eventos de la vista, invoca al modelo y actualiza la vista.

#### Ejemplo Completo del "Saludador MVC":

- **1) El Modelo:**
  ```java
  public class SaludadorModelo {
      private int cantidadSaludos = 0;

      public String generarSaludo(String nombre) {
          cantidadSaludos++;
          if (nombre == null || nombre.isBlank()) {
              return "¡Hola, anónimo!";
          }
          return "¡Hola, " + nombre + "! (saludo N.º " + cantidadSaludos + ")";
      }
      public int getCantidadSaludos() { return cantidadSaludos; }
  }
  ```

- **2) La Vista:**
  ```java
  public class SaludadorVista extends JFrame {
      private JTextField campoNombre = new JTextField(12);
      private JButton botonSaludar = new JButton("Saludar");
      private JLabel resultado = new JLabel("---");

      public SaludadorVista() {
          super("Saludador MVC");
          setDefaultCloseOperation(EXIT_ON_CLOSE);
          setLayout(new FlowLayout());
          add(new JLabel("Nombre:"));
          add(campoNombre);
          add(botonSaludar);
          add(resultado);
          setSize(420, 130);
      }

      public String getNombre() { return campoNombre.getText(); }
      public JButton getBotonSaludar() { return botonSaludar; }
      public void mostrarSaludo(String s) { resultado.setText(s); }
  }
  ```

- **3) El Controlador y el Arranque:**
  ```java
  public class SaludadorControlador {
      private SaludadorModelo modelo;
      private SaludadorVista vista;

      public SaludadorControlador(SaludadorModelo m, SaludadorVista v) {
          this.modelo = m;
          this.vista = v;
          vista.getBotonSaludar().addActionListener(e -> saludar());
      }

      private void saludar() {
          String s = modelo.generarSaludo(vista.getNombre());
          vista.mostrarSaludo(s);
      }
  }

  // En el main (dentro del Event Dispatch Thread):
  public class App {
      public static void main(String[] args) {
          javax.swing.SwingUtilities.invokeLater(() -> {
              SaludadorVista vista = new SaludadorVista();
              new SaludadorControlador(new SaludadorModelo(), vista);
              vista.setVisible(true);
          });
      }
  }
  ```

### 8. Variante avanzada: Action API

> pág. 24 a 27 — Swing_MVC_Interfaces_Java.pdf

- **`AbstractAction`:** Permite empaquetar una operación (texto, tooltip, atajo de teclado, estado habilitado/deshabilitado y la lógica a ejecutar) en un solo objeto.
- Permite asociar una misma acción simultáneamente a un botón y a un ítem de menú, sincronizando su estado (`setEnabled()`) automáticamente.
- El controlador se reduce a un catálogo que crea y "cablea" acciones a la vista mediante métodos como `vista.setAccionSaludar(new SaludarAction(modelo, vista))`.

### 9. Mensaje final del profesor

> pág. 29 — Swing_MVC_Interfaces_Java.pdf

> *"Las interfaces cambian; los buenos diseños permanecen."*

---

## Ampliación de la IA

> [!warning] El hilo de eventos (EDT)
> En Swing, todos los componentes gráficos deben crearse y modificarse exclusivamente en el **Event Dispatch Thread** (EDT). Por eso la buena práctica profesional exige envolver el arranque de la aplicación en `SwingUtilities.invokeLater(...)`, tal como vimos en `NutritionistApp`.

---

## Dudas y vacíos

- [ ] ¿El profesor prefiere lambdas directas para los listeners en la tarea (más sencillas para sistemas chicos) o exige la Action API (recomendada para sistemas grandes con menús y toolbars)?

---

## Relacionado

- [[04-convenciones]] — Convención de getters privados en las vistas de nuestro proyecto
- [[03-vistas-manuales]] — Contingencia de vistas a mano y estrategia `dispose()`
- [[enunciado_rubrica]] — Criterios de evaluación de la interfaz gráfica
- [[00-estado]] — Estado actual de las vistas en `NutritionistApp`
- [[MAP]] — Mapa general del vault

---

**Navegación:** [[index]] · [[MAP]]
