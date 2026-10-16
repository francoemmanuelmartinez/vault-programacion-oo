---
fuente: JPA_Hibernate_DAO.pdf
tags: [tema, practica, jpa, hibernate, dao, persistencia]
estado: completo
---

# Mapeo de Objetos a Bases de Datos (JPA, Hibernate y DAO)

> [!info] Fuente
> `JPA_Hibernate_DAO.pdf` — pág. 1 a 24
> **Profesor:** Esteban Santini — ISFT 204 (Desarrollo de Sistemas Orientado a Objetos)

## Resumen

> pág. 1 — JPA_Hibernate_DAO.pdf

JPA (Java Persistence API / Jakarta Persistence) es la especificación estándar de Java para realizar mapeo objeto-relacional (ORM), permitiendo persistir objetos en bases de datos relacionales sin escribir SQL manual. Hibernate es el proveedor más utilizado que implementa esta especificación, integrándose de forma natural con el patrón DAO.

---

## Lo que dice el profesor

### 1. ¿Qué es JPA?

> pág. 2 — JPA_Hibernate_DAO.pdf

- **Definición:** JPA es la especificación estándar de Java para mapear clases a tablas y trabajar con bases de datos relacionales sin escribir SQL a mano.
- **Mapeo declarativo:** En lugar de armar el SQL y construir los objetos fila por fila como en JDBC, en JPA se anotan las clases (`@Entity`, `@Id`, `@Column`) y el proveedor genera las consultas automáticamente.
- **Especificación vs Proveedor:** JPA es solo un conjunto de interfaces (`jakarta.persistence`); necesita un proveedor que la implemente, como **Hibernate**.

### 2. JPA vs. Jakarta Persistence

> pág. 3 — JPA_Hibernate_DAO.pdf

- **Evolución histórica:**
  1. Hasta 2017 la especificación vivía en `javax.persistence` como parte de Java EE (Oracle).
  2. Oracle donó Java EE a la Eclipse Foundation, que no podía seguir usando la marca "Java" (nació Jakarta EE).
  3. Desde Jakarta EE 9 (2020), los paquetes pasaron de `javax.*` a `jakarta.*`.
- **Imports:**
  ```java
  import javax.persistence.*; // Java EE (hasta 2019)
  import jakarta.persistence.*; // Jakarta EE 9+ (este curso)
  ```
- El nombre "JPA" se mantiene como apodo, pero el nombre oficial actual es **Jakarta Persistence**.

### 3. Implementaciones de JPA

> pág. 4 — JPA_Hibernate_DAO.pdf

- **Hibernate:** El proveedor JPA más usado; en el que se basan los ejemplos de la clase.
- **EclipseLink:** Implementación de referencia de la especificación, mantenida por la Eclipse Foundation.
- **OpenJPA:** Implementación de Apache, usada en entornos empresariales.
- **Spring Data JPA:** Capa sobre JPA que genera automáticamente el DAO a partir de una interfaz.

### 4. Agregar Hibernate con Maven

> pág. 5 — JPA_Hibernate_DAO.pdf

Las tres dependencias esenciales que van dentro de `<dependencies>` en el `pom.xml`:

```xml
<dependency>
    <groupId>jakarta.persistence</groupId>
    <artifactId>jakarta.persistence-api</artifactId>
    <version>3.1.0</version>
</dependency>
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.6.22.Final</version>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.5.0</version>
</dependency>
```

### 5. Ant vs. Maven

> pág. 6 — JPA_Hibernate_DAO.pdf

| Ant | Maven |
|---|---|
| **Dependencias manuales:** Cada `.jar` se descarga y se agrega al proyecto a mano. | **Dependencias declaradas:** Cada librería se declara una vez en el `pom.xml` con su versión exacta. |
| **Sin resolución transitiva:** Hay que buscar e incluir dependencias secundarias una por una. | **Resolución automática:** Descarga automáticamente las dependencias transitivas (lo que Hibernate necesita). |
| **build.xml a medida:** Pasos de compilación y empaquetado escritos y mantenidos a mano. | **Estándar en Java:** Cualquiera compila el proyecto igual con el mismo `pom.xml`. |

### 6. Flujo básico de trabajo con JPA

> pág. 7 — JPA_Hibernate_DAO.pdf

Los pasos que se repiten en cualquier operación:
1. Configurar la unidad de persistencia en `persistence.xml` (conexión y proveedor).
2. Crear un `EntityManagerFactory` a partir de esa unidad de persistencia.
3. Obtener un `EntityManager` desde la factory.
4. Iniciar una transacción para operaciones que modifican datos (`begin`).
5. Ejecutar la operación (`persist`, `find`, consulta JPQL) y confirmar la transacción (`commit`).
6. Cerrar el `EntityManager` (y el `EntityManagerFactory` al finalizar la aplicación).

### 7. Configuración de JPA (`persistence.xml`)

> pág. 8 — JPA_Hibernate_DAO.pdf

- **Ubicación:** `src/main/resources/META-INF/persistence.xml`
- **Ejemplo de configuración:**
  ```xml
  <persistence-unit name="escuelaPU">
      <properties>
          <property name="jakarta.persistence.jdbc.url"
                    value="jdbc:mysql://localhost:3306/escuela"/>
          <property name="jakarta.persistence.jdbc.user" value="root"/>
          <property name="jakarta.persistence.jdbc.password" value="clave"/>
          <property name="hibernate.hbm2ddl.auto" value="update"/>
      </properties>
  </persistence-unit>
  ```
- **Inicialización en código:**
  ```java
  EntityManagerFactory emf = Persistence.createEntityManagerFactory("escuelaPU");
  EntityManager em = emf.createEntityManager();
  ```

### 8. La entidad Persona con JPA y anotaciones

> pág. 9 y 11 — JPA_Hibernate_DAO.pdf

- **Anotaciones básicas:**
  - `@Entity`: Marca la clase como una entidad persistente (mapeada a una tabla).
  - `@Table(name = "personas")`: Indica el nombre de la tabla si es distinto al de la clase.
  - `@Id`: Marca el atributo clave primaria.
  - `@GeneratedValue(strategy = GenerationType.IDENTITY)`: Indica que el ID lo genera automáticamente la base de datos (auto-incremental).
  - `@Column(name = "...")`: Mapea un atributo a una columna con otro nombre o ajuste de tipo/longitud.
- **Ejemplo de clase entidad:**
  ```java
  import jakarta.persistence.*;

  @Entity
  @Table(name = "personas")
  public class Persona {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
      private String nombre;
      private String apellido;
      private String documento;
      // ...
  }
  ```

### 9. Constructores y Getters

> pág. 10 — JPA_Hibernate_DAO.pdf

- **Constructor vacío obligatorio:** JPA exige un constructor sin argumentos (`public Persona() {}`) para poder instanciar la entidad mediante reflexión cuando lee de la base de datos.
- Se suelen proveer constructores con parámetros para el uso normal en la aplicación, junto con todos los getters y setters.

### 10. Anotaciones de relaciones

> pág. 12 — JPA_Hibernate_DAO.pdf

- `@ManyToOne`: Varios registros de esta entidad se relacionan con uno de otra (ej: muchos pedidos, un cliente).
- `@OneToMany`: El lado inverso de `@ManyToOne` (una entidad tiene una colección de otra; usa `mappedBy`).
- `@JoinColumn`: Define el nombre de la columna que guarda la clave foránea en la tabla.
- `@ManyToMany`: Relación de muchos a muchos; JPA la resuelve con una tabla intermedia automática.
- *Nota del profesor:* La entidad `Persona` de este ejemplo inicial no usa relaciones; quedan para más adelante.

### 11. Operaciones CRUD con JPA

> pág. 13 a 16 — JPA_Hibernate_DAO.pdf

- **Insertar (Alta):**
  ```java
  EntityManager em = emf.createEntityManager();
  Persona p = new Persona("Juan", "Pérez", "30111222");
  em.getTransaction().begin();
  em.persist(p);
  em.getTransaction().commit();
  em.close();
  ```
- **Listar todos (Consulta JPQL):**
  ```java
  List<Persona> personas = em.createQuery("SELECT p FROM Persona p", Persona.class)
                             .getResultList();
  ```
- **Consultas con WHERE (JPQL):** JPQL trabaja sobre entidades y atributos en lugar de tablas y columnas.
  ```java
  TypedQuery<Persona> query = em.createQuery(
      "SELECT p FROM Persona p WHERE p.apellido = :apellido", Persona.class);
  query.setParameter("apellido", "Pérez");
  List<Persona> personas = query.getResultList();
  ```
- **Named Queries:** JPQL reutilizable declarado junto a la entidad con `@NamedQuery`:
  ```java
  @Entity
  @NamedQuery(name = "Persona.buscarPorApellido",
              query = "SELECT p FROM Persona p WHERE p.apellido = :apellido")
  ```

### 12. JDBC vs. JPA

> pág. 17 — JPA_Hibernate_DAO.pdf

| JDBC | JPA |
|---|---|
| **Control total del SQL:** Cada consulta se escribe a mano; útil para casos de alto rendimiento. | **Mapeo automático:** Las anotaciones describen la correspondencia entre clase y tabla. |
| **Mapeo manual:** Hay que armar cada objeto fila por fila desde el `ResultSet`. | **Menos código:** `persist()`, `find()` y JPQL reemplazan el código repetitivo de JDBC. |
| **Más código repetitivo:** Conexión, sentencia y mapeo se repiten en cada operación. | **Portabilidad:** Cambiar de motor de base de datos implica mucho menos trabajo. |

### 13. Integración con el Patrón DAO

> pág. 18 a 22 — JPA_Hibernate_DAO.pdf

- **Contrato genérico (`DAO<T, ID>`):** Se mantiene exactamente igual que con JDBC.
- **Especialización (`PersonaDAO`):** Extiende el contrato genérico agregando métodos específicos.
- **Implementación JPA (`PersonaDAOJPA`):**
  ```java
  public class PersonaDAOJPA implements PersonaDAO {
      private final EntityManager em;

      public PersonaDAOJPA(EntityManager em) {
          this.em = em;
      }

      @Override
      public Persona findById(Long id) throws DaoException {
          try {
              return em.find(Persona.class, id);
          } catch (PersistenceException e) {
              throw new DaoException("No se pudo buscar la persona con id " + id, e);
          }
      }

      @Override
      public void insert(Persona p) throws DaoException {
          EntityTransaction tx = em.getTransaction();
          try {
              tx.begin();
              em.persist(p);
              tx.commit();
          } catch (PersistenceException e) {
              if (tx.isActive()) tx.rollback();
              throw new DaoException("No se pudo insertar la persona", e);
          }
      }
      // ...
  }
  ```

### 14. Ventajas de JPA

> pág. 23 — JPA_Hibernate_DAO.pdf

- Menos código repetitivo.
- Mapeo declarativo mediante anotaciones.
- Portabilidad entre motores SQL.
- Caché de primer y segundo nivel y optimizaciones automáticas del proveedor.

### 15. Mensaje final del profesor

> pág. 24 — JPA_Hibernate_DAO.pdf

> *"La mejor herramienta es la que te deja pensar en el dominio, no en el SQL."*
> 
> *JPA no reemplaza a JDBC: lo usa por debajo, pero te permite programar contra objetos en lugar de contra tablas.*

---

## Ampliación de la IA

> [!warning] Esto NO es del profesor
> Conexión con el proyecto `NutritionistApp` y uso estratégico de JPA.

- **Alternativa a JDBC puro (`JDBC_DAO_MySQL.pdf`):** La rúbrica de la materia permite elegir el mecanismo de persistencia. Usar JPA/Hibernate con `NutritionistApp` simplifica enormemente los DAOs de `Patient`, `Aliment` y la futura entidad `Diet`.
- **Característica extra para Nota 10:** Implementar JPA/Hibernate en un proyecto NetBeans con Ant (en lugar de Maven, o configurando las librerías a mano) o agregar relaciones complejas (`@OneToMany` entre `Diet` y sus ítems) califica perfectamente como la característica extra de investigación que exige el nivel 10 de la rúbrica.

---

## Dudas y vacíos

- [ ] ¿El profesor exige estrictamente JDBC a mano (`java.sql`) o acepta JPA/Hibernate como reemplazo directo para el cumplimiento del patrón DAO? El enunciado dice *"utilizando el patrón DAO"* sin imponer la tecnología de base de datos o driver.

---

## Relacionado

- [[JDBC_DAO_MySQL]] — La alternativa basada en JDBC puro
- [[SOLID_Principios]] — Principios aplicados al diseño en capas
- [[enunciado_rubrica]] — Requisitos del trabajo práctico integrador
- [[estado]] — Estado de la persistencia en `NutritionistApp`
- [[MAP]] — Mapa general del vault

---

**Navegación:** [[index]] · [[MAP]]
