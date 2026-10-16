---
fuente: JDBC_DAO_MySQL.pdf
tags: [tema, practica, jdbc, dao, mysql, persistencia]
estado: completo
---

# Acceso a Bases de Datos SQL en Java (JDBC y Patrón DAO)

> [!info] Fuente
> `JDBC_DAO_MySQL.pdf` — pág. 1 a 20
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

> pág. 1 — JDBC_DAO_MySQL.pdf

JDBC (Java Database Connectivity) es la API estándar de Java para conectarse a bases de datos relacionales y ejecutar consultas SQL. El patrón DAO (Data Access Object) encapsula el acceso a los datos detrás de una interfaz, desacoplando la lógica de negocio de los detalles de persistencia y de la tecnología SQL utilizada.

---

## Lo que dice el profesor

### 1. ¿Qué es JDBC?

> pág. 2 — JDBC_DAO_MySQL.pdf

- **Definición:** API estándar de Java en el paquete `java.sql` para conectar con bases de datos relacionales.
- **Drivers JDBC:** Cada motor (MySQL, PostgreSQL, Oracle) provee su propio driver (una biblioteca que implementa las interfaces `Connection`, `Statement`, `ResultSet` para ese motor).
- **Programación contra interfaces:** El código Java programa siempre contra las interfaces estándar; el driver traduce las llamadas al protocolo específico de la base de datos.

### 2. Tipos de Drivers JDBC

> pág. 3 — JDBC_DAO_MySQL.pdf

- **Tipo 1 (Puente JDBC-ODBC):** Traduce llamadas JDBC a ODBC del S.O. (en desuso).
- **Tipo 2 (API Nativa):** Convierte JDBC a llamadas nativas del cliente del motor (en desuso).
- **Tipo 3 (Protocolo de Red):** Servidor intermedio traduce JDBC al protocolo del motor (poco usado).
- **Tipo 4 (Protocolo Nativo / 100% Java):** Habla directo con el motor por su protocolo propio. Es el que usan MySQL y PostgreSQL en la actualidad.

### 3. Agregar el Driver JDBC en Ant vs. Maven

> pág. 4 a 6 — JDBC_DAO_MySQL.pdf

- **En proyectos Ant (NetBeans clásico):**
  1. Descargar `mysql-connector-j-9.5.0.jar` desde la web oficial de MySQL (Platform Independent).
  2. Agregar el `.jar` a mano en las *Properties* del proyecto -> categoría *Libraries* -> pestaña *Compile* -> *Add JAR/Folder*.
  3. Para compartir el proyecto, se suele copiar a una carpeta `lib/` propia.
- **En proyectos Maven:**
  Agregar la dependencia en el `pom.xml`:
  ```xml
  <dependency>
      <groupId>com.mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <version>9.5.0</version>
  </dependency>
  ```

### 4. Flujo Básico de Trabajo con JDBC

> pág. 7 — JDBC_DAO_MySQL.pdf

Los 6 pasos que se repiten en cualquier operación JDBC:
1. Registrar u obtener el driver del motor de base de datos.
2. Abrir una `Connection` con `DriverManager.getConnection(url, usuario, clave)`.
3. Crear un `Statement` o, preferentemente, un `PreparedStatement` (para prevenir inyección SQL).
4. Ejecutar la sentencia SQL (`executeUpdate` para alta/baja/modificación, `executeQuery` para consultas).
5. Si hay resultados, recorrer el `ResultSet` fila por fila con `rs.next()`.
6. Cerrar los recursos (`Connection`, `Statement`, `ResultSet`), idealmente utilizando el bloque **`try-with-resources`** para evitar fugas de memoria y conexiones colgadas.

### 5. La Entidad Persona

> pág. 8 — JDBC_DAO_MySQL.pdf

Clase de dominio que representa un registro de la tabla `personas`:

```java
public class Persona {
    private Long id;
    private String nombre;
    private String apellido;
    private String documento;

    public Persona(String nombre, String apellido, String documento) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.documento = documento;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getNombre() { return nombre; }
    public String getApellido() { return apellido; }
    public String getDocumento() { return documento; }
}
```

### 6. Operaciones JDBC directas (`PreparedStatement` y `ResultSet`)

> pág. 9 y 10 — JDBC_DAO_MySQL.pdf

- **Inserción con PreparedStatement:**
  ```java
  String url = "jdbc:mysql://localhost:3306/escuela";
  String sql = "INSERT INTO personas (nombre, apellido, documento) VALUES (?, ?, ?)";
  try (Connection cn = DriverManager.getConnection(url, "root", "clave");
       PreparedStatement ps = cn.prepareStatement(sql)) {
      ps.setString(1, "Juan");
      ps.setString(2, "Pérez");
      ps.setString(3, "30111222");
      ps.executeUpdate();
  } catch (SQLException e) {
      e.printStackTrace();
  }
  ```

- **Listado recorriendo un ResultSet:**
  ```java
  String sql = "SELECT nombre, apellido, documento FROM personas";
  try (Connection cn = DriverManager.getConnection(url, "root", "clave");
       Statement st = cn.createStatement();
       ResultSet rs = st.executeQuery(sql)) {
      while (rs.next()) {
          String nombre = rs.getString("nombre");
          String apellido = rs.getString("apellido");
          System.out.println(apellido + ", " + nombre);
      }
  } catch (SQLException e) {
      e.printStackTrace();
  }
  ```

### 7. JDBC Directo vs. Patrón DAO

> pág. 11 y 12 — JDBC_DAO_MySQL.pdf

- **JDBC directo:** Rápido para prototipos, pero mezcla el SQL con la lógica de negocio y repite código de conexión en cada método.
- **Patrón DAO (Data Access Object):** Patrón de diseño que encapsula el acceso a la fuente de datos detrás de una interfaz. Separa las operaciones CRUD del resto de la aplicación, facilitando la reutilización, el mantenimiento y los tests (se puede simular un DAO en memoria).

### 8. La Excepción Checked `DaoException`

> pág. 13 — JDBC_DAO_MySQL.pdf

Para evitar propagar `SQLException` (que es técnica y propia de JDBC) hacia la lógica de negocio, se envuelve en una excepción propia de la aplicación de tipo *checked*:

```java
public class DaoException extends Exception {
    public DaoException(String message, Throwable cause) {
        super(message, cause);
    }
    public DaoException(String message) {
        super(message);
    }
}
```

### 9. El Contrato Genérico: `DAO<T, ID>` y `PersonaDAO`

> pág. 14 y 15 — JDBC_DAO_MySQL.pdf

- **Interfaz genérica:**
  ```java
  public interface DAO<T, ID> {
      void insert(T entity) throws DaoException;
      T findById(ID id) throws DaoException;
      List<T> findAll() throws DaoException;
      void update(T entity) throws DaoException;
      void delete(ID id) throws DaoException;
  }
  ```
- **Especialización para la entidad Persona:**
  ```java
  public interface PersonaDAO extends DAO<Persona, Long> {
      List<Persona> findByLastName(String lastName) throws DaoException;
  }
  ```

### 10. Implementación MySQL del DAO (`PersonaDAOMySQL`)

> pág. 16 a 18 — JDBC_DAO_MySQL.pdf

- **Buscar por ID:**
  ```java
  @Override
  public Persona findById(Long id) throws DaoException {
      String sql = "SELECT id, nombre, apellido, documento FROM personas WHERE id = ?";
      try (Connection cn = DriverManager.getConnection(url, usuario, clave);
           PreparedStatement ps = cn.prepareStatement(sql)) {
          ps.setLong(1, id);
          try (ResultSet rs = ps.executeQuery()) {
              if (rs.next()) {
                  Persona p = new Persona(
                      rs.getString("nombre"),
                      rs.getString("apellido"),
                      rs.getString("documento"));
                  p.setId(rs.getLong("id"));
                  return p;
              }
          }
          return null;
      } catch (SQLException e) {
          throw new DaoException("No se pudo buscar la persona con id " + id, e);
      }
  }
  ```

- **Insertar recuperando el ID autogenerado (`RETURN_GENERATED_KEYS`):**
  ```java
  @Override
  public void insert(Persona p) throws DaoException {
      String sql = "INSERT INTO personas (nombre, apellido, documento) VALUES (?, ?, ?)";
      try (Connection cn = DriverManager.getConnection(url, usuario, clave);
           PreparedStatement ps = cn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
          ps.setString(1, p.getNombre());
          ps.setString(2, p.getApellido());
          ps.setString(3, p.getDocumento());
          ps.executeUpdate();
          
          try (ResultSet keys = ps.getGeneratedKeys()) {
              if (keys.next()) {
                  p.setId(keys.getLong(1));
              }
          }
      } catch (SQLException e) {
          throw new DaoException("No se pudo insertar la persona", e);
      }
  }
  ```

- **Listar todos (`findAll`):**
  ```java
  @Override
  public List<Persona> findAll() throws DaoException {
      List<Persona> personas = new ArrayList<>();
      String sql = "SELECT id, nombre, apellido, documento FROM personas";
      try (Connection cn = DriverManager.getConnection(url, usuario, clave);
           Statement st = cn.createStatement();
           ResultSet rs = st.executeQuery(sql)) {
          while (rs.next()) {
              Persona p = new Persona(
                  rs.getString("nombre"),
                  rs.getString("apellido"),
                  rs.getString("documento"));
              p.setId(rs.getLong("id"));
              personas.add(p);
          }
      } catch (SQLException e) {
          throw new DaoException("No se pudo listar las personas", e);
      }
      return personas;
  }
  ```

### 11. Ventajas del Patrón DAO

> pág. 19 — JDBC_DAO_MySQL.pdf

- **Desacoplamiento:** La lógica de negocio no depende de JDBC ni del motor SQL.
- **Reutilización:** Las operaciones CRUD se escriben una vez.
- **Mantenibilidad:** Un cambio de esquema o motor se resuelve en un solo lugar.
- **Testeabilidad:** Permite mocks e implementaciones en memoria para pruebas unitarias.

### 12. Mensaje final del profesor

> pág. 20 — JDBC_DAO_MySQL.pdf

> *"Separar el qué del cómo es la esencia del buen diseño."*
> 
> *El patrón DAO aplica esa idea al acceso a datos: la aplicación pide personas sin preguntar cómo se guardan.*

---

## Ampliación de la IA

> [!warning] Instanciación y manejo de conexiones
> En aplicaciones reales basadas en capas, abrir y cerrar una conexión en cada método del DAO (como muestran los ejemplos didácticos del PDF) es ineficiente. En arquitecturas robustas se utiliza un **Pool de Conexiones** (como HikariCP) o se gestiona una única conexión por transacción.

---

## Dudas y vacíos

- [ ] ¿El profesor exige patrón Singleton o fábrica de conexiones (`ConnectionFactory`) para evitar repetir la URL, el usuario y la clave en cada método DAO? Los ejemplos del PDF abren `DriverManager.getConnection` directo en cada método, pero en la práctica se centraliza.

---

## Relacionado

- [[JPA_Hibernate_DAO]] — La alternativa moderna basada en ORM y JPA
- [[SOLID_Principios]] — Principios SOLID aplicados a la separación de capas
- [[enunciado_rubrica]] — Requisitos del trabajo práctico integrador
- [[estado]] — Estado del acceso a datos en `NutritionistApp`
- [[MAP]] — Mapa general del vault

---

**Navegación:** [[index]] · [[MAP]]
