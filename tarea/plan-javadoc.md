---
fuente: Javadoc_en_Java.pdf (pág. 1 a 13) + enunciado_rubrica.pdf (pág. 1 y 2) + auditoría con `javadoc -Xdoclint:all` sobre el commit `9a1b06a`
tags: [tarea, plan, javadoc, documentacion, rubrica]
estado: propuesto
actualizado: 2026-11-12
aliases: [plan-javadoc, javadoc, documentacion-api]
---

# Plan: Javadoc en clases y métodos públicos

> [!info] Qué es esta nota
> La rúbrica pide **Javadoc para las clases y los métodos públicos**, y el nivel máximo de *Documentación y Código Limpio* dice "documentado en **todas las entidades**". Hoy el proyecto tiene **87 elementos públicos sin un solo comentario**, **8 sin descripción principal** y **7 `{@link}` que apuntan a clases que no existen**.
>
> Este plan ordena el trabajo por archivo, fija la plantilla de cada tipo de clase y define cómo se verifica. Todo el marco viene de `Javadoc_en_Java.pdf`.

> [!warning] Procedencia
> - **Del profesor:** qué es Javadoc, la diferencia con los comentarios comunes, el formato, el orden de las etiquetas y cómo se genera (pág. 2, 3, 6, 7, 8 y 12 — `Javadoc_en_Java.pdf`), y el requisito de la rúbrica (`enunciado_rubrica.pdf`, pág. 1 y 2).
> - **Criterio de equipo/IA:** las plantillas concretas, el orden de trabajo, el uso de `@author` y el criterio de aceptación.

---

## 0. El requisito

> pág. 1 y 2 — enunciado_rubrica.pdf
> **Javadoc:** será necesaria la realización de Javadoc para los **métodos y clases públicas**.

Y en la rúbrica de criterios:

> pág. 2 — enunciado_rubrica.pdf
> *Documentación y Código Limpio* — **Satisfactorio/Avanzado:** "código bien estructurado y documentado en **todas las entidades**, fácil de leer, con buenas prácticas de programación".

Traducido: no alcanza con un par de comentarios. Se evalúa el conjunto, y "todas las entidades" son `Patient`, `Aliment` y `Diet` (con `DietDetail` como pivot).

---

## 1. Estado actual (medido, no estimado)

Generado con el JDK:

```powershell
& "C:\Program Files\Java\jdk-26\bin\javadoc.exe" --release 25 -encoding UTF-8 `
    -d <salida> -sourcepath src -subpackages appNutritionist `
    -subpackages excepciones -subpackages util -classpath "lib/*" `
    -Xdoclint:all -quiet
```

**Resultado:** `javadoc` termina con **exit 1**, **7 errores** y **100 warnings**.

| Tipo de warning | Cantidad | Qué significa |
|---|---|---|
| `no comment` | **87** | clase o método público sin ningún `/** ... */` |
| `no main description` | **8** | tiene etiquetas pero le falta la descripción de apertura |
| `use of default constructor…` | 5 | constructor implícito sin comentario |

### 1.1 Los 87 sin comentario, por archivo

| Archivo | Faltantes |
|---|---|
| `view/DietView.java` | **26** |
| `view/AlimentView.java` | **22** |
| `model/Aliment.java` | 9 |
| `model/DietDetail.java` | 7 |
| `model/Diet.java` | 7 |
| `view/AlimentTableModel.java` | 3 |
| `view/DietTableModel.java` | 3 |
| `AppNutritionist.java` | 2 |
| `controller/AlimentController.java` | 2 |
| `view/DietAlimentTableModel.java` | 2 |
| `dao/AlimentDAOJPA.java` · `dao/DietDAOJPA.java` · `dao/DietDetailDAOJPA.java` · `controller/DietController.java` | 1 cada uno |

> [!tip] Lo que ya está bien
> `Patient`, `PatientView`, `PatientController` y `PatientTableModel` **no aparecen** en la lista: ya están documentados. Sirven de molde y demuestran que el estilo se puede sostener.

### 1.2 Los 7 `{@link}` rotos

Son **errores** (no warnings): cortan la generación del sitio. Apuntan a nombres que quedaron de una versión anterior del modelo.

| Archivo | Línea | Dice | Debería decir |
|---|---|---|---|
| `dao/AlimentDAO.java` | 10 | `{@link Alimento}` | `{@link Aliment}` |
| `dao/AlimentDAOJPA.java` | 8 | `{@link AlimentoDAO}` | `{@link AlimentDAO}` |
| `dao/DietDAO.java` | 8 | `{@link Dieta}` | `{@link Diet}` |
| `dao/DietDAOJPA.java` | 8 | `{@link DietaDAO}` | `{@link DietDAO}` |
| `dao/DietDetailDAO.java` | 8 | `{@link DetalleDieta}` | `{@link DietDetail}` |
| `dao/PatientDAO.java` | 8 | `{@link Paciente}` | `{@link Patient}` |
| `model/Diet.java` | 11 | `{@link DetalleDieta}` | `{@link DietDetail}` |

> [!note] Uno menos que en [[bug-jpql-dao]]
> Esa nota contaba **ocho**: `DAOJpa:10` decía `{@link GenericDAO}` y ya está corregido a `{@link DAO}`.

### 1.3 Los 8 "no main description"

`AlimentView`, `DietView`, `DAOJpa`, `Navigator`, `Patient` y dos en `DuplicateRecordException`. Tienen etiquetas (`@param`, `@return`) pero les falta la primera oración que describe qué hace.

---

## 2. Las reglas del profesor

> pág. 2 — Javadoc_en_Java.pdf
> Javadoc es la herramienta oficial del JDK que genera documentación HTML navegable a partir de comentarios en el código. **No es un comentario interno: es un contrato legible** sobre el comportamiento del código para quien lo utiliza.

> pág. 3 — Javadoc_en_Java.pdf
> Los comentarios comunes (`//`, `/* */`) explican decisiones internas. Los de Javadoc (`/** ... */`) documentan la **API pública**: clases, métodos y atributos públicos o protegidos.

> pág. 6 — Javadoc_en_Java.pdf
> Formato `/** ... */`. **La primera oración es el resumen** que aparece en los índices y **debe terminar en punto**. Orden convencional de etiquetas: `@author`, `@param`, `@return`, `@throws`/`@exception`, `@see`, `@since`.

> pág. 7 — Javadoc_en_Java.pdf
> `@param` es **obligatorio** y va en el orden de la firma. `@return` es obligatorio **salvo en `void`**. `@throws` documenta cada excepción y **bajo qué condición** se lanza. `@see` referencia otra clase o método.

> pág. 8 — Javadoc_en_Java.pdf
> `{@code ...}` da formato de código en línea; `{@link ...}` inserta un **hipervínculo interno** (y falla si el destino no existe).

> pág. 12 — Javadoc_en_Java.pdf
> Se genera con el comando `javadoc` del JDK o desde el IDE (**NetBeans: *Project → Generate Javadoc***).

---

## 3. Convenciones de este proyecto

| Decisión | Valor |
|---|---|
| Idioma | **Español**, con tildes (es documentación, no texto de UI) |
| Primera oración | resumen de una línea, termina en punto |
| Largo | 1 a 3 líneas; los getters, una |
| `@author` | **no** se usa: sería el mismo en 30 clases y no aporta |
| `@since` | **no** se usa |
| `@see` | sólo donde hay una relación real (entidad ↔ DAO) |
| Campos privados | no se documentan, salvo que expliquen una regla (como `Patient.diet`) |
| Métodos privados | no (no forman la API) |

---

## 4. Plantillas por tipo

### 4.1 Clase de modelo

```java
/**
 * Entidad que representa un alimento del catálogo.
 * Cada alimento pertenece a una categoría y tiene un valor calórico por porción.
 */
@Entity
@Table(name = "Aliments")
public class Aliment implements Serializable { ... }
```

### 4.2 Método con parámetros y retorno

```java
    /**
     * Calcula el indice de masa corporal del paciente.
     *
     * @return el IMC redondeado a dos decimales
     */
    public double calcularIMC() { ... }
```

Con parámetros y excepción:

```java
    /**
     * Busca los alimentos que pertenecen a una categoria.
     *
     * @param categoria nombre exacto de la categoria
     * @return la lista de alimentos de esa categoria; vacia si no hay ninguno
     * @throws DAOException si falla la consulta a la base
     */
    List<Aliment> buscarPorCategoria(String categoria) throws DAOException;
```

### 4.3 Interfaz DAO

```java
/**
 * Operaciones de acceso a datos para {@link Aliment}.
 * Extiende las operaciones comunes de {@link DAO}.
 */
public interface AlimentDAO extends DAO<Aliment, Integer> { ... }
```

### 4.4 Implementación JPA

```java
/**
 * Implementacion JPA de {@link AlimentDAO}.
 */
public class AlimentDAOJPA extends DAOJpa<Aliment, Integer> implements AlimentDAO { ... }
```

### 4.5 Controlador

```java
/**
 * Controlador de la pantalla de alimentos. Conecta los botones de la vista
 * con el DAO y mantiene el formulario sincronizado con la tabla.
 */
public class AlimentController { ... }
```

### 4.6 Getter y setter de una vista

```java
    /**
     * Devuelve el texto del campo nombre.
     *
     * @return el nombre escrito por el usuario
     */
    public String getNameText() { return nameTextField.getText(); }

    /**
     * Carga el nombre en el formulario.
     *
     * @param s nombre a mostrar
     */
    public void setNameText(String s) { nameTextField.setText(s); }
```

> [!tip] Los 48 getters de las vistas son el grueso del trabajo…
> …y también los más rápidos: una línea de resumen, `@return` o `@param`, y listo. `DietView` (26) y `AlimentView` (22) suman más de la mitad de los 87.

### 4.7 Excepción

```java
/**
 * Se lanza cuando se intenta eliminar un registro que otros datos referencian.
 */
public class RecordInUseException extends DAOException { ... }
```

---

## 5. Orden de trabajo

De lo que más valor documental aporta a lo más mecánico:

| # | Bloque | Archivos | Elementos | Por qué primero |
|---|---|---|---|---|
| 1 | **Arreglar los `{@link}`** | 6 archivos | 7 líneas | sin esto el `javadoc` no genera nada |
| 2 | **Modelos** | `Aliment`, `Diet`, `DietDetail` | 23 | es la API del dominio |
| 3 | **DAO + excepciones** | 4 `DAOJPA`, `DAOJpa`, `DuplicateRecordException` | ~5 + 4 | contrato de persistencia |
| 4 | **Controladores** | `AlimentController`, `DietController`, `Navigator` | 4 | reglas de negocio |
| 5 | **Modelos de tabla** | `AlimentTableModel`, `DietTableModel`, `DietAlimentTableModel` | 8 | breve |
| 6 | **Vistas** | `AlimentView`, `DietView` | 48 | mecánico y voluminoso |
| 7 | **Main** | `AppNutritionist` | 2 | cierre |

> [!warning] El bloque 1 no es opcional
> Los `{@link}` rotos son **errores**: `javadoc` corta y no produce el sitio. Se arreglan primero, en una sola pasada, y después se documenta.

---

## 6. Cómo se verifica

```powershell
& "C:\Program Files\Java\jdk-26\bin\javadoc.exe" --release 25 -encoding UTF-8 `
    -d C:\ruta\salida -sourcepath src -subpackages appNutritionist `
    -subpackages excepciones -subpackages util -classpath "lib/*" `
    -Xdoclint:all
```

| Métrica | Hoy | Meta |
|---|---|---|
| `error:` | 7 | **0** |
| `warning: no comment` | 87 | **0** |
| `warning: no main description` | 8 | **0** |
| constructor por defecto | 5 | **0** |
| Exit code | 1 | **0** |

Y una verificación visual: abrir `index.html` del sitio generado y comprobar que:

- las tres entidades aparecen en el índice de clases;
- los `{@link}` son enlaces azules y **no** texto plano;
- cada método público muestra su resumen en la tabla del índice.

> [!tip] Desde NetBeans
> *Project → Generate Javadoc* con `-Xdoclint:all` en las opciones. Es la misma herramienta y sirve para ver el resultado sin salir del IDE.

---

## 7. Criterio de aceptación

1. `javadoc` con `-Xdoclint:all` → **0 errores y 0 warnings**.
2. Toda clase, interfaz y excepción pública tiene resumen.
3. Todo método público tiene resumen; **todos** los parámetros tienen `@param`; todo método no `void` tiene `@return`; todo método que lanza una checked tiene `@throws` con la condición.
4. Los 7 `{@link}` resuelven a clases existentes.
5. El sitio generado abre y navega.

---

## 8. Esfuerzo estimado

| Bloque | Tiempo |
|---|---|
| `{@link}` (7 líneas) | 10 min |
| Modelos + DAO + controladores | 1 h |
| Vistas (48 getters/setters) | 1 h 30 min |
| Generar, revisar y corregir warnings | 30 min |

**Total: ~3 horas**, repartibles entre los tres integrantes (uno por entidad, como pide la rúbrica: "documentado en todas las entidades").

---

## 9. Lo que queda afuera

- **`pom.xml` o Maven**: el proyecto es Ant/NetBeans; no se agrega plugin.
- **Incluir el sitio HTML en el zip**: la duda está anotada en [[Javadoc_en_Java]] §Dudas. Alcanza con los `/** */` en el fuente y, si se quiere, una carpeta `doc/` generada. Decisión pendiente del equipo.
- **Documentar métodos privados**: no los pide la rúbrica.

---

## 10. Lo que dice el profesor

> pág. 2 — Javadoc_en_Java.pdf
> «No es un comentario interno: es un contrato legible sobre el comportamiento del código para quien lo utiliza.»

> pág. 6 — Javadoc_en_Java.pdf
> «La primera oración es el resumen que aparece en las tablas de índice y debe terminar en punto.»

> pág. 7 — Javadoc_en_Java.pdf
> «`@param`: describe cada parámetro en el orden de la firma (obligatorio). `@return`: obligatorio salvo en `void`.»

**Qué se toma:** el formato, el orden de etiquetas, la obligatoriedad de `@param`/`@return` y la generación con `javadoc` o desde NetBeans. **Qué agrega el equipo:** las plantillas por tipo, el orden de trabajo y el criterio de aceptación medible.

---

## 11. Ampliación de la IA

> [!note] `doclint` es el juez objetivo
> `-Xdoclint:all` convierte el "está documentado" en un número. Es la diferencia entre "creo que falta poco" y "faltan 87". Vale la pena correrlo **antes** de empezar y **después** de cada bloque: los 87 bajan rápido y los errores de `{@link}` se ven enseguida.

> [!note] Por qué no `@author` en cada clase
> El profesor lo lista en el orden de etiquetas (pág. 6), pero en un TP de tres integrantes termina siendo el mismo dato repetido 30 veces. Si prefieren incluirlo, va **antes** de `@param` y no rompe nada.

> [!note] `{@inheritDoc}`
> En las implementaciones JPA (`AlimentDAOJPA`), el contrato está en la interfaz. Se podría usar `{@inheritDoc}` para no repetir la descripción. Es válido y elegante, pero para un TP que se lee en el IDE conviene la descripción breve en cada clase: se entiende sin saltar de archivo.

> [!note] El orden importa más de lo que parece
> Documentar primero las vistas (48 elementos mecánicos) y dejar los modelos para el final da la sensación de avance pero deja sin documentar lo que la cátedra llama "las entidades". Empezar por el dominio también hace que los `{@link}` de los DAO ya tengan destino cuando se escriban.

---

## Relacionado

- [[Javadoc_en_Java]] — la fuente del profesor, pág. 1 a 13
- [[bug-jpql-dao]] — la sección *Extra: Javadoc*, con los `{@link}` rotos
- [[plan-de-pruebas-crud]] — la otra deuda de calidad antes de la entrega
- [[convenciones]] — nombres, getters y estilo del proyecto
- [[estado]] — qué falta en general
- [[enunciado_rubrica]] — el requisito
- [[MAP]] · [[index]]
