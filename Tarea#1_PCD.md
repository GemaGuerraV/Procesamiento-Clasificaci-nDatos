# Análisis Estadístico y Estilométrico Comparativo en la Obra de Edgar Allan Poe

Este repositorio contiene un estudio cuantitativo basado en Procesamiento de Lenguaje Natural (PLN) y estilometría computacional sobre dos de las obras más emblemáticas del autor estadounidense Edgar Allan Poe: el cuento gótico **"The Masque of the Red Death" (1842)** y el poema lírico **"The Raven" (1845)**.

---

## 1. Estadística Descriptiva Básica

A partir del procesamiento de los textos planos, se obtuvieron las siguientes métricas globales de centralización y diversidad:

| Métrica | *The Masque of the Red Death* | *The Raven* |
| :--- | :---: | :---: |
| **Total de Caracteres** | 13,742 | 9,587 |
| **Total de Palabras** | 2,370 | 1,429 |
| **Vocabulario Único** | 824 | 431 |
| **Total de Oraciones** | 101 | 66 |
| **Riqueza Léxica (TTR)** | **0.3477** | **0.3016** |
| **Longitud Promedio de Palabra** | 4.48 | 4.48 |
| **Longitud Promedio de Oración** | **23.47** | **21.65** |

### Análisis de los Datos:
* **Consistencia del Idiolecto:** La *Longitud Promedio de la Palabra* es exactamente idéntica en ambos textos (**4.48 caracteres**). Esto es un indicador estilométrico clásico del "sello" o huella digital del autor; a pesar de cambiar de género literario, su preferencia subconsciente por la complejidad morfológica de las palabras se mantiene estática.
* **Diversidad Léxica (TTR):** El cuento (*The Masque*) presenta una riqueza léxica superior (0.3477) en comparación con el poema (0.3016). Esto se explica analíticamente porque la poesía lírica, y específicamente *"The Raven"*, depende de la repetición deliberada de estribillos (*refrains*) y rimas internas para construir su atmósfera musical (por ejemplo, la constante repetición de *"Nevermore"*).
* **Complejidad Sintáctica:** La *Longitud Promedio de la Oración* es más elevada en la prosa (23.47 palabras) que en el poema (21.65 palabras), lo cual evidencia el estilo barroco y descriptivo de la narrativa de Poe, caracterizada por cláusulas subordinadas complejas.

---

## 2. Distribución de Frecuencias y N-gramas

El análisis de frecuencias aislando las *stopwords* (palabras vacías) permite mapear los núcleos temáticos y los patrones de co-ocurrencia sintáctica de cada obra.

### Top 5 Bigramas Más Frecuentes
* **"The Masque of the Red Death":**
  1. `('red', 'death')` — Frecuencia: 6
  2. `('prince', 'prospero')` — Frecuencia: 6
  3. `('ebony', 'clock')` — Frecuencia: 3
  4. `('fifth', 'sixth')` — Frecuencia: 2
  5. `('chamber', 'purple')` — Frecuencia: 2

* **"The Raven":**
  1. `('chamber', 'door')` — Frecuencia: 13
  2. `('maiden', 'angels')` — Frecuencia: 5
  3. `('angels', 'name')` — Frecuencia: 5
  4. `('name', 'lenore')` — Frecuencia: 5
  5. `('quoth', 'raven')` — Frecuencia: 5

### Histograma Comparativo de Frecuencias Monogramas
El comportamiento de las palabras individuales más frecuentes resalta la fijación temática de Poe en ambos entornos. Mientras que el cuento se ancla en los símbolos espaciales y personajes (`prince`, `clock`, `death`, `chamber`), el poema concentra su peso en los umbrales transicionales (`door`, `chamber`) y la pérdida mítica (`lenore`, `nevermore`, `raven`).

<img width="1489" height="590" alt="image" src="https://github.com/user-attachments/assets/56c06a52-6253-4ad2-b461-444ba11d6383" />


### Nube de Palabras Combinada
La visualización unificada mediante el *WordCloud* expone los conceptos transversales a la estética gótica del autor: el espacio claustrofóbico (`chamber`, `window`, `door`, `room`), la presencia metafórica (`bird`, `clock`, `soul`) y la fatalidad dramática (`horror`, `velvet`, `red death`).

<img width="790" height="447" alt="image" src="https://github.com/user-attachments/assets/ed9344c2-8225-4146-9cb8-f89e93642cdf" />


---

## 3. Uso de Signos de Puntuación (Estilo Expresivo)

La puntuación determina el ritmo psicológico del texto. El análisis cuantitativo arrojó una divergencia drástica en las herramientas expresivas empleadas por el autor:

* **"The Masque of the Red Death" (Top 5):**
  1. Comas (`,`) — 171
  2. Puntos (`.`) — 100
  3. Puntos y comas (`;`) — 18
  4. Guiones bajos / Énfasis (`_`) — 14
  5. Guiones (`-`) — 12

* **"The Raven" (Top 5):**
  1. Comas (`,`) — 139
  2. Guiones (`-`) — 107
  3. Comillas (`"`) — 106
  4. Puntos (`.`) — 49
  5. Apóstrofes (`'`) — 30

### Análisis del Ritmo:
* **La Prosa Descriptiva:** En *The Masque*, el uso predominante de la coma (171) seguido de una alta cantidad de puntos (100) y puntos y comas (18) demuestra una progresión lineal, pausada y explicativa, diseñada para construir una atmósfera gótica mediante la acumulación de detalles arquitectónicos.
* **El Drama Poético:** En *The Raven*, la densidad de guiones (`-` con 107) y comillas (`"` con 106) supera por mucho a los puntos estructurales. El guion en la poesía de Poe funciona como una cesura rítmica que denota interrupción brusca, agitación psicológica o jadeo del narrador. Asimismo, la alta frecuencia de comillas evidencia la naturaleza dialógica y teatral del poema (el intercambio desesperado entre el protagonista y el cuervo).

---

## 4. Conclusiones Basadas en los Hallazgos

1. **Género vs. Estilo:** Los datos demuestran que las restricciones formales de la poesía lírica obligan al autor a reducir su diversidad léxica global (menor TTR) en favor de la estructura métrica, musical y repetitiva, mientras que la prosa le otorga mayor libertad de dispersión lingüística.
2. **Estabilidad del Idiolecto:** El hallazgo de que la longitud media de las palabras sea matemáticamente idéntica (4.48) valida la teoría estilométrica de que ciertas variables subconscientes del estilo de un autor permanecen inmunes al cambio de formato o género literario.
3. **Puntuación Psicológica:** El análisis computacional de la puntuación logra capturar el estado emocional plasmado en las obras. La transición de una puntuación canónica en el cuento hacia una puntuación disruptiva y conversacional (guiones y comillas) en el poema ratifica cuantitativamente la evolución del terror: de la observación pasiva de una plaga a la locura activa de un corazón roto.
