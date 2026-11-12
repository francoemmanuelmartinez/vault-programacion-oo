---
fuente: Javadoc_en_Java.pdf (pág. 1 a 13) + enunciado_rubrica.pdf (pág. 1 y 2) + auditoría del código en el working copy sobre `9a1b06a` (2026-11-12)
tags: [tarea, plan, javadoc, documentacion, rubrica]
estado: en curso
actualizado: 2026-11-12
aliases: [plan-javadoc, javadoc, documentacion-api]
---

# Plan: Javadoc en clases y métodos públicos

> [!info] Qué es esta nota
> La rúbrica pide **Javadoc para las clases y los métodos públicos**, y el nivel máximo de *Documentación y Código Limpio* dice "documentado en **todas las entidades**".
>
> **Estado:** los 7 `{@link}` rotos ya están arreglados, y están documentados los modelos, los DAO, los controladores, los modelos de tabla, el main y `AlimentView`. Faltan **62**: `DietView` (29) y `PatientView` (33).

> [!warning] Procedencia
> - **Del profesor:** qué es Javadoc, la diferencia con los comentarios comunes, el formato, el orden de las etiquetas y cómo se genera (pág. 2, 3, 6, 7, 8 y 12 — `Javadoc_en_Java.pdf`), y el requisito de la rúbrica (`enunciado_rubrica.pdf`, pág. 1 y 2).
> - **Criterio de equipo/IA:** las plantillas concretas, el orden de trabajo y el criterio de aceptación.

---

## 0. El requisito

> pág. 1 y 2 — enunciado_rubrica.pdf
> **Javadoc:** será necesaria la realización de Javadoc para los **métodos y clases públicas**.

> pág. 2 — enunciado_rubrica.pdf
> *Documentación y Código Limpio* — **Avanzado:** "código bien estructurado y documentado en **todas las entidades**, fácil de leer, con buenas prácticas de programación".

Traducido: no alcanza con un par de comentarios. Se evalúa el conjunto, y "todas las entidades" son `Patient`, `Aliment` y `Diet` (con `DietDetail` como pivot).

---

## 1. Cómo se mide (y por qué no con `doclint`)

Primer intento: correr `javadoc` con `-Xdoclint:all` y contar los warnings `no comment`. **No sirve** por dos motivos, y los dos se descubrieron a los golpes:

1. **`doclint` cuenta los campos privados.** Da 204 faltantes, cuando la rúbrica pide clases y métodos **públicos**. Documentar 55 campos privados no suma nada.
2. **`javadoc` corta a los 100 warnings por defecto** (`-Xmaxwarns`). Las primeras corridas siempre daban "100" y parecía que el arreglo no bajaba el número: en realidad estaba ocultando el resto.

La métrica que sí sirve es un conteo directo sobre el fuente: **elemento público sin un bloque `/** ... */` inmediatamente antes** (salteando anotaciones y líneas en blanco).

```powershell
$files = Get-ChildItem -Recurse src -Filter *.java
foreach ($f in $files) {
    $lines = Get-Content $f.FullName
    for ($i = 0; $i -lt $lines.Count; $i++) {
        $l = $lines[$i]
        if ($l -match '^\s*public\s' -and ($l -match '\(' -or $l -match '\b(class|interface|enum)\b')) {
            # retrocede hasta 15 lineas buscando el cierre del Javadoc,
            # sin cruzar otra declaracion (tolera @Table(...) multilinea)
            $j = $i - 1
            $lim = [Math]::Max(0, $i - 15)
            $doc = $false
            while ($j -ge $lim) {
                $t = $lines[$j].Trim()
                if ($t.EndsWith('*/')) { $doc = $true; break }
                if ($t -match '^\s*(public|private|protected)\b') { break }
                $j--
            }
            if (-not $doc) { "$($f.Name):$($i+1): $($l.Trim())" }
        }
    }
}
```

> [!warning] El script también tiene su trampa
> La primera versión salteaba sólo líneas en blanco y anotaciones de **una** línea. Con `@Table(name = …,` partido en varias líneas se detenía antes de llegar al `*/` y marcaba como "sin documentar" una clase que sí lo estaba (`DietDetail`). Por eso ahora retrocede hasta 15 líneas y corta al cruzar otra declaración.

### 1.1 El número real: 86

| Archivo | Faltan |
|---|---|
| `view/PatientView.java` | **33** |
| `view/DietView.java` | **29** |

**Total: 62**, todos getters y setters de las dos vistas que faltan.

> [!note] Los `isCellEditable` generados
> Cada vista tiene un `public boolean isCellEditable(...)` dentro del `DefaultTableModel` anónimo que genera el diseñador, en la zona `GEN-`. **No se documentan**: el próximo guardado del `.form` los reescribe. Son 3 en total y quedan fuera del conteo a propósito.

### 1.2 Lo ya resuelto

- Los **7 `{@link}` rotos** (bloque 1): `{@link Alimento}` → `Aliment`, `{@link Dieta}` → `Diet`, `{@link Paciente}` → `Patient`, `{@link DetalleDieta}` → `DietDetail`, y los DAO equivalentes. `javadoc` pasó de **exit 1** a **exit 0**.
- **`Aliment`, `Diet`, `DietDetail` y `Patient` completos**: constructores, todos los getters/setters, `calcularIMC` y los `toString`, con `@param` y `@return`.

### 1.3 Un archivo que queda fuera del conteo

`dao/PatientDAOInMemory.java` es el remanente de la etapa con DAO en memoria que se descartó al pasar a JPA. **Nadie lo usa**, y tiene 7 métodos públicos sin Javadoc.

**Decisión del 2026-11-12:** no se borra ni se documenta. El Estudiante le va a poner un comentario explicando **por qué y para qué** se hizo la clase, para que el profesor entienda que fue parte del proceso y no código olvidado.

Por eso esos 7 elementos **no cuentan** en los 86 pendientes.

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
| Campos privados | **no** se documentan (la rúbrica habla de públicos) |
| Métodos privados | **no** (no forman la API) |

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

### 4.2 Método con retorno

```java
    /**
     * Calcula el índice de masa corporal con la fórmula peso / altura².
     *
     * @return el IMC, o {@code 0} si la altura no es mayor a cero
     */
    public double calcularIMC() { ... }
```

### 4.3 Método con parámetros y excepción

```java
    /**
     * Busca los alimentos que pertenecen a una categoría.
     *
     * @param categoria nombre exacto de la categoría
     * @return la lista de alimentos de esa categoría; vacía si no hay ninguno
     * @throws DAOException si falla la consulta a la base
     */
    List<Aliment> buscarPorCategoria(String categoria) throws DAOException;
```

### 4.4 Interfaz DAO

```java
/**
 * Operaciones de acceso a datos para {@link Aliment}.
 * Extiende las operaciones comunes de {@link DAO}.
 */
public interface AlimentDAO extends DAO<Aliment, Integer> { ... }
```

### 4.5 Implementación JPA

```java
/**
 * Implementación JPA de {@link AlimentDAO}.
 */
public class AlimentDAOJPA extends DAOJpa<Aliment, Integer> implements AlimentDAO { ... }
```

### 4.6 Controlador

```java
/**
 * Controlador de la pantalla de alimentos. Conecta los botones de la vista
 * con el DAO y mantiene el formulario sincronizado con la tabla.
 */
public class AlimentController { ... }
```

### 4.7 Getter y setter de una vista

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

> [!tip] Los 86 getters de las vistas son el grueso…
> …y también lo más rápido: una línea de resumen, `@return` o `@param`, y listo. `PatientView` (33), `DietView` (29) y `AlimentView` (24) concentran el 58% del trabajo.

### 4.8 Excepción

```java
/**
 * Se lanza cuando se intenta eliminar un registro que otros datos referencian.
 */
public class RecordInUseException extends DAOException { ... }
```

---

## 5. Orden de trabajo

| # | Bloque | Archivos | Elementos | Estado |
|---|---|---|---|---|
| 1 | **Los `{@link}`** | 6 archivos | 7 líneas | ✅ **hecho** |
| 2 | **Modelos del dominio** | `Aliment`, `Diet`, `DietDetail`, `Patient` | 29 | ✅ **hecho** |
| 3 | **`PatientDAOInMemory`** | lo comenta el Estudiante (por qué y para qué se hizo) | — | ✅ decidido, fuera del conteo |
| 4 | **DAO** | `DAOJpa` 5, `PatientDAOJPA` 3, `AlimentDAOJPA` 2, `DietDAOJPA` 2, `DietDetailDAOJPA` 1 | 13 | ✅ **hecho** |
| 5 | **Controladores y main** | `AlimentController` 2, `PatientController` 2, `DietController` 1, `AppNutritionist` 2 | 7 | ✅ **hecho** |
| 6 | **Modelos de tabla** | los 4 | 31 | ✅ **hecho** |
| 7 | **Vistas** | `AlimentView` ✅ · `DietView` 29 · `PatientView` 33 | 62 | en curso |

> [!warning] El bloque 1 no era opcional
> Los `{@link}` rotos eran **errores**: `javadoc` cortaba y no producía el sitio. Se arreglaron primero, en una sola pasada.

---

## 6. Cómo se verifica

**1. El conteo de públicos** (el script de §1): tiene que dar **0**.

**2. `javadoc` sin errores de referencia**, ignorando el grupo `missing` (que es el que cuenta privados):

```powershell
& "C:\Program Files\Java\jdk-26\bin\javadoc.exe" --release 25 -encoding UTF-8 `
    -d C:\ruta\salida -sourcepath src -subpackages appNutritionist `
    -subpackages excepciones -subpackages util -classpath "lib/*" `
    -Xdoclint:all,-missing -quiet
```

| Métrica | Al empezar | Hoy | Meta |
|---|---|---|---|
| Errores (`{@link}`) | 7 | **0** | 0 |
| Públicos sin Javadoc | 166 | **62** | **0** |
| `javadoc` exit code | 1 | **0** | 0 |

**3. El sitio generado**: abrir `index.html` y comprobar que las tres entidades aparecen en el índice, que los `{@link}` son enlaces y que cada método público muestra su resumen.

> [!tip] Desde NetBeans
> *Project → Generate Javadoc*. Es la misma herramienta y sirve para ver el resultado sin salir del IDE.

---

## 7. Criterio de aceptación

1. El script de conteo da **0** elementos públicos sin Javadoc.
2. `javadoc` con `-Xdoclint:all,-missing` → **0 errores y 0 warnings**.
3. Toda clase, interfaz y excepción pública tiene resumen.
4. Todo método público tiene resumen; todos los parámetros tienen `@param`; todo método no `void` tiene `@return`; todo método que lanza una checked tiene `@throws` con la condición.
5. Los `{@link}` resuelven a clases existentes.
6. El sitio generado abre y navega.

---

## 8. Esfuerzo estimado

| Bloque | Tiempo |
|---|---|
| `{@link}` + modelos del dominio (36 elementos) | ✅ **hecho** (~1 h) |
| DAO (13) | ✅ **hecho** (~20 min) |
| Controladores, main y modelos de tabla (38) | ✅ **hecho** (~1 h) |
| Vistas (62 de 86) | `AlimentView` ✅ · `DietView` y `PatientView` 1 h 30 min |
| Generar, revisar y corregir | 30 min |

**Resta: ~2 horas**, repartibles entre los tres integrantes (uno por entidad, como pide la rúbrica: "documentado en todas las entidades").

---

## 9. Lo que queda afuera

- **Campos privados y métodos privados**: la rúbrica pide públicos.
- **`pom.xml` o Maven**: el proyecto es Ant/NetBeans.
- **Incluir el sitio HTML en el zip**: la duda está anotada en [[Javadoc_en_Java]] §Dudas. Alcanza con los `/** */` en el fuente y, si se quiere, una carpeta `doc/` generada.

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

> [!note] `-Xmaxwarns` es una trampa silenciosa
> `javadoc` corta en **100 warnings** por defecto. Si el conteo no baja al arreglar cosas, no es que el arreglo no sirva: es que hay más warnings escondidos. Se sube con `-Xmaxwarns 10000`. Nos pasó dos veces.

> [!note] `doclint` mide el código, no la rúbrica
> `-Xdoclint:all` incluye el grupo `missing`, que cuenta campos privados. Para la rúbrica (clases y métodos **públicos**) eso infla el número. `-Xdoclint:all,-missing` deja los chequeos que sí importan: referencias, HTML y sintaxis.

> [!note] Por qué no `@author` en cada clase
> El profesor lo lista en el orden de etiquetas (pág. 6), pero en un TP de tres integrantes termina siendo el mismo dato repetido 30 veces. Si prefieren incluirlo, va **antes** de `@param` y no rompe nada.

> [!note] `{@inheritDoc}`
> En las implementaciones JPA (`AlimentDAOJPA`), el contrato está en la interfaz. Se podría usar `{@inheritDoc}` para no repetir la descripción. Es válido, pero para un TP que se lee en el IDE conviene la descripción breve en cada clase.

---

## Relacionado

- [[Javadoc_en_Java]] — la fuente del profesor, pág. 1 a 13
- [[bug-jpql-dao]] — la sección *Extra: Javadoc*, con los `{@link}` rotos
- [[plan-de-pruebas-crud]] — la otra deuda de calidad antes de la entrega
- [[cambios-sobre-codigo-de-romi]] — qué código ajeno tocamos
- [[convenciones]] — nombres, getters y estilo del proyecto
- [[enunciado_rubrica]] — el requisito
- [[MAP]] · [[index]]
