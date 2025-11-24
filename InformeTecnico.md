
# Escuela Politécnica Nacional

## Métodos Numéricos

## Informe Técnico: Proyecto Blackbox S

---
**Integrantes:** Alangasí Anthony, Nicole Achote, Danny Caiza
**Curso:** GR1CC
**Grupo:** 8

---

## 1. Introducción

Las redes neuronales han adquirido un papel muy importante en la modelación de fenómenos complejos debido a su capacidad para representar relaciones no lineales entre variables. En este tipo de sistemas, se dispone únicamente de sus entradas y salidas, pero no de una expresión analítica exacta que determine su comportamiento interno entre sus entradas y salidas. En estos casos, surge la necesidad de emplear métodos numéricos que permitan inferir o aproximar la relación analítica que existe entre ellas.

El presente proyecto tiene como objetivo estudiar la red neuronal Blackbox S, la cual implementa una función paramétrica $$fθ:R2→B$$ que asigna un valor binario a cada par de valores $$(x_1, x_2)$$con la condición $$x_1 \ge 0$$ Si bien el modelo permite obtener predicciones para cualquier punto del dominio, la dependencia analítica entre las variables $x_1$ y $x_2$ sigue sin ser conocida explícitamente. Por ello, el objetivo principal es determinar la relación matemática que existe entre estas ambas variables utilizando técnicas de optimización no lineal, fundamentadas en métodos numéricos.

Para ello se aplican dos métodos numéricos utilizados comúnmente en problemas de optimización y ajuste de modelos que son Gauss-Newton y Levenberg–Marquardt. Estos métodos permiten estimar parámetros mediante la minimización del error entre valores observados y valores generados por la función objetivo. La comparación de estos valores permite evaluar diferencias en precisión, velocidad de convergencia y estabilidad numérica.

Finalmente, se hace la comparación y la interpretación de los resultados obtenidos por cada método, determinando cual proporciona una aproximación más adecuada a la relación analítica buscada.  Se destaca también la importancia de los métodos numéricos para el estudio de modelos cuyo comportamiento no se presenta en forma explícita.

---

## 2. Metodología 💡

Este apartado es responsabilidad del **Analista Matemático y de Implementación (AMI)** y el **Coordinador (CDT)**.

* **2.1. Desarrollo Matemático y Modelo Analítico:**
    * Identificación de la función subyacente (Sinc Amortiguada) basada en la visualización.
    * Formulación de las dos ecuaciones de la frontera superior e inferior.
* **2.2. Descripción de la Implementación:**
    * **2.2.1. Muestreo de la Frontera (Doble Bisección):** Explicación del algoritmo para encontrar los puntos de alta precisión.
    * **2.2.2. Método Numérico 1: Levenberg-Marquardt (L-M):** Implementación de la regresión no lineal (usando `scipy.optimize.curve_fit`).
    * **2.2.3. Método Numérico 2: Gauss-Newton (GN):** Implementación manual para comparación.
* **2.3. Diagrama de Flujo / Pseudocódigo.**
* **2.4. Análisis de Estabilidad y Convergencia (CDT):** Análisis teórico de L-M y GN.

---

## 3. Resultados

* **3.1. Ejecución y Descripción de Casos de Prueba.**
    Se ha realizado un muestreo de varios puntos utilizando el modelo para identificar la región donde $f(x_1,x_2) = 1$. En la Figura n, se observa que la región tiene una forma senoidal hasta $x_1 \approx 0.9$ y luego, se mantiene de forma constante.
    <br>
    ![Muestreo de datos](image.png)
    *Figura n Gráfica del muestreo de datos resultante*
    <br>

    Debido a que el conjunto de puntos está contenido en un área limitada, se aplicó el método de bisección para encontrar los puntos ubicados en la frontera de decisión donde el modelo cambia de 0 a 1 con una tolerancia de $10^{-5}$. Esto permitió obtener dos conjuntos de puntos que representan las fronteras superior e inferior del conjunto donde el modelo predice 1. Estos puntos se muestran en la Figura n.
    <br>
    ![Fronteras de decisión](image-1.png)
    *Figura n Gráfica de las fronteras de decisión obtenidas a través del método de bisección*
    <br>

* **3.2. Comparación con Soluciones Analíticas.**
    Con base en la forma presentada en la anterior figura y la función real utilizada por el modelo Blackbox S, se propuso el siguiente modelo de regresión no lineal basado en una variante de la función $\frac{sin(10x)}{10x}$: <br>
    $x_2 = \frac{Asin(Bx_1 + C)}{x_1 + 0.1} + D$<br>
    donde A, B, C y D son parámetros a ajustar. Se aplicaron los métodos de Gauss-Newton y Levenberg-Marquardt para ajustar estos parámetros utilizando los puntos obtenidos de la frontera inferior puesto que era la más parecida a la forma de la función original.
    <br>
* **3.3. Análisis de Resultados**
    Ambos métodos convergieron a soluciones similares, obteniendo los siguientes parámetros:
    - Levenberg-Marquardt: A = 0.13543566, B = 8.95118215, C = 0.63047492, D = -0.05461683
    - Gauss-Newton: A = 0.13543742, B = 8.95141279, C = 0.63044158, D = -0.05461769

    Las funciones obtenidas son las siguientes:
    - Levenberg-Marquardt:
    $x_2 = \frac{0.13543566 \cdot sin(8.95118215 \cdot x_1 + 0.63047492)}{x_1 + 0.1} - 0.05461683$
    - Gauss-Newton:
    $x_2 = \frac{0.13543742 \cdot sin(8.95141279 \cdot x_1 + 0.63044158)}{x_1 + 0.1} - 0.05461769$

    A continuación, se presenta la comparación gráfica entre las funciones obtenidas por ambos métodos y la función real en la Figura n.<br>
    ![Gráfica de comparación de modelos ajustados y la función original](image-2.png)
    *Figura n Comparación del ajuste de la frontera inferior utilizando Gauss-Newton y Levenberg-Marquardt*<br>
    Para comparar los métodos utilizados, se utilizó el error cuadrático medio (MSE). Se empleó esta métrica porque el objetivo principal de los métodos empleados es reducir el error cuadrático entre los puntos trazados por la función real y los valores predichos por el modelo ajustado. Los resultados obtenidos son:<br>
    ***MSE Levenberg-Marquardt:** 0.0094185092*
    ***MSE Gauss-Newton:** 0.0094183050*<br>
    Con base al error presentado, se concluye que ambos métodos generan resultados muy similares en cuanto a su presición y solo se presentan diferencias en los valores de los parámetros obtenidos.
    <br>
* **3.4. Análisis de Complejidad Computacional Experimental**
    Se midió el tiempo de ejecución de ambos métodos para evaluar su eficiencia computacional. Los resultados obtenidos fueron:
    - Tiempo de ejecución Levenberg-Marquardt: 0.004609 segundos
    - Tiempo de ejecución Gauss-Newton: 0.007342 segundos

    Estos resultados indican que el método de Levenberg-Marquardt es más eficiente en términos de tiempo de ejecución en comparación con el método de Gauss-Newton.

---

## 4. Conclusiones y Trabajo Futuro

* **4.1. Resumen de los Hallazgos más Importantes.**
Este proyecto permitió identificar la relación funcional que establece el modelo Blackbox S entre las variables $x_1$ y $x_2$, a pesar de no conocer una expresión analítica interna explícita del modelo. Mediante un muestreo sistemático y el uso del método de bisección, se determinó con alta precisión la frontera en la cual el modelo cambia su salida entre 0 y 1, obteniendo dos curvas continuas y suaves que representan los límites superior e inferior del conjunto donde el modelo predice 1. Una vez obtenidos los puntos experimentales de dichas fronteras, se propuso un modelo funcional basado en una variante de la función $sin(x)/x$ o también llamada seno cardinal dependiente únicamente de $x_1$ Con el ajuste de parámetros utilizando los métodos Gauss-Newton y Levenberg–Marquardt se obtuvo una función analítica aproximada que describe dicha frontera con gran precisión. Ambos métodos convergieron prácticamente al mismo conjunto de parámetros, lo que valida la estabilidad del modelo matemático elegido y confirma la consistencia de los procedimientos aplicados.

<br>

* **4.2. Dificultades Encontradas y Soluciones (CDT y ARV).**
Durante el desarrollo del proyecto surgieron varias dificultades relevantes:
  * Se nos dificultó la identificación precisa del punto de cambio entre 0 y 1, inicialmente el muestreo directo no era lo suficientemente preciso para localizar el punto exacto donde el modelo cambia su salida, pero se implemento un procedimiento de bisección para definir los límites hasta alcanzar una tolerancia adecuada
  * También fue complicado la elección de un modelo analítico adecuado ya que no se conocía la forma de la función que debía aproximarse, pero tras analizar la estructura oscilatoria decreciente de los datos, llevó a proponer un modelo basado en una función sinc, esta elección permitió que ambos métodos de ajuste convergieran correctamente teniendo un ajuste estable y coherente.
  * Otra dificultad fue encontrar una estabilidad numérica en los ajustes, esto porque el método Gauss-Newton puede divergir si los valores iniciales no son buenos, para esto se tomaron como valores iniciales parámetros razonables basados en la forma visual de los datos, esto evitó inestabilidad numérica y mejoró la convergencia.

<br>


* **4.3. Limitaciones y Restricciones del Enfoque.**
El enfoque implementado presenta varias limitaciones como, por ejemplo:

  * La dependencia total del comportamiento del modelo neuronal ya que la relación encontrada no proviene de una deducción teórica, sino de observar cómo responde el modelo. Si el modelo tuviera ruido, o comportamientos erráticos, la aproximación sería menos confiable.
  * La función ajustada no es la única posible, existen infinitas funciones que pueden aproximar los puntos obtenidos, la que se escogió es la adecuada, pero no necesariamente la única o la óptima en términos matemáticos.
  * Tener un dominio restringido ya que el análisis se realizó dentro de un rango limitado de $x_1$ y $x_2$, fuera de ese rango, no se garantiza la validez del modelo.
  * Uso de métodos sensibles a los valores iniciales propuestos, porque tanto Gauss-Newton como Levenberg–Marquardt requieren buenas condiciones iniciales para converger adecuadamente.
  
<br>

* **4.4. Posibles Mejoras y Trabajos Futuros.**
Existen varias opciones en las que el proyecto se puede ampliar y profundizar el analisis:
  * La principal es extender el dominio de análisis ya que, se debe realizar un muestreo más amplio para verificar el comportamiento global del modelo
  * Probar utilizando métodos de regresión más avanzados como: regresión polinomial adaptativa, redes neuronales inversas, modelos simbólicos (Symbolic Regression), etc, estos podrían describir una función más precisa.
  * Evaluar otras funciones base para el ajuste como: fracciones racionales, funciones B-spline, polinomios de Chebyshev, etc, para comparar su desempeño frente al modelo tipo $sin(x)/x.$
  * Analizar la sensibilidad del modelo, observar cómo pequeñas variaciones en los datos pueden afectar la frontera y el ajuste, esto ayudaría a medir la estabilidad del método.
