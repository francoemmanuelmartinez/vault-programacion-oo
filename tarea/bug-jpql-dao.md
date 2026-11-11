---
fuente: revisión del código real de `NutritionistApp` en `origin/master` (commits `0f060f5`, `bf52102` y `1f785b6` de RomiM, 2026-11-10 y 2026-11-11)
tags: [tarea, dao, jpa, jpql, bug, revision]
estado: pendiente
actualizado: 2026-11-11
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
>
> Y después fue un paso más allá: en `1f785b6` renombró **también las entidades**
> para que coincidieran con el script (`Aliment.name` → `aliment`, `Diet.nombre` →
> `name`). Eso dejó el esquema alineado y **achicó el arreglo pendiente**, pero no
> tocó una sola línea de JPQL: el `FROM` sigue apuntando a tablas.

> [!note] Alcance de esta revisión
> Los números de línea corresponden a **`origin/master` (`1f785b6`, 2026-11-11
> 12:16)**, o sea al estado que queda **después** del `git pull`. Si todavía no lo
> hiciste, tu copia local tiene la generación anterior del bug y los números no
> coinciden.

**Historia del archivo:**

| Generación | Commit | Cómo estaban los JPQL |
|---|---|---|
| 1.ª | `797b39c` (RomiM) | entidades y campos en **español**: `Paciente`, `apellido`, `Alimento`, `categoria` |
| 2.ª | `0f060f5` (RomiM, rebase del 2026-11-10) | nombres de **tabla SQL**: `patients`, `aliments`, `diets`, `p.id_dieta` |
| — | `bf52102` (RomiM, 2026-11-11) | **no los tocó**: siguen igual |
| — | `1f785b6` (RomiM, 2026-11-11 12:16) | **no los tocó**: siguen igual, pero renombró las entidades de abajo |

## Qué cambió con los pushes del 2026-11-11

RomiM pusheó dos veces el mismo día: `ef8be3a`+`bf52102` (*"conecte las vistas…"*) y `1f785b6` (*"Vistas de alimentos y pacientes ya estan funcionales"*). **Ninguno de los cinco JPQL se corrigió**, pero sí se movió todo lo demás de esta nota:

| Tema | Estado al `1f785b6` |
|---|---|
| Los 5 JPQL con nombres de tabla | ❌ **siguen rotos, idénticos** |
| Desalineado `nutricion.sql` vs entidades | ✅ **resuelto al revés**: RomiM alineó las **entidades** con el script (`Aliment.aliment`, `Diet.name` + `description`) |
| `PatientDAOInMemory.buscarPorDieta` con `UnsupportedOperationException` | ❌ sigue — y la clase quedó sin uso (`AppNutritionist` pasó a `PatientDAOJPA` en `bf52102`) |
| `DietDetailDAOJPA.buscarPorDieta` sin consulta | ❌ sigue |
| Nombre de la unidad de persistencia | ✅ **arreglado** en `ef8be3a`: `nutricionPU` (antes `persistence.xml`) |
| Entidades declaradas en el XML | ✅ **arreglado** en `ef8be3a`: `<class>` ×4 + `<exclude-unlisted-classes>` |
| Generación del esquema | ✅ **arreglado** en `ef8be3a`: `create-or-extend-tables` (antes `create`) |
| `DAOJpa` con `em` fuera del `try` | ✅ **arreglado** en `bf52102`: la base caída se reporta como `DAOException` y no como crash |
| `DietDetailDAO` con `Long` | ✅ ya estaba arreglado en `0f060f5` |

> [!danger] Y apareció un problema nuevo: el build apunta a la máquina de RomiM
> `nbproject/project.properties` trae rutas absolutas a `C:\Users\DELL01\Downloads\`
> para `eclipselink-2.7.16.jar` y `mysql-connector-j-8.2.0.jar`, y esos jars **no
> están en `lib/`**. Como `AppNutritionist` ahora usa los DAO de JPA, el proyecto
> **compila pero no arranca** en cualquier otra máquina
> (`No Persistence provider for EntityManager named nutricionPU`).

## Resumen: 5 consultas rotas (antes eran 4)

| # | Archivo | Línea | Consulta actual | Qué está mal | Consulta correcta |
|---|---|---|---|---|---|
| 1 | `dao/PatientDAOJPA.java` | **19** | `SELECT p FROM patients p WHERE LOWER(p.lastName) LIKE :ap` | entidad `patients` no existe | `SELECT p FROM Patient p WHERE LOWER(p.lastName) LIKE :ap` |
| 2 | `dao/PatientDAOJPA.java` | **26** | `SELECT p FROM patients p WHERE p.id_dieta = :idDieta` | entidad `patients` **y** campo `id_dieta` | `SELECT p FROM Patient p WHERE p.diet.id = :idDieta` |
| 3 | `dao/AlimentDAOJPA.java` | **19** | `SELECT a FROM aliments a WHERE a.category = :cat` | entidad `aliments` no existe | `SELECT a FROM Aliment a WHERE LOWER(a.category) LIKE :cat` |
| 4 | `dao/DietDAOJPA.java` | **19** | `SELECT d FROM diets d WHERE LOWER(d.name) LIKE :nom` | entidad `diets` no existe (`d.name` **sí** es correcto desde `1f785b6`) | `SELECT d FROM Diet d WHERE LOWER(d.name) LIKE :nom` |
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

### 4. Diet, el caso que se arregló solo a medias

`src/appNutritionist/dao/DietDAOJPA.java` — método `buscarPorNombre` (declarado en `dao/DietDAO.java:21`):

```java
// línea 19 — ACTUAL
"SELECT d FROM diets d WHERE LOWER(d.name) LIKE :nom",

// CORRECTO
"SELECT d FROM Diet d WHERE LOWER(d.name) LIKE :nom",
```

Esta consulta tiene una historia que vale contar, porque fue **el error que se movió de lugar**:

| Commit | `Diet` tenía | La consulta decía | Veredicto |
|---|---|---|---|
| `797b39c` | `nombre` | `d.nombre` | ✅ campo bien, entidad mal (`Dieta`) |
| `0f060f5` | `nombre` | `d.name` | ❌ **dos** errores: RomiM cambió el campo creyendo que arreglaba |
| `1f785b6` | **`name`** | `d.name` | ⚠️ **un** error: el campo ahora sí coincide |

O sea: en `0f060f5` esta consulta era la peor del lote; desde `1f785b6` es igual al resto. RomiM renombró `Diet.nombre` → `Diet.name` y agregó `description`, y de paso **validó el campo de la consulta sin tocar la consulta**.

> [!tip] La lección
> El bug no desapareció: se **movió al modelo**. Mientras el JPQL usaba `d.name` y la entidad decía `nombre`, había que arreglar el string. Ahora la entidad dice `name`, así que el único error que queda es el `FROM diets`. **El mismo arreglo es cada vez más chico**, pero sigue sin estar hecho.

Lo único que falta es la entidad: `diets` (tabla) → `Diet` (entidad, `model/Diet.java:17`).

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

## El script SQL ya no está desalineado (lo resolvió `1f785b6`)

Este era el motivo de fondo de la nota anterior. **Ya está cerrado, y al revés de lo que se había recomendado.** RomiM eligió el script como fuente de verdad y adaptó las entidades:

| Tabla en `nutricion.sql` | Columna | Entidad | Atributo Java | ¿Coincide? |
|---|---|---|---|---|
| `diets` (:9) | `name` | `Diet` | **`name`** (antes `nombre`) | ✅ |
| `diets` (:10) | `description` | `Diet` | **`description`** (nueva) | ✅ |
| `aliments` (:15) | `aliment` | `Aliment` | **`aliment`** (antes `name`) | ✅ |
| `patients` (:23) | `lastName` | `Patient` | `lastName` | ✅ |
| `patients` (:27) | `id_diet` | `Patient` | `diet` + `@JoinColumn("id_diet")` | ✅ |
| `dietsdetails` (:35-36) | `id_diet`, `id_aliment` | `DietDetail` | `diet`, `aliment` (`@JoinColumn`) | ✅ |

Ninguna entidad necesitó `@Column(name = ...)`: el nombre del atributo **es** el de la columna. O sea que el mapeo quedó 1:1 con el script, y el script es ahora la referencia.

> [!danger] No des este consejo al revés
> La versión anterior de esta nota recomendaba renombrar las **columnas del script** (`diets.name` → `nombre`, `aliments.aliment` → `name`). Con `1f785b6` eso **rompe el proyecto**: la entidad dice `aliment` y `name`, y el `INSERT` volvería a fallar con `Field 'aliment' doesn't have a default value`. Si alguien te pasa ese consejo, ignorálo.

> [!warning] Consecuencia del renombre que sí te puede morder
> `Aliment.getName()` y `setName()` **ya no existen**: ahora son `getAliment()` y `setAliment()` (`model/Aliment.java:38-39`). `Diet` perdió `getNombre()` y el constructor pasó de `Diet(String nombre)` a `Diet(String name, String description)`. Copiar ejemplos viejos —del vault, de commits anteriores o de internet— da `cannot find symbol`.

> [!warning] `create-or-extend-tables` ya no reconcilia nada, pero deja basura
> En `0f060f5` el XML tenía `javax.persistence.schema-generation.database.action=create`,
> que **borraba y recreaba** el esquema en cada arranque: los `INSERT` de
> `nutricion.sql` se perdían. En `ef8be3a` RomiM lo cambió por
> `eclipselink.ddl-generation=create-or-extend-tables`, que **no borra datos**.
> Decisión correcta.
>
> El detalle: si tu base se creó **antes** de `1f785b6`, EclipseLink le había sumado
> `diets.nombre` y `aliments.name` como columnas extra. Ahora la entidad lee `name` y
> `aliment`, así que esas columnas quedan **huérfanas y vacías**: no rompen nada, pero
> ensucian el `SHOW COLUMNS` de la verificación y confunden. Si ves columnas de más,
> `DROP DATABASE nutricion` y corré el script: es un minuto y deja el esquema idéntico
> al de RomiM.

## Otros `UnsupportedOperationException`: la misma bomba, en otra capa

Los DAO en memoria también quedaron con stubs sin implementar:

| Archivo | Línea | Método |
|---|---|---|
| `dao/PatientDAOInMemory.java` | **66-67** | `buscarPorDieta(Integer)` |
| `dao/DietDetailDAOJPA.java` | **18** | `buscarPorDieta(Integer)` |

Es probable que **este** haya sido el error que RomiM vio después del rebase: al agregar `buscarPorDieta` a la interfaz `PatientDAO`, `PatientDAOInMemory` dejó de compilar (*"la clase no es abstracta y no sobrescribe el método abstracto"*) hasta que NetBeans le generó el stub. El error era suyo y quedó resuelto en lo que pusheó, pero **el stub quedó adentro**.

`PatientDAOInMemory.buscarPorDieta` **ya no lo usa nadie**: desde `bf52102`, `AppNutritionist` construye `PatientDAOJPA`. La clase quedó huérfana pero sigue compilando, y el stub sigue ahí adentro. Si alguien la vuelve a enchufar (o el profesor la abre para leer el patrón DAO), revienta sin base de datos de por medio.

## Extra: Javadoc que apunta a clases que no existen

**Ocho** `{@link}` (no cinco: la versión anterior de esta nota estaba incompleta) quedaron apuntando a clases que no existen en el proyecto. Verificado sobre `1f785b6` con `Select-String -Pattern '\{@link '`:

| Archivo | Línea | Dice | Debería decir |
|---|---|---|---|
| `dao/AlimentDAO.java` | 10 | `{@link Alimento}` | `{@link Aliment}` |
| `dao/AlimentDAOJPA.java` | 8 | `{@link AlimentoDAO}` | `{@link AlimentDAO}` |
| `dao/DAOJpa.java` | 10 | `{@link GenericDAO}` | `{@link DAO}` |
| `dao/DietDAO.java` | **8** | `{@link Dieta}` | `{@link Diet}` |
| `dao/DietDAOJPA.java` | 8 | `{@link DietaDAO}` | `{@link DietDAO}` |
| `dao/DietDetailDAO.java` | 8 | `{@link DetalleDieta}` | `{@link DietDetail}` |
| `dao/PatientDAO.java` | 8 | `{@link Paciente}` | `{@link Patient}` |
| `model/Diet.java` | 11 | `{@link DetalleDieta}` | `{@link DietDetail}` |

Los **tres** que faltaban en la versión anterior eran `AlimentDAOJPA:8`, `DAOJpa:10` y `DietDAOJPA:8` (los tres son `@link` a interfaces que se llaman distinto). Aparte, la línea de `DietDAO` estaba mal: **corrió de 11 a 8** porque `1f785b6` le borró el header de licencia al archivo.

No rompen `javac` normal, pero **sí rompen `Generate Javadoc`** (y `javac -Xdoclint:all`). Es la misma deuda de nombres que ya se está limpiando en el resto del proyecto.

> [!info] Por qué NetBeans los pinta en rojo
> `{@link X}` no es texto: es una referencia que el IDE **resuelve** contra las clases del proyecto para poder generar el hipervínculo. Si `X` no existe, NetBeans lo muestra en rojo (y `javadoc` lo reporta como *"reference not found"*). El código compila igual: `javac` sobre `1f785b6` da `ExitCode: 0`. Si un compañero te dice "me aparece un nombre en rojo", casi seguro es esto y no un error de compilación.
>
> `EntityManager` en `DAOJpa:11` **no** está en esta lista: está importado (`javax.persistence.EntityManager`) y resuelve bien. `DietDetail:15` (`{@link Diet}` + `{@link Aliment}`) también resuelve.

Ninguna de las ocho afecta la ejecución ni la conexión a la base: se arreglan en 8 reemplazos de texto, sin tocar lógica.

## Lo que ya está resuelto (respecto de la versión anterior de esta nota)

| Deuda anterior | Estado hoy |
|---|---|
| `DietDetailDAO.buscarPorDieta(Long)` con un DAO de `Integer` | ✅ **Corregido**: ahora es `Integer` (`dao/DietDetailDAO.java:18`) |
| `DietDetail` sin relación con `Diet` | ✅ **Corregido**: `@ManyToOne` a `Diet` y a `Aliment` (`model/DietDetail.java:30-36`) |
| Unidad de persistencia sin nombre coincidente con `JPAUtil` | ✅ **Corregido** en `ef8be3a`: ahora es `nutricionPU`, igual que `util/JPAUtil.java:12` |
| Entidades sin declarar en el `persistence.xml` | ✅ **Corregido** en `ef8be3a`: cuatro `<class>` + `<exclude-unlisted-classes>` |
| `schema-generation.database.action=create` (borraba datos) | ✅ **Corregido** en `ef8be3a`: `eclipselink.ddl-generation=create-or-extend-tables` |
| `EntityManager` creado fuera del `try` en `DAOJpa` | ✅ **Corregido** en `bf52102`: la base caída ahora sale como `DAOException`, no como crash |
| Los cuatro JPQL en español | ⚠️ **Reemplazados por otros cuatro igual de inválidos** (tablas en vez de entidades), y en `bf52102` no se tocaron |

> [!tip] Lo que estos arreglos habilitan
> Antes de `ef8be3a` el `EntityManagerFactory` **no se podía crear nunca**, porque
> `JPAUtil` pedía `nutricionPU` y el XML se llamaba `persistence.xml`. Nadie lo notó
> porque la app corría con `PatientDAOInMemory`. Desde `bf52102` la app arranca
> contra MySQL de verdad, así que **el JPQL roto pasó de teoría a deuda urgente**:
> es lo único que separa el CRUD de las búsquedas.

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
  la columna; ahí es donde el script y las entidades se desalinearon hasta `1f785b6`.
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
- **`@Column` es la alternativa que NO se eligió.** Se podría haber dejado `Diet.nombre` con `@Column(name = "name")` y `Aliment.name` con `@Column(name = "aliment")`, y el script quedaba intacto. Funciona, pero deja dos nombres para el mismo dato en todo el proyecto — exactamente lo que se viene limpiando desde `b357923`. El equipo eligió lo contrario en `1f785b6`: renombrar los atributos para que el nombre Java **sea** el de la columna. Las dos son válidas; la del equipo tiene menos vocabulario pero acopla el modelo al script.
- Como esto lo decidió otra integrante del equipo, no lo reviertas de prepo: si te parece mejor el otro camino, plantearlo como issue o PR propio, no en el medio de otro commit.

## Checklist

- [ ] `PatientDAOJPA:19` → `FROM Patient p WHERE LOWER(p.lastName) LIKE :ap`
- [ ] `PatientDAOJPA:26` → `FROM Patient p WHERE p.diet.id = :idDieta`
- [ ] `AlimentDAOJPA:19` → `FROM Aliment a WHERE LOWER(a.category) LIKE :cat`
- [ ] `DietDAOJPA:19` → `FROM Diet d WHERE LOWER(d.name) LIKE :nom`
- [ ] `DietDetailDAOJPA:17-18` → escribir el `consultar(...)` y sacar el `throw`
- [ ] `PatientDAOInMemory:66-67` → implementar `buscarPorDieta` (o borrar la clase: quedó sin uso)
- [x] Alinear el esquema: RomiM alineó las **entidades** con el script en `1f785b6` (`Aliment.aliment`, `Diet.name` + `description`)
- [x] Decidir la generación de esquema: `create-or-extend-tables` (hecho en `ef8be3a`)
- [ ] **Librerías**: copiar `eclipselink-2.7.16.jar` y el driver MySQL a `lib/` y sacar las rutas absolutas de `nbproject/project.properties`
- [ ] Arreglar los **ocho** `{@link}` rotos (`Alimento`, `AlimentoDAO`, `GenericDAO`, `Dieta`, `DietaDAO`, `DetalleDieta` ×2, `Paciente`) — la tabla está en [[#Extra: Javadoc que apunta a clases que no existen]]
- [ ] Hablar con RomiM el campo `cantidad` de `DietDetail` antes de cablear dietas
- [ ] Compilar y probar cada búsqueda con la base arriba
- [ ] Acordar con RomiM quién hace el commit

## Relacionado

- [[plan-cableado-patient]] — la búsqueda por apellido que va a estrenar esta consulta en la Etapa B
- [[plan-cableado-aliment]] — la búsqueda por categoría (`AlimentDAOJPA:19`)
- [[hoja-de-ruta-7-a-9]] — receta C.6 (pasar a MySQL) y el estado de cada bloque
