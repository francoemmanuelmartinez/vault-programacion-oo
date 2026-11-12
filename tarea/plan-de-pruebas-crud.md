---
fuente: auditoría del código en `origin/master` (`0d47764`) + `Sql/nutricion.sql` + las pruebas ya anotadas en [[plan-crud-bd]] y [[plan-cableado-diet]] + enunciado_rubrica.pdf (pág. 1 y 2)
tags: [tarea, pruebas, qa, crud, patient, aliment, diet]
estado: en uso
actualizado: 2026-11-12
aliases: [plan-de-pruebas, pruebas-crud, qa-crud]
---

# Plan de pruebas del CRUD: `Patient`, `Aliment` y `Diet`

> [!info] Qué es esta nota
> Un plan de pruebas **manuales** para verificar que el CRUD de las tres entidades está completo y que no quedaron agujeros. No reemplaza las pruebas que ya están escritas en [[plan-crud-bd]] (23 casos) y [[plan-cableado-diet]] (16): las **consolida por entidad** y agrega lo que faltaba.
>
> **Punto de partida:** `origin/master` = `0d47764` (después del merge con Romi). Todo lo que sigue se verificó leyendo ese código.

> [!warning] Procedencia del contenido
> - **Del profesor:** los requisitos de la GUI y del CRUD de la rúbrica (`enunciado_rubrica.pdf`, pág. 1 y 2), y el patrón Vista → Controlador → Modelo (`Swing_MVC_Interfaces_Java.pdf`, pág. 14 y 20 a 23).
> - **De la cátedra no sale:** el catálogo de casos, las prioridades y los huecos detectados. Son criterio de equipo/IA, marcados como tales.

---

## 0. Para qué sirve

Tres preguntas concretas:

1. ¿Cada operación del CRUD de las tres entidades **funciona** contra MySQL?
2. ¿Los **caminos de error** muestran un mensaje y no un stacktrace?
3. ¿Quedó algo **sin cubrir**: métodos escritos que nadie llama, validaciones ausentes, JPQL que nunca se ejecutó?

La tercera es la que más rinde. **Un método roto que ningún controlador llama no rompe la demo… hasta que alguien lo llama.**

---

## 1. Entorno y preparación

### 1.1 Requisitos

| Pieza | Estado esperado |
|---|---|
| MySQL 9.7 | servicio iniciado, base `nutricion` |
| JDK 26 | el proyecto usa `--release 25` |
| Jars | `lib/` con los 4 (ya versionados en `0d47764`) |
| Proyecto | abierto en NetBeans, **sin** *Compile on Save* (guardar con `Ctrl+S` antes de correr) |

### 1.2 Datos de prueba

La base tiene que tener al menos:

- **5 alimentos** (`Manzana`, `Banana`, `Pechuga de pollo`, `Arroz integral`, `Yogur descremado`).
- **3 pacientes**, uno de ellos **sin** dieta.
- **2 dietas**, una **con 2 alimentos** y otra **sin** alimentos.

Si la base está vacía, correr los `INSERT` de `Sql/nutricion.sql` (líneas 41-62).

### 1.3 Datos extra que agregan los casos

| Dato | Para qué prueba |
|---|---|
| Un paciente **asignado** a la dieta 1 | D7 (no se puede borrar la dieta) |
| Un alimento **dentro** de una dieta | A9 (no se puede borrar el alimento) |
| Un alimento **libre** | A10 (se borra sin problema) |
| Una dieta **sin** pacientes | D6 (se borra con sus detalles) |

### 1.4 Entre corridas

Cada prueba que borra deja la base distinta. Antes de un bloque, reconstruir con:

```sql
-- volver al estado inicial (sin tocar el esquema)
DELETE FROM DietsDetails;
DELETE FROM Patients;
DELETE FROM Diets;
DELETE FROM Aliments;
-- y volver a correr los INSERT del script
```

> [!tip] Anotá el id de cada cosa
> Casi todas las pruebas usan el campo **ID** de la tabla. Anotá en un papel los ids de `Palta` (está en la dieta 2), del paciente con dieta y de la dieta con alimentos antes de empezar.

---

## 2. Cómo leer los casos

| Prioridad | Significado |
|---|---|
| **P0** | si falla, la entrega no está terminada |
| **P1** | debería pasar; si falla, se documenta y se arregla |
| **P2** | borde o cosmético |

Columna **Cubre**: `feliz` (camino normal), `borde` (valor límite), `error` (mensaje al usuario), `regresión` (algo que ya se rompió una vez).

La columna **Origen**: `nuevo` = no estaba en las notas anteriores; si dice `crud-bd #N` o `diet #N`, ya estaba y se repite acá por completitud.

---

## 3. Cobertura: qué hay que poder demostrar

| Operación | `Patient` | `Aliment` | `Diet` |
|---|---|---|---|
| Insert | P1-P6 | A1-A6 | D1-D4 |
| Search por ID | P7-P9 | A7 | D5 |
| Modify | P10-P13 | A8 | D6 |
| Delete | P14-P15 | A9-A10 | D7-D9 |
| Selección de fila | P16 | A11 | D10 |
| Combo / relación | P17-P18 | A12-A14 | D11-D16 |
| Persistencia | todas | todas | todas |

---

## 4. Casos comunes (C)

| # | P | Acción | Resultado esperado | Cubre |
|---|---|---|---|---|
| C1 | P1 | Abrir la app con la base **vacía** | las tres pantallas abren, tablas vacías, combos con sólo el placeholder | feliz |
| C2 | P0 | Abrir la app con MySQL **detenido** | mensaje de error legible, **sin** stacktrace en consola | error |
| C3 | P1 | Patients → Aliments → Diets → Patients | cada pantalla se abre **una sola vez** y los datos siguen ahí | feliz |
| C4 | P1 | Cerrar con la X en Patients | la app termina; no quedan ventanas huérfanas | borde |
| C5 | P1 | Redimensionar y maximizar cada pantalla | el autozoom no rompe el layout ni esconde botones | borde |
| C6 | P2 | Mirar los tooltips de Remove / Search | aparecen y son coherentes | borde |

> [!warning] C2 es la que más se pifia
> Si `AppNutritionist` construye los DAO en el `main` y la base no responde, el stacktrace aparece en la consola y la ventana no abre. Hay que ver qué muestra hoy antes de decidir si es aceptable para la entrega.

---

## 5. `Patient`

| # | P | Pasos | Resultado esperado | Cubre | Origen |
|---|---|---|---|---|---|
| P1 | P0 | Insert con nombre, apellido, peso `70`, altura `1.75`, sin dieta | guarda, mensaje con el id, formulario limpio, fila en la tabla | feliz | crud-bd #2 |
| P2 | P0 | Insert sin apellido | *"El nombre y el apellido son obligatorios."* y **no** guarda | error | crud-bd #3 |
| P3 | P1 | Insert con peso `abc` | *"Peso y altura deben ser numeros."* | error | nuevo |
| P4 | P1 | Insert con peso `0` | *"El peso debe ser mayor a cero."* | borde | crud-bd #4 |
| P5 | P1 | Insert con peso `62,5` | acepta la coma, guarda `62.5` | borde | crud-bd #5 |
| P6 | P1 | Insert con altura `-1` | *"La altura debe ser mayor a cero."* | error | nuevo |
| P7 | P0 | Search con el id de un paciente existente | carga los 6 campos, el combo y el IMC con 2 decimales | feliz | nuevo |
| P8 | P1 | Search con `9999` | *"No existe un paciente con id 9999"* | error | nuevo |
| P9 | P2 | Search con `abc` | *"El ID debe ser un numero."* | error | nuevo |
| P10 | P0 | Modify **sin** seleccionar fila | *"Selecciona un paciente de la tabla primero"* | error | nuevo |
| P11 | P0 | Seleccionar, cambiar el peso y Modify | persiste al reabrir; el IMC cambia | feliz | crud-bd #8 |
| P12 | P0 | Seleccionar un paciente **con dieta**, cambiar sólo el nombre, Modify | la dieta **se conserva** | regresión | crud-bd #9 |
| P13 | P1 | Modify e **Cancelar** la confirmación | no cambia nada | borde | nuevo |
| P14 | P0 | Delete → **Cancelar** | no borra | borde | crud-bd #10 |
| P15 | P0 | Delete → **OK** | desaparece de la tabla y de la base; el formulario se limpia | feliz | crud-bd #11 |
| P16 | P1 | Ordenar por *Last Name* y hacer clic en una fila | carga el paciente **de esa fila** | regresión | crud-bd #7 |
| P17 | P1 | Paciente **sin** dieta | la columna Diet muestra `-` y el combo queda en *-- Sin dieta --* | borde | crud-bd #18 |
| P18 | P1 | Asignar la **misma dieta** a dos pacientes | permite; la dieta queda compartida | borde | nuevo |
| P19 | P2 | Insertar dos pacientes con el mismo nombre y apellido | permite el duplicado (no hay restricción) | borde | nuevo |
| P20 | P2 | Insertar un nombre de **81 caracteres** | error de MySQL (la columna es `VARCHAR(80)`) → ver **H5** | borde | nuevo |
| P21 | P1 | Delete de un paciente que **tiene** dieta | borra (la FK va en el sentido paciente → dieta) | borde | nuevo |
| P22 | P1 | Cerrar y reabrir la app | todos los datos siguen | feliz | nuevo |

---

## 6. `Aliment`

| # | P | Pasos | Resultado esperado | Cubre | Origen |
|---|---|---|---|---|---|
| A1 | P0 | Insert `Lentejas` / `Legumbres` / `120` | guarda, la tabla muestra `Legumbres` | feliz | nuevo |
| A2 | P0 | Insert con el combo en *-- Elegí una categoría --* | *"El nombre es obligatorio y hay que elegir una categoria."* | error | nuevo |
| A3 | P0 | Insert sin nombre, con categoría | mismo mensaje; no guarda | error | nuevo |
| A4 | P1 | Insert con calorías `abc` | *"Las calorias deben ser un numero."* | error | crud-bd #13 |
| A5 | P1 | Insert con calorías `-5` | *"Las calorias no pueden ser negativas."* | error | nuevo |
| A6 | P2 | Insert con calorías `0` | **guarda** (ver **H7**) | borde | nuevo |
| A7 | P1 | Search por id existente / `9999` / `abc` | carga / *"No existe un alimento con id 9999"* / *"El ID debe ser un numero."* | feliz+error | nuevo |
| A8 | P1 | Modify de `Manzana` a `Otros` | persiste al reabrir | feliz | crud-bd #14 |
| A9 | P0 | Delete de `Palta` (está en la dieta 2) | *"No se puede eliminar el alimento: forma parte de una o más dietas."* y **no** borra | error | crud-bd #15 |
| A10 | P0 | Delete de un alimento que **no** está en ninguna dieta | borra; confirmación previa | feliz | crud-bd #16 |
| A11 | P1 | Ordenar por *Category* y clic en una fila | carga el alimento correcto en el combo | regresión | nuevo |
| A12 | P0 | Seleccionar `Yogur descremado` | el combo muestra **`Lácteos`** (con tilde) | regresión | nuevo |
| A13 | P1 | Insertar y borrar seguidos | el combo vuelve al placeholder, no a `Frutas` | borde | nuevo |
| A14 | P2 | Categoría en la base que **no** está en la lista | el combo queda en placeholder; no rompe | borde | nuevo |
| A15 | P2 | Desplegar el combo | exactamente 7 categorías y no se puede tipear | feliz | nuevo |
| A16 | P1 | Cerrar y reabrir | la categoría persiste | feliz | nuevo |

---

## 7. `Diet` y el panel de alimentos

| # | P | Pasos | Resultado esperado | Cubre | Origen |
|---|---|---|---|---|---|
| D1 | P0 | Insert `Vegetariana` sin descripción | guarda, id en el mensaje, tabla con 3 columnas | feliz | diet #1 |
| D2 | P0 | Insert sin nombre | *"El nombre de la dieta es obligatorio."* | error | diet #2 |
| D3 | P1 | Insert con descripción de 3 renglones (con `Enter`) | guarda; ver **H6** | borde | nuevo |
| D4 | P2 | Insert con descripción de 150 caracteres | error de MySQL (`VARCHAR(100)`) → ver **H5** | borde | nuevo |
| D5 | P1 | Search por id existente / `9999` / `abc` | carga y **muestra sus alimentos** / mensaje / mensaje | feliz+error | nuevo |
| D6 | P0 | Modify del nombre y la descripción | persiste; la tabla se actualiza | feliz | diet #4 |
| D7 | **P0** | Delete de una dieta **sin** pacientes, **con** alimentos | se borran sus `DietsDetails` y después la dieta; mensaje *"Dieta eliminada"* | feliz | diet #7 |
| D8 | **P0** | Delete de una dieta **con** un paciente asignado | *"No se puede eliminar la dieta: 1 paciente(s) la tienen asignada."* y **no** borra | error | diet #6 |
| D9 | P1 | Delete → **Cancelar** | no borra | borde | nuevo |
| D10 | P0 | Clic en la fila de una dieta con alimentos | la tabla de abajo muestra sus alimentos; sin dieta seleccionada queda vacía | feliz | nuevo |
| D11 | P0 | Con una dieta seleccionada, elegir un alimento y **Add** | aparece en la tabla de abajo | feliz | diet #9 |
| D12 | P0 | Repetir el mismo alimento y **Add** | *"Ese alimento ya forma parte de la dieta."* | error | diet #10 |
| D13 | P0 | Seleccionar un alimento de la dieta y **Remove** | desaparece de la tabla; el alimento **sigue** en Aliments | feliz | diet #11 |
| D14 | P1 | **Remove** sin fila seleccionada | *"Selecciona un alimento de la dieta primero"* | error | nuevo |
| D15 | P1 | **Add** sin dieta seleccionada | *"Selecciona una dieta de la tabla primero"* | error | nuevo |
| D16 | P1 | Ordenar la tabla de abajo por *Calories* y **Remove** | quita la fila correcta | regresión | diet #13 |
| D17 | P1 | Cambiar de dieta con otra seleccionada | la tabla de abajo se reemplaza, no se acumula | regresión | nuevo |
| D18 | P2 | Base **sin** alimentos | *"No hay alimentos para agregar. Cargalos primero en la pantalla de Aliments."* | error | nuevo |
| D19 | P1 | Borrar los dos alimentos de una dieta y después la dieta | se borra sin huérfanos (ver S3) | borde | nuevo |
| D20 | P1 | Cerrar y reabrir | dietas y sus alimentos siguen | feliz | nuevo |

> [!danger] D7 y D8 son las pruebas que hoy **no pueden pasar**
> `DietController.delete()` empieza llamando `patientDao.buscarPorDieta(id)`, y ese método tiene un JPQL inválido (**H1**). La excepción se lanza *siempre*, incluso cuando no hay ningún paciente, así que **ninguna dieta se puede borrar** y el mensaje que aparece es técnico. Hay que arreglar H1 antes de ejecutar este bloque.

---

## 8. Verificación en la base (S)

No alcanza con que la pantalla muestre lo correcto: hay que mirar la base.

| # | P | Consulta | Esperado |
|---|---|---|---|
| S1 | P0 | `SELECT * FROM Patients WHERE id = <nuevo>;` | una fila con los valores del formulario y `id_diet` correcto (o `NULL`) |
| S2 | P0 | `SELECT * FROM DietsDetails WHERE id_diet = <dieta>;` | una fila por alimento agregado, con `id_aliment` correcto |
| S3 | P0 | Después de D7: `SELECT COUNT(*) FROM DietsDetails WHERE id_diet = <dieta>;` | **0** (sin huérfanos) |
| S4 | P1 | `SELECT * FROM Aliments WHERE id = <borrado>;` | 0 filas |
| S5 | P1 | `SELECT p.* FROM Patients p WHERE p.id_diet = <dieta>;` | los pacientes que impiden D8 |
| S6 | P1 | `INSERT INTO DietsDetails (id_diet, id_aliment) VALUES (1, 1);` dos veces | el segundo tira **1062 Duplicate entry** |

---

## 9. Huecos detectados en la auditoría

Esto es lo que la revisión del código encontró y que **ninguna prueba de las anteriores cubría**.

### H1 — `PatientDAOJPA.buscarPorDieta` tiene el JPQL roto 🔴

```java
// src/appNutritionist/dao/PatientDAOJPA.java:24-27
"SELECT p FROM patients p WHERE p.id_dieta = :idDieta"
```

Dos errores en una línea:

- La entidad se llama **`Patient`**, no `patients`.
- El campo es **`diet`**, no `id_dieta` (`id_diet` es el nombre de la columna en MySQL).

Debería ser:

```java
"SELECT p FROM Patient p WHERE p.diet.id = :idDieta"
```

**Impacto:** `DietController.delete()` (línea 121) lo llama **siempre**, antes de borrar. Resultado: **no se puede eliminar ninguna dieta** y el usuario ve un error de JPQL. Es el hueco más grave de todo el plan.

### H2 — `AlimentDAOJPA.buscarPorCategoria` 🟠

```java
"SELECT a FROM aliments a WHERE a.category = :cat"   // línea 19
```

Igual: `Aliment`, no `aliments`. **Impacto:** ninguno hoy, porque ningún controlador lo llama (el Search es por ID). Rompe el día que se implemente el filtro por categoría.

### H3 — `DietDAOJPA.buscarPorNombre` 🟠

```java
"SELECT d FROM diets d WHERE LOWER(d.name) LIKE :nom"   // línea 19
```

Misma clase de error. Sin uso actual.

### H4 — `PatientDAOJPA.buscarPorApellido` 🟠

```java
"SELECT p FROM patients p WHERE LOWER(p.lastName) LIKE :ap"   // línea 19
```

Misma clase de error. Sin uso actual: el Search es por ID, así que el `Search by Last Name` de la rúbrica **no está implementado**.

### H5 — No hay validación de longitudes 🔴

`Patients.name`/`lastName` son `VARCHAR(80)`, `Patients.observations` es `VARCHAR(500)`, `Aliments.aliment` es `VARCHAR(50)` y `Diets.description` es `VARCHAR(100)`. El controlador valida que no estén vacíos, pero no el largo. Con MySQL en modo estricto, pasarse devuelve un error técnico (`Data too long for column`), no un mensaje pensado para el usuario.

### H6 — La descripción no normaliza los saltos de línea 🟡

`DietController.readForm()` hace `view.getDescriptionText().trim()`. Si el usuario aprieta `Enter` en la textarea, entra un `\n` en un `VARCHAR(100)`. No rompe, pero guarda algo que después se ve raro en cualquier herramienta. El plan original proponía `replaceAll("\\s+", " ")`.

### H7 — Las calorías aceptan `0` 🟡

`parseNonNegative` rechaza negativos pero permite `0`. Para un alimento es discutible; si se decide que no, es una línea.

### H8 — No hay control de duplicados lógicos 🟡

Se pueden cargar dos `Manzana` o dos pacientes idénticos. No hay `UNIQUE` en `aliments.aliment` ni en `patients`. No lo pide la rúbrica, pero conviene saberlo antes de la defensa.

### H9 — El combo de alimentos no filtra los ya agregados 🟡

`loadAliments()` carga todos. Si elegís uno que ya está, el `UNIQUE (id_diet, id_aliment)` lo frena y muestra *"Ese alimento ya forma parte de la dieta."* Funciona, pero obliga a descubrir el error en vez de prevenirlo.

### H10 — `Add` y `Remove` no piden confirmación 🟡

El resto del CRUD confirma antes de modificar. Estos dos no: un clic mal dado quita un alimento. No es un bug, es una inconsistencia.

### H11 — No hay pruebas automatizadas 🟠

No hay JUnit ni un `main` de pruebas; `test/` está vacío. Todo lo de esta nota es manual. Para un TP alcanza, pero significa que **cada cambio se revalida a mano**.

### H12 — Detalles menores 🟢

- `AlimentView.showInfo` dice `"Informacion"` sin tilde.
- `idTextField` es editable en las tres vistas; no afecta porque los controladores usan `selectedX.getId()`, pero confunde.

---

## 10. Criterio de salida

Se considera cubierto cuando:

1. **Todos los P0 en verde**, incluidos D7 y D8 (que hoy fallan por H1).
2. Cada entidad tiene al menos un caso **feliz**, uno de **borde** y uno de **error** por operación.
3. Las verificaciones **S1 a S3** dan lo esperado en la base.
4. H1 está arreglado. H2, H3 y H4 se arreglan en la misma pasada: son la misma línea y el mismo tipo de error (ver [[bug-jpql-dao]]).
5. Los huecos que no se arreglen quedan **documentados** acá, no descubiertos en la defensa.

---

## 11. Orden de ejecución sugerido

| Bloque | Casos | Tiempo |
|---|---|---|
| 1. Arranque y navegación | C1-C6 | 5 min |
| 2. `Patient` | P1-P22 | 15 min |
| 3. `Aliment` | A1-A16 | 12 min |
| 4. `Diet` sin panel | D1-D10 | 10 min |
| 5. `Diet` con panel | D11-D20 | 10 min |
| 6. Base de datos | S1-S6 | 5 min |

Antes del bloque 4: **arreglar H1**, o D7/D8 no se pueden ni intentar.

---

## 12. Registro de resultados

Copiar esta tabla a mano (o a `estado.md`) y completar en la corrida:

| Caso | Resultado | Observación |
|---|---|---|
| P1 | | |
| P2 | | |
| … | | |

> [!tip] Anotá el mensaje **literal** que aparece
> La mitad de los bugs de este proyecto se ven en el texto: tildes faltantes, mensajes que dicen una cosa y el controlador otra, o el `getMessage()` del `DAOException` que devuelve el wrapper y no el motivo SQL.

---

## 13. Lo que dice el profesor

> pág. 1 y 2 — enunciado_rubrica.pdf
> El trabajo pide el CRUD completo de las tres entidades sobre la base de datos, con interfaz gráfica **intuitiva y funcional**, validaciones de datos y manejo de errores.

> pág. 14 y 20 a 23 — Swing_MVC_Interfaces_Java.pdf
> La vista captura y muestra; las reglas y el acceso a datos van en el controlador y el modelo. Cada botón se conecta a una acción del controlador.

**Qué se toma de ahí:** el alcance de las pruebas (las tres entidades, el CRUD completo, las validaciones) y el hecho de que la prueba se hace **desde la GUI**, no llamando métodos. **Qué agrega el equipo:** el catálogo de casos, las prioridades, las verificaciones SQL y los huecos H1 a H12.

---

## 14. Ampliación de la IA

> [!note] Pruebas manuales vs. automatizadas
> Un plan manual es lo correcto para este TP: la rúbrica evalúa la aplicación funcionando. Pero el costo se paga en cada cambio — y este proyecto ya lo mostró: el merge con Romi cambió firmas, excepciones y navegación, y **ninguna prueba automática avisó** que el `delete()` de `Diet` había quedado dependiendo de un JPQL inválido.
>
> Con JUnit 5 alcanzarían tres clases (`PatientDaoTest`, `AlimentDaoTest`, `DietDaoTest`) con una base de prueba, y los H1-H4 se habrían detectado en segundos. Queda como mejora posible, no como requisito.

> [!note] Por qué los errores de JPQL son tan caros acá
> Un JPQL inválido **compila**: es un `String`. El error aparece recién en runtime, cuando esa consulta se ejecuta. Por eso un método roto puede vivir meses sin que nadie lo note, y por eso la prueba D7 (borrar una dieta) es la que más información da: es la única que ejecuta `buscarPorDieta`.

---

## Relacionado

- [[bug-jpql-dao]] — los cinco JPQL rotos, con el detalle de cada uno
- [[plan-crud-bd]] — las 23 pruebas originales de las fases 4 y 5
- [[plan-cableado-diet]] — las 16 pruebas del panel de alimentos
- [[plan-categoria-combo]] — las 10 pruebas del combo de categoría
- [[plan-cableado-patient]] · [[plan-cableado-aliment]] — el cableado de cada entidad
- [[ejemplo-de-excepciones]] — qué excepción lanza cada método y qué mensaje llega al usuario
- [[enunciado_rubrica]] — el alcance que se está verificando
- [[MAP]] · [[index]]
