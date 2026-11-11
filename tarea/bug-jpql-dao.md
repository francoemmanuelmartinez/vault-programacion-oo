---
fuente: revisión del código real de `NutritionistApp` en `origin/master` (commit `0f060f5`, el que llega con el pull de RomiM del 2026-11-10)
tags: [tarea, dao, jpa, jpql, bug, revision]
estado: pendiente
actualizado: 2026-11-10
aliases: [bug-jpql-dao, jpql-roto]
---

# Bug de JPQL: las consultas usan nombres de tabla en vez de entidades

> [!danger] Compila, pero explota en runtime — y sigue igual de roto que antes
> Los DAO de JPA escriben las consultas con **nombres de tabla** (`patients`,
> `aliments`, `diets`, y el campo `p.id_dieta`) en vez de **entidades y atributos**
> (`Patient`, `Aliment`, `Diet`, `p.diet.id`). `javac` no puede verlo porque son
> strings: el error aparece recién cuando el controlador ejecuta la consulta.
>
> **Lo importante:** esto no es el mismo bug de antes con otro texto. En la versión
> anterior los nombres estaban en **español** (`Paciente`, `apellido`); RomiM los
> cambió y los dejó **alineados con el script SQL** (`nutricion.sql`), que es
> justamente la capa equivocada. El bug cambió de disfraz, no de naturaleza.

> [!note] Alcance de esta revisión
> Los números de línea corresponden a **`origin/master` (`0f060f5`)**, o sea al
> estado que queda **después** del `git pull`. Si todavía no lo hiciste, tu copia
> local tiene la generación anterior del bug y los números no coinciden.

**Historia del archivo:**

| Generación | Commit | Cómo estaban los JPQL |
|---|---|---|
| 1.ª | `797b39c` (RomiM) | entidades y campos en **español**: `Paciente`, `apellido`, `Alimento`, `categoria` |
| 2.ª | `0f060f5` (RomiM, rebase del 2026-11-10) | nombres de **tabla SQL**: `patients`, `aliments`, `diets`, `p.id_dieta` |

## Resumen: 5 consultas rotas (antes eran 4)

| # | Archivo | Línea | Consulta actual | Qué está mal | Consulta correcta |
|---|---|---|---|---|---|
| 1 | `dao/PatientDAOJPA.java` | **19** | `SELECT p FROM patients p WHERE LOWER(p.lastName) LIKE :ap` | entidad `patients` no existe | `SELECT p FROM Patient p WHERE LOWER(p.lastName) LIKE :ap` |
| 2 | `dao/PatientDAOJPA.java` | **26** | `SELECT p FROM patients p WHERE p.id_dieta = :idDieta` | entidad `patients` **y** campo `id_dieta` | `SELECT p FROM Patient p WHERE p.diet.id = :idDieta` |
| 3 | `dao/AlimentDAOJPA.java` | **19** | `SELECT a FROM aliments a WHERE a.category = :cat` | entidad `aliments` no existe | `SELECT a FROM Aliment a WHERE LOWER(a.category) LIKE :cat` |
| 4 | `dao/DietDAOJPA.java` | **19** | `SELECT d FROM diets d WHERE LOWER(d.name) LIKE :nom` | entidad `diets` **y** campo `d.name` | `SELECT d FROM Diet d WHERE LOWER(d.nombre) LIKE :nom` |
| 5 | `dao/DietDetailDAOJPA.java` | **17-18** | *(ya no hay consulta)* | RomiM borró el JPQL y dejó `UnsupportedOperationException` | ver [[#5. DietDetail, el JPQL que desapareció]] |

Todas se ejecutarían a través de `DAOJpa.consultar(...)`, en `dao/DAOJpa.java:116-125`:

```java
protected List<T> consultar(String jpql, String parametro, Object valor) throws DAOException {
    EntityManager em = JPAUtil.crearEntityManager();
    try {
        return em.createQuery(jpql, clase).setParameter(parametro, valor).getResultList();
    } catch (RuntimeException e) {
        throw new DAOException("Error al consultar " + clase.getSimpleName(), e);
    } finally {
        em.close();
    }
}
```

## Detalle por archivo

### 1. Patient, búsqueda por apellido

`src/appNutritionist/dao/PatientDAOJPA.java` — método `buscarPorApellido` (declarado en `dao/PatientDAO.java:18`):

```java
// línea 19 — ACTUAL
"SELECT p FROM patients p WHERE LOWER(p.lastName) LIKE :ap",

// CORRECTO
"SELECT p FROM Patient p WHERE LOWER(p.lastName) LIKE :ap",
```

El campo `lastName` **sí está bien**: la entidad lo tiene así (`model/Patient.java:22`). Lo único roto es el nombre de la entidad, que quedó en minúscula como la tabla `patients` de `nutricion.sql:20`.

### 2. Patient, búsqueda por dieta — **nueva**

`src/appNutritionist/dao/PatientDAOJPA.java:24-27` — método `buscarPorDieta`, agregado en este commit (declarado en `dao/PatientDAO.java:27`):

```java
// línea 26 — ACTUAL
"SELECT p FROM patients p WHERE p.id_dieta = :idDieta"

// CORRECTO
"SELECT p FROM Patient p WHERE p.diet.id = :idDieta"
```

Acá hay **dos** errores: la entidad (`patients` → `Patient`) y el camino de la navegación. `id_dieta` es el nombre de la **columna** en la tabla (`@JoinColumn(name = "id_diet")`), no el nombre del atributo. El atributo se llama `diet` (`model/Patient.java:30`), así que en JPQL se navega `p.diet.id`.

> [!tip] Esta consulta ya tiene a quién apuntar
> A diferencia de la versión anterior de esta nota, la relación **sí existe**:
> `@ManyToOne(fetch = FetchType.EAGER)` + `@JoinColumn(name = "id_diet")` sobre
> `Patient.diet` (`model/Patient.java:28-30`). Así que el arreglo es de una línea,
> no hay que tocar el modelo.

### 3. Aliment

`src/appNutritionist/dao/AlimentDAOJPA.java` — método `buscarPorCategoria` (declarado en `dao/AlimentDAO.java:20`):

```java
// línea 19 — ACTUAL
"SELECT a FROM aliments a WHERE a.category = :cat",

// CORRECTO
"SELECT a FROM Aliment a WHERE LOWER(a.category) LIKE :cat",
```

El campo `category` **está bien** (`model/Aliment.java:22`). Sólo falla la entidad. De paso: es la única de las tres búsquedas que no normaliza a minúsculas ni usa comodines; conviene dejarla igual que las otras dos para que las tres se comporten igual.

### 4. Diet, el caso donde el "arreglo" agregó un error

`src/appNutritionist/dao/DietDAOJPA.java` — método `buscarPorNombre` (declarado en `dao/DietDAO.java:21`):

```java
// línea 19 — ACTUAL
"SELECT d FROM diets d WHERE LOWER(d.name) LIKE :nom",

// CORRECTO
"SELECT d FROM Diet d WHERE LOWER(d.nombre) LIKE :nom",
```

Esta es la más instructiva del lote, porque RomiM **cambió `d.nombre` por `d.name` creyendo que arreglaba algo**, y empeoró: agregó un error donde no había ninguno.

- **Entidad:** `diets` → `Diet` (`model/Diet.java:17`).
- **Atributo:** `d.name` → `d.nombre`. La clase `Diet` nunca se renombró: sigue con `private String nombre;` y `getNombre()` (`model/Diet.java:22` y `:35`).

El campo **sí** se llama `name`… pero **en la tabla** (`nutricion.sql:9`), no en la entidad. Ese es exactamente el error conceptual de todo este commit: confundir el vocabulario de JPQL con el de SQL.

### 5. DietDetail, el JPQL que desapareció

`src/appNutritionist/dao/DietDetailDAOJPA.java` — método `buscarPorDieta` (declarado en `dao/DietDetailDAO.java:18`):

```java
// líneas 17-18 — ACTUAL
public List<DietDetail> buscarPorDieta(Integer idDiet) throws DAOException {
    throw new UnsupportedOperationException("Not supported yet.");    }
```

RomiM **borró la consulta** en vez de corregirla. El método sigue compilando (el stub lo genera NetBeans) pero explota apenas lo llamen.

La buena noticia: acá el modelo **ya está listo**. `DietDetail` se reescribió como tabla pivot con las dos relaciones (`model/DietDetail.java:30-36`):

```java
@ManyToOne(optional = false, fetch = FetchType.EAGER)
@JoinColumn(name = "id_diet", nullable = false)
private Diet diet;

@ManyToOne(optional = false, fetch = FetchType.EAGER)
@JoinColumn(name = "id_aliment", nullable = false)
private Aliment aliment;
```

Así que la consulta es una sola línea:

```java
// en dao/DietDetailDAOJPA.java
@Override
public List<DietDetail> buscarPorDieta(Integer idDiet) throws DAOException {
    return consultar("SELECT dd FROM DietDetail dd WHERE dd.diet.id = :idDiet",
                     "idDiet", idDiet);
}
```

> [!warning] Detalle de formato
> La llave quedó pegada al `throw` (`...yet.");    }`). Compila, pero ensucia el diff
> y hace parecer que el método está sin terminar. Y así está.

> [!warning] Lo que cambió respecto de la nota anterior
> La 1.ª versión de esta nota decía que `DietDetail` no tenía campo `dieta` y que
> había que crearlo. **Eso ya está resuelto.** Lo que sigue pendiente es otra cosa:
> el campo `cantidad` no existe, y el diseño del equipo ([[plan-cableado-patient]])
> definía `DietDetail` como *un alimento + cantidad*. Hoy es un pivot puro
> `Diet`–`Aliment` con `UniqueConstraint({"id_diet", "id_aliment"})`
> (`model/DietDetail.java:21-22`), o sea que **un alimento no puede repetirse en la
> misma dieta**, pero no hay forma de decir "200 g de arroz". Hay que hablarlo antes
> de cablear dietas.

## El script SQL no coincide con las entidades

Este es el motivo de fondo del commit, y conviene tenerlo escrito porque va a morder en la Etapa B. `Sql/nutricion.sql` (nuevo, 62 líneas) crea las tablas con nombres de columna que **no son** los atributos de las entidades:

| Tabla en `nutricion.sql` | Columna | Entidad | Atributo Java | ¿Coincide? |
|---|---|---|---|---|
| `diets` (:9) | `name` | `Diet` | `nombre` | ❌ |
| `aliments` (:15) | `aliment` | `Aliment` | `name` | ❌ |
| `patients` (:23) | `lastName` | `Patient` | `lastName` | ✅ |
| `patients` (:27) | `id_diet` | `Patient` | `diet` + `@JoinColumn("id_diet")` | ✅ |
| `dietsdetails` (:35-36) | `id_diet`, `id_aliment` | `DietDetail` | `diet`, `aliment` (`@JoinColumn`) | ✅ |

O sea: **dos de las cinco tablas están desalineadas en una columna**. Y como ninguna entidad usa `@Column(name = ...)`, EclipseLink busca `Diet.nombre` en una tabla que tiene `name`.

> [!danger] Y hay algo peor: `create`
> `src/META-INF/persistence.xml` tiene
> `<property name="javax.persistence.schema-generation.database.action" value="create"/>`.
> Con `create`, EclipseLink **borra y recrea** el esquema a partir de las entidades
> en **cada arranque**: los datos de los `INSERT` de `nutricion.sql` se pierden, y
> las tablas quedan con las columnas de las entidades (`nombre`, `name`), no con las
> del script. Para la Etapa B hay que decidir una sola fuente de verdad: o el script
> (action `none`) o las entidades (action `update`), pero no las dos peleando.

## Otros `UnsupportedOperationException`: la misma bomba, en otra capa

Los DAO en memoria también quedaron con stubs sin implementar:

| Archivo | Línea | Método |
|---|---|---|
| `dao/PatientDAOInMemory.java` | **66-67** | `buscarPorDieta(Integer)` |
| `dao/DietDetailDAOJPA.java` | **18** | `buscarPorDieta(Integer)` |

Es probable que **este** haya sido el error que RomiM vio después del rebase: al agregar `buscarPorDieta` a la interfaz `PatientDAO`, `PatientDAOInMemory` dejó de compilar (*"la clase no es abstracta y no sobrescribe el método abstracto"*) hasta que NetBeans le generó el stub. El error era suyo y quedó resuelto en lo que pusheó, pero **el stub quedó adentro**.

`PatientDAOInMemory.buscarPorDieta` es el más peligroso de los dos: es el DAO que la app usa **hoy** (`AppNutritionist.java:23`). Si algún día un controlador llama a ese método, la app revienta sin base de datos de por medio.

## Extra: Javadoc que apunta a clases que no existen

Cinco `{@link}` quedaron con los nombres viejos en español:

| Archivo | Línea | Referencia rota |
|---|---|---|
| `dao/AlimentDAO.java` | 10 | `{@link Alimento}` |
| `dao/DietDAO.java` | 11 | `{@link Dieta}` |
| `dao/PatientDAO.java` | 8 | `{@link Paciente}` |
| `dao/DietDetailDAO.java` | 8 | `{@link DetalleDieta}` |
| `model/Diet.java` | 11 | `{@link DetalleDieta}` |

No rompen `javac` normal, pero **sí rompen `Generate Javadoc`** (y `javac -Xdoclint:all`). Es la misma deuda de nombres que ya se está limpiando en el resto del proyecto.

## Lo que ya está resuelto (respecto de la versión anterior de esta nota)

| Deuda anterior | Estado hoy |
|---|---|
| `DietDetailDAO.buscarPorDieta(Long)` con un DAO de `Integer` | ✅ **Corregido**: ahora es `Integer` (`dao/DietDetailDAO.java:18`) |
| `DietDetail` sin relación con `Diet` | ✅ **Corregido**: `@ManyToOne` a `Diet` y a `Aliment` (`model/DietDetail.java:30-36`) |
| Los cuatro JPQL en español | ⚠️ **Reemplazados por otros cuatro igual de inválidos** (tablas en vez de entidades) |

## Por qué `javac` no lo detecta

La consulta es un `String`. Para el compilador es un texto cualquiera; el que lo
interpreta es el proveedor JPA (EclipseLink, según `src/META-INF/persistence.xml`),
y eso pasa **en tiempo de ejecución**, dentro del `EntityManager`.

## Quién lo va a disparar

| Consulta | Se ejecuta cuando… |
|---|---|
| `PatientDAOJPA.buscarPorApellido` | **primero que nada** en la Etapa B: es la búsqueda que va a usar el botón `Search by Last Name` (ver [[plan-cableado-patient]]) |
| `PatientDAOJPA.buscarPorDieta` | se quiera listar los pacientes de una dieta (pantalla de dietas) |
| `AlimentDAOJPA.buscarPorCategoria` | se cablee la búsqueda por categoría en `Aliment` (ver [[plan-cableado-aliment]]) |
| `DietDAOJPA.buscarPorNombre` | exista `DietView` / la pantalla de dietas |
| `DietDetailDAOJPA.buscarPorDieta` | se arme el detalle de una dieta — **hoy ni siquiera hay consulta** |

El `insert`, `update`, `delete` y `findById` **no** están afectados: `DAOJpa` genérico
usa `clase.getSimpleName()` (`dao/DAOJpa.java:60`), que sí devuelve `Aliment`,
`Diet`, `DietDetail` y `Patient`. O sea que **el CRUD básico funciona y sólo
fallan las búsquedas**.

## Cómo verificarlo sin adivinar

1. **Compilar** — da 0 errores, y eso es justamente el problema:

   ```powershell
   javac --release 25 -cp "lib/jakarta.persistence-2.2.3.jar;lib/flatlaf-3.5.4.jar" `
         -d build/classes (Get-ChildItem src -Recurse -Filter *.java).FullName
   ```

2. **Buscar los nombres de tabla dentro de los strings**, que es un `grep` de dos segundos:

   ```powershell
   Select-String -Path "src\appNutritionist\dao\*.java" -Pattern 'FROM (patients|aliments|diets|dietsdetails|dietsDetails)\b'
   ```

   Todo lo que aparezca es un JPQL inválido, sin excepciones.

3. **Correr el método** con la base levantada: EclipseLink tira
   *"Problem compiling query… The abstract schema type 'patients' is unknown"*.

## Base en la cátedra

> pág. 13 a 16 — JPA_Hibernate_DAO.pdf
> «JPQL trabaja sobre entidades y atributos en lugar de tablas y columnas.»

Es decir: el nombre que va después de `FROM` es el **de la clase** (o el `name`
del `@Entity`, si se lo pone), y el que va después del alias es el **del campo
Java**, no el de la columna. Los dos ejemplos del PDF usan `Persona`, `p.apellido`,
que son los nombres reales de esa clase.

## Ampliación de la IA

Lo que sigue es conocimiento general de JPA, **no** contenido del profesor:

- **JPQL y SQL tienen vocabularios distintos.** `FROM patients` es SQL válido y JPQL
  inválido. El puente entre los dos mundos son las anotaciones: `@Entity` define el
  nombre de entidad, `@Table` el de la tabla, y `@Column` (o el nombre del atributo
  por defecto) el de la columna. Cuando no hay `@Column`, el nombre Java **es** el de
  la columna, y ahí es donde el script SQL se desalinea.
- **Mensaje de EclipseLink:** `The abstract schema type 'patients' is unknown`.
  Hibernate dice `patients is not mapped`. Los dos apuntan a lo mismo.
- **Mayúsculas en nombres de tabla:** `nutricion.sql` crea `diets`, `aliments`,
  `patients` y `dietsdetails` en minúscula, mientras las entidades declaran
  `@Table(name = "Diets")`, `"Aliments"`, `"Patients"` y `"DietsDetails"`. En Windows
  MySQL es case-insensitive (`lower_case_table_names=1`) y esto pasa inadvertido;
  en Linux o en un servidor real **no**. La tabla `dietsDetails` además mezcla
  camelCase con minúscula, lo que la vuelve frágil incluso en Windows.
- **`schema-generation.database.action`**: `create` borra y recrea en cada arranque;
  `update` agrega lo que falta sin borrar; `none` no toca nada y deja el esquema en
  manos del script. Para una entrega con datos de prueba cargados a mano, `none`
  (o `update`) es lo razonable.
- Si se decidiera mantener el script como fuente de verdad, la alternativa es
  `@Column(name = "name")` en `Diet.nombre` y `@Column(name = "aliment")` en
  `Aliment.name`. Funciona, pero deja dos nombres para el mismo dato en todo el
  proyecto — exactamente lo que se viene limpiando desde `b357923`.
- Como esto lo escribió otra integrante del equipo, conviene proponerlo como issue
  o PR propio, no cambiarlo de prepo en el medio de otro commit.

## Checklist

- [ ] `PatientDAOJPA:19` → `FROM Patient p WHERE LOWER(p.lastName) LIKE :ap`
- [ ] `PatientDAOJPA:26` → `FROM Patient p WHERE p.diet.id = :idDieta`
- [ ] `AlimentDAOJPA:19` → `FROM Aliment a WHERE LOWER(a.category) LIKE :cat`
- [ ] `DietDAOJPA:19` → `FROM Diet d WHERE LOWER(d.nombre) LIKE :nom`
- [ ] `DietDetailDAOJPA:17-18` → escribir el `consultar(...)` y sacar el `throw`
- [ ] `PatientDAOInMemory:66-67` → implementar `buscarPorDieta` sobre la lista en memoria
- [ ] Alinear `Sql/nutricion.sql` con las entidades (`diets.name` → `nombre`, `aliments.aliment` → `name`)
- [ ] Decidir `schema-generation.database.action`: `create` hoy pisa los datos en cada arranque
- [ ] Arreglar los cinco `{@link}` en español (`Alimento`, `Dieta`, `Paciente`, `DetalleDieta`)
- [ ] Hablar con RomiM el campo `cantidad` de `DietDetail` antes de cablear dietas
- [ ] Compilar y probar cada búsqueda con la base arriba
- [ ] Acordar con RomiM quién hace el commit

## Relacionado

- [[plan-cableado-patient]] — la búsqueda por apellido que va a estrenar esta consulta en la Etapa B
- [[plan-cableado-aliment]] — la búsqueda por categoría (`AlimentDAOJPA:19`)
- [[hoja-de-ruta-7-a-9]] — receta C.6 (pasar a MySQL) y el estado de cada bloque
