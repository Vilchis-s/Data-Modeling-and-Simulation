# 2.4 Descomposición y suavizamiento exponencial

Antes de los modelos paramétricos tipo ARIMA conviene revisar dos familias de técnicas más directas e intuitivas: la descomposición, que separa la serie en sus componentes, y el suavizamiento exponencial, que pondera el pasado otorgando mayor peso a lo reciente. Son métodos que en producción siguen siendo competitivos y que con frecuencia superan a modelos más sofisticados, sobre todo en horizontes cortos.

## Descomposición STL

La descomposición STL, por seasonal-trend decomposition using loess, separa una serie en tendencia, estacionalidad y residuo de forma robusta. Es la versión moderna y flexible de la descomposición clásica vista en el capítulo 2.1 (tema-2-series-de-tiempo/01-componentes.md).

```python
import wbgapi as wb
import pandas as pd
import numpy as np
from statsmodels.tsa.seasonal import STL
import matplotlib.pyplot as plt

# Serie anual de PIB de Brasil; se usa STL con periodo corto para ilustrar el ciclo
pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "BRA", time=range(1970, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
idx = pd.period_range("1970", "2022", freq="Y")
pib = pd.Series(pib.values, index=idx)

resultado = STL(np.log(pib), period=5, robust=True).fit()
resultado.plot()
plt.show()
```

Se trabaja sobre el logaritmo del PIB para que la descomposición sea aditiva en escala logarítmica, lo que equivale a multiplicativa en escala original, apropiado para una variable cuyas oscilaciones crecen con el nivel. El componente de tendencia que devuelve STL es una estimación suave del crecimiento de largo plazo de Brasil, libre de fluctuaciones, y el residuo aísla los choques, donde se aprecian con claridad las crisis de la economía brasileña.

La descomposición no es un modelo de pronóstico por sí misma, pero constituye un diagnóstico excelente: indica cuánta de la varianza proviene de la tendencia, cuánta del ciclo y cuánta del ruido, lo que orienta el grado de ambición admisible al pronosticar.

## Suavizamiento exponencial simple

El suavizamiento exponencial parte de una idea razonable: para predecir el próximo valor, se promedian los valores pasados, otorgando mayor peso a los recientes y menor a los lejanos, con pesos que decaen geométricamente. El parámetro alfa, entre 0 y 1, controla con qué rapidez se olvida el pasado. Un alfa cercano a 1 reacciona rápido a los cambios recientes; uno cercano a 0 produce un pronóstico muy suave y lento.

```python
from statsmodels.tsa.holtwinters import SimpleExpSmoothing

modelo = SimpleExpSmoothing(pib, initialization_method="estimated").fit()
print(f"alfa estimado = {modelo.params['smoothing_level']:.3f}")
pronostico = modelo.forecast(5)
```

El suavizamiento simple sirve para series sin tendencia ni estacionalidad. El PIB tiene tendencia clara, por lo que el suavizamiento simple resulta insuficiente: su pronóstico es plano y subestima de forma sistemática una serie creciente. Esto motiva las extensiones.

## El método de Holt: incorporando tendencia

Holt extiende el suavizamiento exponencial con un segundo componente que suaviza la tendencia, no solo el nivel. Ahora hay dos parámetros, alfa para el nivel y beta para la tendencia, y el pronóstico ya no es plano, sino que proyecta la pendiente reciente.

```python
from statsmodels.tsa.holtwinters import Holt

modelo_holt = Holt(pib, initialization_method="estimated").fit()
pronostico_holt = modelo_holt.forecast(5)
print(pronostico_holt)
```

Para una serie de PIB en niveles, con tendencia y sin estacionalidad anual, Holt suele ser un punto de partida sorprendentemente adecuado. Su debilidad es que extrapola la tendencia de forma lineal e indefinida, lo que sobreestima a largo plazo, igual que el exponencial puro del Tema 1. Por ello existe la variante con tendencia amortiguada, que aplana la proyección con el tiempo, más realista para economías que maduran.

## Holt-Winters: incorporando estacionalidad

El método completo, Holt-Winters, agrega un tercer componente para la estacionalidad, con su parámetro gamma. Es el método de elección cuando la serie tiene los tres componentes: nivel, tendencia y un patrón estacional repetido. En datos anuales de PIB la estacionalidad no aplica, pero en datos trimestrales de comercio, producción industrial o consumo energético resulta indispensable.

```python
from statsmodels.tsa.holtwinters import ExponentialSmoothing

# Esquema general; con datos trimestrales se usaría seasonal_periods=4
modelo_hw = ExponentialSmoothing(
    pib, trend="add", seasonal=None, damped_trend=True,
    initialization_method="estimated"
).fit()
print(modelo_hw.forecast(5))
```

Se activó la tendencia amortiguada con `damped_trend=True`, opción recomendada por defecto para variables económicas en niveles, pues evita la extrapolación lineal indefinida.

## Por qué se comienza por aquí antes de ARIMA

El suavizamiento exponencial tiene una virtud muy apreciable en el trabajo: es robusto, rápido y difícil de romper. Al entregar un pronóstico conviene ejecutar siempre un Holt-Winters como referencia, pues si un modelo ARIMA sofisticado no supera a un suavizamiento exponencial bien calibrado, la complejidad adicional no se justifica. Es la misma lógica de parsimonia que la guía de estudio establece como propiedad de un buen modelo: máxima fidelidad con el mínimo de parámetros. El modelo simple no es un premio de consolación, sino el estándar a superar.

## Bibliografía

La descomposición STL separa la serie en componentes y diagnostica el origen de su variabilidad. El suavizamiento exponencial, en sus versiones simple, Holt y Holt-Winters, pronostica ponderando el pasado con pesos que decaen, e incorpora tendencia y estacionalidad según corresponda. Constituyen la referencia obligada frente a la cual juzgar modelos más complejos. El siguiente paso es entrar a la familia paramétrica que domina las series de tiempo clásicas: los modelos AR, MA y ARMA.

## Referencias

1. Cleveland, R. B., Cleveland, W. S., McRae, J. E., & Terpenning, I. (1990). STL: A seasonal-trend decomposition procedure based on loess. *Journal of Official Statistics, 6*(1), 3-73.
2. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
