# Clasificación de Riesgo en Notificaciones RASFF
### Proyecto Final — Procesamiento y Clasificación de Datos (PCD)

---

## 1. Introducción y planteamiento del problema

El sistema **RASFF** (*Rapid Alert System for Food and Feed*) de la
Unión Europea recopila notificaciones cada vez que un país miembro
detecta un producto alimenticio (o de contacto con alimentos) que
representa un riesgo para la salud pública. Cada notificación incluye
información **estructurada** (país que notifica, país de origen,
categoría del producto, tipo de peligro) y **no estructurada**
(una descripción en texto libre del motivo de la alerta, el campo
`subject`).

El objetivo de este proyecto es **predecir el nivel de riesgo
(`risk_decision`)** de una notificación combinando ambos tipos de
datos. Este es precisamente el tipo de problema que estudia la clase
de Procesamiento y Clasificación de Datos: partir de datos crudos,
heterogéneos y con ruido, y llevarlos —mediante un pipeline de
limpieza, transformación y codificación— hasta un formato que un
algoritmo de clasificación pueda aprovechar.

Lo interesante de este dataset es que **no es un problema de
clasificación "clásico"** con solo variables numéricas o categóricas:
mezcla texto libre en lenguaje natural con variables tabulares, lo
que obliga a combinar dos técnicas de procesamiento distintas dentro
del mismo pipeline.

---

## 2. Descripción del dataset

El dataset (`RASFF_window.csv`) contiene una notificación por fila,
con las siguientes columnas relevantes para el modelo:

| Columna | Tipo de dato | Rol en el pipeline |
|---|---|---|
| `subject` | Texto libre (lenguaje natural) | Variable de entrada — se tokeniza |
| `category` | Categórica nominal | Variable de entrada — se codifica |
| `type` | Categórica nominal | Variable de entrada — se codifica |
| `notifying_country` | Categórica nominal | Variable de entrada — se codifica |
| `origin` | Categórica nominal | Variable de entrada — se codifica |
| `hazards` | Categórica nominal (texto semi-estructurado) | Variable de entrada — se codifica |
| `risk_decision` | Categórica nominal | **Variable objetivo (y)** |
| `forAttention`, `forFollowUp`, `operator`, `distribution` | Diversos | Se descartan (alta proporción de nulos / no aportan al problema de clasificación) |

Esta mezcla de tipos de dato es justo lo que hace interesante al
proyecto desde el punto de vista de PCD: hay que decidir, para cada
columna, **qué tipo de transformación le corresponde** según su
naturaleza (texto vs. categórica) y según cómo la va a consumir el
modelo.

---

## 3. Pipeline de procesamiento de datos

El pipeline completo sigue las etapas clásicas de un flujo de
Procesamiento de Datos: **carga → limpieza → transformación →
codificación → partición → modelado → evaluación**. A continuación se
explica cada una, ligándola directamente al código del repositorio
(`src/`).

### 3.1 Carga y limpieza (`src/data_loader.py`)

1. **Manejo de valores nulos en la variable objetivo.**
   No todas las notificaciones tienen un `risk_decision` explícito.
   En vez de eliminar esas filas (lo que reduciría el tamaño del
   dataset), se imputa el valor `"not serious"` como categoría por
   defecto, asumiendo que la ausencia de una decisión de riesgo
   suele corresponder a casos de bajo impacto.

   ```python
   df["risk_level"] = df[TARGET_COLUMN].fillna("not serious")
   ```

2. **Eliminación de columnas irrelevantes o con demasiados nulos**
   (`forAttention`, `forFollowUp`, `operator`, `distribution`).
   Esto reduce la dimensionalidad del problema y evita que el modelo
   intente aprender de columnas que aportan ruido o que en la
   práctica están casi vacías — un criterio de limpieza básico pero
   fundamental en PCD: *no toda columna disponible debe usarse como
   feature*.

3. **Normalización de texto (`strip`).**
   Se eliminan espacios en blanco invisibles al inicio/final de las
   columnas categóricas. Este tipo de "ruido silencioso" es un
   problema clásico en datasets reales: dos valores como
   `"France"` y `"France "` se verían como categorías *distintas*
   para un `LabelEncoder` si no se limpian antes.

4. **Función auxiliar `clean_hazard`.**
   El campo `hazards` en RASFF suele venir en un formato del tipo
   `"Aflatoxin - {mycotoxins}"`, donde el texto entre llaves `{}`
   representa la *categoría general* del peligro. Se incluye una
   función que extrae ese valor simplificado como utilidad de
   limpieza de texto, disponible para quien quiera trabajar con una
   versión más agregada de `hazards` en vez del texto completo.

### 3.2 Codificación de variables categóricas (`src/preprocessing.py`)

Aquí es donde se aplica uno de los temas centrales del curso:
**cómo convertir datos categóricos en una representación numérica que
un modelo pueda procesar.**

Se eligió **Label Encoding** (`sklearn.preprocessing.LabelEncoder`)
para las 5 variables categóricas de entrada (`category`, `type`,
`notifying_country`, `origin`, `hazards`), en lugar de **One-Hot
Encoding**. La razón es principalmente de **cardinalidad**: columnas
como `hazards` o `notifying_country` pueden tener decenas o cientos
de categorías distintas, y un One-Hot Encoding las convertiría en
igual número de columnas binarias, disparando la dimensionalidad del
problema (la llamada *"maldición de la dimensionalidad"*). Con Label
Encoding cada categoría se mapea a un entero, y ese vector de 5
enteros se **escala** después con `StandardScaler` para que el modelo
no interprete los códigos numéricos como si tuvieran una relación de
orden o magnitud real.

```python
encoders[col] = LabelEncoder()
df[col] = encoders[col].fit_transform(df[col].astype(str))
```

**Trade-off importante para discutir en clase:** Label Encoding
introduce una relación ordinal artificial (por ejemplo, si `"China"`
se codifica como 3 y `"France"` como 7, el modelo podría — en teoría
— interpretar que "France" es "mayor" que "China", lo cual no tiene
ningún sentido semántico). Esto se mitiga en este proyecto porque las
variables categóricas no entran solas a un modelo lineal, sino como
parte del vector de entrada de una red neuronal que aprende
combinaciones no lineales; aun así, es una limitación válida a
mencionar en la defensa del proyecto, y un punto de comparación
natural contra alternativas como One-Hot Encoding o *embeddings*
categóricos aprendidos.

La variable objetivo `risk_decision` se codifica por separado con su
propio `LabelEncoder` (`le_risk`), ya que conceptualmente no es una
variable de entrada sino la clase a predecir.

Finalmente, el vector de 5 variables ya codificadas se **escala**
con `StandardScaler` (media 0, desviación estándar 1). Esto es
importante porque las redes neuronales entrenan mejor y más rápido
cuando las variables de entrada están en rangos comparables entre sí
— sin esto, una variable con códigos numéricos grandes podría dominar
el gradiente frente a otra con códigos pequeños, aunque ambas tengan
la misma relevancia real para el problema.

### 3.3 Procesamiento de texto (`src/text_processing.py`)

El campo `subject` requiere un tratamiento distinto al de las
variables tabulares, porque es **texto libre en lenguaje natural**, no
una categoría cerrada. Aquí se aplican dos técnicas típicas de
Procesamiento de Lenguaje Natural (PLN) dentro de PCD:

1. **Tokenización con vocabulario limitado**
   (`Tokenizer(num_words=1000)`): se construye un vocabulario con las
   1000 palabras más frecuentes del corpus de `subject`. Palabras
   fuera de ese vocabulario se descartan. Esto controla el tamaño del
   problema y evita que palabras muy raras (que aparecen una sola
   vez) generen columnas/dimensiones que el modelo no puede aprender
   a usar de forma confiable.

2. **Padding a longitud fija** (`maxlen=20`): las oraciones tienen
   longitudes distintas, pero las redes neuronales requieren tensores
   de tamaño fijo. Se recortan las oraciones más largas de 20 tokens
   y se rellenan con ceros las más cortas. El valor de 20 se eligió
   como un límite razonable para capturar la idea principal de un
   `subject` de RASFF (suelen ser frases cortas y descriptivas), sin
   generar vectores innecesariamente largos y dispersos.

```python
tokenizer = tf.keras.preprocessing.text.Tokenizer(num_words=max_words)
tokenizer.fit_on_texts(df[TEXT_COLUMN].astype(str))
```

### 3.4 Partición de datos

Se reserva un 20% del dataset como conjunto de prueba
(`train_test_split(..., test_size=0.2, random_state=42)`), y dentro
del entrenamiento se separa además un 20% adicional como conjunto de
validación (`validation_split=0.2` en `model.fit`). Esto sigue el
principio de PCD de **nunca evaluar un modelo con los mismos datos
con los que fue entrenado**, para obtener una estimación realista de
qué tan bien generalizará a notificaciones RASFF nuevas.

---

## 4. Modelo de clasificación

### 4.1 ¿Por qué un modelo híbrido y no un clasificador clásico?

Un enfoque "clásico" de PCD (por ejemplo, un `RandomForestClassifier`
sobre las variables categóricas codificadas) ignoraría por completo
la información del texto en `subject`, que en muchos casos contiene
las pistas más directas sobre la gravedad del riesgo (por ejemplo, la
mención explícita de un patógeno peligroso). Por otro lado, un modelo
que solo use el texto ignoraría el contexto tabular (país, categoría
de producto, tipo de peligro), que también correlaciona con el nivel
de riesgo. La arquitectura de este proyecto busca **aprovechar ambas
fuentes de información al mismo tiempo**, en vez de tratarlas como
problemas separados.

### 4.2 Arquitectura

```
Texto (subject)                         Variables tabulares
      │                                  (category, type,
      ▼                                   notifying_country,
 Embedding (64 dim)                       origin, hazards)
      │                                        │
      ▼                                        ▼
 Multi-Head Attention                   StandardScaler
 (auto-atención, 2 heads)                      │
      │                                        │
      ▼                                        │
 GlobalAveragePooling1D                        │
      │                                        │
      └───────────────► Concatenate ◄──────────┘
                              │
                              ▼
                      Dense(32, relu)
                              │
                              ▼
                   Dense(n_clases, softmax)
                              │
                              ▼
                    Predicción de risk_decision
```

**Rama de texto — Embedding + Multi-Head Attention:**
Cada palabra del `subject` se convierte primero en un vector denso de
64 dimensiones (`Embedding`), en lugar de usar directamente el índice
del token — así el modelo puede aprender relaciones semánticas entre
palabras (por ejemplo, que *"salmonella"* y *"listeria"* son más
parecidas entre sí que *"salmonella"* y *"packaging"*).

Después, la capa de **Multi-Head Attention** (en modo
*self-attention*: la secuencia se compara consigo misma) permite que
cada palabra "mire" a las demás palabras de la oración y pondere
cuáles son más relevantes para la predicción final, sin importar su
posición. Esto es justamente lo que en clase se discute como una
alternativa moderna a los enfoques puramente secuenciales (como
RNN/LSTM): la atención captura relaciones entre palabras distantes en
la oración de forma más directa. `GlobalAveragePooling1D` resume esa
secuencia atendida en un solo vector de tamaño fijo, listo para
concatenarse con las variables tabulares.

**Rama tabular:**
Las 5 variables categóricas ya codificadas y escaladas entran
directamente como un vector numérico de 5 posiciones — no requieren
una arquitectura especial, ya que no son secuenciales como el texto.

**Fusión y clasificación:**
Ambos vectores (el resumen de atención del texto y las variables
tabulares) se concatenan en un solo vector, que pasa por una capa
densa de 32 neuronas con activación `ReLU` (introduce no linealidad)
y finalmente por una capa densa con activación `softmax`, cuyo tamaño
es igual al número de clases de `risk_decision`. `softmax` convierte
las salidas en una distribución de probabilidad sobre las clases, y
la clase predicha es la de mayor probabilidad (`argmax`).

### 4.3 Entrenamiento

- **Función de pérdida:** `sparse_categorical_crossentropy` — apropiada
  porque `y` son enteros (clases codificadas), no vectores one-hot.
- **Optimizador:** `adam`, elección estándar por su convergencia
  rápida y estable en la mayoría de problemas de clasificación con
  redes neuronales.
- **Épocas / batch size:** 20 épocas, lotes de 32 ejemplos —
  valores de partida razonables para un dataset de tamaño moderado;
  ver sección 6 para recomendaciones de ajuste.

---

## 5. Evaluación

El código original importa `classification_report` de
`sklearn.metrics` pero no lo utiliza. Para que el proyecto cumpla con
el ciclo completo de PCD (entrenar → **evaluar** → interpretar), se
recomienda agregar, después del entrenamiento en `train.py`:

```python
from sklearn.metrics import classification_report
import numpy as np

y_pred_probs = model.predict([X_text_test, X_meta_test])
y_pred = np.argmax(y_pred_probs, axis=1)

print(classification_report(
    y_test, y_pred, target_names=le_risk.classes_
))
```

Esto entrega, por cada clase de `risk_decision`:

- **Precisión (precision):** de las notificaciones que el modelo
  clasificó como una clase X, ¿qué porcentaje realmente lo era?
- **Sensibilidad (recall):** de todas las notificaciones que
  realmente eran clase X, ¿qué porcentaje detectó el modelo?
- **F1-score:** media armónica entre precisión y sensibilidad —
  especialmente útil aquí porque `risk_decision` probablemente esté
  **desbalanceado** (es de esperarse que haya muchas más
  notificaciones "not serious" que "serious"), y el *accuracy* global
  puede ser engañoso en ese escenario.

Para la defensa del proyecto ante el profesor, conviene explícitamente
reportar la matriz de confusión y discutir en qué clase(s) el modelo
falla más — típicamente será la clase minoritaria de mayor riesgo,
que es además la más importante de detectar correctamente en un
sistema de alerta real (un falso negativo en riesgo "serious" es
mucho más costoso que uno en riesgo "not serious").

---

## 6. Limitaciones y posibles mejoras

Estos son puntos útiles para la sección de "trabajo futuro" o para
anticipar preguntas del profesor:

1. **Label Encoding vs. otras codificaciones.** Como se discutió en
   3.2, Label Encoding introduce una relación ordinal artificial.
   Una mejora sería reemplazarlo por capas `Embedding` categóricas
   (aprendidas junto con el modelo, igual que se hace con el texto),
   lo cual es más consistente con la arquitectura de red neuronal ya
   usada.
2. **Desbalance de clases en `risk_decision`.** Si una clase domina
   el dataset, conviene aplicar técnicas de PCD para manejarlo:
   `class_weight` en `model.fit`, sobremuestreo (oversampling) de la
   clase minoritaria, o submuestreo de la mayoritaria.
3. **Tamaño de vocabulario y longitud de secuencia.** `max_words=1000`
   y `max_len=20` son valores fijos; un análisis exploratorio de la
   distribución real de longitudes de `subject` y de la frecuencia de
   palabras ayudaría a justificar (o ajustar) estos hiperparámetros
   con evidencia del propio dataset.
4. **Modelos de comparación (baseline).** El código original importa
   `RandomForestClassifier`, lo cual sugiere que la intención original
   era comparar el modelo híbrido de atención contra un modelo
   clásico de árboles usando solo las variables tabulares. Incluir
   esa comparación fortalece mucho la defensa del proyecto, porque
   permite argumentar *cuantitativamente* si la complejidad extra del
   modelo de atención realmente se traduce en mejor desempeño.
5. **Validación cruzada.** Actualmente se usa una sola partición
   train/validation/test. Con un dataset de tamaño moderado, k-fold
   cross-validation daría una estimación más robusta del desempeño
   real del modelo.

---

## 7. Conclusión

Este proyecto ilustra, de principio a fin, un pipeline realista de
Procesamiento y Clasificación de Datos sobre un dataset con **datos
mixtos**: texto en lenguaje natural y variables categóricas
tabulares. Cada decisión de preprocesamiento (imputación de nulos,
elección de Label Encoding sobre One-Hot, escalado, tokenización con
vocabulario y longitud limitados) responde a una razón concreta
ligada a las características del dataset, y el modelo resultante
combina dos técnicas de representación (embeddings + atención para
texto, escalado para variables tabulares) en una sola arquitectura de
clasificación. Las secciones 5 y 6 dejan explícito el siguiente paso
natural del proyecto — evaluar rigurosamente el modelo y comparar
contra un baseline — que es lo que normalmente cierra el ciclo de
Procesamiento y Clasificación de Datos: no basta con entrenar un
modelo, hay que poder argumentar, con métricas, qué tan bien resuelve
el problema.
