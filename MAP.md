# MAP — Mapa del Vault

## Estructura de carpetas

```text
Programacion Orientada a Objetos/
├── ME.md                 ← contexto del Estudiante y la materia (leer primero)
├── MAP.md                ← este archivo: índice y mapa
├── SKILLS.md             ← qué sabe hacer el asistente con este material
├── index.md              ← catálogo de temas procesados
├── log.md                ← bitácora de trabajo (append-only)
├── AGENTS.md             ← reglas para assistants
├── Sincronizacion_Notebook.md ← guía de sync con GitHub en la notebook
├── Bienvenido.md         ← nota original de Obsidian (intocable)
│
├── fuentes/
│   └── pdf/              ← PDFs originales, INTOCABLES (21 archivos)
│
├── temas/                ← una nota por PDF procesado (21 notas)
│   ├── Teoria/           ← conceptos POO, SOLID, código limpio y diagramas UML
│   ├── practica/         ← Swing, JDBC y JPA
│   └── tarea/            ← enunciado_rubrica.md
│
├── tarea/                ← trabajo práctico del Estudiante
│   ├── entregas/              ← una nota por entrega (metodología incremental)
│   │   ├── entrega-01-interfaces-graficas.md
│   │   └── entrega-02-dao-una-entidad.md
│   ├── hoja-de-ruta-7-a-9.md  ← checklist 0-G con guía de PDFs para 7-9 y el 10
│   ├── tutorial-dao-desde-cero.md ← guía paso a paso de DAO (JPA y JDBC) para el equipo
│   ├── ejemplo-de-excepciones.md  ← mensajes de DaoException por método y su uso en el controlador
│   ├── plan-cableado-patient.md   ← plan de cableado Vista ↔ Controlador ↔ Modelo de Patient
│   ├── plan-cableado-aliment.md   ← el mismo plan aplicado a Aliment
│   ├── plan-cableado-diet.md      ← el tercer cableado: DietView + DietController + DietTableModel
│   ├── plan-categoria-combo.md    ← la categoría de Aliment como JComboBox cerrado
│   ├── plan-crud-bd.md            ← plan para dejar el CRUD andando contra MySQL
│   ├── plan-de-pruebas-crud.md    ← 70 casos del CRUD de las tres entidades y los huecos H1-H12
│   ├── plan-arreglos-h1-h10.md    ← los diez arreglos: JPQL, validaciones, panel y confirmaciones
│   ├── plan-busquedas-hibridas.md ← búsqueda por texto o por ID según el campo cargado
│   ├── cambios-sobre-codigo-de-romi.md ← qué se tocó del código ajeno y el aviso para Romi
│   ├── plan-javadoc.md            ← documentación de la API pública: reglas, plantillas y 155 comentarios
│   ├── estado.md              ← estado general del código y la tarea
│   ├── vistas-manuales.md     ← contingencia: vistas Swing a mano
│   ├── convenciones.md        ← convenciones de código del proyecto
│   └── bug-jpql-dao.md        ← bug de las consultas JPQL (nombres de tabla en vez de entidades)
│
└── plantillas/
    └── tema.md
```

### Notas citadas que todavía no existen

| Nota | Estado | Donde se la cita |
|---|---|---|
| `00-enunciado` | ⚠️ reemplazada por [[enunciado_rubrica]] | `log.md` (histórico) |

[[enunciado_rubrica]] **sí** existe, pero vive en `temas/tarea/` porque es la
síntesis de un PDF, no una nota de aplicación. Cualquier cita a `00-enunciado`
debería apuntar ahí.

## Las tres capas

El vault separa deliberadamente la **fuente** de la **síntesis** de la **aplicación**:

| Capa | Carpeta | Naturaleza | Se edita |
|---|---|---|---|
| **Fuente** | `fuentes/pdf/` | Los PDFs del profesor, verbatim | Nunca |
| **Síntesis** | `temas/` | Notas que explican lo que dice la fuente | Sí |
| **Aplicación** | `tarea/` | El trabajo práctico del Estudiante | Sí |

Regla de oro: si una afirmación no se puede rastrear hasta `fuentes/pdf/`, no
pertenece a la capa de síntesis sin estar marcada como ampliación de la IA.

## Índice de fuentes

### Bloque 1 — Teoría y conceptos

| # | Archivo | Tema | Nota |
|---|---|---|---|
| 10 | `POO_Abstraccion.pdf` | Abstracción | ✅ [[POO_Abstraccion]] |
| 14 | `POO_Metodos.pdf` | Métodos | ✅ [[POO_Metodos]] |
| 12 | `POO_Herencia.pdf` | Herencia | ✅ [[POO_Herencia]] |
| 13 | `POO_HerenciaMultiple_Interfaces.pdf` | Interfaces y herencia múltiple | ✅ [[POO_HerenciaMultiple_Interfaces]] |
| 15 | `POO_Polimorfismo.pdf` | Polimorfismo | ✅ [[POO_Polimorfismo]] |
| 11 | `POO_Equals.pdf` | `equals`, `hashCode`, identidad | ✅ [[POO_Equals]] |
| 6 | `Errores_y_Excepciones_Java.pdf` | Excepciones | ✅ [[Errores_y_Excepciones_Java]] |
| 16 | `POO_Repaso_y_Diagrama_de_Clases.pdf` | Repaso + diagrama de clases | ✅ [[POO_Repaso_y_Diagrama_de_Clases]] |
| 17 | `SOLID_Principios.pdf` | Principios SOLID | ✅ [[SOLID_Principios]] |
| 2 | `Codigo_Limpio_Nombres_y_Estructura.pdf` | Código limpio, nombres y estructura | ✅ [[Codigo_Limpio_Nombres_y_Estructura]] |
| 3 | `Diagrama_Casos_de_Uso_UML.pdf` | Diagrama de casos de uso | ✅ [[Diagrama_Casos_de_Uso_UML]] |
| 4 | `DiagramaClases_Ej2_Biblioteca.pdf` | Diagrama de clases (ejercicio biblioteca) | ✅ [[DiagramaClases_Ej2_Biblioteca]] |
| 19 | `Diagrama_de_Secuencia_UML.pdf` | Diagrama de secuencia UML | ✅ [[Diagrama_de_Secuencia_UML]] |
| 20 | `Javadoc_en_Java.pdf` | Javadoc y documentación en Java | ✅ [[Javadoc_en_Java]] |

### Bloque 2 — Práctica y herramientas

| # | Archivo | Tema | Nota |
|---|---|---|---|
| 18 | `Swing_MVC_Interfaces_Java.pdf` | Swing, MVC, interfaces | ✅ [[Swing_MVC_Interfaces_Java]] |
| 9 | `Listas_y_ComboBox_Swing.pdf` | Listas y ComboBox en Swing | ✅ [[Listas_y_ComboBox_Swing]] |
| 8 | `JTable_en_Swing.pdf` | JTable | ✅ [[JTable_en_Swing]] |
| 1 | `Checkbox_y_RadioButton_Swing.pdf` | Checkbox y RadioButton | ✅ [[Checkbox_y_RadioButton_Swing]] |
| 7 | `JDBC_DAO_MySQL.pdf` | JDBC, DAO, MySQL | ✅ [[JDBC_DAO_MySQL]] |
| — | `JPA_Hibernate_DAO.pdf` | Mapeo objeto-relacional con JPA, Hibernate y DAO | ✅ [[JPA_Hibernate_DAO]] |

### Tarea

| # | Archivo | Tema | Nota |
|---|---|---|---|
| 5 | `enunciado_rubrica.pdf` | Enunciado y rúbrica del trabajo práctico | ✅ [[enunciado_rubrica]] |

## Orden de procesamiento sugerido

1. **Primero `enunciado_rubrica.pdf`** — define qué hay que hacer y cómo se corrige.
   Todo lo demás se estudia en función de eso.
2. **Luego la teoría y los conceptos** (Bloque 1) — el vocabulario que la tarea exige,
   incluyendo POO, SOLID y código limpio.
3. **Después las herramientas** (Bloque 2) — sólo las que la tarea use de verdad.
4. **Por último los diagramas UML** (`temas/Teoria/`, también Bloque 1) — conviene
   verlos cuando ya se entiende el modelo de la tarea, aunque formen parte de la teoría.

## Convenciones

- Nombres de nota = nombre del PDF sin extensión (ej. `POO_Herencia.md`).
- Toda cita del profesor: `> pág. N — 12-POO_Herencia.pdf`.
- Los diagramas van como imágenes en `temas/_assets/`, embebidas en la nota.
- Frontmatter mínimo: `fuente` y `tags`.

## Navegación

[[ME]] · [[SKILLS]] · [[index]] · `log.md`