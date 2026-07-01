# Optimización de Clasificación de Texto mediante Diseño de Experimentos (DOE)

**Estudiante:** Gema Guerra Valdez  1819110

**Dataset Utilizado:** Capterra Reviews (Análisis de Sentimiento Basado en Aspectos)  

---

## 1. Introducción y Objetivo
El enrutamiento automatizado y la detección de opinión en sistemas de gestión de tickets (*Helpdesk*) representan un componente crítico en la automatización de operaciones de TI. El objetivo de este experimento es aplicar el rigor del método científico mediante un **Diseño de Experimentos Factorial** para evaluar y demostrar estadísticamente qué combinación de preparación de texto (N-gramas) y complejidad del modelo (Número de Árboles) maximiza el rendimiento del algoritmo en la detección de opiniones sobre el módulo **"Automated Ticket Routing"**.

---

## 2. Metodología Experimental

Se implementó un enfoque de **Diseño de Experimentos (DOE)** tratando al pipeline de Machine Learning como un proceso industrial controlable.

### 2.1 Definición de Factores y Niveles
Se seleccionaron dos factores clave con dos niveles cada uno (Alto y Bajo), generando un diseño factorial completo $2^2$ (4 tratamientos únicos):

* **Factor A: Representación del Texto (N-Grams)**
    * *Nivel Bajo (-1):* Unigramas (`ngram_range=(1, 1)`). El modelo analiza palabras aisladas.
    * *Nivel Alto (+1):* Bigramas (`ngram_range=(1, 2)`). El modelo analiza palabras sueltas y pares de términos consecutivos.
* **Factor B: Complejidad del Clasificador (Estimators)**
    * *Nivel Bajo (-1):* 50 árboles de decisión (`n_estimators=50`).
    * *Nivel Alto (+1):* 150 árboles de decisión (`n_estimators=150`).

### 2.2 Variables del Sistema
* **Variable de Entrada ($X$):** Texto libre de la reseña (`overall_text`), preprocesado mediante conversión a minúsculas y eliminación de caracteres especiales.
* **Variable de Respuesta ($Y$):** Rendimiento del modelo medido a través del **Macro F1-Score**. Se seleccionó esta métrica debido al desbalance natural de las etiquetas de aspecto ($-1$: Negativo, $0$: No mencionado, $1$: Positivo).
* **Réplicas Experimentales:** Para asegurar la validez estadística y capturar la variabilidad del proceso, cada tratamiento fue evaluado con **3 réplicas** utilizando Validación Cruzada (`3-Fold Cross Validation`).

---

## 3. Resultados Numéricos (Tabla del DOE)

A continuación se presentan los resultados obtenidos tras la ejecución de la matriz experimental automatizada con `GridSearchCV`:

| Tratamiento | Factor A (N-Grams) | Factor B (Árboles) | F1 Réplica 1 | F1 Réplica 2 | F1 Réplica 3 | F1 Promedio ($Y$) | Error Estándar |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **1** | (1, 1) | 50 | 0.335193 | 0.353962 | 0.336726 | **0.341960** | 0.008510 |
| **2** | (1, 2) | 50 | 0.358921 | 0.369776 | 0.335064 | **0.354587** | 0.014498 |
| **3** | (1, 1) | 150 | 0.342174 | 0.358065 | 0.329395 | **0.343211** | 0.011727 |
| **4** | (1, 2) | 150 | 0.347789 | 0.371777 | 0.332274 | **0.350614** | 0.016250 |

**Configuración Óptima Detectada:** Tratamiento 2 (N-Grams: `(1, 2)` con `n_estimators=50`) con un Macro F1-Score máximo de **0.354587**.

---

## 4. Principales Hallazgos y Análisis Estadístico

### 4.1 Análisis de Efectos Principales

* **Impacto del Factor A (N-gramas):** Es el factor dominante. Al activar los Bigramas `(1, 2)`, el F1-Score promedio pasa de ~0.342 a ~0.352, lo que representa un incremento neto de **+0.01001**. Esto demuestra que en el dominio de soporte técnico, capturar el contexto de palabras contiguas (ej. *"bad routing"*, *"easy setup"*) aporta información estadísticamente crucial para discriminar la opinión del usuario.
* **Impacto del Factor B (Árboles):** Sorprendentemente, incrementar el número de estimadores de 50 a 150 tuvo un impacto marginal neutro o ligeramente negativo. El promedio global del nivel bajo (50 árboles) es **0.34827**, mientras que el del nivel alto (150 árboles) es **0.34691** (una caída de **-0.00136**). 

### 4.2 Efecto de Interacción (A x B)
Al observar la relación cruzada, el incremento de complejidad matemática en el clasificador no se justifica:
* El uso de 150 árboles no mejora la extracción de características simples (Unigramas) y, de hecho, reduce el rendimiento cuando se combina con Bigramas (cayendo de 0.3545 a 0.3506). Esto indica una ligera saturación o sobreajuste (overfitting) en las clases minoritarias del aspecto.

---

## 5. Conclusiones
1.  **Validación del Método:** La incorporación de técnicas de diseño industrial (DOE) al desarrollo de Machine Learning permitió aislar la variabilidad por pliegues y demostrar que aumentar la complejidad del modelo (`n_estimators`) no garantiza un mejor resultado.
2.  **Recomendación de Despliegue:** Para la tarea de clasificación de texto basado en aspectos en Capterra, se recomienda configurar el pipeline utilizando el **Tratamiento 2: Bigramas (1, 2) con 50 árboles**, ya que no solo maximiza la métrica de rendimiento estadístico, sino que optimiza drásticamente el costo computacional y los tiempos de ejecución en producción.
