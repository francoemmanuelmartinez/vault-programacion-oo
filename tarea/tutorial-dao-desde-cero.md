---
fuente: síntesis didáctica basada en JDBC_DAO_MySQL.pdf, JPA_Hibernate_DAO.pdf, Errores_y_Excepciones_Java.pdf y Codigo_Limpio_Nombres_y_Estructura.pdf
tags: [tarea, tutorial, dao, jdbc, jpa, excepciones, estudio]
estado: completo
---

# Tutorial Paso a Paso: Cómo construir el Patrón DAO desde cero (JDBC y JPA)

> [!info] Propósito de esta nota
> Guía pensada para que todos los integrantes del equipo entiendan **qué hace cada archivo del patrón DAO**, **por qué se divide en interfaces y clases**, y cómo implementarlo tanto en **JDBC** como en **JPA** siguiendo las convenciones y correcciones del profesor Esteban Santini.

---

## Parte 1 — Conceptos Fundamentales (comunes a JDBC y JPA)

### 1. ¿Qué es el Patrón DAO y por qué no escribimos el SQL en los botones?

> pág. 11, 12 y 20 — JDBC_DAO_MySQL.pdf

Cuando se empieza a programar con bases de datos, es común querer escribir el `INSERT INTO` o el `SELECT` directamente adentro del botón "Guardar" de la vista (`PatientView`) o en el controlador (`PatientController`).
El problema de hacer eso es que **mezclás la lógica de negocio con la tecnología de persistencia**. Si mañana cambia una columna en la tabla o decidís pasar de JPA a JDBC, tenés que buscar y arreglar código por todas las ventanas del sistema.

El patrón **DAO (Data Access Object)** separa el **qué** del **cómo**:

> *"Separar el qué del cómo es la esencia del buen diseño. El patrón DAO aplica esa idea al acceso a datos: la aplicación pide personas sin preguntar cómo se guardan."*
> `> pág. 20 — JDBC_DAO_MySQL.pdf`

Para cumplir con los **"Archivos deseados"** de la consigna y las correcciones de la cátedra, la solución se divide en **5 archivos**:

| # | Archivo | Paquete | Rol | ¿Para qué sirve? |
|---|---|---|---|---|
| **1** | `Patient.java` | `appNutritionist.model` | Entidad | Molde de datos en Java que representa una fila de la tabla `patients`. |
| **2** | `DaoException.java` | `appNutritionist.dao` | Excepción propia | Envuelve los errores técnicos (`SQLException` o `PersistenceException`) para que las capas superiores reciban una excepción limpia del dominio. |
| **3** | `DAO.java` | `appNutritionist.dao` | Interfaz Genérica (`DAO<T, ID>`) | Declara las 5 operaciones básicas (CRUD) una sola vez para **cualquier** entidad (`Patient`, `Aliment`, `Diet`). |
| **4** | `PatientDAO.java` | `appNutritionist.dao` | Interfaz Específica | Fija los tipos genéricos a `<Patient, Integer>` y permite agregar búsquedas propias de un paciente (ej. `findByLastName`). |
| **5** | `PatientDAOJPA.java` o `PatientDAOMySQL.java` | `appNutritionist.dao` | Implementación | Es la única clase que conoce la librería de persistencia (JPA o JDBC) y habla con la base de datos. |

---

### 2. Regla de oro: Nombres de paquetes y clases (Corrección de la cátedra)

> pág. 3 a 7 — Codigo_Limpio_Nombres_y_Estructura.pdf

El profesor corrigió sobre la primera entrega:
> *"Traten de mejorar los nombres de las clases: `modelPatient` es en realidad la clase `Patient` en el paquete `model` (algo similar con el `GenericDAO`). Implementen una excepción para el DAO."*

Por lo tanto:
1. **Los paquetes van siempre en minúscula y singular**:
   - `appNutritionist.model`
   - `appNutritionist.dao`
   - `appNutritionist.view`
   - `appNutritionist.controller`
2. **Las clases e interfaces van en `PascalCase` sin repetir el paquete en el nombre**:
   - ❌ `Models.modelPatient` → ✅ `model.Patient`
   - ❌ `Dao.GenericDAO` → ✅ `dao.DAO` (`> pág. 14 — JDBC_DAO_MySQL.pdf` y `> pág. 18 — JPA_Hibernate_DAO.pdf`)
   - ❌ `DaoImpl.PatientDAOImpl` → ✅ `dao.PatientDAOJPA` (si usa JPA) o `dao.PatientDAOMySQL` (si usa JDBC).

---

### 3. Los 3 archivos que son idénticos en JDBC y en JPA

> pág. 18 — JPA_Hibernate_DAO.pdf

#### A. La Excepción del DAO (`DaoException.java`)

> pág. 13 — JDBC_DAO_MySQL.pdf y pág. 9 a 11 — Errores_y_Excepciones_Java.pdf

**¿Por qué hace falta?**
Si ocurre un fallo en la base de datos, JDBC lanza `SQLException` y JPA lanza `PersistenceException`. Si dejamos que esas excepciones salgan del DAO hacia el Controlador, acoplamos el Controlador a la tecnología de base de datos.
Al crear `DaoException extends Exception` (excepción *checked*), el compilador obliga a quien llame al DAO a manejar el posible fallo con `try/catch`, y el DAO traduce cualquier error técnico en una `DaoException` conservando la causa original (`cause`).

```java
package appNutritionist.dao;

public class DaoException extends Exception {

    public DaoException(String message) {
        super(message);
    }

    public DaoException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

#### B. La Interfaz DAO Genérica (`DAO.java`)

> pág. 14 — JDBC_DAO_MySQL.pdf y pág. 18 — JPA_Hibernate_DAO.pdf

**¿Para qué sirven los genéricos `<T, ID>`?**
En lugar de copiar y pegar los 5 métodos CRUD en cada entidad, los definimos una sola vez:
- **`T`**: Tipo de la entidad (`Patient`, `Aliment`, `Diet`).
- **`ID`**: Tipo de su clave primaria (`Integer`, `Long`).

```java
package appNutritionist.dao;

import java.util.List;

public interface DAO<T, ID> {

    void insert(T entity) throws DaoException;

    T findById(ID id) throws DaoException;

    List<T> findAll() throws DaoException;

    void update(T entity) throws DaoException;

    void delete(ID id) throws DaoException;
}
```

#### C. La Interfaz DAO Específica (`PatientDAO.java`)

> pág. 15 — JDBC_DAO_MySQL.pdf y pág. 19 — JPA_Hibernate_DAO.pdf

**¿Para qué sirve si ya existe `DAO<T, ID>`?**
1. Reemplaza `T` por `Patient` e `ID` por `Integer` (`extends DAO<Patient, Integer>`).
2. Permite declarar consultas que **solo existen para pacientes** (como buscar por apellido), las cuales no tendrían sentido en `Aliment`.

```java
package appNutritionist.dao;

import appNutritionist.model.Patient;
import java.util.List;

public interface PatientDAO extends DAO<Patient, Integer> {

    List<Patient> findByLastName(String lastName) throws DaoException;
}
```

---

## Parte 2 — Implementación con JPA (`Patient` + `PatientDAOJPA`)

> pág. 1 a 24 — JPA_Hibernate_DAO.pdf

### 1. ¿Cómo funciona JPA?

En JPA usamos **Mapeo Objeto-Relacional (ORM)**: anotamos la clase `Patient` con etiquetas (`@Entity`, `@Table`, `@Id`) y el proveedor JPA genera el SQL por debajo.

### 2. Paso 1 en JPA: La Entidad anotada (`Patient.java`)

> pág. 9 a 11 — JPA_Hibernate_DAO.pdf

- **`@Entity`**: Indica que la clase es una entidad persistente.
- **`@Table(name = "Patients")`**: Nombre de la tabla en la base de datos.
- **`@Id`**: Marca el atributo clave primaria.
- **`@GeneratedValue(strategy = GenerationType.IDENTITY)`**: Indica que la base de datos genera el ID automáticamente (`AUTO_INCREMENT`).
- **Constructor vacío obligatorio (`public Patient() {}`)**: JPA lo necesita para instanciar objetos mediante reflexión cuando lee registros de la base de datos.

```java
package appNutritionist.model;

import java.io.Serializable;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "Patients")
public class Patient implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;
    private String lastName;
    private double weight;
    private double height;
    private String observations;

    public Patient() {
    }

    public Patient(String name, String lastName, double weight, double height, String observations) {
        this.name = name;
        this.lastName = lastName;
        this.weight = weight;
        this.height = height;
        this.observations = observations;
    }

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }

    public double getWeight() { return weight; }
    public void setWeight(double weight) { this.weight = weight; }

    public double getHeight() { return height; }
    public void setHeight(double height) { this.height = height; }

    public String getObservations() { return observations; }
    public void setObservations(String observations) { this.observations = observations; }
}
```

### 3. Paso 2 en JPA: La Implementación (`PatientDAOJPA.java`)

> pág. 13 a 16 y 20 a 22 — JPA_Hibernate_DAO.pdf

**Operaciones clave del `EntityManager` (`em`):**
- `em.persist(patient)` → inserta el objeto (`INSERT`).
- `em.find(Patient.class, id)` → busca por ID (`SELECT ... WHERE id = ?`).
- `em.merge(patient)` → actualiza el objeto (`UPDATE`).
- `em.remove(patient)` → elimina el objeto (`DELETE`).
- **Transacciones (`EntityTransaction tx`)**: Las operaciones que modifican datos (`insert`, `update`, `delete`) deben iniciar con `tx.begin()`, confirmar con `tx.commit()` y, si ocurre un `PersistenceException`, deshacer los cambios con `if (tx.isActive()) tx.rollback()` antes de lanzar `new DaoException(...)`.
- **Consultas JPQL (`SELECT p FROM Patient p`)**: En JPQL se consulta por el **nombre de la clase Java (`Patient`)** y sus **atributos (`p.lastName`)**, nunca por el nombre de la tabla.

```java
package appNutritionist.dao;

import appNutritionist.model.Patient;
import java.util.List;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.PersistenceException;
import javax.persistence.TypedQuery;

public class PatientDAOJPA implements PatientDAO {

    private final EntityManagerFactory emf;

    public PatientDAOJPA(EntityManagerFactory emf) {
        this.emf = emf;
    }

    @Override
    public void insert(Patient patient) throws DaoException {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        try {
            tx.begin();
            em.persist(patient);
            tx.commit();
        } catch (PersistenceException e) {
            if (tx.isActive()) {
                tx.rollback();
            }
            throw new DaoException("No se pudo insertar el paciente", e);
        } finally {
            em.close();
        }
    }

    @Override
    public Patient findById(Integer id) throws DaoException {
        EntityManager em = emf.createEntityManager();

        try {
            return em.find(Patient.class, id);
        } catch (PersistenceException e) {
            throw new DaoException("No se pudo buscar el paciente con id " + id, e);
        } finally {
            em.close();
        }
    }

    @Override
    public List<Patient> findAll() throws DaoException {
        EntityManager em = emf.createEntityManager();

        try {
            return em.createQuery("SELECT p FROM Patient p", Patient.class)
                     .getResultList();
        } catch (PersistenceException e) {
            throw new DaoException("No se pudo listar los pacientes", e);
        } finally {
            em.close();
        }
    }

    @Override
    public void update(Patient patient) throws DaoException {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        try {
            tx.begin();
            em.merge(patient);
            tx.commit();
        } catch (PersistenceException e) {
            if (tx.isActive()) {
                tx.rollback();
            }
            throw new DaoException("No se pudo actualizar el paciente con id " + patient.getId(), e);
        } finally {
            em.close();
        }
    }

    @Override
    public void delete(Integer id) throws DaoException {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        try {
            tx.begin();
            Patient patient = em.find(Patient.class, id);
            if (patient != null) {
                em.remove(patient);
            }
            tx.commit();
        } catch (PersistenceException e) {
            if (tx.isActive()) {
                tx.rollback();
            }
            throw new DaoException("No se pudo eliminar el paciente con id " + id, e);
        } finally {
            em.close();
        }
    }

    @Override
    public List<Patient> findByLastName(String lastName) throws DaoException {
        EntityManager em = emf.createEntityManager();

        try {
            TypedQuery<Patient> query = em.createQuery(
                "SELECT p FROM Patient p WHERE p.lastName = :lastName",
                Patient.class
            );
            query.setParameter("lastName", lastName);
            return query.getResultList();
        } catch (PersistenceException e) {
            throw new DaoException("No se pudo buscar pacientes con apellido " + lastName, e);
        } finally {
            em.close();
        }
    }
}
```

---

## Parte 3 — Alternativa con JDBC (`Patient` + `PatientDAOMySQL`)

> pág. 1 a 20 — JDBC_DAO_MySQL.pdf

### 1. ¿Cómo funciona JDBC?

En JDBC (`java.sql.*`) no se usan anotaciones en la entidad `Patient` (queda una clase Java pura) y las consultas SQL se escriben a mano dentro de `PatientDAOMySQL` usando:
- `DriverManager.getConnection(url, user, password)` para abrir la conexión.
- `PreparedStatement` con parámetros `?` para evitar inyección SQL.
- `Statement.RETURN_GENERATED_KEYS` para recuperar el `id` autogenerado tras un `INSERT`.
- `ResultSet` para recorrer las filas devueltas por un `SELECT`.
- `try-with-resources` (`try (Connection cn = ...; PreparedStatement ps = ...)`) para que Java cierre los recursos automáticamente.
- `catch (SQLException e)` envolviendo el error en `throw new DaoException("...", e)`.

### 2. Implementación `PatientDAOMySQL.java`

> pág. 16 a 18 — JDBC_DAO_MySQL.pdf

```java
package appNutritionist.dao;

import appNutritionist.model.Patient;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

public class PatientDAOMySQL implements PatientDAO {

    private final String url;
    private final String user;
    private final String password;

    public PatientDAOMySQL(String url, String user, String password) {
        this.url = url;
        this.user = user;
        this.password = password;
    }

    @Override
    public void insert(Patient patient) throws DaoException {
        String sql = "INSERT INTO Patients (name, lastName, weight, height, observations) VALUES (?, ?, ?, ?, ?)";
        try (Connection cn = DriverManager.getConnection(url, user, password);
             PreparedStatement ps = cn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {

            ps.setString(1, patient.getName());
            ps.setString(2, patient.getLastName());
            ps.setDouble(3, patient.getWeight());
            ps.setDouble(4, patient.getHeight());
            ps.setString(5, patient.getObservations());
            ps.executeUpdate();

            try (ResultSet keys = ps.getGeneratedKeys()) {
                if (keys.next()) {
                    patient.setId(keys.getInt(1));
                }
            }
        } catch (SQLException e) {
            throw new DaoException("No se pudo insertar el paciente", e);
        }
    }

    @Override
    public Patient findById(Integer id) throws DaoException {
        String sql = "SELECT id, name, lastName, weight, height, observations FROM Patients WHERE id = ?";
        try (Connection cn = DriverManager.getConnection(url, user, password);
             PreparedStatement ps = cn.prepareStatement(sql)) {

            ps.setInt(1, id);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    return mapRowToPatient(rs);
                }
            }
            return null;
        } catch (SQLException e) {
            throw new DaoException("No se pudo buscar el paciente con id " + id, e);
        }
    }

    @Override
    public List<Patient> findAll() throws DaoException {
        List<Patient> patients = new ArrayList<>();
        String sql = "SELECT id, name, lastName, weight, height, observations FROM Patients";
        try (Connection cn = DriverManager.getConnection(url, user, password);
             Statement st = cn.createStatement();
             ResultSet rs = st.executeQuery(sql)) {

            while (rs.next()) {
                patients.add(mapRowToPatient(rs));
            }
        } catch (SQLException e) {
            throw new DaoException("No se pudo listar los pacientes", e);
        }
        return patients;
    }

    @Override
    public void update(Patient patient) throws DaoException {
        String sql = "UPDATE Patients SET name = ?, lastName = ?, weight = ?, height = ?, observations = ? WHERE id = ?";
        try (Connection cn = DriverManager.getConnection(url, user, password);
             PreparedStatement ps = cn.prepareStatement(sql)) {

            ps.setString(1, patient.getName());
            ps.setString(2, patient.getLastName());
            ps.setDouble(3, patient.getWeight());
            ps.setDouble(4, patient.getHeight());
            ps.setString(5, patient.getObservations());
            ps.setInt(6, patient.getId());
            ps.executeUpdate();
        } catch (SQLException e) {
            throw new DaoException("No se pudo actualizar el paciente con id " + patient.getId(), e);
        }
    }

    @Override
    public void delete(Integer id) throws DaoException {
        String sql = "DELETE FROM Patients WHERE id = ?";
        try (Connection cn = DriverManager.getConnection(url, user, password);
             PreparedStatement ps = cn.prepareStatement(sql)) {

            ps.setInt(1, id);
            ps.executeUpdate();
        } catch (SQLException e) {
            throw new DaoException("No se pudo eliminar el paciente con id " + id, e);
        }
    }

    @Override
    public List<Patient> findByLastName(String lastName) throws DaoException {
        List<Patient> patients = new ArrayList<>();
        String sql = "SELECT id, name, lastName, weight, height, observations FROM Patients WHERE lastName = ?";
        try (Connection cn = DriverManager.getConnection(url, user, password);
             PreparedStatement ps = cn.prepareStatement(sql)) {

            ps.setString(1, lastName);
            try (ResultSet rs = ps.executeQuery()) {
                while (rs.next()) {
                    patients.add(mapRowToPatient(rs));
                }
            }
        } catch (SQLException e) {
            throw new DaoException("No se pudo buscar pacientes con apellido " + lastName, e);
        }
        return patients;
    }

    private Patient mapRowToPatient(ResultSet rs) throws SQLException {
        Patient p = new Patient(
            rs.getString("name"),
            rs.getString("lastName"),
            rs.getDouble("weight"),
            rs.getDouble("height"),
            rs.getString("observations")
        );
        p.setId(rs.getInt("id"));
        return p;
    }
}
```

---

## Parte 4 — ¿Cómo hacer el DAO de la siguiente entidad (`Aliment`)?

Para agregar el DAO de `Aliment`, **no se vuelven a crear `DAO.java` ni `DaoException.java`** (se reutilizan los mismos). Solo hay que crear 3 archivos:
1. `appNutritionist.model.Aliment` (con sus atributos `id`, `name`, `category`, `caloriesPerServing`).
2. `appNutritionist.dao.AlimentDAO extends DAO<Aliment, Integer>`.
3. `appNutritionist.dao.AlimentDAOJPA` (o `AlimentDAOMySQL`).

## Relacionado

- [[entrega-02-dao-una-entidad]] — consigna y correcciones de la Entrega 2
- [[JDBC_DAO_MySQL]] — nota de cátedra sobre JDBC y DAO
- [[JPA_Hibernate_DAO]] — nota de cátedra sobre JPA, Hibernate y DAO
- [[Errores_y_Excepciones_Java]] — nota de cátedra sobre excepciones propias
- [[estado]] — estado actual del proyecto `NutritionistApp`
