# SKILLS — Qué puedo hacer con este material

Capacidades disponibles para el asistente que trabaja en este vault.

## 1. Ingesta (procesar un PDF)

**Comando natural**: "procesá `POO_Herencia.pdf`" o "procesá el bloque 1".

Pasos:
1. Leo el PDF completo (soporte nativo, incluye diagramas).
2. Extraigo el contenido **separando** lo que dice el profesor de lo que agrega la IA.
3. Reconstruyo los diagramas como imágenes si son relevantes.
4. Creo la nota en `temas/` usando `plantillas/tema.md`.
5. Registro en `index.md`, `MAP.md` y `log.md`.

Restricción: **nunca invento paginación**. Si no la puedo determinar, va `pág. ?`.

## 2. Consulta (responder sobre la materia)

**Comandos naturales**: "¿qué dice el profe sobre herencia?", "dame el ejemplo de
polimorfismo", "¿cómo se usa JTable?"

- Respondo **con cita a la página** cuando la info viene de un PDF.
- Si la respuesta es conocimiento general mío, lo marco como ampliación.
- Si hay discrepancia entre lo que dice el profesor y lo que sé, **el profesor gana**.

## 3. Auditoría de fidelidad

**Comando natural**: "auditá las notas contra los PDFs".

Verifico que:
- Toda afirmación atribuida al profesor tenga cita real.
- Los números de página correspondan al PDF correcto.
- No haya contenido de IA presentado como del profesor.

## 4. Trabajo práctico

**Comandos naturales**: "armame el modelo de la tarea", "derivame el diagrama
de clases", "revisá mi código contra lo que pidió el profe".

- Traduzco el enunciado a un modelo de clases.
- Derivo el diagrama de clases siguiendo el estilo de
  `DiagramaClases_Ej2_Biblioteca.pdf`.
- Contrasto el código del Estudiante contra los criterios de la rúbrica.

## 5. Mantenimiento del vault

- Actualizo `index.md`, `MAP.md` y `log.md` cuando agrego material.
- Mantengo la separación fuente / síntesis / aplicación.

## Límites conocidos

- **Videos**: OpenCode no adjunta `.mp4` al modelo. Para procesar videos
  del instituto hay que transcribirlos primero (fuera de esta herramienta) y
  guardar la transcripción como Markdown.
- No puedo verificar nada contra un PDF que no esté en `fuentes/pdf/`.