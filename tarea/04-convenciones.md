---
fuente: convención propia del proyecto (no proviene de ningún PDF)
tags: [tarea, convenciones, encapsulamiento, mvc, swing]
estado: vigente
---

# 04 — Convenciones del proyecto

> [!warning] Esto NO es una exigencia del profesor
> Revisé `enunciado_rubrica.pdf` completo (pág. 1 y 2) y **no menciona getters,
> encapsulamiento ni nivel de acceso** de los componentes de la vista. Tampoco
> aparece en [[Swing_MVC_Interfaces_Java]].
>
> Lo que sigue es una convención que NOSOTROS adoptamos. El apoyo del profesor
> es indirecto y tenuous, así que queda dicho explícitamente en cada sección en
> lugar de presentarlo como regla de la cátedra.

## La regla

**Los componentes de la vista que el controlador necesita quedan `private` y se
exponen con un getter público. Siempre que sea posible.**

El controlador nunca toca un campo de la vista directamente:

```java
// NO: el controlador depende de un nombre que generó NetBeans
view.alimentBt.addActionListener(e -> { ... });

// SÍ: el contrato es un método con nombre propio
view.getBtnAliment().addActionListener(e -> { ... });
```

### Por qué importa acá

`alimentBt` se llamaba así porque el Estudiante lo renombró en el designer.
`jButton1` es el nombre que genera NetBeans. Si el profesor —o cualquier
integrante del equipo— abre el designer y renombra ese botón, el controlador
deja de **compilar**. El getter convierte el nombre autogenerado en un contrato
estable, y de paso el nombre dice qué es.

## Lo que sí dice la rúbrica

No pide getters, pero hay dos puntos donde esta convención nos favorece:

> pág. 1 — enunciado_rubrica.pdf

- **Javadoc:** "Será necesaria la realización de Javadoc para los métodos y
  clases públicas."

> pág. 1 y 2 — enunciado_rubrica.pdf, criterio *Documentación y Código Limpio*

- "nombres descriptivos", "comentarios explicativos", "buenas prácticas de
  programación".

`jButton1` no es un nombre descriptivo: es exactamente el tipo de nombre que ese
criterio penaliza en la columna de *Insuficiente*. `getBtnAliment()` sí lo es.

> [!info] Ojo con el criterio de *"Documentación y Código Limpio"*
> Puntúa **por entidad**. Si el getter no tiene Javadoc, el criterio queda en
> *Insuficiente* en cada una de las tres entidades. Son 3 getters mal
> documentados que se multiplican.

## Los dos niveles

No todos los getters deberían devolver el componente. Hay dos niveles y la
elección depende de qué hace el controlador con el campo.

### Nivel 1 — el getter devuelve el componente

Alcanza cuando el controlador solo **cuelga listeners**, no toca los valores.

```java
public javax.swing.JButton getBtnAliment() {
    return alimentBt;
}
```

Encapsulamiento parcial: el controlador todavía podría hacer `setText(...)` o
`setEnabled(...)`. Se acepta porque para navegación no hay nada que proteger.

### Nivel 2 — el getter devuelve el dato

Preferido en campos de formulario. El controlador nunca ve el `JTextField`.

```java
public String getNombre() {
    return jTextField1.getText();
}
```

Acá sí hay encapsulamiento real: el componente queda como detalle de
implementación de la vista. Si mañana se cambia el `JTextField` por un
`JComboBox` editable, el controlador no cambia.

Una variante aún mejor, cuando el controlador debería poder observar pero no
manipular:

```java
public void addGuardarListener(Runnable accion) {
    btnGuardar.addActionListener(e -> accion.run());
}
```

### Regla de elección

| El controlador... | Nivel | Devuelve |
|---|---|---|
| solo agrega listeners | 1 | el componente |
| lee o escribe valores | 2 | el dato |
| reacciona a un evento | 2 | un método de suscripción |

## Cuándo **no** aplicar la regla

"Siempre que sea posible" tiene que tener límites, o terminamos con cuarenta
getters que nadie usa.

- **Etiquetas, layouts y decorativos.** Un `JLabel` no lo usa el controlador.
  Getter: no.
- **Nada especulativo.** El botón `Diets` no tiene getter porque no existe la
  entidad `Diet`. Se agrega cuando exista, no antes.
- **Con GUI Builder hay costo.** El `.form` **no puede declarar métodos**, sólo
  campos. El getter se escribe a mano en el `.java` y tiene que quedar fuera de
  las regiones `//GEN-BEGIN` / `//GEN-END`, que es lo único que garantiza que
  NetBeans no lo borre al regenerar. Escribilo después de
  `//GEN-END:variables`, nunca antes.
  Si algún día se aplica la contingencia de [[03-vistas-manuales]] (vistas a
  mano), esto deja de ser un problema.

## Abreviaturas: nombres vs comentarios

> [!warning] Aclaración importante
> La regla de "no abreviar" aplica a los **comentarios**, no a los nombres de los
> identificadores. La razón es la que explicó el Estudiante: un comentario con
> abreviaturas no es de ayuda para quien lo lee.
>
> En los **nombres** de métodos y variables las abreviaturas van bien, y
> `getBtnAliment()` es preferible a `getAlimentButton()`:
> - Es más corto y se lee igual de bien.
> - Agrupa por sufijo: todos los getters de botones quedan juntos en el índice
>   de un editor.
> - `Button` repetido en cada nombre es redundante cuando la clase ya es una
>   vista con muchos `JButton`.

Por eso el código usa `getBtnAliment()` y `getBtnPatient()`, y no los nombres
largos. Esto queda escrito para que no se reabra la discusión cada vez que se
toca una vista.

## Javadoc en los getters

Como los getters son públicos, la rúbrica los Obliga a documentar.

```java
/**
 * Devuelve el botón que lleva al formulario de alimentos.
 *
 * @return el botón de navegación, nunca {@code null}
 */
public javax.swing.JButton getBtnAliment() {
    return alimentBt;
}
```

> [!note] Por qué el constructor no lleva esto
> El constructor no es público ni lo expone el controlador, así que la rúbrica
> no lo pide. Igual aplica: lo público se documenta.

## Dudas y vacíos

- [ ] Si la rúbrica de la cátedra tiene una versión más detallada que este PDF,
      puede exigir getters explícitamente. Cuando aparezca, esta nota se
      reclasifica: la regla pasa a ser del profesor y hay que citarla.
- [ ] Falta decidir si en `03-vistas-manuales` los botones se llaman `btnAliment`
      y `btnPatient`. Si se aplica esa contingencia, los getters pasan a devolver
      esos componentes pero **los nombres de los getters no cambian**.

## Relacionado

- [[00-estado]] — auditoría del código, incluye el inventario de vistas
- [[03-vistas-manuales]] — contingencia de vistas a mano y `setVisible(false)`
- [[enunciado_rubrica]] — el criterio *Documentación y Código Limpio*
- [[Swing_MVC_Interfaces_Java]] — Swing, MVC e interfaces
- [[Codigo_Limpio_Nombres_y_Estructura]] — por qué `jButton1` es un mal nombre

---

**Navegación:** [[index]] · [[MAP]]