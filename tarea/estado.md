---
tags: [tarea, estado, auditoria]
fuente: inspección directa del repositorio
aliases: [00-estado]
---

# Estado actual del código

> Auditoría hecha el 2026-10-03 por inspección directa del proyecto.
> **No** es contenido del profesor: es un relevamiento de lo que hay escrito.

## Ubicación

El código **no** vive en este vault. Vive en un proyecto NetBeans aparte:

```text
C:\Users\games\Desktop\Proyectos\NutritionistApp\NutritionistApp\
├── build.xml, manifest.mf, nbproject/    ← proyecto NetBeans (Ant)
├── .gitignore                             ← solo ignora nbproject/private/
└── src/appNutritionist/
    ├── AppNutritionist.java               ← main, cablea MVC de Patient
    ├── Controller/  controllerPatient.java  controllerAliment.java
    ├── Models/      modelPatient.java       modelAliment.java
    └── Views/       viewPatient.java        viewAliment.java
```

- **Java:** `javac.source=25` / `javac.target=25`. JDK instalado: **26**.
  Ojo: compilar a mano **siempre** con `--release 25`, si no los `.class`
  salen versión 70 y NetBeans (JDK 25) no los puede cargar.
- **Entry point:** `appNutritionist.AppNutritionist`
- **Git:** el detalle de commits y el estado del working tree cambian con cada
  tanda de trabajo, así que **no se transcriben acá**. La fuente de verdad es
  `log.md`, y el historial real está en el proyecto con `git log`.

## Inventario

> Actualizado el 2026-10-03, después del commit `7fc3180`.

| Archivo | Líneas | Estado |
|---|---|---|
| `AppNutritionist.java` | 18 | Funciona. Instancia `modelPatient` → `viewPatient` → `controllerPatient` |
| `Models/modelPatient.java` | 5 | **Vacía.** Cero campos |
| `Models/modelAliment.java` | 5 | **Vacía.** Cero campos |
| `Controller/controllerPatient.java` | 30 | Cablea el modelo y la vista. Listener de navegación a `viewAliment` |
| `Controller/controllerAliment.java` | 29 | Cablea el modelo y la vista. Listener de navegación a `viewPatient` |
| `Views/viewPatient.java` | 230 | GUI armada. Todos los componentes privados; `getBtnAliment()` expone el de navegación |
| `Views/viewAliment.java` | 185 | GUI armada. Todos los componentes privados; `getBtnPatient()` expone el de navegación |

**Navegación entre vistas:** funcionando. `Patient` ↔ `Aliment` con `dispose()` y
`SwingUtilities.invokeLater()`. Ver [[vistas-manuales]] para la contingencia de
`setVisible(false)`.

**Diagnóstico:** sigue siendo un **esqueleto con las ventanas cableadas**. La
navegación ya anda, pero no hay ni una entidad de datos ni una regla de negocio.
Los dos modelos siguen vacíos y falta la capa de persistencia.

## Lo que falta, según lo que implican los PDFs

| Capa | Estado | PDF que la governa |
|---|---|---|
| Modelos (entidades) | ❌ Vacío | `POO_Abstraccion.pdf`, `POO_Equals.pdf` |
| Controllers | ⚠️ Cableados, sin lógica de negocio | `Swing_MVC_Interfaces_Java.pdf` |
| Navegación entre vistas | ✅ `Patient` ↔ `Aliment` | `Swing_MVC_Interfaces_Java.pdf` |
| Entidad `Diet` | ❌ No existe | — |
| Persistencia (DAO/JDBC) | ❌ No existe | `JDBC_DAO_MySQL.pdf` |
| Wiring de Swing | ⚠️ Ver nota | `JTable_en_Swing.pdf`, `Listas_y_ComboBox_Swing.pdf`, `Checkbox_y_RadioButton_Swing.pdf` |
| Diagramas | ❌ No hay | `DiagramaClases_Ej2_Biblioteca.pdf`, `Diagrama_Casos_de_Uso_UML.pdf` |

## Riesgos a verificar contra la rúbrica

> [!warning] Estado de esta sección
> El encabezado decía que estos riesgos había que confirmarlos leyendo
> `enunciado_rubrica.pdf`. **Ya se leyó** ([[enunciado_rubrica]], pág. 1 y 2), así
> que lo que sigue está actualizado contra el texto real. El riesgo 1 y el 2
> bajan bastante.

1. **Nombres de clase en minúscula.** `modelPatient`, `viewPatient`,
   `controllerPatient`. La convención de Java pide PascalCase, y el criterio
   *Documentación y Código Limpio* de la rúbrica evalúa "nombres descriptivos" y
   "buenas prácticas de programación". No lo dice de forma explícita, pero es un
   riesgo real de puntaje.
   Además las carpetas son inconsistentes: `Controller` singular, `Models` y
   `Views` plurales.

2. **Las vistas son código generado por el NetBeans GUI Builder.** Se ve en el
   patrón `initComponents()` con ~185 líneas de un solo bloque.
   → **Riesgo bajo.** El criterio *Interfaz Gráfica de Usuario* de la rúbrica
   evalúa exactamente cuatro cosas: que la GUI sea clara, que no tenga errores,
   que sea intuitiva y que esté completa para **todas** las entidades. En ningún
   momento menciona cómo se construyeron las ventanas. El designer no penaliza
   por sí mismo.
   → El riesgo real es otro: el criterio *Documentación y Código Limpio* pide
   comentarios y código propios, y el bloque `initComponents()` no se puede
   documentar. Por eso [[convenciones]] pone los getters fuera de las regiones
   generadas: son las únicas partes de la vista que son código nuestro.
   → **Contingencia ya preparada** en [[vistas-manuales]] por si más adelante
   el profesor o el equipo decide cortar con el designer.

3. **Java 25 con JDK 26 instalado.** Compila, pero si la corrección ocurre en una
   máquina con un JDK anterior, el proyecto no levanta. Conviene fijar la versión
   objetivo a lo que el profesor use.

4. **Sin capa de datos.** La rúbrica exige DAO sobre base SQL para cada entidad
   como requisito, no como extra. Hoy no hay nada.

## Próximo paso

> [!info] La rúbrica ya está procesada
> Este bloque estaba desactualizado: [[enunciado_rubrica]] ya existe completa
> (pág. 1 y 2 leídas). No hay que volver a abrir el PDF para saber qué hay que
> hacer. Lo que queda abierto son las dudas que esa nota no pudo resolver.

**Pendiente de confirmar con el profesor** (están en "Dudas y vacíos" de
[[enunciado_rubrica]]):
- Motor de base de datos: el enunciado dice que es libre. Si el profesor tiene
  preferencia por MySQL, [[JDBC_DAO_MySQL]] es la nota a estudiar.
- Característica extra para aspirar al 10: el enunciado pide "alguna
  característica extra (investigación)", sin especificar cuál.

**Entregas (metodología incremental):**
- ✅ [[entrega-01-interfaces-graficas]] — GUI de `Patient` y `Aliment`.
- ⬜ [[entrega-02-dao-una-entidad]] — DAO de una entidad (JDBC o JPA).

**Trabajo de diseño que sí depende de lo que ya sabemos:**
- Modelar **tres** entidades, no dos. La regla del enunciado es que dos se
  combinan en una tercera: en este dominio la candidata es `Diet`, que relaciona
  paciente y alimentos. Ver la sugerencia en [[enunciado_rubrica]].

**Riesgo a resolver antes de escribir más código:**
- Las vistas son generadas por el NetBeans GUI Builder. La rúbrica **no** las
  penaliza (ver riesgos 2), pero [[vistas-manuales]] queda como contingencia
  si el equipo decide cortar con el designer.
- Los componentes de las vistas ya se exponen por getters según
  [[convenciones]].

## Relacionado

[[convenciones]] · [[enunciado_rubrica]] · [[vistas-manuales]] · [[ME]] · [[MAP]]