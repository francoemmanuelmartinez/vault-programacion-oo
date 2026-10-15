# AGENTS.md — Reglas para assistants en este vault

## Contexto

Este vault contiene una wiki de la materia **Programación Orientada a Objetos (Java)**.
Todo el material de estudio proviene de PDFs del profesor y de videos de clase.
El objetivo es que el Estudiante pueda estudiar y después desarrollar una tarea
usando **los conceptos y convenciones de la cátedra**, no versiones genéricas.

## Reglas que anulan las globales ⚠️

El usuario tiene reglas globales en `C:\Users\games\.config\opencode\agents.md`
escritas para otro proyecto (**FacturadorC**, C++/CLI). Ese archivo se carga en
**todas** las sesiones, así que acá hay que anularlo explícitamente.

**IGNORAR por completo en este vault:**

- El comando de build de MSBuild / Visual Studio.
- `ControladoresImpl.h`, `ref classes`, `Convert::ToInt32`, `safe_cast`.
- La regla "sin lambdas en ref classes".
- Cualquier cosa de C#, WinForms, SQL Server o bases de datos de ese proyecto.

**La regla "sin tildes en textos UI" NO aplica acá.** Este vault está escrito en
español rioplatense **con tildes y Ñ**. Respetá la ortografía normal.

## Español

- Respondé siempre en **español rioplatense** (voseo: "vos", "podés", "fijate").
- Terminología técnica en inglés solo cuando sea el estándar (`interface`,
  `inheritance`, `static`, `null`).
- Los ejemplos de código Java van en bloques ```java.

## Fidelidad a la fuente — regla más importante 🔴

Este proyecto NO es "Java en general". Es "Java según este profesor".

1. **Nunca inventes contenido del profesor.** Si un tema no está en los PDFs,
   no lo escribas como si el profesor lo hubiera dicho.
2. **Todo dato del profesor se cita**:
   `> pág. N — 03-IfElse.pdf`
   Si el PDF no tiene paginación legible, usar `> sin paginación — archivo.pdf`.
3. **Separá siempre las dos fuentes de información** en cada nota:

   ```markdown
   ## Lo que dice el profesor
   [con cita de página]

   ## Ampliación de la IA
   [conocimiento general, claramente marcado como no atribuido a la cátedra]
   ```

4. **Cuando el contenido de la IA contradiga al profesor**, gana el profesor.
   Anotá la discrepancia explícitamente.
5. **No inventes números de página.** Si no estás seguro de la página, poné
   `pág. ?` y avisale al Estudiante para que lo verifique.

## Cómo trabajar

- Presentá un plan y esperá aprobación antes de modificar archivos o crear
  muchas notas de una vez.
- Al agregar PDFs nuevos a `fuentes/pdf/`, actualizá `MAP.md`, `index.md` y `log.md`.
- Usá la plantilla de `plantillas/tema.md` para cada PDF procesado.
- Registrá los cambios en `log.md` (una entrada por tanda de trabajo).
- No borres ni sobrescribas `Bienvenido.md`.

## Estructura

- `fuentes/pdf/` — PDFs originales. **Intocables.** No editar, no renombrar.
- `temas/` — notas por PDF, una por archivo.
- `tarea/` — trabajo práctico del Estudiante.
- `plantillas/` — templates de Obsidian.
- `ME.md` — contexto del Estudiante y de la materia.
- `MAP.md` — índice de fuentes y mapa del vault.
- `SKILLS.md` — qué sabe hacer el asistente con este material.
- `index.md` — catálogo de temas.
- `log.md` — bitácora.

## Estudiante

Estudiante de **ISFT 204 "Desarrollo de Sistemas Orientado a Objetos"**.
Profesor: **Esteban Santini**. Está cursando la materia y/developando una tarea.
Ver `ME.md` para los detalles.