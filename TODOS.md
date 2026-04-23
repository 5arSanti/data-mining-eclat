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
- [ ] Ejecutar notebook completo y validar outputs en entorno local
- [ ] Ajustar `replace_map` con hallazgos exploratorios adicionales

## Bucle de autocorrección y aprendizaje
1. Ejecutar celda por celda y registrar errores.
2. Corregir función/celda con falla manteniendo el contrato del README.
3. Re-ejecutar desde la sección afectada.
4. Documentar ajuste en este archivo.
