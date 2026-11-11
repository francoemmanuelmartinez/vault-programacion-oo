---
fuente: auditoría del código real de `NutritionistApp` en `origin/master` (`1f785b6`) + JPA_Hibernate_DAO.pdf (pág. 9 a 16), JDBC_DAO_MySQL.pdf y enunciado_rubrica.pdf (pág. 1 y 2)
tags: [tarea, plan, jpa, mysql, crud, dao]
estado: vigente
actualizado: 2026-11-11
aliases: [plan-crud-bd, plan-crud-contra-bd, plan-mysql]
---

# Plan para dejar el CRUD de `Patient` y `Aliment` funcionando contra MySQL

> [!info] Qué resuelve esta nota
> Lo que falta entre "las vistas ya hacen select, insert y update" y "el CRUD está cerrado y pusheado". Está escrito para ejecutarlo en NetBeans **sin volver al chat**, y para pushear al final.
>
> **Punto de partida:** `origin/master` = `1f785b6` (RomiM, 2026-11-11 12:16).

> [!warning] Versión 2 de este plan — cambió una fase entera
> La v1 recomendaba **renombrar las columnas del script SQL** para que coincidieran con las entidades. RomiM resolvió el desalineado **al revés**: renombró las **entidades** para que coincidieran con el script (`Aliment.name` → `aliment`, `Diet.nombre` → `name`, y agregó `description`).
>
> **Como el esquema ya está alineado, la Fase 2 de la v1 quedó cancelada.** Si alguien renombra las columnas del script ahora, **rompe lo que funciona**. Lo que sigue del plan está reescrito sobre la realidad de `1f785b6`.

> [!warning] Procedencia del contenido
> - **Del profesor:** que JPQL trabaja sobre entidades y atributos (pág. 13 a 16 — `JPA_Hibernate_DAO.pdf`) y la separación en capas Vista → Controlador → DAO.
> - **Criterio de equipo / IA (NO del profesor):** el reparto del trabajo, el orden de las fases, los textos de los botones y la tabla de troubleshooting.

---

## 0. Diagnóstico al 2026-11-11 12:16

RomiM probó select, insert y update. **Eso está confirmado y funciona.** Lo que queda:

| Operación | Estado | Comentario |
|---|---|---|
| Abrir la app **en la máquina de RomiM** | ✅ | tiene los dos jars en sus `Downloads` |
| Abrir la app **en tu máquina** | ❌ | `project.properties` apunta a `C:\Users\DELL01\Downloads\…` y `lib/` sólo tiene 2 jars |
| Listar pacientes / alimentos | ✅ probado | `findAll()` usa `clase.getSimpleName()` (`DAOJpa:60`) |
| Buscar por ID | ✅ | `em.find(clase, id)`, sin JPQL |
| Insertar | ✅ probado por RomiM | `em.persist()` |
| Modificar | ✅ probado por RomiM | `em.merge()` |
| Asignar dieta a un paciente | ✅ probado por RomiM | combo + `setDieta()` en `readForm()` |
| **Eliminar** | ⚠️ **sin probar** | falta el caso de la clave foránea |
| Búsqueda por apellido | ❌ | `PatientDAOJPA:19` → `FROM patients` |
| Búsqueda por dieta | ❌ | `PatientDAOJPA:26` → `FROM patients … p.id_dieta` |
| Búsqueda por categoría | ❌ | `AlimentDAOJPA:19` → `FROM aliments` |
| Búsqueda por nombre de dieta | ❌ | `DietDAOJPA:19` → `FROM diets` |
| Detalle por dieta | ❌ | `DietDetailDAOJPA:18` → `throw new UnsupportedOperationException` |
| Vista de dietas | ❌ no existe | `DietController` vacío, sin `DietView`, `getButtonDiets()` sin listener |

> [!important] Lo que falta es más chico de lo que parece
> **Cuatro** nombres de entidad en los JPQL, **un** camino de navegación (`p.id_dieta`), **un** método sin cuerpo y **un** classpath. Nada más. El CRUD de `Patient` y `Aliment` está a media hora de quedar cerrado.

---

## 1. Fase 1 — Que el proyecto arranque en tu máquina

Esta fase no cambió: sigue siendo la que te bloquea.

### 1.1 Traer los cambios

```powershell
cd C:\Users\games\Desktop\Proyectos\NutritionistApp
git pull --ff-only
```

Es un fast-forward (`behind 7`, `ahead 0`, árbol limpio). El pull reemplaza `PatientController`, `AlimentController`, `PatientView`, `AlimentView`, `AppNutritionist`, `Aliment`, `Diet` y `DietDAO` por las versiones de RomiM: son una evolución de las tuyas, no hay nada perdido.

### 1.2 Arrancar MySQL

Tenés **MySQL 9.7** instalado pero detenido:

```powershell
Start-Service MySQL97
```

Si falla por permisos, PowerShell como administrador.

> [!warning] La contraseña `2711` está commiteada
> `persistence.xml` trae `password = 2711`, la de RomiM. Si tu `root` usa otra, cambiala **sólo para probar** y **no la commitees** (revisá el `git status` antes de cada commit).

### 1.3 Los dos jars

| Jar | De dónde |
|---|---|
| `eclipselink-2.7.16.jar` | EclipseLink JPA 2.7.16 (implementa JPA 2.2). **No lo tenés en ninguna carpeta.** |
| `mysql-connector-j-9.6.0.jar` | ya lo tenés en `C:\Users\games\Desktop\Proyectos\mysql-connector-j-9.6.0.jar` |

Copiá los dos a `lib/`.

### 1.4 Arreglar el classpath

`nbproject/project.properties` sigue con:

```
file.reference.eclipselink-2.7.16.jar=C:\\Users\\DELL01\\Downloads\\eclipselink-2.7.16.jar
file.reference.mysql-connector-j-8.2.0.jar=C:\\Users\\DELL01\\Downloads\\mysql-connector-j-8.2.0.jar
```

En NetBeans, **no editando el archivo** (lo reescribe):

1. Clic derecho en el proyecto → **Properties**
2. **Libraries** → pestaña **Compile**
3. Seleccioná las dos entradas rotas → **Remove**
4. **Add JAR/Folder** → `lib\eclipselink-2.7.16.jar` y `lib\mysql-connector-j-9.6.0.jar`
5. **OK**

Quedan como rutas relativas. Commiteá el `.properties` **y** los dos jars.

### 1.5 Verificación

```powershell
javac --release 25 -cp "lib/jakarta.persistence-2.2.3.jar;lib/flatlaf-3.5.4.jar" `
      -d build/classes (Get-ChildItem src -Recurse -Filter *.java).FullName
```

0 errores (para **compilar** no hacen falta los jars nuevos; para **ejecutar** sí). Después: **Run Project** y que abra `PatientView` con los 3 pacientes del script.

---

## 2. Fase 2 — Verificar el esquema (ya está alineado, no lo toques)

### 2.1 Lo que hizo RomiM

Renombró las entidades para que coincidan con las columnas del script:

| Antes | Ahora | Archivo |
|---|---|---|
| `Aliment.name` | **`Aliment.aliment`** | `model/Aliment.java:21` |
| `Diet.nombre` | **`Diet.name`** | `model/Diet.java:22` |
| — | **`Diet.description`** (nueva) | `model/Diet.java:23` |
| `getName()` / `setName()` de `Aliment` | **`getAliment()` / `setAliment()`** | `model/Aliment.java:38-39` |
| `Diet(String nombre)` | **`Diet(String name, String description)`** | `model/Diet.java:28` |

O sea: el esquema `Sql/nutricion.sql` **es la referencia** y las entidades se adaptaron a él. Eso deja el mapeo 1:1:

| Tabla | Columnas del script | Atributos de la entidad | ¿Coincide? |
|---|---|---|---|
| `diets` | `id`, `name`, `description` | `id`, `name`, `description` | ✅ |
| `aliments` | `id`, `aliment`, `category`, `caloriesPerServing` | `id`, `aliment`, `category`, `caloriesPerServing` | ✅ |
| `patients` | `id`, `name`, `lastName`, `weight`, `height`, `observations`, `id_diet` | `id`, `name`, `lastName`, `weight`, `height`, `observations`, `diet` + `@JoinColumn("id_diet")` | ✅ |
| `dietsdetails` | `id`, `id_diet`, `id_aliment` | `id`, `diet` + `@JoinColumn("id_diet")`, `aliment` + `@JoinColumn("id_aliment")` | ✅ |

> [!danger] No renombres las columnas del script
> La v1 de este plan recomendaba `diets.name` → `nombre` y `aliments.aliment` → `name`. **Eso ahora es al revés y rompería todo.** Si la base ya está creada, no la borres: el esquema actual es el correcto.

### 2.2 Consecuencias del renombre que te van a morder

| Si escribís… | Compila | Es lo correcto |
|---|---|---|
| `aliment.getName()` | ❌ | `aliment.getAliment()` |
| `aliment.setName(...)` | ❌ | `aliment.setAliment(...)` |
| `diet.getNombre()` | ❌ | `diet.getName()` |
| `new Diet("Vegetariana")` | ❌ | `new Diet("Vegetariana", "Sin carnes")` |

Y cuidado con las notas viejas del vault y los ejemplos de internet: casi todos usan `getName()`. Los archivos del proyecto que ya se actualizaron son `AlimentController` (`readForm`/`showAliment`), `AlimentTableModel` (columna 1) y `PatientTableModel` (columna 6 → `getDieta().getName()`).

### 2.3 Verificación de la fase

Con la base creada por el script:

```sql
USE nutricion;
SHOW COLUMNS FROM Aliments;   -- id, aliment, category, caloriesPerServing
SHOW COLUMNS FROM Diets;      -- id, name, description

SELECT p.lastName, d.name
FROM Patients p LEFT JOIN Diets d ON d.id = p.id_diet;
-- Fernández → Hipocalórica ; Gómez → Hipocalórica ; Pereyra → NULL
```

Y en la app: la columna **Diet** tiene que mostrar `Hipocalórica` dos veces y `-` para Pereyra. Si muestra vacío, el esquema no es el que espera la entidad.

> [!note] Único desalineado que queda: mayúsculas en el nombre de tabla
> El script crea `aliments`, `diets`, `patients`, `dietsdetails`; las entidades declaran `@Table(name = "Aliments")`, `"Diets"`, `"Patients"`, `"DietsDetails"`. En Windows MySQL no distingue mayúsculas, así que funciona. En Linux o en un servidor real **fallaría**. Si querés cerrarlo, normalizá los cuatro `@Table` a minúscula (4 líneas, y hay que recrear la base). No es urgente ni afecta la entrega.

---

## 3. Fase 3 — Los cinco JPQL

### 3.1 Por qué ahora

El CRUD no los usa, así que la app parece terminada con ellos rotos. Pero la rúbrica pide búsquedas, y dejar cuatro métodos que revientan es peor que no tenerlos. **Son cuatro nombres de entidad y un camino.**

> pág. 13 a 16 — JPA_Hibernate_DAO.pdf
> «JPQL trabaja sobre entidades y atributos en lugar de tablas y columnas.»

Después de `FROM` va **la clase** (o el `name` del `@Entity`), no el `@Table`. Y después del alias, **el atributo Java**.

### 3.2 Los cambios

> [!tip] Buena noticia respecto de la v1
> El renombre de RomiM **arregló dos atributos de paso**: `DietDAOJPA` usaba `d.name` (que antes era incorrecto porque el atributo era `nombre`) y ahora es correcto. Lo mismo con `AlimentDAOJPA` y `a.category`. Sólo queda mal el nombre de la entidad.

**`dao/AlimentDAOJPA.java:19`** — sólo la entidad:

```java
// antes
"SELECT a FROM aliments a WHERE a.category = :cat", "cat", categoria);
// después
"SELECT a FROM Aliment a WHERE LOWER(a.category) LIKE :cat","cat", "%" + categoria.toLowerCase() + "%");
```

> [!warning] `LIKE` cambia el comportamiento
> Pasás de igualdad exacta a coincidencia parcial. Es lo que hacen las otras búsquedas y lo que espera un usuario que escribe "frut". Decidilo a conciencia.

**`dao/DietDAOJPA.java:19`** — sólo la entidad:

```java
// antes
"SELECT d FROM diets d WHERE LOWER(d.name) LIKE :nom","nom", "%" + nombre.toLowerCase() + "%");
// después
"SELECT d FROM Diet d WHERE LOWER(d.name) LIKE :nom","nom", "%" + nombre.toLowerCase() + "%");
```

**`dao/PatientDAOJPA.java:19`** — sólo la entidad:

```java
// antes
"SELECT p FROM patients p WHERE LOWER(p.lastName) LIKE :ap",
// después
"SELECT p FROM Patient p WHERE LOWER(p.lastName) LIKE :ap",
```

**`dao/PatientDAOJPA.java:26`** — dos cosas: entidad y camino:

```java
// antes
"SELECT p FROM patients p WHERE p.id_dieta = :idDieta","idDieta", idDieta);
// después
"SELECT p FROM Patient p WHERE p.diet.id = :idDieta","idDieta", idDieta);
```

`id_dieta` es el nombre de la **columna** (`@JoinColumn(name = "id_diet")`); el **atributo** es `diet` (`Patient.java:30`).

**`dao/DietDetailDAOJPA.java:16-18`** — no hay consulta, hay que escribirla:

```java
@Override
public List<DietDetail> buscarPorDieta(Integer idDiet) throws DAOException {
    return consultar("SELECT dd FROM DietDetail dd WHERE dd.diet.id = :idDiet",
                     "idDiet", idDiet);
}
```

`DietDetail` ya tiene `@ManyToOne Diet diet` (`model/DietDetail.java:30-32`): no hay que tocar el modelo.

### 3.3 Verificación de la fase

```powershell
Select-String -Path "src\appNutritionist\dao\*.java" -Pattern 'FROM (patients|aliments|diets|dietsDetails)\b'
```

No tiene que devolver nada. La prueba real llega en la Fase 5.

---

## 4. Fase 4 — Probar lo que falta: el DELETE

Insert, update, select y la asignación de dieta ya los probó RomiM. Falta el borrado, que es donde están las claves foráneas.

> [!important] La prueba que vale
> **Cerrá y volvé a abrir la app después de cada operación**, y confirmá con un `SELECT` en MySQL. Es lo único que distingue "está en la base" de "está en memoria".

| # | Prueba | Resultado esperado |
|---|---|---|
| 1 | Abrir la app | 3 pacientes del script; columna Diet con `Hipocalórica`, `Hipocalórica`, `-` |
| 2 | Insert de un paciente con dieta | fila nueva + mensaje con el id; al reabrir, sigue |
| 3 | Insert sin apellido | "El nombre y el apellido son obligatorios" |
| 4 | Insert con peso `0` | "El peso debe ser mayor a cero" |
| 5 | Insert con peso `62,5` | acepta la coma |
| 6 | Clic en una fila | se cargan los 7 campos y el combo muestra la dieta correcta |
| 7 | Ordenar por *Last Name* y hacer clic | carga el paciente correcto |
| 8 | Modify del peso | persiste al reabrir |
| 9 | Modify de la dieta | persiste; **y no borra la dieta de los otros pacientes** |
| 10 | **Delete de un paciente → Cancelar** | no borra |
| 11 | **Delete de un paciente → OK** | desaparece de la tabla y de la base |
| 12 | Aliments: insert `Manzana` / `Frutas` / `95` | guarda; al reabrir, sigue |
| 13 | Aliments: insert con calorías `abc` | "Las calorias deben ser un numero" |
| 14 | Aliments: Modify | persiste |
| 15 | Aliments: **Delete de `Palta`** (id 7, está en la dieta 2) | mensaje "No se pudo eliminar… puede estar incluido en alguna dieta", **sin crash** |
| 16 | Aliments: **Delete de un alimento recién creado** | borra |
| 17 | Patients → Aliments → Patients | los datos siguen (los DAO se comparten) |
| 18 | Paciente sin dieta | la columna Diet muestra `-` y al reabrir sigue así |

> [!note] El `Delete` de `Aliment` ya tiene mensaje específico
> `AlimentController.delete()` (línea ~114) captura `DAOException` y muestra *"No se pudo eliminar el alimento. Puede estar incluido en alguna dieta."* RomiM quiere reemplazarlo por una excepción propia; si lo hace antes que vos, no toques esa parte.

---

## 5. Fase 5 — Cablear las búsquedas (recomendado)

> [!question] Por qué está en el plan si pediste "el CRUD"
> Porque sin un llamador **no se puede demostrar que los cinco JPQL funcionan**, y en la defensa van a preguntar por las búsquedas. Es media hora y cierra el circuito: la consulta baja hasta el DAO y la tabla muestra el resultado.

### 5.1 La idea

El mismo patrón en las dos pantallas, sin agregar controles: **Search filtra la tabla**, y el clic en la fila carga el formulario (eso ya lo hace `selectRow()`).

| Pantalla | Botón | Lee | Llama a | Resultado |
|---|---|---|---|---|
| `PatientView` | `Search by Last Name` | `lastNameTextField` | `buscarPorApellido` | tabla filtrada |
| `AlimentView` | `Search by Category` | `categoryTextField` | `buscarPorCategoria` | tabla filtrada |

### 5.2 `PatientController.search()`

Reemplazá el método actual (el que parsea el ID). `List` ya está importado.

```java
private void search(){
    String lastName = view.getLastNameText().trim();

    // Campo vacío = quitar el filtro y mostrar todos otra vez
    if(lastName.isEmpty()){
        refreshTable();
        resetFields();
        return;
    }
    try{
        List<Patient> encontrados = patientDao.buscarPorApellido(lastName);
        if(encontrados.isEmpty()){
            view.showError("No hay pacientes con apellido \"" + lastName + "\".");
            return;
        }
        tableModel.setPatients(encontrados);
        selectedPatient = new Patient();
        view.getTablePatients().clearSelection();
        if(encontrados.size() == 1){                 // uno solo: lo deja listo en el formulario
            selectedPatient = encontrados.get(0);
            showPatient(selectedPatient);
            view.getTablePatients().setRowSelectionInterval(0, 0);
        }
        view.showInfo(encontrados.size() + " paciente(s) encontrado(s).");
    } catch(DAOException ex){
        view.showError(ex.getMessage());
    }
}
```

### 5.3 `AlimentController.search()`

`AlimentController` **no importa `java.util.List`**: agregalo arriba, junto a los otros `import`.

```java
import java.util.List;
```

y después:

```java
private void search() {
    String category = view.getCategoryText().trim();

    if (category.isEmpty()) {
        refreshTable();
        resetFields();
        return;
    }
    try {
        List<Aliment> encontrados = alimentDao.buscarPorCategoria(category);
        if (encontrados.isEmpty()) {
            view.showError("No hay alimentos de la categoria \"" + category + "\".");
            return;
        }
        tableModel.setAliments(encontrados);
        selectedAliment = new Aliment();
        view.getTableAliments().clearSelection();
        if (encontrados.size() == 1) {
            selectedAliment = encontrados.get(0);
            showAliment(selectedAliment);
            view.getTableAliments().setRowSelectionInterval(0, 0);
        }
        view.showInfo(encontrados.size() + " alimento(s) encontrado(s).");
    } catch (DAOException ex) {
        view.showError(ex.getMessage());
    }
}
```

### 5.4 En el diseñador

| Vista | Componente | Cambio |
|---|---|---|
| `PatientView` | `searchButton` | texto → `Search by Last Name`; tooltip → `Input a last name and click this button to search` |
| `AlimentView` | `searchButton` | texto → `Search by Category`; tooltip → `Input a category and click this button to search` |
| ambas | campo **ID** | desmarcar `editable` (Properties). Ya nadie lo lee; si queda editable, el profesor va a tipear un ID esperando una búsqueda por ID |

### 5.5 Pruebas de la fase

| # | Prueba | Resultado esperado |
|---|---|---|
| 19 | Search con `fer` | queda sólo Fernández |
| 20 | Search con `PEREZ` (inexistente) | "No hay pacientes con apellido…" |
| 21 | Search con el campo vacío | vuelve a mostrar los 3 |
| 22 | Search de un alimento en `Frutas` | quedan Manzana y Banana |
| 23 | Clic en la fila después de filtrar, y Modify | modifica el alimento correcto |
| 24 | Insert después de un filtro | la tabla se repuebla completa (el filtro se pierde: es esperado) |

> [!note] Mayúsculas y tildes
> `LOWER(...)` resuelve `Perez` vs `PEREZ`, pero **no** `Perez` vs `Pérez`. Si hace falta, se normaliza con `java.text.Normalizer`, pero no es requisito.

---

## 6. Limpieza (deudas chicas)

| # | Qué | Dónde |
|---|---|---|
| 1 | `buscarPorDieta` lanza `UnsupportedOperationException` y la clase quedó **sin uso**. Implementalo (7 líneas) para que no sea una bomba | `dao/PatientDAOInMemory.java:66` |
| 2 | **Ocho** `{@link}` a nombres viejos → rompen `Generate Javadoc` | `AlimentDAO:10` (`Alimento`), `AlimentDAOJPA:8` (`AlimentoDAO`), `DAOJpa:10` (`GenericDAO`), `DietDAO:8` (`Dieta`), `DietDAOJPA:8` (`DietaDAO`), `DietDetailDAO:8` (`DetalleDieta`), `PatientDAO:8` (`Paciente`), `Diet:11` (`DetalleDieta`) — tabla completa en [[bug-jpql-dao]] |
| 3 | `rowCount="4"` en el modelo del diseñador (regenera 4 filas de `null`) | `AlimentView.form:236` |
| 4 | `"Informacion"` sin tilde | `AlimentView.showInfo` |
| 5 | `selectedDiet()` no valida `index - 1 < diets.size()`; hoy es seguro sólo porque `setDietOptions` resetea el combo | `PatientController:208` |
| 6 | Si la base está caída salen **dos** diálogos (uno de `loadDiets`, otro de `refreshTable`) | `PatientController` constructor |
| 7 | Los botones **Diets** no hacen nada todavía; deshabilitarlos hasta que exista la vista | las dos vistas |
| 8 | `Aliment.java` quedó **sin salto de línea final** (y `DietDAO` perdió el header de licencia: cosmético) | `model/Aliment.java` |

Código del punto 1:

```java
@Override
public List<Patient> buscarPorDieta(Integer idDiet) throws DAOException {
    List<Patient> encontrados = new ArrayList<>();
    if (idDiet == null) return encontrados;
    for (Patient p : patients) {
        if (p.getDieta() != null && p.getDieta().getId() == idDiet.intValue()) {
            encontrados.add(p);
        }
    }
    return encontrados;
}
```

---

## 7. Reparto del trabajo con RomiM

Ella ya avisó qué va a hacer: **la vista de dietas** (`DietView` + `DietController`, hoy vacíos) y **excepciones específicas** para el borrado con clave foránea.

> [!warning] Van a tocar los mismos archivos
> Para cablear el botón **Diets**, RomiM tiene que agregar listeners en `PatientController` y `AlimentController` — los mismos archivos donde vos vas a cambiar `search()`. Git probablemente lo resuelva solo (son métodos distintos), pero si los dos editan la misma zona, hay conflicto.

Reparto sugerido:

| Quién | Qué |
|---|---|
| **Vos** | Fase 1 (build portable), Fase 3 (JPQL), Fase 4 (probar delete), Fase 6 (limpieza) |
| **RomiM** | `DietView` + `DietController`, excepciones de FK |
| Después | Fase 5 (búsquedas), cuando su vista esté pusheada |

Si querés hacer la Fase 5 igual, hacé **dos commits separados**: primero el de la Fase 1 + 3 + 6, y después el de las búsquedas. Si aparece un conflicto, se resuelve sólo el segundo.

---

## 8. Commit y push

### 8.1 Antes de commitear

```powershell
git status -s
```

Revisá que **no** aparezcan `dist/`, `build/`, `nbproject/private/`, ni una `persistence.xml` con **tu** contraseña de MySQL.

### 8.2 Qué entra

| Archivo | Por qué |
|---|---|
| `lib/eclipselink-2.7.16.jar`, `lib/mysql-connector-j-9.6.0.jar` | sin esto nadie más puede correr el proyecto |
| `nbproject/project.properties` | saca las rutas absolutas de RomiM |
| `dao/PatientDAOJPA.java`, `AlimentDAOJPA.java`, `DietDAOJPA.java`, `DietDetailDAOJPA.java` | los cinco JPQL |
| `dao/PatientDAOInMemory.java` | `buscarPorDieta` implementado |
| `view/PatientView.java` + `.form`, `AlimentView.java` + `.form` | textos, tooltips e `editable` (si hacés la Fase 5) |
| `controller/*.java` | sólo si hacés la Fase 5 |

### 8.3 Mensaje sugerido

```
Arregle los JPQL de los DAO (usaban nombres de tabla en vez de entidades), complete
DietDetailDAOJPA.buscarPorDieta, agregue los jars de EclipseLink y MySQL a lib/ y
saque las rutas absolutas de project.properties.

Con esto la app corre en cualquier maquina despues del pull y las busquedas por
apellido, categoria, nombre de dieta y dieta funcionan contra MySQL.
```

### 8.4 Qué **no** hacer

- No `amend` ni reescribir commits de RomiM: el tuyo va **encima**.
- No renombrar las columnas del script (Fase 2: el esquema actual es el correcto).
- No commitear tu contraseña de MySQL.
- No borrar `Sql/nutricion.sql`: es la única forma de recrear los datos de prueba.

### 8.5 Avisarle a RomiM

1. **Los jars cambiaron de nombre y de lugar**: ahora en `lib/` con rutas relativas. Ella tenía `mysql-connector-j-8.2.0.jar` desde sus `Downloads`; después del pull tiene que re-agregar el suyo o usar el que commiteás.
2. **Los JPQL quedaron arreglados**: si ella iba a tocarlos, que no.
3. Si su base tiene datos cargados a mano, que no corra ningún script que borre tablas.

---

## 9. Si algo falla

| Mensaje | Causa | Solución |
|---|---|---|
| `No Persistence provider for EntityManager named nutricionPU` | EclipseLink no está en el classpath | Fase 1.4 |
| `Communications link failure` / `Connection refused` | MySQL detenido | `Start-Service MySQL97` |
| `Access denied for user 'root'@'localhost'` | contraseña distinta de `2711` | corregir la property (sin commitearla) |
| `Unknown column 't1.name' in 'field list'` | base creada con el esquema **viejo** (antes del renombre) | recrear la base con el script actual; **no** renombrar el script |
| `Field 'nombre' doesn't have a default value` | base vieja con `diets.nombre` | ídem |
| `The abstract schema type 'patients' is unknown` | JPQL sin arreglar | Fase 3 |
| `Unknown column 'p.id_dieta'` (o similar) | falta cambiar `p.id_dieta` por `p.diet.id` | Fase 3 |
| `Cannot delete or update a parent row: a foreign key constraint fails` | borraste un alimento que está en una dieta | es **esperado**: el controlador ya lo informa |
| `java.lang.reflect.InaccessibleObjectException` | EclipseLink 2.7.x con Java 25 | Properties → **Run** → VM Options: `--add-opens java.base/java.lang=ALL-UNNAMED` |
| La tabla se llena pero al reabrir no está el dato nuevo | estás usando un DAO en memoria | `AppNutritionist` tiene que decir `new PatientDAOJPA()` |
| `cannot find symbol: method getName()` en `Aliment` | quedó código con el nombre viejo | ahora es `getAliment()` (Fase 2.2) |

---

## 10. Orden de trabajo

| Fase | Qué | Se puede probar | Tiempo |
|---|---|---|---|
| 1 | pull + MySQL + jars + classpath | la app abre con los 3 pacientes | 20 min |
| 2 | verificar el esquema (sin tocar nada) | `SHOW COLUMNS` y la columna Diet | 5 min |
| 3 | los cinco JPQL | compila + `grep` vacío | 15 min |
| 4 | las 18 pruebas del CRUD (foco en delete) | todo persiste | 25 min |
| 5 | las dos búsquedas (opcional) | pruebas 19 a 24 | 30 min |
| 6 | limpieza | `Generate Javadoc` sin errores | 20 min |
| 7 | commit y push | `git log` limpio | 10 min |

**Total: entre 1½ y 2 horas**, con la Fase 5 incluida.

---

## Relacionado

- [[bug-jpql-dao]] — el detalle de los cinco JPQL y el error conceptual de usar nombres de tabla
- [[plan-cableado-patient]] · [[plan-cableado-aliment]] — el cableado ya hecho y por qué está hecho así
- [[hoja-de-ruta-7-a-9]] — receta C.6 (pasar a MySQL) y el resto del checklist; falta el bloque de dietas
- [[tutorial-dao-desde-cero]] · [[ejemplo-de-excepciones]] · [[convenciones]]
