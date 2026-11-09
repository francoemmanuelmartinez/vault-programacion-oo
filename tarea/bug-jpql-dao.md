---
fuente: revisión del código real de `NutritionistApp` (commit `008edbf` + cambios locales del 2026-11-08/09)
tags: [tarea, dao, jpa, jpql, bug, revision]
estado: pendiente
aliases: [bug-jpql-dao, jpql-roto]
---

# Bug de JPQL: las consultas usan nombres en español

> [!danger] Compila, pero explota en runtime
> Los cuatro DAO de JPA tienen consultas escritas con **nombres en español**
> (`Alimento`, `Dieta`, `DetalleDieta`, `Paciente`) y **campos que no existen**
> (`categoria`, `dieta`, `apellido`). `javac` no puede verlo porque son strings:
> el error aparece recién cuando el controlador ejecuta la consulta.

**Origen:** commit `797b39c` de RomiM (*"Agregue metodo toString a las entidades…"*).
No lo introdujo el renombrado de las vistas y no está relacionado con él.

## Resumen

| # | Archivo | Línea | Consulta actual | Error | Consulta correcta |
|---|---|---|---|---|---|
| 1 | `dao/AlimentDAOJPA.java` | **19** | `SELECT a FROM Alimento a WHERE a.categoria = :cat` | entidad `Alimento` y campo `categoria` no existen | `SELECT a FROM Aliment a WHERE a.category = :cat` |
| 2 | `dao/DietDAOJPA.java` | **19** | `SELECT d FROM Dieta d WHERE LOWER(d.nombre) LIKE :nom` | entidad `Dieta` no existe (el campo `nombre` **sí**) | `SELECT d FROM Diet d WHERE LOWER(d.nombre) LIKE :nom` |
| 3 | `dao/DietDetailDAOJPA.java` | **19** | `SELECT d FROM DetalleDieta d WHERE d.dieta.id = :idDieta` | entidad `DetalleDieta` y campo `dieta` no existen | ver [[#3. DietDetail, el caso que no se arregla renombrando]] |
| 4 | `dao/PatientDAOJPA.java` | **19** | `SELECT p FROM Paciente p WHERE LOWER(p.apellido) LIKE :ap` | entidad `Paciente` y campo `apellido` no existen | `SELECT p FROM Patient p WHERE LOWER(p.lastName) LIKE :ap` |

Todas se ejecutan a través de `DAOJpa.consultar(...)`, en `dao/DAOJpa.java:117-126`:

```java
protected List<T> consultar(String jpql, String parametro, Object valor) throws DAOException {
    EntityManager em = JPAUtil.crearEntityManager();
    try {
        return em.createQuery(jpql, clase).setParameter(parametro, valor).getResultList();
    ...
```

## Detalle por archivo

### 1. Aliment

`src/appNutritionist/dao/AlimentDAOJPA.java` — método `buscarPorCategoria` (declarado en `dao/AlimentDAO.java:20`):

```java
// línea 19 — ACTUAL
"SELECT a FROM Alimento a WHERE a.categoria = :cat",

// CORRECTO
"SELECT a FROM Aliment a WHERE a.category = :cat",
```

Por qué: la entidad se llama `Aliment` (`model/Aliment.java:16`, sin nombre en el `@Entity`, así que JPA usa el nombre de la clase) y los campos son `name` y `category` (`model/Aliment.java:21-22`).

### 2. Diet

`src/appNutritionist/dao/DietDAOJPA.java` — método `buscarPorNombre` (`dao/DietDAO.java:21`):

```java
// línea 19 — ACTUAL
"SELECT d FROM Dieta d WHERE LOWER(d.nombre) LIKE :nom",

// CORRECTO
"SELECT d FROM Diet d WHERE LOWER(d.nombre) LIKE :nom",
```

Por qué: la entidad es `Diet` (`model/Diet.java:17`) pero el **atributo** `nombre` sí existe tal cual (`model/Diet.java:22`). Es el único de los cuatro donde el campo está bien; sólo falla el nombre de la entidad.

### 3. DietDetail, el caso que no se arregla renombrando

`src/appNutritionist/dao/DietDetailDAOJPA.java` — método `buscarPorDieta` (`dao/DietDetailDAO.java:21`):

```java
// línea 19 — ACTUAL
"SELECT d FROM DetalleDieta d WHERE d.dieta.id = :idDieta",
```

Acá hay **dos** problemas:

1. La entidad es `DietDetail`, no `DetalleDieta`.
2. `DietDetail` **no tiene ningún campo `dieta`**. Sus atributos hoy son sólo `id`, `nombre` y `aliments` (`model/DietDetail.java:22-24`):

```java
private int id;
private String nombre;
private List<Aliment> aliments = new ArrayList<>();
```

O sea que **la relación `Diet 1—N DietDetail` todavía no está mapeada**. El arreglo no es de una línea: primero hay que agregar el campo con su anotación, y después la consulta queda así:

```java
// en model/DietDetail.java
@ManyToOne
@JoinColumn(name = "diet_id")
private Diet diet;

// en dao/DietDetailDAOJPA.java
"SELECT d FROM DietDetail d WHERE d.diet.id = :idDieta",
```

> [!warning] Esto contradice el plan del equipo
> En [[plan-cableado-patient]] la relación quedó definida como `Diet 1—N DietDetail`
> con `cascade = ALL` + `orphanRemoval` del lado de `Diet`, y `DietDetail` como
> **un alimento + cantidad**. La implementación de RomiM tiene `List<Aliment>` y no
> tiene ni la relación ni la cantidad. Hay que hablarlo antes de tocar esta consulta.

### 4. Patient

`src/appNutritionist/dao/PatientDAOJPA.java` — método `buscarPorApellido` (`dao/PatientDAO.java:18`):

```java
// línea 19 — ACTUAL
"SELECT p FROM Paciente p WHERE LOWER(p.apellido) LIKE :ap",

// CORRECTO
"SELECT p FROM Patient p WHERE LOWER(p.lastName) LIKE :ap",
```

Por qué: entidad `Patient` (`model/Patient.java:12`) y el apellido se llama `lastName` (`model/Patient.java:18`), no `apellido`.

## Por qué `javac` no lo detecta

La consulta es un `String`. Para el compilador es un texto cualquiera; el que lo
interpreta es el proveedor JPA (EclipseLink, según `src/META-INF/persistence.xml`),
y eso pasa **en tiempo de ejecución**, dentro del `EntityManager`.

## Quién lo va a disparar

| Consulta | Se ejecuta cuando… |
|---|---|
| `AlimentDAOJPA.buscarPorCategoria` | se cablee el `searchButton` de `AlimentView` y se busque por categoría |
| `PatientDAOJPA.buscarPorApellido` | se cablee el `searchButton` de `PatientView` (paso A del [[plan-cableado-patient]]) |
| `DietDAOJPA.buscarPorNombre` | exista `DietView` / la pantalla de dietas |
| `DietDetailDAOJPA.buscarPorDieta` | se arme el detalle de una dieta |

El `insert`, `update`, `delete` y `find` **no** están afectados: `DAOJpa` genérico
usa `clase.getSimpleName()` (`dao/DAOJpa.java:60`), que sí devuelve `Aliment`,
`Diet`, `DietDetail` y `Patient`. O sea que **el CRUD básico funciona y sólo
fallan las búsquedas**: si el equipo probó el alta, no vio el bug.

## Extra: `Long` en un DAO de `Integer`

`DietDetailDAO.buscarPorDieta(Long idDieta)` (`dao/DietDetailDAO.java:21`) recibe
`Long`, pero el DAO está parametrizado con `Integer`:

```java
public class DietDetailDAOJPA extends DAOJpa<DietDetail, Integer> implements DietDetailDAO
```

Y el `id` de todas las entidades es `int`. Cuando se arregle la consulta, conviene
pasar ese parámetro a `int` / `Integer` para que coincida con el resto del DAO.

## Cómo verificarlo sin adivinar

1. **Compilar** — va a dar 0 errores, y eso es justamente el problema.
2. **Correr el método** con la base levantada: el error de EclipseLink tiene la forma
   *"Problem compiling query… The abstract schema type 'Alimento' is unknown"*.
3. O, más barato: `grep` de los nombres en español dentro de `dao/` y confirmar que
   cada entidad de la consulta coincida con la clase y sus atributos.

## Base en la cátedra

> pág. 13 a 16 — JPA_Hibernate_DAO.pdf
> «JPQL trabaja sobre entidades y atributos en lugar de tablas y columnas.»

Es decir: el nombre que va después de `FROM` es el **de la clase** (o el `name`
del `@Entity`, si se lo pone), y el que va después del alias es el **del campo
Java**, no el de la columna. Los dos ejemplos del PDF usan `Persona`, `p.apellido`,
que son los nombres reales de esa clase.

## Ampliación de la IA

Lo que sigue es conocimiento general de JPA, **no** contenido del profesor:

- Si se quisiera independizar el JPQL del nombre de la clase, se le puede poner
  nombre explícito a la entidad: `@Entity(name = "Alimento")`. Pero eso deja el
  modelo con dos nombres para la misma cosa (contradice lo que venimos limpiando)
  y no arregla el problema de los campos.
- EclipseLink suele reportar `Problem compiling query` + `The abstract schema type
  'X' is unknown`; Hibernate dice `X is not mapped`. Los dos apuntan a lo mismo:
  el nombre de entidad no existe.
- Como esto lo escribió otra integrante del equipo, conviene proponerlo como issue
  o PR propio, no cambiarlo de prepo en el medio de otro commit.

## Checklist

- [ ] `AlimentDAOJPA:19` → `FROM Aliment a WHERE a.category = :cat`
- [ ] `DietDAOJPA:19` → `FROM Diet d WHERE LOWER(d.nombre) LIKE :nom`
- [ ] `PatientDAOJPA:19` → `FROM Patient p WHERE LOWER(p.lastName) LIKE :ap`
- [ ] `DietDetail`: definir primero la relación `@ManyToOne Diet diet` (ver nota 3)
- [ ] `DietDetailDAO.buscarPorDieta`: `Long` → `int`
- [ ] Compilar y probar cada búsqueda con la base arriba
- [ ] Acordar con RomiM quién hace el commit
