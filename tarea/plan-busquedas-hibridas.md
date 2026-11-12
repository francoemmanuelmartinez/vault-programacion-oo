---
fuente: pedido del Estudiante (2026-11-12) + auditoría de `origin/master` (`0d47764`) + las interfaces `PatientDAO`, `AlimentDAO` y `DietDAO` + enunciado_rubrica.pdf (pág. 1 y 2)
tags: [tarea, plan, busquedas, swing, crud, patient, aliment, diet]
estado: implementado
actualizado: 2026-11-12
aliases: [plan-busquedas, busqueda-hibrida, search-hibrido]
---

# Plan: búsqueda híbrida (texto o ID) y campo ID editable

> [!info] Qué se pide
> Dos cosas que van juntas:
>
> 1. El campo **ID** tiene que ser **tipeable en las tres vistas** —hoy `DietView` lo tiene bloqueado— para poder escribir el id a buscar.
> 2. El botón **Search** tiene que decidir solo: si el campo de texto de la entidad (**apellido**, **categoría**, **nombre**) tiene algo, busca por ahí; si está vacío, busca por **ID**; si tampoco hay ID, recarga la lista completa.
>
> El campo ID se puede escribir, pero **el id del registro no se modifica nunca**: los `modify()` ya lo toman de `selectedX.getId()` e ignoran el texto.

> [!warning] Procedencia
> - **Del profesor:** el `Search by Last Name` que pide la rúbrica (`enunciado_rubrica.pdf`, pág. 1 y 2) y la separación Vista → Controlador (`Swing_MVC_Interfaces_Java.pdf`, pág. 14 y 20 a 23).
> - **Del Estudiante:** la regla de decisión (texto primero, ID después) y el ID editable.
> - **Criterio de equipo/IA:** qué pasa cuando hay varias coincidencias, si se filtra la tabla o se selecciona una fila, y los mensajes.

---

## 0. Estado actual

| Vista | Campo ID | Campo de texto | `search()` hoy |
|---|---|---|---|
| `PatientView` | editable (`PatientView.java:135`) | `lastNameTextField` | sólo `findById` |
| `AlimentView` | editable (`AlimentView.java:74`) | `categoryComboBox` | sólo `findById` |
| `DietView` | **no editable** (`DietView.java:135`, `setEditable(false)`) | `nameTextField` | sólo `findById` — **inalcanzable** |

Los tres `search()` hacen lo mismo: `Integer.parseInt(view.getIdText())` → `findById` → si existe, cargan el formulario; si no, mensaje. **Ninguno usa los métodos `buscarPorApellido`, `buscarPorCategoria` ni `buscarPorNombre`**, que existen en las interfaces y tienen el JPQL roto (H2, H3 y H4 de [[plan-arreglos-h1-h10]]).

---

## 1. La regla de decisión

Una sola regla, aplicada a las tres entidades:

```
si el campo de texto tiene algo    → buscar por ese campo (filtra la tabla)
sino si el ID tiene algo           → buscar por id (carga el formulario)
sino                              → recargar todo (findAll)
```

| Entidad | Campo de texto que manda | Método del DAO |
|---|---|---|
| `Patient` | apellido (`getLastNameText()`) | `buscarPorApellido` (H4) |
| `Aliment` | categoría del combo (`getCategoryIndex() > 0`) | `buscarPorCategoria` (H2) |
| `Diet` | nombre (`getNameText()`) | `buscarPorNombre` (H3) |

> [!important] La categoría no es texto libre
> En `Aliment` el criterio no es "hay texto" sino **"el combo no está en el placeholder"**: índice `> 0`. Si el usuario eligió `Frutas`, busca por `Frutas`; si está en *-- Elegí una categoría --*, pasa al ID.

---

## 2. Qué hace cada resultado

| Situación | Qué pasa | Por qué |
|---|---|---|
| Texto con **1 o más** coincidencias | la tabla se **filtra** con esa lista; el formulario se limpia; mensaje *"N paciente(s) encontrados."* | el usuario ve el conjunto, no una fila elegida al azar |
| Texto **sin** coincidencias | tabla vacía; mensaje *"No hay pacientes con apellido fer."* (o el equivalente) | tabla vacía + mensaje es más claro que "quedó la anterior" |
| Texto **y** ID cargados | gana el **texto**; el ID se ignora | es la regla pedida |
| Texto vacío + ID válido | **carga el formulario** y deja el registro seleccionado (como hoy) | mantiene el comportamiento actual y habilita `Modify`/`Delete` |
| Texto vacío + ID inválido | *"No existe un paciente con id 999"* | igual que hoy |
| Ambos vacíos | `findAll()` y tabla completa | es el "volver a ver todos" sin botón extra |

> [!warning] Hay una asimetría, y es a propósito
> Buscar **por ID** deja el registro seleccionado: `Modify` y `Delete` funcionan sin tocar la tabla.
> Buscar **por texto** sólo filtra: para modificar hay que **hacer clic en la fila**.
>
> Es coherente (la búsqueda por texto puede devolver varios) pero hay que saberlo, porque el botón se comporta distinto según lo que haya cargado.

---

## 3. Decisiones confirmadas

> **Confirmadas por el Estudiante el 2026-11-12**: D1 a D5 tal como están propuestas.

| # | Pregunta | Propuesta | Estado |
|---|---|---|---|
| D1 | ¿Si hay texto y ID, gana el texto? | **Sí** — es lo pedido | ✅ |
| D2 | ¿Filtrar la tabla o seleccionar la primera coincidencia? | **Filtrar**: con varias coincidencias, seleccionar una es arbitrario | ✅ |
| D3 | Al filtrar, ¿se limpia el formulario? | **Sí**, con `resetFields()`, para no mostrar datos de alguien que puede no estar en la lista | ✅ |
| D4 | ¿El mensaje de "sin resultados" es error o información? | **Información** (`showInfo`): no es un error del usuario | ✅ |
| D5 | ¿Se restaura el criterio en el campo después de filtrar? | **No**: el mensaje dice por qué está filtrada, y el campo queda libre para la próxima búsqueda | ✅ |

---

## 4. Los cambios

### 4.1 `DietView` — el ID editable

En el diseñador: seleccionar `idTextField` → **Properties** → tildar `editable` (hoy está destildado). En el `.form` desaparece `<Property name="editable" type="boolean" value="false"/>` y el `.java` pierde `setEditable(false)`.

El tooltip actual, *"Asigne automatically"*, hay que cambiarlo: el id **no se asigna a mano**, pero el campo se usa para buscar.

```java
// queda así, sin la linea de setEditable(false)
idTextField.setToolTipText("ID automatico. Escribilo para buscarlo.");
```

> [!tip] "Editable pero no modificable" ya está garantizado
> `modify()` hace `d.setId(selectedDiet.getId())` en las tres entidades. El texto del campo **nunca** viaja al `UPDATE`. Lo único que hay que probar es que un id tipeado no cambie nada (casos B12 y B13).

### 4.2 `PatientController.search()` — apellido o ID

```java
    private void search() {
        String apellido = view.getLastNameText().trim();
        String idTexto = view.getIdText().trim();
        try {
            if (!apellido.isEmpty()) {
                List<Patient> encontrados = patientDao.buscarPorApellido(apellido);
                tableModel.setPatients(encontrados);
                resetFields();
                view.showInfo(encontrados.isEmpty()
                        ? "No hay pacientes con apellido " + apellido + "."
                        : encontrados.size() + " paciente(s) encontrados.");
                return;
            }
            if (!idTexto.isEmpty()) {
                int id = Integer.parseInt(idTexto);
                Patient p = patientDao.findById(id);
                if (p == null) {
                    view.showError("No existe un paciente con id " + id);
                } else {
                    selectedPatient = p;
                    showPatient(p);
                }
                return;
            }
            refreshTable();
            resetFields();
        } catch (NumberFormatException ex) {
            view.showError("El ID debe ser un numero.");
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
    }
```

### 4.3 `AlimentController.search()` — categoría o ID

```java
    private void search() {
        String idTexto = view.getIdText().trim();
        try {
            if (view.getCategoryIndex() > 0) {
                String categoria = CATEGORIAS[view.getCategoryIndex() - 1];
                List<Aliment> encontrados = alimentDao.buscarPorCategoria(categoria);
                tableModel.setAliments(encontrados);
                resetFields();
                view.showInfo(encontrados.isEmpty()
                        ? "No hay alimentos en la categoria " + categoria + "."
                        : encontrados.size() + " alimento(s) encontrados.");
                return;
            }
            if (!idTexto.isEmpty()) {
                int id = Integer.parseInt(idTexto);
                Aliment a = alimentDao.findById(id);
                if (a == null) {
                    view.showError("No existe un alimento con id " + id);
                } else {
                    selectedAliment = a;
                    showAliment(a);
                }
                return;
            }
            refreshTable();
            resetFields();
        } catch (NumberFormatException ex) {
            view.showError("El ID debe ser un numero.");
        } catch (DAOException ex) {
            view.showError(ex.getMessage());
        }
    }
```

> [!note] `CATEGORIAS` ya existe
> Es la constante del combo de categoría ([[plan-categoria-combo]]). `getCategoryIndex() - 1` la mapea al valor real, igual que `selectedCategory()`.

### 4.4 `DietController.search()` — nombre o ID

Mismo esqueleto, con `buscarPorNombre(view.getNameText().trim())` y `dietModel.setDiets(encontrados)`.

> [!warning] Esta búsqueda depende de H3
> `buscarPorNombre` tiene el JPQL roto (`FROM diets`). Sin el commit 0 de [[plan-arreglos-h1-h10]], este `search()` tira excepción en vez de filtrar.

---

## 5. Lo que **no** hace falta tocar

| Pieza | Por qué |
|---|---|
| `PatientTableModel`, `AlimentTableModel`, `DietTableModel` | ya tienen `setPatients` / `setAliments` / `setDiets` que aceptan cualquier lista |
| Las vistas `PatientView` y `AlimentView` | el ID ya es editable |
| Los DAO | los tres métodos existen; el arreglo es de H1-H4 |
| `refreshTable()` | sigue siendo el "mostrar todo" |

---

## 6. Impacto en las pruebas

### Casos que cambian

| Caso | Antes | Después |
|---|---|---|
| P7 / A7 / D5 (search por ID) | se podía tipear el id directo | hay que **vaciar el campo de texto** primero (apellido / combo en placeholder / nombre) |
| P8 / A7 (id inexistente) | mensaje | igual, pero con el campo de texto vacío |
| D5 (search de `Diet`) | **inalcanzable** (el ID no se podía tipear) | funciona |

### Casos nuevos

| # | Caso | Esperado |
|---|---|---|
| B1 | Patient: apellido `fer`, ID vacío, Search | tabla con `Fernández`; mensaje *"1 paciente(s) encontrados."* |
| B2 | Patient: apellido `zzz`, Search | tabla vacía; *"No hay pacientes con apellido zzz."* |
| B3 | Patient: apellido vacío, ID `1`, Search | carga el formulario y el registro queda seleccionado |
| B4 | Patient: ambos vacíos, Search | vuelve a mostrar los 3 |
| B5 | Patient: apellido `fer` **y** ID `2`, Search | gana el apellido: filtra por `fer` |
| B6 | Aliment: combo en `Frutas`, ID vacío, Search | 2 filas (`Manzana`, `Banana`) |
| B7 | Aliment: combo en el placeholder, ID `3`, Search | carga `Pechuga de pollo` |
| B8 | Aliment: placeholder + ID vacío, Search | los 7 alimentos |
| B9 | Diet: nombre `hipo`, Search | 1 fila (`Hipocalórica`) |
| B10 | Diet: nombre `zzz`, Search | tabla vacía + mensaje |
| B11 | Diet: nombre vacío, ID `2`, Search | carga `Vegetariana` |
| B12 | **Las tres**: tipear un ID distinto al de la fila seleccionada y hacer `Modify` | modifica **la fila seleccionada**; el id no cambia |
| B13 | **Las tres**: tipear un ID, no seleccionar fila y hacer `Modify` / `Delete` | *"Selecciona un … de la tabla primero"*; no toca nada |
| B14 | Patient: filtrar por `fer` y hacer `Modify` sin clic | mensaje de "selecciona primero" (la búsqueda por texto no selecciona) |

> [!important] B12 y B13 son las que demuestran "editable pero no modificable"
> Sin ellas, el cambio de `DietView` parece un permiso para tocar el id. Lo que prueban es lo contrario: el texto del campo se ignora al guardar.

---

## 7. Orden de trabajo

| # | Commit | Archivos | Depende de |
|---|---|---|---|
| 1 | `Hago editable el ID de DietView para poder buscar` | `DietView.java` + `.form` | — |
| 2 | `Busqueda de Patient por apellido o ID` | `PatientController` | H4 |
| 3 | `Busqueda de Aliment por categoria o ID` | `AlimentController` | H2 |
| 4 | `Busqueda de Diet por nombre o ID` | `DietController` | H3 |

Los commits 2 a 4 van **después** del commit 0 de [[plan-arreglos-h1-h10]] (los JPQL), porque los tres dependen de esos métodos.

> [!note] Se commitea todo junto
> Decisión del 2026-11-12: un único commit al final, con el mensaje `Corrijo los JPQL y completo validaciones y busquedas del CRUD`. La numeración de arriba es el **orden de implementación y prueba**, no de commits.

---

## 8. Riesgos

| Riesgo | Cómo se mitiga |
|---|---|
| Quedarse con la tabla filtrada y no poder volver | "ambos campos vacíos = `findAll()`", probado en B4 y B8 |
| El combo de categoría siempre tiene algo y nunca se busca por ID | el placeholder cuenta como "vacío" (`índice 0`) |
| `buscarPorCategoria` compara **exacto** (`a.category = :cat`) | el combo manda el valor exacto de `CATEGORIAS`; si hubiera categorías con otra capitalización en la base, no aparecen |
| Confundir "ID editable" con "ID modificable" | casos B12 y B13 + el `setId(selectedX.getId())` de cada `modify()` |
| Buscar con el apellido del paciente que se está editando y perder el formulario | `resetFields()` limpia; la tabla filtrada y el mensaje explican que se buscó |

---

## 9. Lo que dice el profesor

> pág. 1 y 2 — enunciado_rubrica.pdf
> La aplicación tiene que permitir **buscar** los registros, no sólo listarlos; el ejemplo de la cátedra es `Search by Last Name`.

> pág. 14 y 20 a 23 — Swing_MVC_Interfaces_Java.pdf
> El botón dispara una acción del controlador; la vista sólo captura el criterio y muestra el resultado.

**Qué se toma:** la búsqueda por texto como requisito y el botón conectado al controlador. **Qué agrega el equipo:** la regla de fallback al ID, el filtrado de la tabla y los mensajes.

---

## 10. Ampliación de la IA

> [!note] Filtrar vs. seleccionar
> Un `Search` que "salta" al primer resultado es más rápido cuando hay uno solo y frustrante cuando hay varios: el usuario no sabe que había más. Filtrar la tabla muestra el conjunto y deja que él elija. El costo es que hay que acordarse de limpiar el criterio para volver a la lista completa — por eso "ambos vacíos = todos".

> [!note] Por qué el ID no debería ser el criterio principal
> Un id es un detalle interno: nadie se acuerda si `Palta` es el 7. La búsqueda por texto es la que usa una persona. El ID queda como atajo para cuando ya lo conocés (por ejemplo, después de un alta, que el mensaje dice *"guardado con id 7"*).

> [!note] La validación de "texto vacío"
> `isEmpty()` después de `trim()` evita que un espacio haga que la búsqueda por texto se dispare con un criterio vacío y devuelva todo sin avisar.

---

## Relacionado

- [[plan-arreglos-h1-h10]] — los JPQL que estas búsquedas necesitan (H2, H3, H4)
- [[plan-de-pruebas-crud]] — los casos P7, A7 y D5, y los nuevos B1 a B14
- [[plan-categoria-combo]] — la constante `CATEGORIAS` y la API por índices
- [[plan-cableado-patient]] — el `search()` original y `parsePositive`
- [[plan-crud-bd]] — la Fase 5, que proponía estas búsquedas
- [[enunciado_rubrica]] — el `Search by Last Name`
- [[MAP]] · [[index]]
