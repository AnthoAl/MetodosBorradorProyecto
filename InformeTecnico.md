
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

    A partir del análisis gráfico de los puntos donde la red neuronal cambia de salida entre las clases 0 y 1, se observó que la frontera de decisión presenta un comportamiento oscilatorio y decreciente, una característica muy notable de funciones tipo Sinc:

    <p style="text-align:center;">
$$
\text{sinc}(x) = \frac{\sin(kx)}{kx}
$$
</p>

    En particular, la forma de las fronteras sugiere que el patrón subyacente sigue un comportamiento similar a:

        <p style="text-align:center;">
$$
\text{sinc}(x) = \frac{\sin(10x)}{10x}
$$
</p>
    Sin embargo, al comparar esta función ideal con los datos generados por la red, fue necesario introducir dos modificaciones para ajustarla correctamente:

**1.Amortiguación artificial (Blackbox) para evitar la singularidad en x=0:**  

     <p style="text-align:center;">
$$
\frac{1}{x} \;\longrightarrow\; \frac{1}{x + 0.1}
$$
</p>

**2. Ajuste paramétrico general para modelar correctamente la amplitud, frecuencia, desfase y desplzamiento vertical propios de la frontera aprendida:**

  <p style="text-align:center;">
$$
X<sub>2</sub> = A.\frac{\sin(Bx<sub>1</sub> + C)}{X<sub>1</sub> + 0.1} + D
$$

Este modelo constituye una **Sinc amortiguada paramétrica**, que analiza el comportamiento oscilatorio de la frontera, pero a su vez permite adaptarlo a los valores reales detectados por los metodos del algoritmo.

    * Formulación de las dos ecuaciones de la frontera superior e inferior.

Durante el muestreo sistemático del plano (x<sub>1</sub>,x<sub>2</sub>), la red neuronal tenia como clasificación; **0** o **1**. A partir de esta clasificación se identificaron dos tipos de transiciones:

1. **Frontera Superior ($1\longrightarrow0$)**
Corresponde a los puntos en donde, al aumentar x<sub>2</sub>, la red cambia su predicción desde 1 hacia 0.Es decir, se delimita el limite superior de la banda donde la red considera salida = 1.
Esto, en un dialecto matemáticoe,quiere decir que la fronte fue modelada mediante un Sinc amortiguada con parámetros ajustados usando **curve_fit o Metodo de Levenberg-Marquardt:**

  <p style="text-align:center;">
$$
x<sub>2</sub><sup>up</sup>(x<sub>1</sub>) = A<sub>sup</sub> . \frac{\sin(B<sub>sup</sub>x<sub>1</sub> + C<sub>sup</sub>)}{x<sub>1</sub> + 0.1} + D<sub>sup</sub>
$$

Los parametros A<sub>sup</sub>,B<sub>sup</sub>,C<sub>sup</sub>,D<sub>sup</sub> representan el ajuste optimo obetenido a partil del conjunto **frontera_superior**

2. **Frontera Inferior ($0\longrightarrow1$)**
Corresponde a los punto donde, al disminuir x<sub>2</sub>, la red cambia su prediccion desde 0 hacia 1.
Define el **limite inferior** de la region donde la red activa la salida = 1.
Para esta formulacion analitica, se siguio el mismo modelo amortiguado, pero con parametros diferentes:

  <p style="text-align:center;">
$$
x<sub>2</sub><sup>up</sup>(x<sub>1</sub>) = A<sub>inf</sub> . \frac{\sin(B<sub>inf</sub>x<sub>1</sub> + C<sub>inf</sub>)}{x<sub>1</sub> + 0.1} + D<sub>inf</sub>
$$

Los parametros a evaluar se obtuvieron el método de Guss-Newton y se contrastaron numéricamente con la aproximación de Levenberg-Marquardt para validad la equivalencia del ajuste.

* **2.2. Descripción de la Implementación:**
    * **2.2.1. Muestreo de la Frontera (Doble Bisección):** 
    Con la finalización del alcance del objetivo de obtener una representación precisa de las fronteras de decisión de la red Neuronal BlackBox S, se implemento un algoritmo de muestreo mediante una doble bisección. Dado que este método permite localizar con alta exactitud los puntos donde la red cambia su salida entre 0 y , lo cual defina una banda en la que la funcion de la red es igual a 1.

    Dado los pasos a seguir del algoritmo, fueron:

    **a) Exploración inicial** 
    Para cada valor de 𝑥 1 x 1 ​ dentro del intervalo estudiado, se realizó un muestreo preliminar sobre un rango definido de valores de 𝑥 2 x 2 ​ . Este muestreo permite identificar de manera aproximada la región donde ocurre una transición abrupta en la salida de la red, ya sea:

    • **De 1 a 0**(frontera superior)

    • **De 0 a 1**(frontera inferior)

    Este punto inicial sirve como referencia para el refinamiento posterior

    **b) Bisección para la Frontera Superior ($1\longrightarrow0$)** 

    Una vez detectado un punto aproximado donde la red deja de clasificar como 1, se define un intervalo [x<sub>2low</sub>, 𝑥<sub>2high</sub>] que contiene la transición. Sobre este intervalo se aplica el método de bisección clásica, evaluando la red en el punto medio:

      <p style="text-align:center;">
$$
X<sub>2mid</sub> = \frac{X<sub>2</sub> + X<sub>2high</sub>}{2} 
$$


    * **2.2.2. Método Numérico 1: Levenberg-Marquardt (L-M):** Implementación de la regresión no lineal (usando `scipy.optimize.curve_fit`).
    * **2.2.3. Método Numérico 2: Gauss-Newton (GN):** Implementación manual para comparación.
* **2.3. Diagrama de Flujo / Pseudocódigo.**
* **2.4. Análisis de Estabilidad y Convergencia (CDT):** Análisis teórico de L-M y GN.

---

## 3. Resultados

Este es tu apartado principal.

* **3.1. Ejecución y Descripción de Casos de Prueba.**
* **3.2. Comparación con Soluciones Analíticas.**
* **3.3. Análisis de Resultados (Relación Analítica Final).**
* **3.4. Análisis de Complejidad Computacional Experimental.**

---

## 4. Conclusiones y Trabajo Futuro ✅

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
