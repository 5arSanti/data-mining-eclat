# TODOs de implementación ECLAT

## Estado general
- [x] Leer y seguir el plan de implementación de `README.md`
- [x] Crear notebook final `eclat_habitos_compra.ipynb`
- [x] Estructurar secciones 0..10 en Markdown
- [x] Definir configuración global parametrizable
- [x] Implementar carga y renombrado de columnas
- [x] Crear `grupo_edad` con bins y labels requeridos
- [x] Implementar normalización de texto en 3 capas
- [x] Corregir errores críticos de `replace_map`
- [x] Filtrar `ninguno` en el pipeline
- [x] Evitar `applymap()` y usar `.map()`/`.apply()`
- [x] Implementar one-hot encoding y exportar `df_procesado.xlsx`
- [x] Implementar ECLAT core con soporte vertical y `frozenset`
- [x] Implementar reglas con métricas (`confidence`, `lift`, `affinity`, `score`)
- [x] Agregar 4 visualizaciones requeridas
- [x] Agregar segmentación por `estrato_nse`
- [x] Agregar segmentación por `grupo_edad`
- [x] Crear skill de autocontexto para futuras ejecuciones
- [x] Auditar fases de `scripts/eclat.ipynb` para transferencia de buenas prácticas
- [x] Reestructurar `eclat_habitos_compra.ipynb` en formato presentación
- [x] Incorporar métricas de soporte en exploratorio y distribución de métricas de reglas
- [x] Alinear secciones de segmentación y conclusiones a storytelling de negocio
- [x] Refactor fase 5+ separando definición/ejecución en celdas independientes
- [x] Añadir descripciones académicas en Markdown para bloques de modelado y segmentación
- [x] Verificar explícitamente criterios 1..6 solicitados en sección de conclusiones
- [x] Documentar todas las celdas del notebook con redacción explicativa
- [x] Eliminar viñetas de celdas Markdown para estilo narrativo continuo
- [ ] Ejecutar notebook completo y validar outputs en entorno local
- [ ] Ajustar `replace_map` con hallazgos exploratorios adicionales

## Bucle de autocorrección y aprendizaje
1. Ejecutar celda por celda y registrar errores.
2. Corregir función/celda con falla manteniendo el contrato del README.
3. Re-ejecutar desde la sección afectada.
4. Documentar ajuste en este archivo.
