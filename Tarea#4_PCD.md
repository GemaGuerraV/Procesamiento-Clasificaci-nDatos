# Tarea #4: Detección de Centros en Objetos Circulares
**Materia:** Procesamiento y Clasificación de Datos

---

## 1. Introducción
El objetivo de esta práctica es desarrollar un pipeline de visión computacional para la identificación de estructuras circulares y sus respectivos centros de masa en imágenes del mundo real (en este caso, una colección de frutas cítricas y bayas). La detección se basa en la **Transformada de Hough para Círculos**, precedida por etapas esenciales de análisis frecuencial y filtrado espacial para mitigar el ruido de la textura natural.
Para consultar el notebook da click [AQUÍ](https://github.com/GemaGuerraV/Procesamiento-Clasificaci-nDatos/blob/main/Tarea4_PCD.ipynb)

---

## 2. Análisis del Perfil Colorimétrico (Histogramas)

Para comprender la distribución lumínica y de color antes de la detección, se analizaron los histogramas tanto por canales RGB independientes como en la escala de intensidad combinada.

### A. Histogramas por Canal de Color (RGB)
Al descomponer la imagen original en sus canales Rojo (R), Verde (G) y Azul (B), observamos comportamientos altamente contrastantes debido a la naturaleza del objetivo (frutas rojas, naranjas, amarillas y verdes):

<img width="1489" height="490" alt="image" src="https://github.com/user-attachments/assets/22cb56a1-f2d6-4d91-9b36-edfc504a3382" />

* **Canal Rojo:** Presenta una altísima concentración de píxeles en valores cercanos a $0$ (zonas oscuras o carentes de componente roja pura, como las sombras entre las frutas), pero con una dispersión uniforme a lo largo del rango medio-alto que representa la abundancia de fresas, naranjas y toronjas.
* **Canal Verde:** Es el canal más balanceado y con distribución más suave. Muestra picos prominentes en la zona de altas luces (cercanas a 230), lo cual se debe a la presencia de kiwis, limones y las manzanas verdes brillantes.
* **Canal Azul:** Exhibe un pico masivo e inclinado hacia el extremo superior de saturación ($255$). Esto refleja que gran parte de los blancos brillantes de la iluminación reflejada sobre la superficie cerosa de las frutas saturan este canal, mientras que las sombras bajas caen a cero.

### B. Efecto del Filtro Espacial de Suavizado (Escala de Grises)
Al convertir la imagen a escala de grises y aplicar un filtro de suavizado, comparamos el **Histograma Original** frente al **Histograma con Filtro**:

<img width="643" height="260" alt="image" src="https://github.com/user-attachments/assets/bcd9d7b3-4764-4197-8542-e4555a3d8d0a" />

* **Histograma Original (Gris):** Muestra una forma multimodal con picos pronunciados (por ejemplo, cerca de los valores $80$, $170$ y $210$). Las "puntas" o valles muy rápidos representan variaciones de alta frecuencia debidas a la textura rugosa de la piel de las naranjas y las semillas de las fresas (ruido de alta frecuencia).
* **Histograma con Filtro (Suavizado):** Tras aplicar el desenfoque, el histograma conserva la macroestructura (los picos principales), pero las oscilaciones internas de la curva se reducen notoriamente. El perfil se vuelve más continuo, lo que demuestra matemáticamente que los píxeles aislados y ruidosos han sido promediados con su vecindario.

---

## 3. Segmentación de Características: Detector de Bordes Canny

El mapa de bordes generado mediante el algoritmo de Canny revela la complejidad del escenario físico:

<img width="515" height="379" alt="image" src="https://github.com/user-attachments/assets/c8b7472c-46e9-46b5-a85b-e471b106a11a" />

* **Éxito:** Las secciones transversales de las naranjas cortadas y los contornos de los limones son capturados con líneas continuas bien definidas.
* **Problema de textura:** Debido a que las frutas no son esferas perfectas ni lisas, Canny detectó una enorme cantidad de micro-bordes internos (textura de la pulpa expuesta, contornos de las semillas de fresa y las hojas). Esta sobre-segmentación de bordes es la causa directa del comportamiento en la siguiente etapa.

---

## 4. Evaluación de la Transformada de Hough y Diagnóstico

### Diagnóstico del Resultado Actual
En nuestra primera ejecución, la imagen de **"Centros Detectados"** experimenta una **saturación por falsos positivos**. El algoritmo está intentando trazar círculos en prácticamente cada acumulación de micro-bordes detectados por Canny (como los interiores de las rodajas de toronja y los racimos de uvas).

<img width="515" height="195" alt="image" src="https://github.com/user-attachments/assets/d5988249-21d0-4188-85ba-8f36eb0f55f5" />

### Propuesta de Optimización del Código
Para solucionar esta sobre-detección en la función `cv2.HoughCircles()`, se propone ajustar los siguientes hiperparámetros:

1. **`param2` (Umbral del acumulador):** Debe incrementarse considerablemente (por ejemplo, de `30` a `50` o `70`). Cuanto más alto sea este valor, más perfecto y seguro debe ser el círculo para ser tomado en cuenta.
2. **`minDist`:** Aumentar la distancia mínima entre los centros de los círculos para evitar que se dibujen decenas de círculos empalmados sobre una misma fruta.
3. **`minRadius` y `maxRadius`:** Acotar mejor el rango de píxeles basándonos en el tamaño real de las frutas en la imagen para ignorar las bayas pequeñas si solo se buscan las frutas grandes (o viceversa).
