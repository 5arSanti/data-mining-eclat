## ROL
Eres un científico de datos experto en Python, minería de datos,
procesamiento de texto en español y análisis de reglas de asociación.
Dominas pandas, spacy, mlxtend, matplotlib, seaborn y networkx.

---

## OBJETIVO
Construir un único Jupyter Notebook (.ipynb) profesional, limpio y
ejecutable en entorno local que integre:
1. Carga y preprocesamiento del dataset
2. Normalización avanzada de texto sucio en español
3. Codificación one-hot de productos
4. Algoritmo ECLAT completo
5. Generación de reglas de asociación con métricas
6. Visualizaciones
7. Segmentación por estrato socioeconómico y grupo de edad

---

## CONTEXTO DEL PROYECTO

### Archivos de entrada
- `dataset.xlsx` — Encuesta de hábitos de compra de hogares colombianos
- Sheet: "Respuestas de formulario 1"

### Columnas del dataset (después de renombrar):
| Columna original | Nombre normalizado |
|---|---|
| Marca temporal | fecha |
| EDAD | edad |
| GENERO | genero |
| ESTRATO NSE | estrato_nse |
| CIUDAD DE ORIGEN | ciudad_origen |
| Con quien vive actualmente | quien_vive |
| Numero de personas en el hogar | num_personas |
| Quien realiza las compras en el Hogar | quien_compra |
| Tu misión de compra... | mision_compra |
| Lugar principal de compra | canal_compra |
| Presupuesto Mensual aproximado | gasto |
| Alimentos Básicos | alimentos_basicos |
| Proteinas | proteinas |
| Bebidas | bebidas |
| Snacks y otros | snacks |
| Delikatessen | delikatessen |
| Aseo para el Hogar | aseo_hogar |
| Aseo Personal | aseo_personal |
| ultima | otros |

### Columnas de texto con productos (rows_to_cols):
```python
rows_to_cols = [
    "alimentos_basicos", "proteinas", "bebidas",
    "snacks", "delikatessen", "aseo_hogar",
    "aseo_personal", "otros"
]
```

### Columnas demográficas para segmentación:
- `estrato_nse` — segmentación socioeconómica
- `edad` → derivar `grupo_edad` con bins [0,25,35,50,100]
  labels: ['Joven','Adulto joven','Adulto','Senior']

### Columnas a excluir del análisis de productos:
```python
cols_demograficas = [
    'fecha','edad','genero','estrato_nse','ciudad_origen',
    'quien_vive','num_personas','quien_compra','mision_compra',
    'canal_compra','gasto','grupo_edad'
]
```

---

## ERRORES CRÍTICOS DETECTADOS EN EL CÓDIGO EXISTENTE
Corrígelos obligatoriamente en la nueva implementación:

1. `replace_map` actual tiene mapeos INCORRECTOS:
   - ❌ "bananos" → "abanos"   |  ✅ "bananos" → "banano"
   - ❌ "arroz" → "arros"      |  ✅ eliminar esta entrada
   - ❌ "frijoles" → "2 libras de frijol"  |  ✅ "frijoles" → "frijol"

2. El valor "ninguno" aparece frecuentemente en las celdas y debe
   filtrarse antes del procesamiento. No es un producto.

3. El notebook tiene `clean_and_split()` definida dos veces.
   Conservar solo la versión final (segunda definición).

4. `df.applymap()` está deprecado en pandas >= 2.0.
   Reemplazar con `df.map()` o `df.apply(lambda col: col.map(...))`.

5. `from google.colab import files` → eliminar completamente.
   El entorno es Jupyter local.

6. En ECLAT.ipynb, `get_support()` compara tuplas de forma frágil
   con `.values[0]`. Reimplementar usando un dict para O(1) lookup.

---

## ─── SKILL 1: SETUP Y DEPENDENCIAS ───

Instalar y verificar al inicio del notebook:
```python
# Celda 1 — Instalación (ejecutar solo si es necesario)
# !pip install pandas openpyxl spacy rapidfuzz matplotlib seaborn networkx
# !python -m spacy download es_core_news_sm
```

Imports organizados por categoría:
```python
# Datos
import pandas as pd
import numpy as np
from sklearn.preprocessing import MultiLabelBinarizer

# NLP
import unicodedata, re, spacy
from rapidfuzz import process, fuzz

# Algoritmo
from itertools import combinations

# Visualización
import matplotlib.pyplot as plt
import seaborn as sns
import networkx as nx

# Configuración global
pd.set_option('display.max_columns', None)
nlp = spacy.load("es_core_news_sm", disable=["parser","ner"])
```

---

## ─── SKILL 2: CARGA Y RENOMBRADO ───

- Leer `dataset.xlsx` con `pd.read_excel()`
- Normalizar nombres de columnas: `.str.strip().str.lower()`
- Aplicar el diccionario de renombrado completo (ver tabla anterior)
- Crear `grupo_edad` con `pd.cut()` bins y labels definidos
- Mostrar `.shape`, `.info()` y `.head(3)` al final de la celda

---

## ─── SKILL 3: NORMALIZACIÓN DE TEXTO (CRÍTICA) ───

### Estrategia óptima: Pipeline en 3 capas

**Capa 1 — Limpieza base:**
- Eliminar acentos con `unicodedata.normalize('NFKD')`
- Convertir a minúsculas
- Unificar separadores (\\n, -, ;, espacios múltiples) → coma
- Split por coma → lista de tokens
- Filtrar tokens vacíos y el valor literal "ninguno"

**Capa 2 — Lematización con spaCy:**
- Procesar cada token con el modelo `es_core_news_sm`
- Extraer el lema del primer token del doc
- Esto resuelve automáticamente:
  - cereal/cereales → cereal
  - fruta/frutas → fruta
  - frijol/frijoles → frijol
  - blanqueador/blanqueadores → blanqueador

**Capa 3 — Diccionario curado (replace_map):**
Aplicar DESPUÉS de lematización para casos que spaCy no resuelve
(marcas, abreviaciones, regionalismos):
```python
replace_map = {
    # Correcciones de marca/regionalismos
    "arroz diana": "arroz",
    "suavitel": "suavizante",
    "1200k": None,   # valor inválido → filtrar
    "700k": None,
    # Regionalismos
    "banano": "banano",   # ya normalizado por spaCy
    "papa": "papa",
    # Higiene
    "jabon": "jabon tocador",
    # Agregar más según análisis exploratorio del dataset
}
# Los valores None indican que el item debe eliminarse
```

**Implementación de `normalize_product(token)`:**
```python
def normalize_product(token: str) -> str | None:
    if not token or token == "ninguno":
        return None
    # Lematización
    doc = nlp(token)
    lemma = doc[0].lemma_ if doc else token
    # Diccionario curado
    result = replace_map.get(lemma, lemma)
    return result if result else None
```

**Implementación de `clean_and_split(text)`:**
```python
def clean_and_split(text: str) -> list[str]:
    if pd.isna(text) or not isinstance(text, str):
        return []
    # Capa 1: limpieza base
    text = unicodedata.normalize('NFKD', text)
    text = text.encode('ascii','ignore').decode('utf-8','ignore')
    text = text.lower()
    text = re.sub(r'[\n\r]+', ',', text)
    text = re.sub(r'-\s*', ',', text)
    text = re.sub(r';', ',', text)
    text = re.sub(r'\s{2,}', ',', text)
    text = re.sub(r',+', ',', text)
    tokens = [t.strip() for t in text.split(',') if t.strip()]
    # Capas 2 y 3: normalización
    normalized = [normalize_product(t) for t in tokens]
    cleaned = [n for n in normalized if n]
    return list(set(cleaned))
```

---

## ─── SKILL 4: ONE-HOT ENCODING ───

- Aplicar `clean_and_split` sobre `df[rows_to_cols]`
  usando `.apply(lambda col: col.map(clean_and_split))`
- Combinar items de todas las columnas por fila:
  `combined = processed.apply(lambda x: sum(x.tolist(), []), axis=1)`
- Aplicar `MultiLabelBinarizer` → `df_one_hot`
- Separar base demográfica: `df_base = df.drop(columns=rows_to_cols)`
- Concatenar: `df_final = pd.concat([df_base, df_one_hot], axis=1)`
- Mostrar shape, columnas de productos generadas y primeras 3 filas
- Exportar a `df_procesado.xlsx` para auditoría

---

## ─── SKILL 5: ALGORITMO ECLAT CORE ───

### Preparación de transacciones:
```python
# Extraer solo columnas de productos (excluir demográficas)
cols_productos = [c for c in df_final.columns
                  if c not in cols_demograficas]
productos_df = df_final[cols_productos]

# Construir transacciones
transactions = [
    list(productos_df.columns[row.values == 1])
    for _, row in productos_df.iterrows()
]

# Construir TID lists (estructura vertical ECLAT)
tid_lists: dict[str, set] = {}
for tid, trans in enumerate(transactions):
    for item in trans:
        tid_lists.setdefault(item, set()).add(tid)
```

### Búsqueda de itemsets frecuentes:
```python
MIN_SUPPORT = 0.2
MAX_ITEMSET_LEN = 3
n = len(transactions)

# Precalcular soporte individual (optimización)
support_cache: dict[tuple, float] = {}

frequent_itemsets = []
items = sorted(tid_lists.keys())

for size in range(1, MAX_ITEMSET_LEN + 1):
    for combo in combinations(items, size):
        tids = tid_lists[combo[0]].copy()
        for item in combo[1:]:
            tids &= tid_lists[item]
            if not tids:
                break  # early exit
        support = len(tids) / n
        if support >= MIN_SUPPORT:
            frequent_itemsets.append({
                'itemset': frozenset(combo),
                'support': round(support, 4)
            })
            support_cache[frozenset(combo)] = support

freq_df = pd.DataFrame(frequent_itemsets)
```

**IMPORTANTE:** Usar `frozenset` como clave del `support_cache` para
lookup O(1) al calcular confianza y lift. Evitar búsquedas con
`.loc` o comparaciones de tuplas en DataFrame (ineficiente y frágil).

---

## ─── SKILL 6: REGLAS DE ASOCIACIÓN Y MÉTRICAS ───

Generar reglas a partir de itemsets de tamaño >= 2:

```python
rules = []

for _, row in freq_df[freq_df['itemset'].apply(len) >= 2].iterrows():
    items_list = list(row['itemset'])
    for i in range(len(items_list)):
        ant = frozenset([items_list[i]])
        con = row['itemset'] - ant
        sup_itemset = row['support']
        sup_ant = support_cache.get(ant, 0)
        sup_con = support_cache.get(con, 0)
        if sup_ant > 0 and sup_con > 0:
            confidence = sup_itemset / sup_ant
            lift = confidence / sup_con
            rules.append({
                'antecedent': str(sorted(ant)),
                'consequent': str(sorted(con)),
                'support': sup_itemset,
                'confidence': round(confidence, 4),
                'lift': round(lift, 4),
                'affinity': round(confidence * lift, 4),
                'score': round(
                    0.4 * sup_itemset +
                    0.3 * confidence +
                    0.3 * lift, 4
                )
            })

rules_df = pd.DataFrame(rules).sort_values(
    by='affinity', ascending=False
).reset_index(drop=True)
```

Mostrar top 10 por `affinity` y top 10 por `score`.

---

## ─── SKILL 7: VISUALIZACIONES ───

Implementar las 4 visualizaciones siguientes en celdas separadas:

**Viz 1 — Scatter soporte vs confianza (burbujas = lift):**
```python
plt.figure(figsize=(10, 6))
scatter = plt.scatter(
    rules_df['support'], rules_df['confidence'],
    s=rules_df['lift'] * 100,
    alpha=0.6, c=rules_df['lift'], cmap='RdYlGn'
)
plt.colorbar(scatter, label='Lift')
plt.xlabel('Soporte'), plt.ylabel('Confianza')
plt.title('ECLAT — Reglas de Asociación (tamaño = Lift)')
```

**Viz 2 — Heatmap lift (antecedente vs consecuente):**
- Usar top 15 reglas por affinity
- `pivot_table(index='antecedent', columns='consequent', values='lift')`
- `sns.heatmap(annot=True, fmt='.2f', cmap='YlOrRd')`

**Viz 3 — Red de afinidad (networkx):**
- Nodos = productos
- Aristas = reglas con grosor proporcional al lift
- Usar top 20 reglas por affinity
- Layout: `spring_layout` con `seed=42`

**Viz 4 — Frecuencia de productos (bar chart):**
- `productos_df.sum().sort_values(ascending=False).head(20)`
- Barras horizontales con color gradiente

---

## ─── SKILL 8: SEGMENTACIÓN ECLAT ───

Encapsular el pipeline ECLAT en una función reutilizable:
```python
def run_eclat(data: pd.DataFrame,
              min_support: float = 0.2,
              max_len: int = 3) -> pd.DataFrame:
    """
    Ejecuta ECLAT sobre un subset del DataFrame.
    Retorna DataFrame de itemsets frecuentes con soporte.
    Excluye automáticamente columnas demográficas.
    """
    ...
```

**Segmentación por estrato_nse:**
- Iterar sobre `df_final['estrato_nse'].unique()`
- Para cada estrato: `subset = df_final[df_final['estrato_nse'] == e]`
- Ejecutar `run_eclat(subset)`
- Mostrar top 5 itemsets por soporte
- Generar bar chart comparativo: soporte por producto y estrato

**Segmentación por grupo_edad:**
- Iterar sobre `df_final['grupo_edad'].dropna().unique()`
- Misma lógica que por estrato
- Mostrar tabla comparativa de top productos por grupo etario

**Visualización de segmentación:**
- Un gráfico de barras agrupadas por estrato
- Un gráfico de barras agrupadas por grupo_edad
- Ambos mostrando soporte de los 10 productos más frecuentes

---

## ESTRUCTURA FINAL DEL NOTEBOOK

El notebook debe tener las siguientes secciones (usar Markdown headers):
🛒 Análisis ECLAT — Hábitos de Compra
0. Instalación y Setup
1. Carga de Datos
2. Normalización de Texto
3. Codificación One-Hot
4. Análisis Exploratorio (frecuencias)
5. ECLAT — Dataset Completo
6. Reglas de Asociación
7. Visualizaciones
8. Segmentación por Estrato NSE
9. Segmentación por Grupo de Edad
10. Conclusiones

---

## PARÁMETROS CONFIGURABLES (al inicio del notebook)
```python
# ── Configuración global ──────────────────────────────
DATASET_PATH   = "dataset.xlsx"
MIN_SUPPORT    = 0.2
MAX_ITEMSET    = 3
EDAD_BINS      = [0, 25, 35, 50, 100]
EDAD_LABELS    = ['Joven', 'Adulto joven', 'Adulto', 'Senior']
OUTPUT_PATH    = "df_procesado.xlsx"
# ─────────────────────────────────────────────────────
```

---

## REQUISITOS DE CALIDAD
- Todo el código con type hints donde aplique
- Cada celda con comentario de una línea indicando su propósito
- Sin código duplicado: funciones reutilizables para ECLAT y viz
- Sin imports de Google Colab
- Sin `applymap()` (deprecado): usar `.map()` o `.apply()`
- Manejo de errores en la carga del dataset y del modelo spaCy
- Celdas de validación intermedias: mostrar shape y sample
  después de cada transformación mayor

---

## ENTREGABLE
Un único archivo `eclat_habitos_compra.ipynb` completamente
funcional, ejecutable con:
jupyter notebook eclat_habitos_compra.ipynb
Con celdas Markdown explicativas en cada sección y outputs
visibles en el notebook (no solo en consola).