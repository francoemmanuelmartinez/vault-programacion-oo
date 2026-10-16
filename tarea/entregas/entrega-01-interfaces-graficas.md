---
fuente: consigna de la entrega (resumida por el Estudiante; no hay PDF en el vault)
tags: [tarea, entrega, gui, swing, mvc]
estado: entregada
---

# Entrega 1 — Interfaces gráficas (GUI) de dos entidades

> [!info] Metodología de la cátedra
> El profesor pide entregar el proyecto de forma **incremental**: cada trabajo
> agrega funcionalidad y el proyecto va creciendo hasta la **entrega final**, que
> debe cumplir con la [[enunciado_rubrica]].

## Consigna

> Hacer la **interfaz gráfica** de dos entidades del proyecto.
>
> *(El texto exacto del profesor no está en el vault; esto es el resumen de lo
> pedido. Completar el wording si aparece la consigna original.)*

## Entregables

- Pantalla (vista) para la entidad `Patient`.
- Pantalla (vista) para la entidad `Aliment`.
- Controlador de cada vista (cableado MVC).

## Estado: ✅ Entregada

**En el proyecto `NutritionistApp`:**
- `Views/viewPatient.java` — GUI con campos (name, lastName, weight, high, imc,
  observations, id), botones CRUD (Insert/Modify/Delete/Search) y navegación.
- `Views/viewAliment.java` — GUI con campos (aliment, category, caloriesPerServing),
  botones CRUD y navegación.
- `Controller/controllerPatient.java` / `controllerAliment.java` — cablean modelo y
  vista y resuelven la navegación `Patient` ↔ `Aliment`.

**Pendiente de esta entrega:** los botones CRUD están en la UI pero **sin lógica**
(los modelos están vacíos). Eso se completa en entregas posteriores.

## Mapeo a la rúbrica

| Criterio | Aporte de esta entrega |
|---|---|
| Interfaz Gráfica de Usuario (GUI) | Pantallas de las dos entidades |
| Arquitectura en Capas y MVC | Vista + Controlador por entidad (Modelo aún vacío) |

## Relacionado

- [[entrega-02-dao-una-entidad]] — siguiente etapa
- [[estado]] — estado general del proyecto
- [[enunciado_rubrica]] — criterios de evaluación
