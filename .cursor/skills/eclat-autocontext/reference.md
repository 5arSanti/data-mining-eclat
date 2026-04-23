# Referencia rápida del pipeline

## Configuración
- `MIN_SUPPORT = 0.2`
- `MAX_ITEMSET = 3`
- `EDAD_BINS = [0, 25, 35, 50, 100]`
- `EDAD_LABELS = ['Joven', 'Adulto joven', 'Adulto', 'Senior']`

## Funciones clave
- `normalize_product(token)` -> lematiza y aplica `replace_map`
- `clean_and_split(text)` -> limpia y tokeniza
- `run_eclat(data, min_support, max_len)` -> itemsets y cache de soporte
- `generate_rules(freq_df, support_cache)` -> reglas y métricas

## Validaciones
- Revisar `shape` tras carga, one-hot y base final.
- Confirmar que no exista `applymap`.
- Confirmar que `ninguno` no aparezca en columnas one-hot.
