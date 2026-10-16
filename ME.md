# ME — Contexto del Estudiante y de la Materia

> Leé esto primero. Es la identidad del proyecto: qué es esta wiki, quién la
> usa y de dónde sale la información.

## Qué es este vault

Una **wiki de estudio** de la materia *Programación Orientada a Objetos*, construida
a partir del material oficial de la cátedra (PDFs del profesor) y, más adelante,
de los videos de clase.

El objetivo no es aprender "Java genérico", sino **poder desarrollar la tarea de la
materia usando los conceptos, las convenciones y el criterio de evaluación del
profesor**. El profesor **Esteban Santini** define el contrato.

## El Estudiante

- Cursa **ISFT 204 — Desarrollo de Sistemas Orientado a Objetos**.
- Período actual: **2026**.
- La tarea es una app de **nutricionistas** (`NutritionistApp`), con entidades
  `Patient` y `Aliment`, en Java + Swing + MVC.
- El código ya está empezado en un proyecto NetBeans, en
  `C:\Users\games\Desktop\Proyectos\NutritionistApp\NutritionistApp`
  (**fuera del vault**). Hoy es un esqueleto: models y controllers vacíos.
  Ver [[estado]] para el detalle.
- Necesita el material en un formato navegable y consultable desde Obsidian,
  no PDFs sueltos.

## El Profesor

**Esteban Santini** — es la fuente de verdad del proyecto.

- Todo lo que se atribuya a la cátedra debe poder citarse con página.
- Cuando algo sea una agregación de la IA y no del profesor, tiene que estar
  claramente separado.

## Fuentes

- **PDFs**: 21 archivos en `fuentes/pdf/`. Son la base actual de la wiki.
- **Videos**: material del instituto pendiente de incorporar (ver [[SKILLS]] para
  el procedimiento).

## Estado

| Bloque | Estado |
|---|---|
| PDFs copiados al vault (21) | Hecho |
| PDFs procesados en `temas/` (21 notas) | Hecho |
| Estructura y navegación del vault | Hecho |
| `enunciado_rubrica.pdf` analizado | Hecho → [[enunciado_rubrica]] |
| Auditoría del código existente | Hecho → [[estado]] |
| Limpieza de código muerto + navegación `Patient` ↔ `Aliment` | Hecho → commit `7fc3180` |
| Encapsulamiento con getters | Hecho → commit `8fc5218` |
| Vault versionado en git + GitHub | Hecho → [[Sincronizacion_Notebook]] |
| **Entrega 1**: GUI de dos entidades | Hecho → [[entrega-01-interfaces-graficas]] |
| **Entrega 2**: DAO de una entidad | Pendiente → [[entrega-02-dao-una-entidad]] |
| Modelo de clases / entidad `Diet` | Pendiente |
| Entrega final (3 entidades + rúbrica) | Pendiente |
| Videos procesados | Pendiente |

Ver `log.md` para el detalle de cada tanda de trabajo.

## Navegación

- [[MAP]] — índice de fuentes y mapa del vault
- [[SKILLS]] — qué sabe hacer el asistente con este material
- [[index]] — catálogo de temas
- [[estado]] — auditoría del código que ya existe
- `AGENTS.md` — reglas para assistants