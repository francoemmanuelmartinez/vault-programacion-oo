# log — Bitácora

Registro append-only de cada tanda de trabajo. Entradas nuevas arriba.

---

## 2026-10-15 — Correcciones: getter `Diets`, `src.zip` y guía de sync

**Hecho:**
- Eliminado `src.zip` del repo `NutritionistApp` (era un zip manual de `src/`, sin trackear; el repo quedó limpio).
- [[convenciones]]: corregida la afirmación obsoleta «el botón `Diets` no tiene getter». Ahora documenta la **excepción deliberada**: ambas vistas exponen `getBtnDiet()` (commit `8fc5218`) aunque la entidad `Diet` no exista. Nueva subsección «El caso del botón `Diets`» y duda pendiente sobre el nombre (`getBtnDiet` vs campo `dietsBt`/texto «Diets»).
- [[estado]]: callout que aclara que el getter `getBtnDiet()` ya existe en las dos vistas.
- [[Sincronizacion_Notebook]] (Fase D): reescrita. Se aclaró que `community-plugins.json` **no** auto-instala el plugin: hay que commitear también su código (`main.js`, `manifest.json`, `styles.css`). Se documentó excluir `data.json`.
- `.gitignore`: agregado `.obsidian/plugins/*/data.json` (evita filtrar config/token de plugins).
- Instalado el plugin **Obsidian Git** (Vinzent03) en esta PC; su código quedó versionado.

**Fuentes verificadas (no es contenido del profesor):**
- Foro de Obsidian: `community-plugins.json` no reinstala plugins automáticamente al clonar.
- `obsidian-git` (Vinzent03): en escritorio la autenticación la maneja el git del sistema, no `data.json`.

---

## 2026-10-04 — Reestructuración de `tarea/`: esquema por entregas

**Contexto:** el profesor entrega el proyecto de forma **incremental**; cada trabajo agrega funcionalidad hasta la entrega final (rúbrica). El eje de organización de `tarea/` pasa a ser **la entrega**.

**Hecho:**
- Creada la carpeta `tarea/entregas/` con una nota por entrega:
  - [[entrega-01-interfaces-graficas]] (✅ entregada): GUI de `Patient` y `Aliment`.
  - [[entrega-02-dao-una-entidad]] (⬜ pendiente): DAO de una entidad con JDBC o JPA (consigna textual del profesor incluida).
- Renombradas las notas de trabajo (sin número, con `aliases` al nombre viejo y H1 sin prefijo numérico):
  - `00-estado` → [[estado]]
  - `03-vistas-manuales` → [[vistas-manuales]]
  - `04-convenciones` → [[convenciones]]
- Eliminado `02-plan.md` (vacío, mal ubicado en la raíz).
- Reapuntados todos los `[[enlaces]]` a las notas renombradas (13 archivos). `log.md` se dejó intacto (append-only; los enlaces viejos resuelven por `aliases`).
- Limpiadas las referencias muertas a `01-modelo` y `02-plan`.
- Actualizados `index.md` (sección Tarea dividida en Entregas / Notas de trabajo), `ME.md` (tabla de Estado al día) y `MAP.md` (árbol de `tarea/` y tabla de notas inexistentes).

---

## 2026-10-04 — Guía de sincronización en la notebook

**Hecho:**
- Creada [[Sincronizacion_Notebook]] con los pasos para clonar el vault en la notebook (Fase C) y configurar el plugin Obsidian Git en ambas máquinas (Fase D).

---

## 2026-10-04 — Vault versionado y sincronizado con GitHub

**Hecho:**
- Inicializado el vault como repositorio git (rama `main`) y publicado en el repo **privado** `francoemmanuelmartinez/vault-programacion-oo`.
- Creado `.gitignore` del vault excluyendo lo específico de cada máquina: `.obsidian/workspace.json`, `.obsidian/workspace-mobile.json`, `.obsidian/cache/` y `.trash/`.
- Commit inicial `4a2ab7d` (55 archivos) pusheado a `origin/main`.
- Objetivo: copiar el vault a la notebook clonando el repo y mantenerlo sincronizado con el plugin **Obsidian Git**.

---

## 2026-10-03 — Reformulación del orden de procesamiento sugerido

**Hecho:**
- Reformulado el "Orden de procesamiento sugerido" en `MAP.md`: el paso 2 ahora habla de "teoría y conceptos" (incluyendo POO, SOLID y código limpio) y el paso 4 aclara que los diagramas UML viven en `temas/Teoria/` (Bloque 1), coherente con la reorganización de carpetas.

---

## 2026-10-03 — Diagramas UML movidos a `temas/Teoria`

**Hecho:**
- Movidas `Diagrama_Casos_de_Uso_UML.md` y `DiagramaClases_Ej2_Biblioteca.md` de `temas/practica/` a `temas/Teoria/`.
- Actualizados `index.md` y `MAP.md`: ambas notas reubicadas en el Bloque 1 (Teoría y conceptos); comentarios del árbol de carpetas ajustados.
- `temas/practica/` queda sólo con las notas de código (Swing, JDBC y JPA).

---

## 2026-10-03 — Reorganización de carpetas: `temas/POO` → `temas/Teoria`

**Hecho:**
- Renombrada la carpeta `temas/POO/` a `temas/Teoria/`.
- Movidas dos notas de `temas/practica/` a `temas/Teoria/` por ser conceptuales, no de código:
  - [[SOLID_Principios]] y [[Codigo_Limpio_Nombres_y_Estructura]].
- Actualizados `index.md` y `MAP.md`: Bloque 1 renombrado a "Teoría y conceptos" y las dos notas reubicadas en su tabla; árbol de carpetas de `MAP.md` corregido.
- Los `[[links]]` de Obsidian no se tocaron (resuelven por nombre de archivo).

---

## 2026-10-03 — Expansión completa de notas de componentes de Swing (JTable, Listas y Checkboxes)

**Hecho:**
- Ampliadas y enriquecidas las tres notas restantes de componentes gráficos y modelos de Swing en `temas/practica/` para dejarlas al mismo nivel de detalle que JPA y JDBC:
  - [[JTable_en_Swing]]: incorporadas las tres formas de carga (array, `DefaultTableModel`, y `AbstractTableModel` propio), configuración avanzada (`setAutoCreateRowSorter`, ancho de columnas) y el ejemplo MVC completo de calificaciones de alumnos (`Alumno`, `CalificacionesModelo`, `AlumnosTableModel`, `VistaAlumnos`, `CalificacionesControlador`).
  - [[Listas_y_ComboBox_Swing]]: incorporados `JList`, `DefaultListModel`, `JComboBox`, eventos (`ListSelectionListener` vs `ActionListener`), selección múltiple y el ejemplo MVC completo del catálogo por categoría (`CatalogoModelo` con `Map`, `CatalogoVista`, `CatalogoControlador`).
  - [[Checkbox_y_RadioButton_Swing]]: incorporados `JCheckBox`, `JRadioButton`, el rol fundamental de `ButtonGroup` para la exclusividad lógica, `setActionCommand()` y los dos ejemplos MVC completos: el de ingredientes de un pedido (`Ingrediente`, `PedidoModelo`, `PedidoVista`, `PedidoControlador`) y el de métodos de pago (`MetodoPago`, `PagoModelo`, `PagoVista`, `PagoControlador`).

---

## 2026-10-03 — Expansión de notas prácticas con código completo de la cátedra

**Hecho:**
- Ampliadas y enriquecidas dos notas clave del Bloque 2 (`temas/practica/`) para que tengan el mismo nivel de detalle técnico, fragmentos de código completos y ejemplos paso a paso que la nota de JPA:
  - [[JDBC_DAO_MySQL]]: incorporados los bloques de código completos de la entidad `Persona`, la excepción `DaoException`, la interfaz genérica `DAO<T, ID>`, la especialización `PersonaDAO` y la implementación completa `PersonaDAOMySQL` (con `PreparedStatement`, `try-with-resources`, `RETURN_GENERATED_KEYS` y recorrido de `ResultSet`).
  - [[Swing_MVC_Interfaces_Java]]: incorporados los ejemplos completos del ciclo de Swing, el patrón Observer con lambdas, los layout managers y el ejemplo completo del "Saludador MVC" (`SaludadorModelo`, `SaludadorVista`, `SaludadorControlador` con `SwingUtilities.invokeLater`), además de la variante avanzada de la *Action API* (`AbstractAction`, `SaludarAction`).

---

## 2026-10-03 — Incorporación de la fuente 19: JPA_Hibernate_DAO.pdf

**Hecho:**
- Detectado que el PDF `JPA_Hibernate_DAO.pdf` (505 KB) estaba en la carpeta de trabajo del profesor (`Desktop\poo`) pero faltaba en el vault. Los otros 18 eran idénticos byte a byte.
- Copiado a `fuentes/pdf/JPA_Hibernate_DAO.pdf`.
- Creada la nota de síntesis [[JPA_Hibernate_DAO]] en `temas/practica/` (pág. 1 a 24), cubriendo JPA, Jakarta Persistence, Hibernate, configuración con `persistence.xml`, entidades anotadas, operaciones CRUD con `EntityManager`, JPQL y el patrón `PersonaDAOJPA`.
- Actualizados los contadores del vault (19 PDFs, 19 notas):
  - `MAP.md`: agregado al Bloque 2 y actualizado contador de archivos y notas.
  - `ME.md` e `index.md`: actualizado contador a 19.

**Por qué es clave para el proyecto:**
- La rúbrica exige persistencia en base de datos mediante el patrón DAO. Hasta ahora solo contábamos con JDBC puro (`JDBC_DAO_MySQL.pdf`). Este PDF aporta la segunda opción (JPA/Hibernate), que simplifica el mapeo y califica como la característica de investigación avanzada para la Nota 10.

---

## 2026-10-03 — Convención de getters en las vistas

**Hecho:**
- Nueva nota [[04-convenciones]] en `tarea/`. Fija la regla de exponer los
  componentes de la vista por getters y dejar los campos `private`, "siempre que
  sea posible".
- La nota distingue dos niveles, porque no todos los getters deberían devolver el
  componente:
  - **Nivel 1**, devuelve el componente. Alcanza para colgar listeners.
    `getBtnAliment()` → `JButton`.
  - **Nivel 2**, devuelve el dato. Preferido en campos de formulario.
    `getNombre()` → `String`, no el `JTextField`.
  - Regla de elección: si el controlador solo agrega listeners, Nivel 1. Si lee o
    escribe valores, Nivel 2.
- También deja asentado que la regla de no abreviar aplica a los comentarios y no
  a los nombres, que era la duda que había dejado `getBtnAliment()` a medias.

**Hallazgo relevante:**
- La rúbrica **ya estaba procesada** (`temas/tarea/enunciado_rubrica.md`,
  `estado: completo`). `00-estado.md` y el `log.md` la trataban todavía como un
  bloqueante sin leer. Ya no.
- Revisé el texto: **no pide getters, encapsulamiento ni nivel de acceso**. Así
  que la convención se anotó explícitamente como decisión propia del proyecto, no
  como regla de la cátedra. El apoyo real es indirecto:
  - pág. 1 — Javadoc obligatorio en métodos y clases públicas → los getters
    públicos heredan la obligación.
  - pág. 1 y 2 — criterio *Documentación y Código Limpio*: "nombres descriptivos"
    → `jButton1` es justo el tipo de nombre que penaliza en *Insuficiente*.
  - pág. 3 a 7 — `Codigo_Limpio_Nombres_y_Estructura.pdf`: revelar intención.

**Riesgo 2 de `00-estado.md`, reevaluado:**
-_decía_ que las vistas del GUI Builder podían ser un problema si la rúbrica
  penaliza el designer. Ya se leyó: el criterio *Interfaz Gráfica de Usuario*
  evalúa claridad, ausencia de errores, intuición y completitud para todas las
  entidades. **No menciona cómo se construyeron las ventanas.** El riesgo baja a
  bajo. El riesgo real es otro: *Documentación y Código Limpio* pide código
  propio, y `initComponents()` no se puede documentar.

**Limpieza de contenido desactualizado:**
- `00-estado.md`: se sacaron los números de commit fijos y ahora apunta a
  `log.md` como fuente de verdad, para que no vuelva a pudrirse. Se reemplazó el
  "Próximo paso: leer la rúbrica" por las preguntas que siguen abiertas.
- `MAP.md`: el bloque de estructura listaba `tarea/` con `00-enunciado.md`,
  `01-modelo.md` y `02-plan.md`, que no existen. Corregido, con una advertencia
  que aclara que esas dos últimas están pendientes de escribir.
- `index.md`: alta de [[04-convenciones]] y aclaración de que la rúbrica ya no es
  un bloqueante.
- `00-estado.md`: el link de "Relacionado" apuntaba a `[[00-enunciado]]`, que no
  existe. Apunta ahora a `[[enunciado_rubrica]]`, que sí.

**Decisiones:**
- La nota va en `tarea/` (capa "Aplicación") y no en `temas/practica/`. Las notas
  de `temas/` sintetizan al profesor; mezclar ahí una convención propia rompería
  la separación de fuentes que pide `AGENTS.md`.
- No se renombró ningún getter. `getBtnAliment()` y `getBtnPatient()` quedan.

**Pendiente:**
- `01-modelo` y `02-plan` siguen sin escribirse. Los referencia `00-estado.md` e
  `index.md` como `⬜`.

**Fuera de scope:** el getter y el Javadoc de `getBtnAliment()` todavía no están
en el código. La nota dice que hacen falta; el cambio en
`NutritionistApp` quedó pendiente y sin commitear.

---

## 2026-10-03 — Encapsular los botones de navegación detrás de getters

**Hecho:**
- Los botones de navegación dejó de exponerse como campos públicos.
  - `Views/viewPatient.java`: `alimentBt` y `dietsBt` ahora son `private`. Se
    agregó `getBtnAliment()`.
  - `Views/viewAliment.java`: `jButton1` volvió a `private`. Se agregó
    `getBtnPatient()`.
  - `Controller/controllerPatient.java` y `Controller/controllerAliment.java` ahora
    llaman `view.getBtnAliment()` y `view.getBtnPatient()`.
- Se quitó de los tres `.form` el `AuxValue` con
  `JavaCodeGenerator_VariableModifier = 1`, que era lo que forzaba los campos a
  `public`. Sin eso, NetBeans los volvería a generar públicos en la próxima
  regeneración.
- Verificado: `javac --release 25 -Xlint:all` sobre los 7 `.java` → exit 0,
  6 warnings, 0 campos públicos en las vistas, 0 `AuxValue` en los `.form`,
  ambos `.form` con XML válido.

**Por qué (corrección de una decisión propia):**
- El plan original de la transición especificaba getters, pero la implementación los
  reemplazó por campos públicos para replicar el estilo que ya tenía
  `viewPatient`. Fue un error: ataba el controlador a nombres autogenerados por
  NetBeans. Si el designer renombra el componente, el controlador deja de
  compilar.
- Los getters van **fuera** de las regiones `GEN` (`//GEN-BEGIN` /
  `//GEN-END`), que es la única forma de que NetBeans no los borre al regenerar
  el `.java`. El `.form` no tiene forma de declarar métodos, sólo campos.

**Base en la wiki:**
- `> pág. 3 a 7 — Codigo_Limpio_Nombres_y_Estructura.pdf`: revelar intención en
  los nombres. `jButton1` no revela nada.
- `> pág. 4 a 9 — SOLID_Principios.pdf`: una única razón para cambiar. Si el
  controlador toca los campos del designer, un renombre en la vista rompe al
  controlador.

> [!warning] Ampliación de la IA, no del profesor
> Que los widgets de Swing se expongan con getters sale de OOP general. Ninguno de
> los dos PDFs dice eso explícitamente.

**Decisiones:**
- No se agregó `getBtnDiets()`. Sería código sin usar: el botón `Diets` no tiene
  listener porque no existe la entidad `Diet`. Se suma cuando exista.
- Se mantiene el nombre `getBtnAliment()` del plan original y de
  `03-vistas-manuales.md`. **Corrección posterior:** había anotado que
  `getAlimentButton()` sería mejor por no abreviar "button". El Estudiante
  aclaró que la regla de no abreviar aplica a los **comentarios**, no a los
  nombres de identificadores, así que `getBtnAliment()` queda como está.
  Queda asentado en [[04-convenciones]].

**Pendiente:**
- Commit de este cambio: **no hecho**, esperando aprobación del Estudiante.

---

## 2026-10-03 — Navegación entre vistas Patient ↔ Aliment

**Hecho:**
- Implementada la transición recíproca entre las dos ventanas usando los botones
  de la parte superior. Cada controlador cierra la vista actual con `dispose()` y
  crea modelo, vista y controlador del destino dentro de
  `SwingUtilities.invokeLater`.
  - `Controller/controllerPatient.java` → listener sobre `view.alimentBt`.
  - `Controller/controllerAliment.java` → listener sobre `view.jButton1`.
- `Views/viewAliment.form`: se agregó
  `JavaCodeGenerator_VariableModifier = 1` al componente `jButton1` para que el
  campo quede `public` **y siga así** cuando NetBeans regenere el `.java`.
- Limpieza de código muerto: los dos `main()` y los dos `logger` de las vistas,
  el handler vacío `jButton3ActionPerformed` (y su bloque `<Events>` en el
  `.form`), dos imports de `ActionEvent` sin uso, `setLabel` deprecado → `setText`,
  `setToolTipText("")` inútil, y líneas en blanco sueltas.
- Commit `7fc3180` — "Code cleanup and navigation between Patient and Aliment
  buttons". 10 archivos, +46 / −90. El repo queda con 2 commits y el working
  tree limpio.

**Base en la wiki:**
- `> pág. 20 a 23 — Swing_MVC_Interfaces_Java.pdf`: la vista solo compone la
  interfaz; el controlador registra los listeners y coordina las acciones.
- `> pág. 28 — Swing_MVC_Interfaces_Java.pdf`: la interfaz debe crearse en el
  hilo de eventos con `SwingUtilities.invokeLater(() -> ...)`.
- `> pág. 2 — Codigo_Limpio_Nombres_y_Estructura.pdf`: "Dejá el código un poco
  mejor de como lo encontraste."

**Aprendizaje sobre el build (importante):**
- Compilar a mano con el **JDK 26 sin `--release 25`** produce `.class` versión
  70. NetBeans corre con JDK 25, que sólo reconoce hasta la 69, y Ant no
  recompila porque los `.java` son más viejos que los `.class`. Resultado:
  `UnsupportedClassVersionError` con un mensaje engañoso de "no se ha encontrado
  el método principal".
- siempre con `javac --release 25`, y sobre los **7 `.java`**, no sólo
  `AppNutritionist.java`: `controllerAliment` y `modelAliment` todavía no los
  referencia nadie y quedan sin verificar si se compila sólo desde el entry point.
- Comando de verificación usado:
  `javac --release 25 -Xlint:all -d build/classes <los 7 .java>` → exit 0,
  6 warnings (2 `[serial]` + 4 `[this-escape]`, inherentes a heredar de `JFrame`).

**Otro error propio (anotado para no repetirlo):**
- Al reescribir archivos enteros se perdieron los saltos de línea finales y
  quedaron con finales LF. Git avisaba con `LF will be replaced by CRLF`.
  Corregido a CRLF. Verificado que ningún archivo quede sin salto final.

**Decisiones:**
- El botón `Diets` queda **sin listener** en las dos vistas: no existe entidad
  `Diet` todavía, y no se va a inventar código para una vista que no está
  especificada.
- Estrategia de transición: `dispose()` y nueva instancia. La alternativa de
  `setVisible(false)` quedó documentada como contingencia en `03-vistas-manuales`.

**Pendiente:**
- Botón `Diets` inerte en las dos vistas. Falta la entidad `Diet` completa.
- Las ventanas destino no llevan `setLocationRelativeTo(null)`, así que pueden
  aparecer pegadas en el borde superior izquierdo. Es comportamiento de NetBeans,
  no introducido por este cambio.
- El campo `model` de los dos controladores se asigna pero no se lee: es el
  contrato del constructor que la cátedra teach, no se tocó.
- Sigue bloqueante leer `enunciado_rubrica.pdf`.

---

## 2026-10-03 — Creación de la LLM Wiki completa (18 PDFs procesados)

**Hecho:**
- Procesados los **18 PDFs** oficiales de la cátedra de Programación Orientada a Objetos (Prof. Esteban Santini, ISFT 204).
- Creadas todas las notas de síntesis en `temas/POO/` (Bloque 1: Abstracción, Métodos, Herencia, Herencia Múltiple e Interfaces, Polimorfismo, Equals, Excepciones, Repaso y Diagrama de Clases).
- Creadas todas las notas de síntesis en `temas/practica/` (Bloque 2: Principios SOLID, Código Limpio, Swing MVC, Listas y ComboBox, JTable, Checkbox y RadioButton, JDBC & DAO, Casos de Uso UML, Diagrama de Clases - Biblioteca).
- Actualizados `MAP.md` e `index.md` con el estado completo de la wiki (18 notas creadas).

**Pendiente:**
- Esperar instrucciones del Estudiante para avanzar con la capa de aplicación (`tarea/`).

---

## 2026-10-03 — Procesamiento de enunciado_rubrica.pdf

**Hecho:**
- Procesado `enunciado_rubrica.pdf` y creada la nota de síntesis `temas/tarea/enunciado_rubrica.md`.
- Extraídos los requerimientos para la nota 10: 3 entidades (dos combinadas en una tercera), capas MVC, DAO con SQL, Swing GUI, excepciones, Javadoc y diagramas UML (Casos de Uso y Secuencia).
- Actualizados `MAP.md` e `index.md`.

**Pendiente:**
- Diseñar el modelo de clases (`01-modelo.md`) incorporando la tercera entidad (ej. `Diet`) para cumplir con la combinación de entidades.
- Plan de implementación (`02-plan.md`).

---

## 2026-10-03 — Contingencia de vistas Swing

**Decisiones del Estudiante:**
- El código **se queda** en `Desktop\Proyectos\NutritionistApp`, fuera del vault.
- Sobre el GUI Builder: probablemente el profesor lo permite, pero se prepara
  el replacement por las dudas.

**Hecho:**
- Analizadas las dos vistas generadas y extraídos los campos reales que
  escondían detrás de `jTextField1..8`:
  - `Patient`: id, name, lastName, weight, height, imc, observations.
  - `Aliment`: name, category, caloriesPerServing.
- Escritas y **verificadas** las versiones manuales en `tarea/03-vistas-manuales.md`.
  Compilan con `javac --release 25 -Xlint:all -Werror`, exit code 0.
- Cambios respecto a las generadas: `GridBagLayout` en vez de `GroupLayout`,
  nombres con significado (`txtName`, `btnInsert`), clases en PascalCase,
  `private final` agrupadas, IMC en `setEditable(false)`.

**Bug encontrado al compilar (importante):**
- El getter `getHeight()` **no compila**: colisiona con
  `java.awt.Component.getHeight()`, que devuelve `int` (píxeles).
  Renombrado a `getHeightMeters()`, y `getWeightKg()` por simetría.

**Hallazgos nuevos que hay que verificar contra la rúbrica:**
- Hay botón `Diets` en las dos vistas, pero **no existe** `viewDiet`,
  `modelDiet` ni `controllerDiet`. Si la tarea pide dietas, falta una entidad
  entera.
- **No hay ningún `JTable` en el proyecto**, pero existe `JTable_en_Swing.pdf`.
  Casi seguro la tarea pide listar pacientes y alimentos en tabla.

**Pendiente:**
- Leer `enunciado_rubrica.pdf`. Sigue siendo lo bloqueante.

---

## 2026-10-03 — Descubrimiento del código de la tarea

**Hecho:**
- El Estudiante tiene código de la tarea en
  `C:\Users\games\Desktop\Proyectos\NutritionistApp\NutritionistApp`
  (proyecto NetBeans/Ant, Java 25, git con 1 commit).
- Auditoría completa registrada en `tarea/00-estado.md`.

**Diagnóstico:** es un **esqueleto**. Hay `main` y dos ventanas, pero
`modelPatient`, `modelAliment`, `controllerPatient` y `controllerAliment` están
vacíos. No existe capa de persistencia.

**Riesgos marcados para verificar contra la rúbrica:**
1. Nombres de clase en minúscula (`modelPatient`, `viewPatient`).
2. Las vistas son generadas por el NetBeans GUI Builder (`initComponents()`),
   y los PDFs de Swing enseñan armado manual.
3. `javac.target=25` con JDK 26 instalado.
4. Sin capa JDBC/DAO, pese a que existe `JDBC_DAO_MySQL.pdf`.

**Decisiones:**
- El código **permanece fuera del vault**, en su carpeta de proyecto con su
  propio git. El vault lo documenta y lo referencia por ruta.
- `00-estado.md` es la nota puente entre el wiki y el proyecto de código.

**Pendiente:**
- Leer `enunciado_rubrica.pdf`: es lo bloqueante. Define campos de las
  entidades, si hay relación entre `Patient` y `Aliment`, y si la rúbrica
  acepta el GUI Builder o exige Swing manual.

---

## 2026-10-03 — Estructura inicial del vault

**Hecho:**
- Creado el vault en `G:\Apps\Obsidian\vaults\Programacion Orientada a Objetos`.
- Copiados los **18 PDFs** del profesor desde `C:\Users\games\Desktop\poo`
  a `fuentes/pdf/`.
- Creadas las carpetas `temas/POO`, `temas/practica`, `temas/tarea`, `tarea/`,
  `plantillas/`.
- Creados los archivos núcleo:
  - `ME.md` — contexto del Estudiante y la materia.
  - `MAP.md` — mapa del vault e índice de fuentes.
  - `SKILLS.md` — capacidades del asistente.
  - `index.md` — catálogo de temas.
  - `AGENTS.md` — reglas para assistants.
  - `plantillas/tema.md` — template de notas.
- Configurada la carpeta de plantillas de Obsidian (`Templates`) en
  `plantillas/`.
- Configurado `opencode` con `google/gemini-flash-latest` como modelo por
  defecto, porque el modelo anterior no acepta PDFs adjuntos.

**Decisiones:**
- Orden de procesamiento: primero `enunciado_rubrica.pdf`, después los
  conceptos, después las herramientas.
- Separación estricta entre lo que dice el profesor y la ampliación de la IA.
- `Fuentes/pdf/` es intocable: los originales no se editan ni se renombran.

**Pendiente:**
- Procesar los 18 PDFs en `temas/`.
- Analizar el enunciado y la rúbrica de la tarea.
- Incorporar los videos del instituto (requiere transcripción previa).

---

## Plantilla de entrada

```markdown
## YYYY-MM-DD — Título corto

**Hecho:**
- ...

**Decisiones:**
- ...

**Pendiente:**
- ...
```