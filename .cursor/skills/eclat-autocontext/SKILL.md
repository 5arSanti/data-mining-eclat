---
name: eclat-autocontext
description: Ejecuta un pipeline ECLAT end-to-end en notebooks con texto sucio en español, reglas de asociación y segmentación demográfica. Use when the user mentions eclat, hábitos de compra, dataset.xlsx, normalización de productos, reglas de asociación, estrato_nse o grupo_edad.
---

# ECLAT Autocontext

## Objetivo
Construir y mantener un notebook único, limpio y ejecutable para análisis ECLAT de hábitos de compra.

## Flujo obligatorio
1. Leer `README.md` y extraer requisitos, errores críticos y entregables.
2. Mantener un archivo de progreso en `TODOS.md` con checkboxes.
3. Aplicar normalización en 3 capas:
   - Limpieza base (acentos, separadores, minúsculas, split)
   - Lematización con `es_core_news_sm`
   - `replace_map` curado posterior a lematización
4. Filtrar `ninguno` y cualquier token inválido (`None`) antes del one-hot.
5. Evitar `df.applymap()`; usar `.map()` o `.apply(lambda col: col.map(...))`.
6. Ejecutar ECLAT con estructura vertical (TID lists) y `support_cache` con `frozenset`.
7. Generar reglas con `support`, `confidence`, `lift`, `affinity`, `score`.
8. Incluir visualizaciones: scatter, heatmap, red de afinidad y barras de frecuencia.
9. Segmentar por `estrato_nse` y por `grupo_edad`.
10. Cerrar con conclusiones accionables.

## Reglas críticas
- No usar imports de Google Colab.
- No duplicar funciones (especialmente `clean_and_split`).
- Corregir mapeos erróneos:
  - `"bananos" -> "banano"`
  - `"frijoles" -> "frijol"`
  - no mapear `"arroz" -> "arros"`
- Usar type hints en funciones clave.
- Mostrar validación intermedia (`shape`, `head`, conteos) tras transformaciones mayores.

## Salidas requeridas
- Notebook principal: `eclat_habitos_compra.ipynb`
- Dataset procesado: `df_procesado.xlsx`
- Seguimiento: `TODOS.md`

## Bucle de autocorrección
1. Ejecutar notebook completo.
2. Si falla una celda, aislar causa y corregir en función reutilizable.
3. Re-ejecutar desde la sección afectada.
4. Actualizar `TODOS.md` con ajuste y estado.
