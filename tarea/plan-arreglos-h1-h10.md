---
fuente: auditoría del código en `origin/master` (`0d47764`) + `Sql/nutricion.sql` (DDL) + los huecos H1 a H10 de [[plan-de-pruebas-crud]] + [[bug-jpql-dao]]
tags: [tarea, plan, validaciones, jpql, crud, aliment, patient, diet]
estado: implementado
actualizado: 2026-11-12
aliases: [plan-arreglos, arreglos-h1-h10, plan-validaciones, arreglos-h5-h10]
---

# Plan de arreglos: H1 a H10

> [!info] Qué es esta nota
> El paso a paso para cerrar los diez huecos del [[plan-de-pruebas-crud]]:
>
> - **H1 a H4** — los cuatro JPQL rotos. Van **primero**: H1 desbloquea el borrado de dietas.
> - **H5 a H10** — longitudes, saltos de línea, calorías en cero, duplicados, combo filtrado y confirmaciones.
>
> **No se toca** el esquema de la base, las entidades ni la configuración de JPA.

> [!warning] Procedencia
> - **Del profesor:** la rúbrica pide CRUD completo con **validación de datos** y manejo de errores (`enunciado_rubrica.pdf`, pág. 1 y 2), y que los diálogos salgan por la vista mientras las reglas viven en el controlador (`Swing_MVC_Interfaces_Java.pdf`, pág. 14 y 20 a 23).
> - **Criterio de equipo/IA:** los límites concretos, la clave de duplicado, el orden de trabajo y los mensajes.

---

## 0. Alcance

| Hueco | Qué se arregla | Archivos |
|---|---|---|
| **H1** | `buscarPorDieta` de `Patient` — desbloquea el **delete de `Diet`** | `PatientDAOJPA` |
| **H2** | `buscarPorCategoria` de `Aliment` | `AlimentDAOJPA` |
| **H3** | `buscarPorNombre` de `Diet` | `DietDAOJPA` |
| **H4** | `buscarPorApellido` de `Patient` | `PatientDAOJPA` |
| **H5** | Validar el largo de cada texto antes de guardar | los 3 controladores |
| **H6** | Convertir los `Enter` de la descripción en espacios | `DietController` |
| **H7** | Las calorías tienen que ser mayores a cero | `AlimentController` |
| **H8** | Rechazar duplicados lógicos con un mensaje claro | los 3 controladores |
| **H9** | El combo de alimentos ofrece sólo los que faltan en la dieta | `DietController` |
| **H10** | `Add` y `Remove` del panel piden confirmación | `DietView` + `DietController` |

---

## 1. Diagnóstico

### 1.1 Los cuatro JPQL (H1 a H4)

Un JPQL es un `String`: **compila aunque esté mal**. El error aparece recién cuando esa consulta se ejecuta, así que un método roto puede vivir meses sin que nadie lo note.

| # | Archivo | Línea | Dice | Debería decir | ¿Se ejecuta hoy? |
|---|---|---|---|---|---|
| H1 | `PatientDAOJPA` | 26 | `SELECT p FROM patients p WHERE p.id_dieta = :idDieta` | `SELECT p FROM Patient p WHERE p.diet.id = :idDieta` | **Sí**, desde `DietController.delete():121` |
| H2 | `AlimentDAOJPA` | 19 | `SELECT a FROM aliments a WHERE a.category = :cat` | `SELECT a FROM Aliment a WHERE a.category = :cat` | no |
| H3 | `DietDAOJPA` | 19 | `SELECT d FROM diets d WHERE LOWER(d.name) LIKE :nom` | `SELECT d FROM Diet d WHERE LOWER(d.name) LIKE :nom` | no |
| H4 | `PatientDAOJPA` | 19 | `SELECT p FROM patients p WHERE LOWER(p.lastName) LIKE :ap` | `SELECT p FROM Patient p WHERE LOWER(p.lastName) LIKE :ap` | no |

Los tres errores que se repiten:

1. **El nombre de la entidad no es el nombre de la tabla.** Es `Patient`, `Aliment` y `Diet` (las clases), no `patients`, `aliments` ni `diets`.
2. **En JPQL se navega por campos, no por columnas.** `Patient` tiene el campo `diet` (con `@JoinColumn(name = "id_diet")`), así que es `p.diet.id`. `p.id_dieta` no existe para JPA.
3. **Los nombres de campo sí están bien** en H2, H3 y H4: `category`, `name` y `lastName` existen tal cual.

> [!danger] Consecuencia de H1
> `DietController.delete()` llama `patientDao.buscarPorDieta(id)` **siempre**, antes de borrar. La excepción se lanza aunque no haya ningún paciente, así que **hoy no se puede eliminar ninguna dieta**. Los casos **D7 y D8** del plan de pruebas están bloqueados por esto y nada más.

### 1.2 Los seis de validación (H5 a H10)

| # | Dónde | Hoy | Qué pasa |
|---|---|---|---|
| H5 | los 3 `insert()` / `modify()` | sólo valida vacío | un texto más largo que la columna vuelve como `Data too long for column 'name'` |
| H6 | `DietController:215` | `trim()` | los `\n` de la textarea se guardan en un `VARCHAR(100)` |
| H7 | `AlimentController:186` | `if (value < 0)` | `0` kcal se acepta |
| H8 | los tres `insert()` / `modify()` | nada | dos `Manzana`, dos `Lucía Fernández`, dos `Vegetariana` |
| H9 | `DietController:249-261` | `alimentDao.findAll()` completo | el combo ofrece alimentos que ya están |
| H10 | `DietController:173-207` | inserta/borra directo | un clic equivocado quita un alimento |

#### Los límites reales

Salen de `Sql/nutricion.sql`, que es la fuente de verdad (las entidades no declaran `@Column(length)`, así que JPA asumiría 255 y la base rechazaría mucho antes):

| Tabla | Columna | Largo | Archivo |
|---|---|---|---|
| `patients` | `name` | **80** | `nutricion.sql:22` |
| `patients` | `lastName` | **80** | `:23` |
| `patients` | `observations` | **500** | `:26` |
| `aliments` | `aliment` | **50** | `:15` |
| `aliments` | `category` | 60 | `:16` |
| `diets` | `name` | **50** | `:9` |
| `diets` | `description` | **100** | `:10` |

> [!tip] El 100 de `description` no es teórico
> La dieta `Vegetariana` del script tiene una descripción de **99 caracteres**. Un carácter más y el `INSERT` del propio script falla.

---

## 2. Decisiones de diseño

### H1 a H4 — Arreglar primero, y no confundir con "implementar la búsqueda"

Son cuatro líneas de `String`, sin riesgo. Se hacen **antes** que todo lo demás porque:

- **H1 desbloquea D7 y D8**, que son P0 y hoy no se pueden ni intentar.
- H2, H3 y H4 no cambian el comportamiento visible (nadie los llama), pero dejan de ser bombas.

> [!important] Arreglar el JPQL **no** implementa la funcionalidad
> `buscarPorApellido`, `buscarPorCategoria` y `buscarPorNombre` van a funcionar, pero **ninguna pantalla los usa**: los tres botones `Search` buscan por **ID**. El *Search by Last Name* de la rúbrica y el filtro por categoría siguen sin existir como funcionalidad. Eso es un trabajo aparte (ver [[plan-crud-bd]] §5).

### H5 — Un `validarLargo` por controlador

| Opción | Veredicto |
|---|---|
| **Método privado en cada controlador** | ✅ elegida: 6 líneas por clase, mismo estilo que `parsePositive` / `parseNonNegative` |
| Clase utilitaria compartida | más limpia, pero agrega un paquete y refactoriza helpers que hoy funcionan |
| `@Size` + Bean Validation | requiere Hibernate Validator; no lo pide la rúbrica |

Lanza `IllegalArgumentException`, que los tres `insert()`/`modify()` ya atrapan con `view.showError(ex.getMessage())`. **En `Diet` hay que agregar ese `catch`**, porque hoy no lo tiene.

### H6 — Normalizar en `readForm()`, no en la vista

`replaceAll("\\s+", " ")` convierte cualquier secuencia de espacios, tabs y saltos en **un** espacio. Va en el único punto donde el dato pasa a la base. Un `KeyListener` en la textarea sería lógica de negocio en la vista.

### H7 — `>` en vez de `>=`

Un alimento con 0 kcal no existe. Se renombra a `parsePositive` (como en `Patient`) y el mensaje pasa a *"Las calorias deben ser mayores a cero."*

### H8 — Duplicados en el controlador, no con `UNIQUE`

| Opción | Veredicto |
|---|---|
| **Comparar contra `findAll()` antes de guardar** | ✅ elegida: no toca el esquema, mensaje específico, y `findAll()` ya se usa |
| `UNIQUE` en la base | toca el esquema de Romi y todas las bases creadas; el 1062 llega como *"Ya existe un registro igual."*, sin decir qué campo |

| Entidad | Duplicado si coinciden | Ejemplo que se rechaza |
|---|---|---|
| `Patient` | `name` + `lastName`, sin distinguir mayúsculas | `Lucía Fernández` dos veces |
| `Aliment` | `aliment` (el nombre) | `Manzana` dos veces |
| `Diet` | `name` | `Hipocalórica` dos veces |

Al modificar se **excluye el propio registro** por `id`, si no todo `Modify` se rechazaría a sí mismo.

### H9 — Un solo `refreshPanel()`

El combo y la tabla de abajo son dos vistas del mismo dato (los `DietDetail` de la dieta). Refrescarlos por separado los desincroniza tarde o temprano:

```java
private void refreshPanel() {
    refreshDetails();   // tabla de abajo
    loadAliments();     // combo de arriba
}
```

`loadAliments()` deja de cargar "todos" y pasa a cargar "los que faltan", comparando por `id` (porque `Aliment` no define `equals`). La lista se renombra a `alimentsDisponibles` para que nadie la use como "todos".

### H10 — Confirmación en la vista, decisión en el controlador

Mismo patrón que `confirmInsert` / `confirmDelete`: la vista muestra el `JOptionPane`, el controlador decide cuándo preguntar.

---

## 3. Los cambios

### 3.0 Los cuatro JPQL — H1 a H4

Un solo commit, tres archivos, cuatro líneas:

```java
// PatientDAOJPA.java:26  (H1)
return consultar(
    "SELECT p FROM Patient p WHERE p.diet.id = :idDieta", "idDieta", idDieta);

// PatientDAOJPA.java:19  (H4)
return consultar(
    "SELECT p FROM Patient p WHERE LOWER(p.lastName) LIKE :ap",
    "ap", "%" + apellido.toLowerCase() + "%");

// AlimentDAOJPA.java:19  (H2)
return consultar(
    "SELECT a FROM Aliment a WHERE a.category = :cat", "cat", categoria);

// DietDAOJPA.java:19  (H3)
return consultar(
    "SELECT d FROM Diet d WHERE LOWER(d.name) LIKE :nom",
    "nom", "%" + nombre.toLowerCase() + "%");
```

> [!warning] `p.diet.id` y no `p.diet`
> Se puede filtrar por el id sin traer la dieta entera: JPA navega la relación por el campo `diet` y baja a su `id`. Es lo mismo que hace `DietDetailDAOJPA` con `d.diet.id`, que ya funciona.

#### Cómo se verifican

| JPQL | Cómo se prueba |
|---|---|
| H1 | **Desde la UI**: D7 (borrar dieta sin pacientes) y D8 (con pacientes). Es el único con pantalla. |
| H2, H3, H4 | No tienen pantalla. Se verifican con una clase de prueba con `main` (ver abajo). |

Clase de verificación, en `test/appNutritionist/dao/PruebaBusquedas.java` (no entra al build de la app; se corre con *Run File* en NetBeans). **Decisión del 2026-11-12: se queda como herramienta local y no se commitea.**

```java
package appNutritionist.dao;

import appNutritionist.model.*;
import java.util.List;

// Verificacion manual de los JPQL que hoy no tienen pantalla.
public class PruebaBusquedas {
    public static void main(String[] args) throws Exception {
        PatientDAOJPA pacientes = new PatientDAOJPA();
        AlimentDAOJPA alimentos = new AlimentDAOJPA();
        DietDAOJPA dietas = new DietDAOJPA();

        List<Patient> porApellido = pacientes.buscarPorApellido("fer");
        System.out.println("apellido 'fer': " + porApellido.size());

        List<Aliment> porCategoria = alimentos.buscarPorCategoria("Frutas");
        System.out.println("categoria 'Frutas': " + porCategoria.size());

        List<Diet> porNombre = dietas.buscarPorNombre("hipo");
        System.out.println("nombre 'hipo': " + porNombre.size());

        List<Patient> porDieta = pacientes.buscarPorDieta(1);
        System.out.println("dieta 1: " + porDieta.size());
    }
}
```

Con los datos del script, la salida esperada es `1`, `2`, `1` y `1`. Si alguno tira excepción, el JPQL sigue mal.

> [!tip] Antes y después
> Corré la clase **antes** de tocar los JPQL: los cuatro métodos tienen que fallar. Así se ve el bug. Después del arreglo, los cuatro tienen que devolver números.

### 3.1 `AlimentController` — H5, H7 y H8

Constante y helpers, junto a los que ya existen:

```java
    private static final int MAX_ALIMENT = 50;   // aliments.aliment VARCHAR(50)

    private void validarLargo(String texto, int maximo, String campo) {
        if (texto != null && texto.trim().length() > maximo) {
            throw new IllegalArgumentException(campo + " no puede superar los " + maximo + " caracteres.");
        }
    }

    // Rechaza un alimento con el mismo nombre (sin distinguir mayusculas)
    private void validarDuplicado(Aliment a, int idAExcluir) throws DAOException {
        for (Aliment otro : alimentDao.findAll()) {
            if (otro.getId() != idAExcluir && otro.getAliment().equalsIgnoreCase(a.getAliment())) {
                throw new IllegalArgumentException("Ya existe un alimento con ese nombre.");
            }
        }
    }
```

En `insert()` (después de validar vacíos y categoría, **antes** de confirmar):

```java
            Aliment a = readForm();
            validarLargo(a.getAliment(), MAX_ALIMENT, "El nombre");
            validarDuplicado(a, 0);
            if (!view.confirmInsert()) return;
```

En `modify()`, igual pero excluyendo el seleccionado:

```java
            validarLargo(a.getAliment(), MAX_ALIMENT, "El nombre");
            validarDuplicado(a, selectedAliment.getId());
            if (!view.confirmModify()) return;
```

Y `parseNonNegative` pasa a `parsePositive`:

```java
    private double parsePositive(String text, String field) {
        double value = Double.parseDouble(text.trim().replace(',', '.'));
        if (value <= 0) {
            throw new IllegalArgumentException(field + " deben ser mayores a cero.");
        }
        return value;
    }
```

### 3.2 `PatientController` — H5 y H8

```java
    private static final int MAX_NOMBRE = 80;         // patients.name / lastName
    private static final int MAX_OBSERVACIONES = 500; // patients.observations

    private void validarDuplicado(Patient p, int idAExcluir) throws DAOException {
        for (Patient otro : patientDao.findAll()) {
            if (otro.getId() != idAExcluir
                    && otro.getName().equalsIgnoreCase(p.getName())
                    && otro.getLastName().equalsIgnoreCase(p.getLastName())) {
                throw new IllegalArgumentException("Ya existe un paciente con ese nombre y apellido.");
            }
        }
    }
```

En `insert()` y `modify()`, después de `readForm()`:

```java
            validarLargo(p.getName(), MAX_NOMBRE, "El nombre");
            validarLargo(p.getLastName(), MAX_NOMBRE, "El apellido");
            validarLargo(p.getObservations(), MAX_OBSERVACIONES, "Las observaciones");
            validarDuplicado(p, selectedPatient.getId());   // 0 en insert
```

> [!tip] `getObservations()` puede ser null
> Por eso `validarLargo` empieza con `texto != null`.

### 3.3 `DietController` — H5, H6 y H8

```java
    private static final int MAX_NOMBRE = 50;        // diets.name
    private static final int MAX_DESCRIPCION = 100;  // diets.description
```

`readForm()`:

```java
    private Diet readForm() {
        Diet d = new Diet();
        d.setName(view.getNameText().trim());
        // Un Enter en la textarea no aporta nada en un VARCHAR(100)
        String descripcion = view.getDescriptionText().replaceAll("\\s+", " ").trim();
        d.setDescription(descripcion);
        return d;
    }
```

En `insert()` / `modify()`, después de `readForm()`:

```java
            Diet d = readForm();
            validarLargo(d.getName(), MAX_NOMBRE, "El nombre");
            validarLargo(d.getDescription(), MAX_DESCRIPCION, "La descripcion");
            validarDuplicado(d, selectedDiet.getId());
            if (!view.confirmInsert()) return;
```

Y hay que **agregar el `catch` que falta**:

```java
        } catch (IllegalArgumentException ex) {
            view.showError(ex.getMessage());
        } catch (DAOException ex) {
```

> [!warning] El orden importa
> Primero `readForm()` (que normaliza), después las validaciones, y **al final** `confirmInsert()`. Al revés, el usuario confirma y después le llega el error.

### 3.4 `DietController` — H9

```java
    private List<Aliment> alimentsDisponibles = new ArrayList<>(); // los que faltan en la dieta
    private int totalAliments = 0;                                 // para distinguir los dos vacios

    private void loadAliments() {
        List<Aliment> todos;
        try {
            todos = alimentDao.findAll();
        } catch (DAOException ex) {
            todos = new ArrayList<>();
            view.showError(ex.getMessage());
        }
        totalAliments = todos.size();

        List<Integer> yaEstan = new ArrayList<>();
        if (selectedDiet.getId() != 0) {
            try {
                for (DietDetail d : detailDao.buscarPorDieta(selectedDiet.getId())) {
                    yaEstan.add(d.getAlimento().getId());
                }
            } catch (DAOException ex) {
                view.showError(ex.getMessage());
            }
        }

        alimentsDisponibles = new ArrayList<>();
        for (Aliment a : todos) {
            if (!yaEstan.contains(a.getId())) {
                alimentsDisponibles.add(a);
            }
        }

        List<String> names = new ArrayList<>();
        for (Aliment a : alimentsDisponibles) {
            names.add(String.valueOf(a));
        }
        view.setAlimentOptions(names);
    }
```

`addAliment()` con los dos mensajes:

```java
    private void addAliment() {
        if (selectedDiet.getId() == 0) {
            view.showError("Selecciona una dieta de la tabla primero");
            return;
        }
        if (alimentsDisponibles.isEmpty()) {
            view.showError(totalAliments == 0
                    ? "No hay alimentos cargados. Cargalos primero en la pantalla de Aliments."
                    : "Ya estan todos los alimentos cargados en esta dieta.");
            return;
        }
        int index = view.getAlimentIndex();
        if (index < 0 || index >= alimentsDisponibles.size()) {
            view.showError("Elegi un alimento de la lista.");
            return;
        }
        if (!view.confirmAddAliment()) return;
        try {
            detailDao.insert(new DietDetail(selectedDiet, alimentsDisponibles.get(index)));
            refreshPanel();
        } catch (DuplicateRecordException ex) {
            view.showError("Ese alimento ya forma parte de la dieta.");
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
    }
```

`refreshPanel()` reemplaza a `refreshDetails()` en **todos** estos puntos:

| Método | Línea actual | Cambia a |
|---|---|---|
| constructor | 51 (`refreshTable`) | `refreshPanel()` después de la tabla |
| `search()` | 152 | `refreshPanel()` |
| `selectRow()` | 168 | `refreshPanel()` |
| `addAliment()` | 185 | `refreshPanel()` |
| `removeAliment()` | 203 | `refreshPanel()` |
| `delete()` | 133 | `refreshPanel()` |
| `resetFields()` | 268 | `refreshPanel()` |

### 3.5 `DietView` — H10

Dos métodos nuevos, después de `getAlimentIndex()` (fuera de las zonas `GEN-`):

```java
    //Confirmaciones del panel de alimentos
    public boolean confirmAddAliment() { return javax.swing.JOptionPane.showConfirmDialog(
            this, "Desea agregar este alimento a la dieta?", "Confirmar",
            javax.swing.JOptionPane.OK_CANCEL_OPTION) == javax.swing.JOptionPane.OK_OPTION; }
    public boolean confirmRemoveAliment() { return javax.swing.JOptionPane.showConfirmDialog(
            this, "Desea quitar este alimento de la dieta?", "Confirmar",
            javax.swing.JOptionPane.OK_CANCEL_OPTION) == javax.swing.JOptionPane.OK_OPTION; }
```

No toca el `.form`: son métodos escritos a mano. En `removeAliment()` la confirmación va **después** de validar que hay fila:

```java
        int modelRow = view.getTableDietAliments().convertRowIndexToModel(viewRow);
        DietDetail detail = detailModel.getDetail(modelRow);
        if (!view.confirmRemoveAliment()) return;
        try {
            detailDao.delete(detail.getId());
            refreshPanel();
```

---

## 4. Qué NO se toca

| Cosa | Por qué |
|---|---|
| El DDL y `Sql/nutricion.sql` | los límites ya existen; el arreglo es validar antes, no cambiar columnas |
| Las entidades | no hace falta `@Column(length)` si el controlador valida |
| `DietDetailDAOJPA`, `DAOJpa` | funcionan (el JPQL del detalle ya está bien) |
| `persistence.xml` | local, con credenciales |

Y algo que **no** se arregla con esto:

> [!important] La búsqueda por apellido, por categoría y por nombre sigue sin pantalla
> H2, H3 y H4 dejan los métodos funcionando, pero los tres botones `Search` siguen buscando por **ID**. Implementar los filtros es la Fase 5 de [[plan-crud-bd]] y es un trabajo distinto: tocar la vista, el controlador y el modelo de tabla.

---

## 5. Impacto en el plan de pruebas

### Casos que se desbloquean

| Caso | Antes | Después |
|---|---|---|
| **D7** (borrar dieta sin pacientes) | imposible, error de JPQL | borra los `DietDetail` y después la dieta |
| **D8** (borrar dieta con pacientes) | imposible, error de JPQL | mensaje *"No se puede eliminar la dieta: 1 paciente(s) la tienen asignada."* |

### Casos que cambian de resultado

| Caso | Antes | Después |
|---|---|---|
| P20 (nombre de 81) | error de MySQL | *"El nombre no puede superar los 80 caracteres."* |
| D4 (descripción de 150) | error de MySQL | *"La descripcion no puede superar los 100 caracteres."* |
| D3 (3 renglones) | guarda con `\n` | guarda en **una sola línea** |
| A6 (calorías 0) | guarda | *"Las calorias deben ser mayores a cero."* |
| D12 (agregar repetido) | error al agregar | el alimento **ya no aparece en el combo** |
| D18 (sin alimentos) | un mensaje | dos mensajes según haya o no alimentos en el sistema |
| D13 (Remove) | borra directo | pide confirmación: **Cancelar** no borra |

### Casos nuevos

| # | Caso |
|---|---|
| Q1 | `PruebaBusquedas`: los cuatro métodos devuelven listas, no excepción |
| A17 | Insertar `Manzana` cuando ya existe → *"Ya existe un alimento con ese nombre."* |
| A18 | Insertar `manzana` en minúsculas → también se rechaza |
| P23 | Insertar `Lucía Fernández` repetido → *"Ya existe un paciente con ese nombre y apellido."* |
| P24 | Modify de un paciente sin cambiar el nombre → **no** se rechaza a sí mismo |
| D21 | Insertar `Hipocalórica` repetida → *"Ya existe una dieta con ese nombre."* |
| D22 | Con todos los alimentos ya en la dieta, el combo queda vacío y el mensaje es el de "ya están todos" |

---

## 6. Orden de trabajo

| # | Commit | Archivos | Probás |
|---|---|---|---|
| 0 | `Corrijo los JPQL de busqueda de Patient, Aliment y Diet` | los 3 `DAOJPA` | Q1, **D7 y D8** |
| 1 | `Ajustes en Aliment: largo del nombre, calorias mayores a cero y nombre duplicado` | `AlimentController` | A1-A8, A17, A18 |
| 2 | `Ajustes en Patient: largos y nombre+apellido duplicado` | `PatientController` | P1-P13, P23, P24 |
| 3 | `Ajustes en Diet: largos, descripcion en una linea y nombre duplicado` | `DietController` | D1-D6, D21 |
| 4 | `Panel de alimentos: combo filtrado y confirmaciones` | `DietController` + `DietView` | D10-D20, D22 |

> [!important] Decisión del 2026-11-12: **un solo commit al final**
> El Estudiante decidió agrupar todo en un commit único, con el mensaje corto:
> `Corrijo los JPQL y completo validaciones y busquedas del CRUD`
>
> La tabla de arriba sigue siendo el **orden de implementación y de prueba**, no de commits. El bloque 0 se corre y se verifica primero igual: si el borrado de dietas falla después de arreglar H1, conviene descubrirlo antes de escribir las otras 300 líneas.

El **bloque 4** va último porque es el más riesgoso: si algo sale mal, todo lo anterior ya está estable y probado.

> [!tip] Un bloque, una idea
> Si el 4 se complica, se puede separar en "combo filtrado" y "confirmaciones" para probar. El `.java` de la vista nunca viaja solo: si se toca el diseñador, va con el `.form`.

---

## 7. Riesgos

| Riesgo | Cómo se mitiga |
|---|---|
| Arreglar H1 y descubrir que el delete de `Diet` falla por otra cosa | D7/D8 se corren **inmediatamente** después del commit 0, antes de tocar nada más |
| H9 deja el combo desincronizado con la lista | un solo `refreshPanel()`; ningún `refreshDetails()` suelto |
| H8 rechaza el `Modify` de un registro sin cambios | excluir por `id` en las tres entidades (caso P24) |
| H5 rompe el `INSERT` del script | la descripción de `Vegetariana` tiene 99 caracteres: **pasa**, pero conviene contarla antes de tocar el script |
| H7 cambia un mensaje que ya estaba en las notas | actualizar [[plan-cableado-aliment]] y el plan de pruebas |
| H10 duplica el diálogo con el `confirm` del CRUD | son botones distintos; no se pisan |

**Plan B:** si H9 no cierra antes de la entrega, se revierte sólo ese punto (el combo vuelve a mostrar todos) y se dejan el resto, que son independientes.

---

## 8. Lo que dice el profesor

> pág. 1 y 2 — enunciado_rubrica.pdf
> El CRUD tiene que validar los datos y manejar los errores sin que el usuario vea mensajes técnicos. El trabajo incluye las tres entidades sobre la base de datos.

> pág. 14 y 20 a 23 — Swing_MVC_Interfaces_Java.pdf
> Los diálogos y la captura de datos son responsabilidad de la vista; la regla que decide si un dato es válido, del controlador.

**Qué se toma:** validaciones y mensajes en el controlador, `JOptionPane` en la vista, y el CRUD de las tres entidades como alcance. **Qué agrega el equipo:** los límites, la clave de duplicado y el orden de trabajo.

---

## 9. Ampliación de la IA

> [!note] Por qué un JPQL roto compila
> Es un `String`. Ni el compilador ni NetBeans lo revisan: el error aparece en el `createQuery` de runtime. Por eso los cuatro huecos sobrevivieron a un merge y a varias tandas de pruebas manuales — la única forma de detectarlos es **ejecutar** el método. De ahí la clase `PruebaBusquedas`: convierte tres métodos sin pantalla en algo observable.

> [!note] Validar en el controlador y no en la vista
> Un `DocumentFilter` impediría tipear el carácter 81, pero el usuario no entendería por qué, la regla quedaría en la vista y al pegar texto se comportaría distinto. Validar al guardar permite explicar el límite.

> [!note] Duplicados: controlador hoy, `UNIQUE` mañana
> Comparar contra `findAll()` funciona con 50 registros y no toca el esquema. Con miles sería una consulta por alta. La solución definitiva es un `UNIQUE` en la base **más** un mensaje específico; el día que se migre, el único punto que cambia es `validarDuplicado`.

---

## Relacionado

- [[plan-de-pruebas-crud]] — los 70 casos y los huecos H1 a H12
- [[bug-jpql-dao]] — los cinco JPQL rotos y los `{@link}` de la documentación
- [[plan-crud-bd]] — la Fase 3 (los JPQL) y la Fase 5 (las búsquedas, que esto **no** implementa)
- [[plan-cableado-aliment]] — validaciones y mensajes de `Aliment`
- [[plan-cableado-patient]] — el molde de `parsePositive`
- [[plan-cableado-diet]] — el panel de alimentos y su `refreshDetails`
- [[ejemplo-de-excepciones]] — `IllegalArgumentException` vs `DAOException`
- [[enunciado_rubrica]] — el requisito de validación
- [[MAP]] · [[index]]
