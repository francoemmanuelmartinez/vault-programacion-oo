---
fuente: enunciado_rubrica.pdf (pág. 2), JDBC_DAO_MySQL.pdf (pág. 13 a 18), JPA_Hibernate_DAO.pdf (pág. 18 a 22) y Errores_y_Excepciones_Java.pdf (pág. 9 a 11)
tags: [tarea, excepciones, dao, jpa, tutorial]
estado: completo
---

# Ejemplo de excepciones: un mensaje personalizado por método del DAO

> [!info] Propósito de esta nota
> Responder **por qué** cada método del DAO lanza su propia `DaoException` con un mensaje distinto, y dejar el **código final exacto** de los archivos del DAO para el momento en que se conectan los botones desde el controlador.
> Complementa a [[tutorial-dao-desde-cero]], que explica el patrón completo desde cero.

---

## 1. La pregunta: ¿por qué un mensaje distinto en cada método?

### Lo que pide la rúbrica

> pág. 2 — enunciado_rubrica.pdf

> *"Excelente manejo de excepciones en todas las entidades, con excepciones personalizadas y mensajes claros que mejoran la experiencia."*

Ese es el criterio **Manejo de Excepciones** en el nivel 7-9. Ojo con las tres exigencias que hay adentro de una sola frase:

1. **"en todas las entidades"** → no alcanza con `Patient`: tienen que estar `Aliment`, `Diet` y `DietDetail`.
2. **"excepciones personalizadas"** → `DaoException`, no `RuntimeException` ni `SQLException` sueltas.
3. **"mensajes claros que mejoran la experiencia"** → un mensaje genérico tipo `"Error"` no cumple.

### Lo que hace el profesor en el código

> pág. 16 a 18 — JDBC_DAO_MySQL.pdf
> pág. 18 a 22 — JPA_Hibernate_DAO.pdf (los mensajes aparecen en la implementación, pág. 20 a 22)

El patrón es siempre el mismo: **el mensaje dice qué operación falló** y, cuando tiene el ID a mano, **con qué ID falló**.

```java
// JDBC_DAO_MySQL.pdf, pág. 16 a 18
catch (SQLException e) {
    throw new DaoException("No se pudo buscar la persona con id " + id, e);
}
```

```java
// JPA_Hibernate_DAO.pdf, pág. 20 a 22
catch (PersistenceException e) {
    throw new DaoException("No se pudo insertar la persona", e);
}
```

Fijate en el segundo argumento `e`: ahí viaja el error técnico. El mensaje es para el humano; la causa es para el programador.

### La respuesta corta: un mensaje, dos destinatarios

Una excepción personalizada con mensaje específico sirve a **dos públicos distintos** al mismo tiempo:

| Dato | Dónde vive | Quién lo lee | Ejemplo |
|---|---|---|---|
| **Qué operación falló** | `ex.getMessage()` | El usuario final, en un `JOptionPane` | `"No se pudo insertar el paciente"` |
| **Con qué ID falló** | `ex.getMessage()` | El usuario y quien da soporte | `"No se pudo eliminar el paciente con id 7"` |
| **Qué dijo la base** | `ex.getCause()` | El programador (log / `printStackTrace()`) | `PersistenceException: could not execute statement` |
| **Dónde falló** | *stack trace* | El programador | `PatientDAOJPA.java:34` |

Sin mensaje específico, los dos públicos pierden:

- **El usuario** ve `"org.hibernate.PersistenceException: could not execute statement"` en un cartel y no sabe si falló al guardar, al borrar o al buscar.
- **El programador** ve `"Error al guardar"` en cinco métodos distintos y no sabe cuál se ejecutó.

Con un mensaje por método, el mismo `catch (DaoException ex)` del controlador sirve para los cinco botones, porque **la excepción ya dice qué pasó**.

---

## 2. Mapa de mensajes por entidad

Los cinco mensajes de `Patient` y cómo se traducen a las demás entidades. Las tres formas son: *sin ID* (en `insert`, `findAll` y `update`), *con el ID del parámetro* (`findById`, `delete`) y *con el ID del objeto* (`update`).

| Método | `Patient` | `Aliment` | `Diet` | `DietDetail` |
|---|---|---|---|---|
| `insert` | No se pudo insertar el paciente | No se pudo insertar el alimento | No se pudo insertar la dieta | No se pudo insertar el detalle de la dieta |
| `findById` | No se pudo buscar el paciente con id *N* | No se pudo buscar el alimento con id *N* | No se pudo buscar la dieta con id *N* | No se pudo buscar el detalle de la dieta con id *N* |
| `findAll` | No se pudo listar los pacientes | No se pudo listar los alimentos | No se pudo listar las dietas | No se pudo listar los detalles de la dieta |
| `update` | No se pudo actualizar el paciente con id *N* | No se pudo actualizar el alimento con id *N* | No se pudo actualizar la dieta con id *N* | No se pudo actualizar el detalle de la dieta con id *N* |
| `delete` | No se pudo eliminar el paciente con id *N* | No se pudo eliminar el alimento con id *N* | No se pudo eliminar la dieta con id *N* | No se pudo eliminar el detalle de la dieta con id *N* |

> [!note] Corrección al borrador original
> El borrador dejaba `update` **sin** ID en `Aliment`, `Diet` y `DietDetail` porque "las clases todavía no tienen `getId()`". **Eso ya no es cierto:** `Aliment` tiene `getId()` desde el commit `d8e40df`, y `Diet` ya declara su `id`. Conviene usar `+ aliment.getId()` y `+ diet.getId()` para que los cinco métodos queden simétricos. En `DietDetail`, si todavía no tiene `getId()`, se deja sin ID o se agrega el getter.

---

## 3. Los archivos, completos

### 3.1 `appNutritionist.dao.DaoException`

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

> pág. 13 — JDBC_DAO_MySQL.pdf
> También: pág. 9 y 10 — Errores_y_Excepciones_Java.pdf (cómo se crea una excepción propia).

Los **dos** constructores son necesarios: el de un argumento se usa cuando no hay causa técnica, el de dos cuando se envuelve `PersistenceException`.

### 3.2 `appNutritionist.dao.DAO`

```java
package appNutritionist.dao;

import java.util.List;

public interface DAO<T, ID> {

    void insert(T entidad) throws DaoException;
    T findById(ID id) throws DaoException;
    List<T> findAll() throws DaoException;
    void update(T entidad) throws DaoException;
    void delete(ID id) throws DaoException;
}
```

> pág. 14 y 15 — JDBC_DAO_MySQL.pdf

Ese `throws DaoException` es el que obliga a todo el resto del sistema a decidir qué hacer con el error. Al ser **checked** (`extends Exception`), el compilador no deja ignorarla: por eso la rúbrica premia este diseño.

### 3.3 `appNutritionist.dao.PatientDAOJPA`

```java
package appNutritionist.dao;

import appNutritionist.model.Patient;
import java.util.List;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.PersistenceException;

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
            throw new DaoException(
                "No se pudo actualizar el paciente con id " + patient.getId(), e);

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
}
```

### 3.4 `appNutritionist.dao.AlimentDAOJPA`

Estructura idéntica, cambiando sólo los textos:

```java
// insert
throw new DaoException("No se pudo insertar el alimento", e);

// findById
throw new DaoException("No se pudo buscar el alimento con id " + id, e);

// findAll  (alias sugerido: "a", no "p")
return em.createQuery("SELECT a FROM Aliment a", Aliment.class)
         .getResultList();
// catch
throw new DaoException("No se pudo listar los alimentos", e);

// update  (Aliment YA tiene getId(), commit d8e40df)
throw new DaoException("No se pudo actualizar el alimento con id " + aliment.getId(), e);

// delete
throw new DaoException("No se pudo eliminar el alimento con id " + id, e);
```

### 3.5 `DietDAOJPA` y `DietDetailDAOJPA`

Misma estructura, cambiando sólo el texto de los cinco mensajes: `"la dieta"` y `"el detalle de la dieta"` (ver tabla de la sección 2).

> [!tip] Detalle de idioma en `Aliment`
> El campo de `Aliment` se llama `aliment` y su getter es `getAliment()`, así que la tabla de `AlimentView` muestra `aliment.getAliment()`. No es un error, pero a la hora de leer el código conviene tenerlo presente.

---

## 4. Cómo llega el mensaje hasta el usuario

> [!warning] Esto NO es del profesor
> Es el criterio de equipo para conectar la capa DAO con la vista. El profesor define los mensajes (pág. 16 a 18), no el `JOptionPane`.

Cuando se conecten los botones de `PatientController`:

```java
private void insertBtActionPerformed(java.awt.event.ActionEvent evt) {
    try {
        Patient nuevoPaciente = buildPatientFromForm();
        patientDAO.insert(nuevoPaciente);

        JOptionPane.showMessageDialog(view,
            "Paciente guardado correctamente.",
            "Alta exitosa",
            JOptionPane.INFORMATION_MESSAGE);

    } catch (DaoException ex) {
        JOptionPane.showMessageDialog(view,
            ex.getMessage(),                 // "No se pudo insertar el paciente"
            "Error al guardar",
            JOptionPane.ERROR_MESSAGE);

        ex.printStackTrace();                // el detalle técnico, para el programador
    }
}
```

Lo importante: **el `catch` es uno solo para los cinco botones**. No hay `if` mirando qué método se llamó, porque el mensaje ya viene armado desde el DAO. Eso es exactamente lo que la excepción personalizada compra.

Y el flujo completo queda así:

```text
Botón "Insert"
   └── PatientController  (arma el Patient, decide qué cartel mostrar)
        └── PatientDAOJPA.insert()
             └── em.persist()  ──►  PersistenceException
                  └── throw new DaoException("No se pudo insertar el paciente", e)
                       ├── getMessage() → JOptionPane  (usuario)
                       └── getCause()    → printStackTrace / log  (programador)
```

---

## 5. Detalles finos

> [!warning] Esto NO es del profesor
> Observaciones sobre el código de la sección 3. Si alguna contradice el material de la cátedra, gana el profesor.

1. **El mensaje describe la operación, nunca la causa técnica.** `"No se pudo insertar el paciente"` sí; `"Duplicate entry '3' for key 'PRIMARY'"` no. Ese texto va en el `cause`.
2. **`catch (PersistenceException e)` no atrapa todo.** Un `NullPointerException` o un `IllegalArgumentException` de nuestra propia aplicación no es un problema de persistencia y **no** debe convertirse en `DaoException`. Está bien que escape.
3. **Cuidado con `patient.getId()` dentro del `catch`.** Si `patient` fuera `null`, el armado del mensaje lanza `NullPointerException` y tapa el error original. Si querés blindarlo, guardá el id antes del `try`:
   ```java
   int id = patient != null ? patient.getId() : -1;
   ```
4. **`tx.rollback()` también puede fallar.** Si el `rollback()` lanza, se pierde la excepción original. El código del profesor no lo blinda; si querés ser prolijo, envolvelo en su propio `try`/`catch`.
5. **`findById` que devuelve `null` no es un error de persistencia.** La búsqueda funcionó: no había nadie con ese id. El mensaje `"No se pudo buscar el paciente con id 5"` sería mentira. Ese caso lo resuelve **el controlador**, con su propio texto: `"No existe un paciente con id 5"`. Distinguir *"falló"* de *"no existe"* es la diferencia entre un programa que comunica y uno que confunde.
6. **Consistencia gramatical.** `"la dieta"`, `"el alimento"`, `"el detalle de la dieta"`. Un mensaje con el género cambiado se nota más que un error de SQL.
7. **`printStackTrace()` sirve para la entrega, no para producción.** En producción va a un archivo de log (SLF4J / Log4j).
8. **No hace falta una jerarquía de excepciones** (`PatientNotFoundException`, `DuplicateKeyException`…). La rúbrica pide *"excepciones personalizadas y mensajes claros"*, y una sola `DaoException` con buenos mensajes ya cumple.

---

## 6. Estado real del proyecto (relevado el 2026-11-06)

| Pieza | Estado | Qué falta |
|---|---|---|
| `DaoException.java` | **No existe** en `src/appNutritionist/dao/` | Crearlo (está listo en `Desktop\Entrega`) |
| `DAO.java` | Sin `throws DaoException` en ningún método | Agregarlo a los 5 métodos |
| `PatientDAOJPA` | `catch (RuntimeException e) { ... throw e; }` | Envolver en `DaoException` |
| `AlimentDAOJPA` | Ídem | Ídem |
| `DietDAOJPA` / `DietDetailDAOJPA` | Ídem | Ídem |
| `Aliment.java` | **Completa** (`id`, `aliment`, `category`, `caloriesPerServing`) con `getId()` | Nada |
| `Diet.java` / `DietDetail.java` | Vacías / parciales | Ver [[hoja-de-ruta-7-a-9]], Bloque A |
| Interfaces `*DAO` | Extienden `DAO<T, Integer>` sin `throws` | Agregar `throws DaoException` donde declare métodos propios |

### Checklist para aplicar este ejemplo

- [ ] Crear `src/appNutritionist/dao/DaoException.java`.
- [ ] Agregar `throws DaoException` a los 5 métodos de `DAO`.
- [ ] Agregar `throws DaoException` en `PatientDAO`, `AlimentDAO`, `DietDAO`, `DietDetailDAO` (incluido `findByLastName`).
- [ ] Reemplazar `catch (RuntimeException e)` por `catch (PersistenceException e)` en las 4 implementaciones.
- [ ] Poner el mensaje específico en cada uno de los 20 `catch` (5 métodos × 4 entidades).
- [ ] Compilar con `javac --release 25 -cp "lib/jakarta.persistence-2.2.3.jar;lib/flatlaf-3.5.4.jar"`.

> [!warning] No compila hasta que esté completo
> Al agregar `throws DaoException` en la interfaz, **todas** las implementaciones dejan de compilar hasta que declaren el `throws`. Es esperable: conviene hacer el cambio de los 5 archivos en una sola pasada.

---

## 7. Dudas y vacíos

- [ ] **¿En qué idioma van los mensajes?** El profesor los escribe en español (pág. 16 a 18). Nuestra UI está en inglés (`Aliments`, `Search`, `Insert`, `Patients`). Si el cartel muestra `ex.getMessage()`, en la misma pantalla conviven dos idiomas. Hay que decidir: traducir la UI o los mensajes. **Recomendación:** mensajes en el idioma de la interfaz.
- [ ] ¿`DietDetail` necesita su propio `DAOJPA` si los detalles se guardan por `cascade` desde `Diet`? (Ver la decisión de diseño en [[hoja-de-ruta-7-a-9]], Bloque A.)
- [ ] ¿La cátedra quiere ver el error técnico en algún lado (log, archivo) o alcanza con `printStackTrace()` en consola?

## Relacionado

- [[tutorial-dao-desde-cero]] — el patrón DAO completo, desde cero
- [[Errores_y_Excepciones_Java]] — checked vs. unchecked y excepciones personalizadas
- [[JDBC_DAO_MySQL]] — `DaoException` y DAO en JDBC (pág. 13 a 18)
- [[JPA_Hibernate_DAO]] — el mismo patrón con `PersistenceException` (pág. 18 a 22)
- [[enunciado_rubrica]] — criterio "Manejo de Excepciones" (pág. 2)
- [[estado]] — estado actual del código
- [[hoja-de-ruta-7-a-9]] — Bloque 0: dónde encaja este paso

---

**Navegación:** [[index]] · [[MAP]]
