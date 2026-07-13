# Clasificación de enfermedades en plantas mediante redes neuronales convolucionales y aprendizaje por transferencia

**Estudiante:** Gema Guerra Valdez  
**Matrícula:** 1819110  

---

## 1. Introducción
La identificación temprana y precisa de patologías en cultivos agrícolas es un factor crítico para la seguridad alimentaria mundial y la mitigación de pérdidas económicas en el sector agropecuario. El presente trabajo académico aborda el desarrollo, optimización y evaluación de un sistema de visión computacional basado en Redes Neuronales Convolucionales (CNN) para clasificar enfermedades foliares en plantas, utilizando como base el catálogo de imágenes *PlantVillage*. 

A través de un enfoque incremental de Diseño de Experimentos (DOE), se contrastó una arquitectura convolucional construida desde cero (*Baseline*) contra técnicas avanzadas de Inteligencia Artificial: Aprendizaje por Transferencia (*Transfer Learning*) y Ajuste Fino (*Fine-Tuning*) utilizando la arquitectura de red móvil **MobileNetV2**.

---

## 2. Descripción del Dataset y Pipeline de Datos
El directorio analizado corresponde a una subcarpeta del catálogo completo que expone múltiples estados patológicos y saludables en cultivos como pimientos (`Pepper__bell`), papas (`Potato`) y tomates (`Tomato`).

*   **Volumen de Datos:** Se detectó un universo total de **20,638 archivos** distribuidos de manera categórica.
*   **Espacio de Clases:** El pipeline identificó de manera robusta **15 clases reales** (ej. *Pepper__bell___Bacterial_spot*, *Potato___healthy*, *Tomato_Leaf_Mold*, *Tomato__Tomato_YellowLeaf__Curl_Virus*, *Tomato_Bacterial_spot*, entre otras).
*   **Estrategia de Partición (Hold-Out Validation):**
    *   **Conjunto de Entrenamiento:** 80% de los datos disponibles, equivalente a **16,511 imágenes**.
    *   **Conjunto de Validación:** 20% de los datos restantes, equivalente a **4,127 imágenes**.
*   **Optimización del Flujo:** El pipeline se configuró utilizando la API `tf.data`, procesando imágenes reescaladas a una dimensión homogénea de **128 × 128 × 3 píxeles** con un tamaño de lote (*Batch Size*) de **64**. Se aplicaron optimizaciones de rendimiento mediante la persistencia en memoria (`.cache()`), barajado dinámico (`.shuffle(1000)`) y precarga en paralelo (`.prefetch(buffer_size=AUTOTUNE)`).

---

## 3. Desarrollo de Experimentos y Arquitecturas

### Experimento 1: CNN Secuencial desde Cero (Baseline)
Se diseñó un modelo secuencial compacto con tres bloques convolucionales alternados con capas de reducción espacial (`MaxPooling2D`). Para mitigar cuellos de botella computacionales y el sobreajuste masivo característico de las capas totalmente conectadas tras imágenes de alta dimensionalidad, se sustituyó la capa tradicional de aplanado (`Flatten`) por una capa de **GlobalAveragePooling2D**.

*   **Total de Parámetros:** 61,455 (Todos entrenables).
*   **Resultados destacados (Época 10):** Alcanzó un **83.77% de accuracy en validación** y una pérdida de **0.4906**. Las curvas de aprendizaje demuestran un entrenamiento estable y sin divergencias tempranas.

### Experimento 2: Transfer Learning (MobileNetV2 Base)
Se importaron los pesos preentrenados del modelo **MobileNetV2** obtenidos del enorme banco de imágenes *ImageNet*. Se procedió al congelamiento total de las capas convolucionales base (`base_model.trainable = False`), funcionando estrictamente como un extractor de características geométricas, texturales y cromáticas abstractas. Al final de la arquitectura, se acopló una capa de regularización `Dropout(0.2)` y una capa densa de salida con activación Softmax ajustada a las 15 clases del problema.

*   **Total de Parámetros:** 2,277,199 (Parámetros entrenables: 19,215).
*   **Resultados destacados (Época 8):** Elevó significativamente el desempeño del sistema, consolidando un **92.25% de accuracy en validación** y reduciendo la pérdida a **0.2346**.

### Experimento 3: Fine-Tuning (Ajuste Fino Avanzado)
Partiendo del modelo de Transfer Learning consolidado, se descongelaron las últimas **20 capas convolucionales** de MobileNetV2. Para evitar la destrucción del conocimiento previo (gradientes explosivos) y ajustar los filtros terminales de la red hacia las microestructuras particulares de las manchas en las hojas, el modelo se recompiló utilizando el optimizador Adam con una tasa de aprendizaje reducida e hipersensible ($\alpha = 10^{-4}$).

*   **Resultados destacados (Época 5):** El modelo convergió de manera óptima hacia un excelente **93.68% de accuracy en validación**, con una pérdida remanente mínima de **0.1905**, mientras que la precisión en el set de entrenamiento rozó la perfección estadística (**99.99%**).

---

## 4. Tabla Comparativa de Resultados (DOE)

La siguiente matriz resume la evaluación del Diseño de Experimentos ejecutado a lo largo del pipeline de ciencia de datos:

| Enfoque Arquitectural | Parámetros Entrenables | Épocas | Loss (Train) | Accuracy (Train) | Loss (Val) | Accuracy (Val) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **CNN Secuencial (Baseline)** | 61,455 | 10 | 0.5453 | 81.41% | 0.4906 | 83.77% |
| **Transfer Learning (MobileNetV2)** | 19,215 | 8 | 0.2157 | 92.98% | 0.2346 | 92.25% |
| **Fine-Tuning (Últimas 20 Capas)** | ~115,000 | 5 | 0.0060 | 99.99% | **0.1905** | **93.68%** |

---

## 5. Análisis de Resultados y Conclusiones
1. **Validación de Estrategia:** La CNN secuencial propia demostró ser un gran punto de partida, logrando superar el 83% de precisión general. Sin embargo, las texturas complejas de las necrosis foliares y manchas bacterianas requerían filtros más sofisticados.
2. **El Impacto de MobileNetV2:** Al introducir el Aprendizaje por Transferencia se logró un salto cualitativo de casi un 9% extra de precisión con una fracción de esfuerzo computacional en entrenamiento (solo 19,215 parámetros densos optimizados por época).
3. **Convergencia del Fine-Tuning:** El ajuste de las últimas capas convolucionales permitió especializar el modelo en el dominio botánico del problema. La precisión final del **93.68%** valida el sistema como un clasificador altamente confiable y listo para ser desplegado en arquitecturas embebidas o aplicaciones móviles de monitoreo agrícola en tiempo real.
