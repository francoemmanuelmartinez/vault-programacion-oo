---
fuente: auditoría del código real de `NutritionistApp` en `origin/master` (`751bb35`) + `Sql/nutricion.sql` + Listas_y_ComboBox_Swing.pdf (pág. 6 a 8, 9 y 11 a 17) + Swing_MVC_Interfaces_Java.pdf (pág. 14 y 20 a 23)
tags: [tarea, plan, swing, jcombobox, aliment, categoria, mvc]
estado: propuesto
actualizado: 2026-11-12
aliases: [plan-categoria-combo, plan-combo-categoria, categoria-aliment]
---

# Plan: `Category` de `Aliment` como `JComboBox` cerrado

> [!info] Qué es esta nota
> `AlimentView` tiene hoy un `JTextField` para la categoría, así que acepta `Verdur`, `Verduras` y `Verduraaaaaas` como si fueran tres categorías distintas. Este plan lo reemplaza por un `JComboBox` **no editable** con una lista cerrada de **7 valores**, un ítem vacío al principio y la validación en el controlador.
>
> **Punto de partida:** `origin/master` = `751bb35`. Es un cambio de **una sola pantalla**: no toca la base, ni el DAO, ni el modelo, ni la tabla.
>
> **Esfuerzo:** media hora en el diseñador y unas 20 líneas de código.

> [!warning] Procedencia del contenido
> - **Del profesor:** el uso de `JComboBox` para listas cerradas, el modelo mutable y el registro del evento (pág. 6 a 8, 9 y 11 a 17 — `Listas_y_ComboBox_Swing.pdf`), y la separación Vista → Controlador → Modelo (pág. 14 y 20 a 23 — `Swing_MVC_Interfaces_Java.pdf`).
> - **Criterio de equipo / IA (NO del profesor):** los 7 valores concretos, el placeholder, dónde vive la lista y el mensaje de validación. Van marcados con `[!tip]` o `[!warning]`.

---

## 0. Diagnóstico: qué hay hoy

| Pieza | Estado | Comentario |
|---|---|---|
| `AlimentView.categoryTextField` | `JTextField` editable | `AlimentView.java:32`; `setHorizontalAlignment(CENTER)` en la línea 62 |
| `getCategoryText()` / `setCategoryText()` | ✅ existen | `AlimentView.java:365` y `371` |
| `clearForm()` | ✅ existe | `AlimentView.java:387` llama a `setCategoryText("")` |
| `AlimentController` validaciones | ⚠️ texto libre | líneas 59 y 84: `getCategoryText().trim().isEmpty()` |
| `AlimentController.readForm()` | ⚠️ texto libre | línea 153: `a.setCategory(view.getCategoryText().trim())` |
| `AlimentController.showAliment()` | ⚠️ texto libre | línea 162: `view.setCategoryText(a.getCategory())` |
| `AlimentTableModel` | ✅ no se toca | sigue mostrando `Category` como texto |
| `AlimentDAO.buscarPorCategoria` | ✅ no se toca | sigue sin usarse en el CRUD |
| `Sql/nutricion.sql` | ✅ no se toca | `category VARCHAR(60) NOT NULL` |

Los valores que hoy tiene la base (`Sql/nutricion.sql:45-51`) son exactamente cinco:

```
Frutas · Verduras · Cereales · Carnes · Lácteos
```

> [!warning] `Lácteos` lleva tilde
> Es el detalle que más se paga: si en la lista escribís `Lacteos`, al seleccionar `Yogur descremado` el combo no encuentra el ítem y queda mostrando el placeholder. La pantalla miente y, si guardás, cambiás la categoría sin querer.

---

## 1. Alcance

### Entra

- Reemplazar el `JTextField` por un `JComboBox<String>` **no editable**.
- Lista cerrada de 7 valores: los 5 del script + `Legumbres` + `Otros`.
- Placeholder `-- Elegí una categoría --` en el índice 0, que **no** es una categoría válida.
- Getters y setters nuevos en la vista, con el mismo patrón de índices que `setDietOptions` / `getDietIndex` de `PatientView`.
- Validación en el controlador: no se guarda sin categoría elegida.
- Normalización de datos viejos, si hubiera.

### No entra

- **Una tabla `categories` con clave foránea.** Es la solución "de manual", pero toca el esquema de RomiM, agrega una carga inicial o un CRUD, y no lo pide la rúbrica. Si algún día se quiere, sólo cambia el origen de `setCategoryOptions`.
- **Búsqueda por categoría.** El botón Search sigue buscando por ID. `AlimentDAOJPA.buscarPorCategoria` sigue esperando su turno (ver [[bug-jpql-dao]]).
- **Cambiar el script SQL.** La columna sigue igual; los valores nuevos son opciones de interfaz, no filas.

> [!tip] Por qué es la decisión correcta para este trabajo
> El conjunto de categorías es cerrado y chico. Un combo de 7 ítems elimina los errores de tipeo **sin** agregar una tabla, un DAO y una pantalla más. Es el mismo criterio con el que `PatientView` muestra las dietas en un combo en vez de pedir el id a mano.

---

## 2. La lista

| Índice del combo | Valor | Origen |
|---|---|---|
| 0 | `-- Elegí una categoría --` | placeholder, **no** es un dato |
| 1 | `Frutas` | script |
| 2 | `Verduras` | script |
| 3 | `Cereales` | script |
| 4 | `Carnes` | script |
| 5 | `Lácteos` | script (con tilde) |
| 6 | `Legumbres` | nueva |
| 7 | `Otros` | nueva |

`Legumbres` y `Otros` son categorías reales para el dominio: sin ellas, un alimento como lentejas o una comida procesada no tiene dónde ir y el usuario termina eligiendo cualquier cosa. `Otros` es la válvula de escape, no texto libre.

---

## 3. Paso 1 — La vista (`AlimentView`)

### 3.1 En el diseñador

1. `Ctrl+F` por `categoryTextField` y anotar dónde se usa (hoy: declaración, `initComponents`, getter, setter y `clearForm`).
2. En el **Navigator**, seleccionar `categoryTextField` → **Delete**.
3. Arrastrar un **Combo Box** desde la paleta al mismo lugar, en la fila de `Category:`, alineado con `nameTextField`.
4. Clic derecho → **Change Variable Name…** → `categoryComboBox`.
5. En **Properties**: destildar **`editable`** y escribir el `toolTipText`: `Elegí una categoría de la lista`.
6. En el modelo del combo, borrar los `Item 1..4` que agrega NetBeans. El controlador lo llena en el constructor.
7. `Ctrl+S` y compilar. **Todavía no va a compilar** el controlador: sigue llamando a `getCategoryText()`. Se arregla en el paso 2.

> [!warning] Se va el `setHorizontalAlignment(CENTER)`
> Esa propiedad es de `JTextField`; un `JComboBox` no la tiene. Si quedó la línea en `initComponents()`, no compila. No la busques para el combo: se descarta.

> [!tip] El par `.java` + `.form` va siempre junto
> Si commiteás uno sin el otro, el diseñador reescribe el `.java` la próxima vez que abras el form. Es la misma trampa 4 de [[plan-cableado-diet]].

### 3.2 Los métodos de la vista

Se **reemplazan** `getCategoryText()` y `setCategoryText(String)`. Todo va después de `// End of variables declaration//GEN-END:variables`, junto a los getters que ya existen.

```java
// --- Categoria: combo de lista cerrada ---

// Llena el combo. El indice 0 es el placeholder, no una categoria valida.
public void setCategoryOptions(java.util.List<String> categories) {
    javax.swing.DefaultComboBoxModel<String> model = new javax.swing.DefaultComboBoxModel<>();
    model.addElement("-- Elegí una categoría --");
    for (String c : categories) {
        model.addElement(c);
    }
    categoryComboBox.setModel(model);
}

// 0 = el usuario todavia no eligio nada
public int getCategoryIndex() { return categoryComboBox.getSelectedIndex(); }
public void setCategoryIndex(int index) { categoryComboBox.setSelectedIndex(index); }
```

Y en `clearForm()`:

```java
public void clearForm() {
    setIdText("");
    setNameText("");
    setCategoryIndex(0);       // vuelve al placeholder
    setCaloriesText("");
}
```

> [!note] Por qué índices y no el texto
> Es el patrón que ya usa `PatientView` con las dietas: `setDietOptions(List<String>)`, `getDietIndex()` y `setDietIndex(int)`. La vista no conoce los valores reales; el controlador traduce índice ↔ categoría y queda como único dueño de la lista. Además evita el `ClassCastException` de `(String) getSelectedItem()` cuando no hay selección.

### 3.3 Opcional en la misma pasada

`AlimentView.showInfo` dice `"Informacion"` sin tilde (`AlimentView.java:395`). Ya está anotado como deuda; si estás tocando el archivo, corregilo.

---

## 4. Paso 2 — El controlador (`AlimentController`)

### 4.1 La constante y la carga inicial

Arriba de la clase, junto a los campos:

```java
    // Lista cerrada de categorias. Tiene que coincidir EXACTAMENTE con los valores
    // de la base, tildes incluidas (Lácteos).
    private static final String[] CATEGORIAS = {
        "Frutas", "Verduras", "Cereales", "Carnes", "Lácteos", "Legumbres", "Otros"
    };
```

En el constructor, después de `view.getTableAliments().setModel(tableModel);`:

```java
        view.setCategoryOptions(java.util.Arrays.asList(CATEGORIAS));
```

### 4.2 Los dos helpers

```java
    // Devuelve la categoria elegida, o null si esta en el placeholder.
    private String selectedCategory() {
        int index = view.getCategoryIndex();
        return index <= 0 ? null : CATEGORIAS[index - 1];
    }

    // Devuelve la posicion en el combo de una categoria (0 = no esta en la lista).
    private int categoryIndexOf(String category) {
        for (int i = 0; i < CATEGORIAS.length; i++) {
            if (CATEGORIAS[i].equals(category)) {
                return i + 1;
            }
        }
        return 0;
    }
```

### 4.3 `readForm()` y `showAliment()`

```java
    private Aliment readForm() {
        Aliment a = new Aliment();
        a.setAliment(view.getNameText().trim());
        a.setCategory(selectedCategory());          // ya validado, nunca null aca
        a.setCaloriesPerServing(parseNonNegative(view.getCaloriesText(), "Las calorias"));
        return a;
    }

    private void showAliment(Aliment a) {
        view.setIdText(String.valueOf(a.getId()));
        view.setNameText(a.getAliment());
        view.setCategoryIndex(categoryIndexOf(a.getCategory()));   // 0 si es un valor viejo
        view.setCaloriesText(String.valueOf(a.getCaloriesPerServing()));
    }
```

### 4.4 Las validaciones

En `insert()` (línea 59) y `modify()` (línea 84), el bloque:

```java
            if (view.getNameText().trim().isEmpty() || view.getCategoryText().trim().isEmpty()) {
                view.showError("El nombre y la categoria son obligatorios.");
                return;
            }
```

pasa a:

```java
            if (view.getNameText().trim().isEmpty() || view.getCategoryIndex() <= 0) {
                view.showError("El nombre es obligatorio y hay que elegir una categoria.");
                return;
            }
```

> [!warning] El `<= 0` incluye el placeholder
> Si usás `getCategoryIndex() < 0`, el placeholder (índice 0) pasa la validación y se guarda `selectedCategory() == null` en una columna `NOT NULL`. MySQL responde con un error técnico en inglés. El chequeo correcto es `<= 0`.

### 4.5 Lo que **no** cambia

- `refreshTable()`, `selectRow()`, `delete()`, `search()`, `resetFields()`: no saben nada de la categoría.
- `parseNonNegative`, las calorías y el resto del formulario: intactos.
- El DAO y la entidad: intactos. `Aliment.getCategory()` sigue devolviendo un `String`.

---

## 5. Datos existentes

Antes de probar, confirmá que la base no tenga variantes:

```sql
SELECT DISTINCT category FROM aliments ORDER BY category;
```

Esperado: exactamente las cinco del script. Si aparece una variante (`Verdur`, `verdura`, `Lácteos ` con espacio), normalizala:

```sql
UPDATE aliments SET category = 'Verduras' WHERE category IN ('Verdur', 'verdura', 'Verduras ');
```

> [!important] Por qué esto va **antes** de probar
> Una categoría fuera de la lista no rompe nada: al seleccionar la fila, el combo muestra el placeholder (`categoryIndexOf` devuelve 0). Pero si el usuario guarda sin darse cuenta, la validación lo frena. Es seguro, pero confuso: mejor que no existan datos viejos.

> [!note] El script no se toca
> `Sql/nutricion.sql` ya usa los valores canónicos. Los dos valores nuevos (`Legumbres`, `Otros`) son opciones de la interfaz: no hace falta `INSERT` ni cambiar la columna.

---

## 6. Pruebas

| # | Acción | Resultado esperado |
|---|---|---|
| 1 | Abrir **Aliments** | el combo muestra `-- Elegí una categoría --` y no se puede escribir |
| 2 | Desplegar el combo | exactamente 7 categorías, en el orden de la tabla de §2 |
| 3 | Seleccionar `Yogur descremado` | el combo queda en **Lácteos** (con tilde) |
| 4 | Insert sin elegir categoría | mensaje *"El nombre es obligatorio y hay que elegir una categoria."*, no guarda |
| 5 | Insert de `Lentejas` con `Legumbres` | guarda; la tabla muestra `Legumbres` |
| 6 | **Cerrar y reabrir la app** | la categoría sigue en `Legumbres` (está en MySQL) |
| 7 | Modify de `Manzana` a `Otros` | persiste al reabrir |
| 8 | Delete + Insert seguidos | el formulario vuelve al placeholder, no a `Frutas` |
| 9 | `SELECT DISTINCT category FROM aliments;` | los valores son sólo los de la lista |
| 10 | Ordenar la tabla y hacer clic en una fila | el combo muestra la categoría de **esa** fila (`convertRowIndexToModel`) |

> [!important] Las pruebas 3 y 8 son las que valen
> La 3 demuestra que el dato de la base **preselecciona** el ítem correcto (el bug de la tilde aparece acá). La 8 demuestra que no hay un default silencioso: después de limpiar, el combo no elige `Frutas` por vos.

---

## 7. Trampas

| # | Trampa | Qué pasa |
|---|---|---|
| 1 | Dejar `editable` tildado | el combo acepta texto libre: es un `JTextField` con flechita |
| 2 | Escribir `Lacteos` en la lista | `Lácteos` no se preselecciona al editar el yogur |
| 3 | No llamar a `setCategoryOptions` en el constructor | el combo muestra `Item 1..4` del diseñador |
| 4 | Cambiar la vista y no el controlador | `cannot find symbol: method getCategoryText()` en 4 lugares |
| 5 | Poner la lista en el diseñador **y** en el controlador | se desincronizan; una agrega una categoría que la otra no conoce |
| 6 | Validar con `< 0` en vez de `<= 0` | el placeholder pasa y se intenta guardar `null` en `NOT NULL` |
| 7 | Usar `(String) getSelectedItem()` sin chequeo | `ClassCastException` o `NullPointerException` si el modelo está vacío |
| 8 | `setCategoryIndex(-1)` para "limpiar" | el combo queda sin selección y `getSelectedIndex()` devuelve -1: funciona, pero rompe la convención de que 0 es el placeholder |
| 9 | No normalizar datos viejos | al editar una fila con categoría rara, el combo queda en placeholder y parece que el dato se perdió |
| 10 | Commitear `AlimentView.java` sin el `.form` | el diseñador reescribe el `.java` al abrirlo |

> [!danger] La trampa 5 es la más silenciosa
> Si la lista vive en dos lugares, todo compila y todo anda… hasta que alguien agrega una categoría en el diseñador y el controlador valida contra la otra lista. Una sola fuente de verdad: el `static final` del controlador.

---

## 8. Si algo falla

| Mensaje / síntoma | Causa | Solución |
|---|---|---|
| `cannot find symbol: method getCategoryText()` | el controlador no se actualizó | §4.3 y §4.4 |
| El combo muestra `Item 1..4` | falta `setCategoryOptions` en el constructor | §4.1 |
| Al elegir `Yogur` el combo no muestra `Lácteos` | la lista dice `Lacteos` o el orden no coincide | §2 y §5 |
| Guarda con categoría vacía | validación con `< 0` o sin validación | §4.4 |
| `NullPointerException` en `getSelectedItem()` | se usó la API de texto en vez de índices | §3.2 |
| `Data too long for column 'category'` | se editó un valor viejo más largo que 60 | normalizar (§5) |
| El combo quedó gris / no responde | `enabled` destildado en el diseñador | Properties → `enabled` |

---

## 9. Lo que dice el profesor

> pág. 6 a 8 — Listas_y_ComboBox_Swing.pdf
> El `JComboBox` combina un campo visible con una lista desplegable y puede ser de **solo selección** o editable (`setEditable(true)`). El `DefaultComboBoxModel` permite manipular las opciones en tiempo de ejecución y el ítem elegido se obtiene con `getSelectedItem()`.

> pág. 9 — Listas_y_ComboBox_Swing.pdf
> El `JComboBox` se escucha con `addActionListener`, igual que un `JButton`; el evento se dispara al confirmar una opción.

> pág. 11 a 17 — Listas_y_ComboBox_Swing.pdf
> El ejemplo MVC "Catálogo por categoría" usa un `JComboBox` de categorías (Frutas, Bebidas, Lácteos) para filtrar los productos de una `JList`. La categoría se trata como una lista cerrada, no como texto libre.

> pág. 14 y 20 a 23 — Swing_MVC_Interfaces_Java.pdf
> La vista sólo muestra y captura; los datos y las reglas van en el controlador y el modelo.

**Qué toma este plan del profesor:** el componente (`JComboBox` no editable), el modelo (`DefaultComboBoxModel`) y el lugar de la lista (el controlador la provee, la vista la muestra). **Qué agrega el equipo:** los 7 valores, el placeholder y el mensaje de validación.

---

## 10. Ampliación de la IA

> [!note] Lista hardcodeada vs tabla `categories`
> Lo "correcto" en un sistema que crece es una tabla `categories` con `id_categoria` en `aliments` y una FK. Eso permite agregar categorías sin recompilar, pero cuesta: cambiar el esquema (que es de RomiM), recrear la base, cargar los datos y —si se quiere administrar— una pantalla más.
>
> Para este TP, con 7 valores estables, la constante en el controlador es la relación costo/beneficio correcta. Si el día de mañana se migra a tabla, el único punto que cambia es de dónde salen los `String` que recibe `setCategoryOptions`.

> [!note] Por qué el placeholder y no un default
> Si el combo arranca en `Frutas`, todo alimento nuevo sale `Frutas` salvo que el usuario se fije. Un placeholder que la validación rechaza convierte un default silencioso en una decisión explícita.

> [!note] Por qué la validación va en el controlador
> La vista no decide si una categoría es válida: sólo informa qué índice está seleccionado. La regla (`índice > 0`) vive en el controlador, igual que el largo del nombre y las calorías no negativas.

---

## 11. Commit

```powershell
git add src/appNutritionist/view/AlimentView.java `
        src/appNutritionist/view/AlimentView.form `
        src/appNutritionist/controller/AlimentController.java
git commit -m "Cambie la categoria de Aliment por un JComboBox con lista cerrada"
```

> [!tip] Un solo commit
> Vista y controlador cambian juntos: separarlos deja un commit que no compila. El `.form` va en el mismo `git add` que el `.java`.

---

## Relacionado

- [[plan-cableado-aliment]] — el cableado de `Aliment`; su sección "decisión previa" queda resuelta por esta nota
- [[Listas_y_ComboBox_Swing]] — la fuente del profesor sobre `JComboBox` y `DefaultComboBoxModel`
- [[plan-cableado-diet]] — el mismo patrón de combo con placeholder (`PatientView`) y el panel de alimentos
- [[plan-cableado-patient]] — `setDietOptions` / `getDietIndex`: el molde de índices que se copia acá
- [[bug-jpql-dao]] — `buscarPorCategoria` sigue con el JPQL roto; no entra en este cambio
- [[enunciado_rubrica]] — la GUI tiene que ser intuitiva y funcional para las tres entidades
- [[MAP]] · [[index]] — mapa y catálogo del vault
