---
fuente: `git log -S` sobre `origin/master` (`0d47764`) + estado del working copy al 2026-11-12
tags: [tarea, coordinacion, equipo, merge, romi, caro]
estado: en curso
actualizado: 2026-11-12
aliases: [cambios-sobre-romi, aviso-romi, codigo-ajeno]
---

# Cambios sobre código de Romi (y de Caro)

> [!info] Para qué es esta nota
> Todo lo que tocamos de código que **no escribimos nosotros**, para avisarle a Romi antes de la entrega y para no enterarnos por un conflicto de merge.
>
> Se actualiza a medida que avanzamos. **Estado al 2026-11-12:** los cambios están en el working copy, **sin commitear** (decisión: un commit único al final).

> [!warning] La autoría está verificada con `git log -S`
> No es "me parece que era de Romi": cada fila de abajo tiene el commit que introdujo ese código.

---

## 1. Autoría de los archivos que tocamos

| Archivo | Autor original | Commit |
|---|---|---|
| `dao/PatientDAOJPA.java` | **Romi** | `bc0f5b6` (y `797b39c`) |
| `dao/AlimentDAOJPA.java` | **Romi** | `bc0f5b6` |
| `dao/DietDAOJPA.java` | **Romi** | `bc0f5b6` |
| `controller/DietController.java` | **Romi** | `374b8e6` (y `452d1c4`) |
| `controller/AlimentController.java` | mixto: Romi `374b8e6` + Franco | `29d873a`, `374b8e6` |
| `controller/PatientController.java` | mixto: Romi `374b8e6` + Franco | `1f785b6`, `374b8e6` |
| `view/DietView.java` | **Caro** (`6f6a449`), getters de Romi (`374b8e6`) | — |

---

## 2. Cambios sobre código de Romi

### 2.1 Los cuatro JPQL — `bc0f5b6`

| Archivo | Método | Antes | Ahora |
|---|---|---|---|
| `PatientDAOJPA:19` | `buscarPorApellido` | `FROM patients p` | `FROM Patient p` |
| `PatientDAOJPA:26` | `buscarPorDieta` | `FROM patients p WHERE p.id_dieta = …` | `FROM Patient p WHERE p.diet.id = …` |
| `AlimentDAOJPA:19` | `buscarPorCategoria` | `FROM aliments a` | `FROM Aliment a` |
| `DietDAOJPA:19` | `buscarPorNombre` | `FROM diets d` | `FROM Diet d` |

**Por qué:** un JPQL usa el nombre de la **entidad**, no el de la tabla, y navega por **campos** (`diet`), no por columnas (`id_dieta`). H1 bloqueaba el borrado de dietas.

**Alcance:** 4 líneas. Sin cambios de firma, de tipos ni de lógica.

### 2.2 `DietController` — `374b8e6`

| Método | Qué cambió | Por qué |
|---|---|---|
| `addAliment()` | valida el combo vacío con dos mensajes distintos, usa la lista de disponibles y **pide confirmación** | H9 y H10 |
| `removeAliment()` | **pide confirmación** antes de borrar | H10 |
| `loadAliments()` | pasó de traer **todos** los alimentos a traer **sólo los que faltan** en la dieta | H9 |
| `refreshPanel()` | **método nuevo**: refresca tabla de abajo y combo juntos | H9 |
| 7 llamadas a `refreshDetails()` | pasaron a `refreshPanel()` | H9 |
| `insert()` / `modify()` | validan largo y duplicado, y atrapan `IllegalArgumentException` | H5 y H8 |
| `readForm()` | la descripción se normaliza a una línea | H6 |
| `search()` | búsqueda híbrida: **nombre → ID → todos** (filtra la tabla si hay nombre) | [[plan-busquedas-hibridas]] |

**Lo que NO tocamos de su controlador** (importante para el aviso):

- `delete()` con la **guarda de pacientes** (`patientDao.buscarPorDieta`) y la **cascada manual** de `DietDetail`.
- `selectRow()` con `convertRowIndexToModel`.
- El manejo de `DuplicateRecordException` en el panel.
- La navegación por `Navigator`.

### 2.3 Un comentario que dejó pegado

`DietView.java:391` tenía, dentro del archivo:

```java
// Pegar en DietView.java, DESPUES de "// End of variables declaration//GEN-END:variables"
// (fuera de las zonas GEN-). Siguen la misma convencion que PatientView y AlimentView.
```

Son instrucciones de armado, no código. Las borramos (decisión del 2026-11-12).

---

## 3. Cambios sobre código de Caro — `DietView`

| Qué | Quién lo hizo | Detalle |
|---|---|---|
| Tabla de dietas de 3 columnas | Franco (§2 bis) | `ID`, `Name`, `Description` |
| `descriptionTextArea` con `lineWrap`, `wrapStyleWord` y 3 filas | Franco | evita el scroll horizontal |
| Getters y setters de la vista | Franco | los necesitaba el controlador |
| Columna `Name` → `Aliment` en la tabla del panel | Franco | para que coincida con `DietAlimentTableModel` |
| Campo ID editable | Franco | para poder buscar por ID |
| `confirmAddAliment()` / `confirmRemoveAliment()` | Franco | **métodos nuevos**, no existían |

**Lo que NO tocamos de la vista de Caro:** la disposición de los paneles, los colores, los `TitledBorder`, los botones y el `GroupLayout`.

---

## 4. Lo que no tocamos de nadie

| Pieza | Autor | Estado |
|---|---|---|
| `controller/Navigator.java` | Romi | intacto |
| `AppNutritionist.java` | Romi | intacto |
| `PatientController`: constructor y navegación con `Navigator` | Romi | intacto (sólo se agregaron validaciones a `insert`/`modify`) |
| `AlimentController`: constructor y navegación | Romi | intacto (sólo se agregaron validaciones y el combo de categoría) |
| `dao/DietDetailDAOJPA.java` | Romi | intacto (su JPQL ya estaba bien) |
| `dao/DAOJpa.java` | Romi | intacto |
| `excepciones/ExceptionTranslator.java` | Romi | intacto |
| `excepciones/RecordInUseException.java` | Romi | intacto |
| `excepciones/DuplicateRecordException.java` | Romi | intacto |
| `view/DietTableModel.java` | Romi | intacto |
| `view/DietAlimentTableModel.java` | Romi | intacto |

---

## 5. Riesgo si Romi sigue trabajando

| Archivo | Zona | Riesgo de conflicto |
|---|---|---|
| `DietController` | `addAliment`, `removeAliment`, `loadAliments` | **medio** — son las tres zonas más reescritas |
| `DietController` | `insert`, `modify`, `readForm` | medio |
| `DietView` | bloque de getters al final | bajo |
| Los 3 `DAOJPA` | una línea cada uno | trivial |

**Recomendación:** avisarle **antes** de que vuelva a tocar `DietController` o `DietView`. Si ya lo hizo, el merge lo resolvemos juntos: las zonas están identificadas.

---

## 6. Mensaje sugerido para mandarle

> Romi, en la tanda de hoy toqué un poco de código tuyo. Te resumo para que no te sorprenda:
>
> - **Los 4 JPQL** de `PatientDAOJPA`, `AlimentDAOJPA` y `DietDAOJPA`: usaban el nombre de la tabla (`FROM patients`) en vez del de la entidad (`FROM Patient`), y `buscarPorDieta` usaba `p.id_dieta` en vez de `p.diet.id`. Sin eso no se podía borrar ninguna dieta.
> - **`DietController`**: el panel de alimentos ahora ofrece sólo los que faltan (`loadAliments`), y Add/Remove piden confirmación. Agregué `refreshPanel()` para refrescar la tabla y el combo juntos. `insert`/`modify` validan largos y duplicados.
> - **No toqué** `delete()` con la guarda de pacientes, ni `search()`, ni `selectRow()`, ni las excepciones nuevas, ni `DietTableModel` / `DietAlimentTableModel`.
>
> Si vas a tocar `DietController` o `DietView` avisame y lo mergeamos juntos.

---

## 7. Pendiente de esta nota

- ~~Búsquedas híbridas~~ — **hechas el 2026-11-12**: `search()` de los tres controladores. El de `Diet` es de Romi; los de `Patient` y `Aliment` son mixtos.
- Confirmar todo junto en el commit final.
- Registrar las notas nuevas en `index.md`, `MAP.md` y `log.md`.

---

## Relacionado

- [[plan-arreglos-h1-h10]] — el detalle de H1 a H10
- [[plan-de-pruebas-crud]] — los casos que verifican estos cambios
- [[plan-busquedas-hibridas]] — lo que falta y también toca `search()`
- [[bug-jpql-dao]] — el diagnóstico original de los cuatro JPQL
- [[log]] — bitácora de la tanda
- [[MAP]] · [[index]]
