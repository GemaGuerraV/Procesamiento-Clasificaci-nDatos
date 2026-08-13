# Clasificación de Audios Ambientales (ESC-50) utilizando Transformada Wavelet (DWT)

**Universidad Autónoma de Nuevo León**  
**Maestría en Ciencia de Datos**  
**Asignatura:** Procesamiento y Clasificación de Datos  

---

## 1. Introducción

El procesamiento de señales acústicas no estacionarias presenta retos significativos al utilizar transformadas tradicionales como la Transformada Rápida de Fourier (FFT), la cual pierde la localización temporal de los eventos de alta frecuencia. En este proyecto, se implementa un pipeline de procesamiento y clasificación de audios ambientales perteneciente al dataset **ESC-50** (*Environmental Sound Classification*) utilizando la **Transformada de Ondícula Discreta (DWT)** para el análisis multirresolución (MRA) y un modelo de aprendizaje automático (*Random Forest*).

El objetivo principal es evaluar la capacidad de representación de las ondículas Daubechies (`db1`) al extraer características espectro-temporales para clasificar eventos acústicos complejos en cinco categorías representativas:
- `dog` (ladrido de perro)
- `chainsaw` (motosierra)
- `rooster` (canto de gallo)
- `rain` (lluvia)
- `sea_waves` (olas del mar)

---

## 2. Metodología

El flujo de trabajo se dividió en cuatro etapas principales:

```text
[Audios ESC-50 (.wav)] ──> [Preprocesamiento (22.05 kHz)] ──> [Descomposición DWT (db1, Nivel 5)]
                                                                          │
[Reporte PDF & Markdown] <── [Evaluación & Matriz Confusión] <── [Random Forest Classifier]
```
### 2.1. Preprocesamiento de la Señal
- **Fuente de Datos:** Subconjunto de audios monocanal del dataset **ESC-50**.
- **Frecuencia de Muestreo ($f_s$):** Re-muestreo uniforme a $22,050\text{ Hz}$.
- **Duración:** Segmentación/normalización a $5.0\text{ segundos}$ por muestra.

### 2.2. Extracción de Características con Wavelets
Para cada señal temporal $y(t)$, se aplicó la descomposición discreta de ondículas a $5$ niveles de resolución:
$$\text{Coeffs} = \text{DWT}(y(t), \text{wavelet}=\text{'db1'}, \text{level}=5)$$

De cada vector de coeficientes de aproximación y detalle ($A_5, D_5, D_4, D_3, D_2, D_1$), se calcularon $4$ métricas estadísticas clave:
1. **Media ($\mu$):** Tendencia central del nivel de descomposición.
2. **Desviación Estándar ($\sigma$):** Dispersión de las oscilaciones.
3. **Varianza ($\sigma^2$):** Potencia de la señal en la banda de frecuencia.
4. **Energía ($\sum c^2$):** Energía total contenida en la escala espectral.

Esto generó un vector consolidado de $24$ características por muestra de audio ($6\text{ sub-bandas} \times 4\text{ métricas}$).

### 2.3. Entrenamiento del Modelo
- **Clasificador:** *Random Forest Classifier* ($100$ árboles de decisión).
- **Partición:** $75\%$ entrenamiento, $25\%$ prueba, con estratificación por clase.

---

## 3. Resultados y Hallazgos

### 3.1. Rendimiento del Clasificador
El modelo alcanzó una exactitud global (**Accuracy**) del **90.0%** en el conjunto de validación, demostrando que la representación mediante coeficientes DWT discrimina eficazmente tanto señales continuas como impulsos acústicos breves.

| Clase | Precision | Recall | F1-Score | Support |
| :--- | :---: | :---: | :---: | :---: |
| **chainsaw** | 0.90 | 0.90 | 0.90 | 10 |
| **dog** | 0.82 | 0.90 | 0.86 | 10 |
| **rain** | 1.00 | 0.90 | 0.95 | 10 |
| **rooster** | 0.90 | 0.90 | 0.90 | 10 |
| **sea_waves** | 0.90 | 0.90 | 0.90 | 10 |
| **Promedio Global** | **0.90** | **0.90** | **0.90** | **50** |

---

## 4. Experiencias y Conclusiones

1. **Eficiencia Espectro-Temporal:** La Transformada Wavelet Discreta (DWT) demuestra ser ampliamente superior a la STFT tradicional para señales no estacionarias, ya que no sufre del compromiso fijo entre resolución temporal y frecuencial.
2. **Compresión de Información:** Con tan solo $24$ características estadísticas derivadas de los coeficientes de la ondícula `db1`, el modelo Random Forest logró separar las $5$ clases con una precisión promedio del $90\%$.
3. **Escalabilidad:** El tiempo de extracción de características con `pywt` es sumamente rápido, lo que permite su ejecución ligera sin requerir cómputo intensivo en GPU.

---
