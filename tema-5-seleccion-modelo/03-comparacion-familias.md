# 5.3 ODE contra series de tiempo contra GBM

Este es el capítulo donde se ponen a competir las tres familias de modelos continuos que el árbol de decisión dejó como candidatas para el PIB: la ODE logística del Tema 1, el ARIMA del Tema 2 y el movimiento browniano geométrico, GBM, que aparece en la guía de estudio como el modelo de una variable continua con tendencia y ruido. La comparación se realiza sobre una moneda común, el error de pronóstico fuera de muestra, pues, como se vio en el capítulo anterior, el AIC no compara bien entre familias.

## Las tres filosofías en una tabla

Antes del código conviene tener claro qué representa cada familia, pues no son lo mismo con distinta cara, sino tres formas distintas de comprender el fenómeno.

```
Familia    Qué supone                          Da incertidumbre   Da mecanismo
ODE log.   Motor de crecimiento + techo         No (determinista)  Sí, interpretable
ARIMA      Memoria lineal sobre serie estac.    Sí, intervalos     No
GBM        Crecimiento proporcional + ruido     Sí, distribución   Parcial (drift)
```

La guía de estudio resume el GBM con su fórmula y su lectura: los cambios son proporcionales al valor actual, el drift es la tasa de crecimiento esperada y la volatilidad mide la incertidumbre, con la corrección de Ito en el exponente. Es el puente natural entre la ODE determinista y la serie de tiempo estadística, pues es una ODE con ruido.

## Preparación común

```python
import wbgapi as wb
import numpy as np
import pandas as pd

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]
df = wb.data.DataFrame("NY.GDP.MKTP.CD", brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib = (df.sum(axis=0) / 1e12).sort_index()
pib.index = pd.period_range("1995", "2022", freq="Y")

corte = 2017
train = pib[pib.index.year <= corte]
test = pib[pib.index.year > corte]
h = len(test)
```

Se utiliza un corte temporal: las tres familias se entrenan con los datos hasta 2017 y se mide su error sobre los años posteriores, que ninguna observó. Es la regla de validación del capítulo 2.8 (tema-2-series-de-tiempo/08-validacion-metricas.md).

## Candidato 1: ODE logística

```python
from scipy.optimize import curve_fit

tau = np.arange(len(train))
def logistica(tau, r, K, y0):
    return K / (1 + ((K - y0) / y0) * np.exp(-r * tau))

(r, K, y0), _ = curve_fit(logistica, tau, train.values,
                          p0=[0.1, 80, train.values[0]], maxfev=10000)
tau_test = np.arange(len(train), len(train) + h)
pred_ode = logistica(tau_test, r, K, y0)
```

## Candidato 2: ARIMA

```python
from statsmodels.tsa.arima.model import ARIMA

m = ARIMA(np.log(train), order=(1, 1, 1)).fit()
pred_arima = np.exp(m.forecast(steps=h).values)
```

## Candidato 3: GBM

El GBM se calibra estimando el drift y la volatilidad de los log-retornos históricos, y se proyecta con su media teórica. Se sigue la fórmula de la guía de estudio, cuidando la corrección de Ito.

```python
log_ret = np.log(train / train.shift(1)).dropna()
mu = log_ret.mean()
sigma = log_ret.std()
s0 = train.values[-1]

pasos = np.arange(1, h + 1)
# La media esperada del GBM es S0 * exp(mu * t), sin la corrección de Ito,
# porque la corrección vive en el exponente de las trayectorias, no en la media
pred_gbm = s0 * np.exp(mu * pasos)
```

Se respeta la advertencia de la guía de estudio: la media esperada del GBM usa `exp(mu*t)`, mientras que la corrección de Ito `sigma^2/2` aparece en el exponente de las trayectorias simuladas, no en la media. Confundir esto es uno de los errores que la guía de estudio señala como típicos de examen.

## La comparación sobre la moneda común

```python
def mape(real, pred):
    real, pred = np.asarray(real), np.asarray(pred)
    return np.mean(np.abs((real - pred) / real))

resultados = pd.DataFrame({
    "real": test.values,
    "ODE": pred_ode,
    "ARIMA": pred_arima,
    "GBM": pred_gbm,
}, index=test.index)

for modelo in ["ODE", "ARIMA", "GBM"]:
    print(f"MAPE {modelo}: {mape(test.values, resultados[modelo]):.2%}")
print(resultados.round(1))
```

El MAPE fuera de muestra es la moneda común que sí permite comparar familias distintas, algo que el AIC no podía hacer. El modelo con menor MAPE es el mejor para predecir este tramo, y esa es la evidencia dura que se lleva a la decisión.

## Cómo se lee el resultado sin distorsionarlo

Aquí cabe una precisión metodológica. El modelo ganador en este corte específico no es necesariamente el mejor en general, pues un solo corte puede favorecer a uno por azar. Por ello, el ranking de este capítulo es provisional, y se confirma con el backtesting de múltiples cortes del capítulo 5.4 (tema-5-seleccion-modelo/04-validacion-cruzada-temporal.md). Además, el MAPE no lo es todo: la ODE puede perder en MAPE pero ganar en interpretabilidad, y el ARIMA puede ganar en MAPE pero ser el único que ofrece intervalos. La elección final pondera precisión, incertidumbre e interpretabilidad según la decisión del Tema 4.

## La lectura de fondo

Cada familia ofrece una narrativa distinta sobre la transición BRICS+. La ODE indica que el bloque sigue lejos de su techo, de modo que el crecimiento continúa. El GBM indica que, si el régimen de drift y volatilidad reciente se mantiene, el bloque crece de forma multiplicativa con un cono de incertidumbre que se ensancha. El ARIMA indica que la memoria de corto plazo de la serie proyecta cierta inercia con intervalos. Las tres apuntan en la misma dirección de fondo, el bloque sigue ganando, y esa convergencia de tres métodos distintos resulta más convincente que cualquiera de ellos por separado.

## Bibliografía

Se compararon ODE, ARIMA y GBM sobre el error de pronóstico fuera de muestra, la única moneda que compara familias distintas con justicia. Se respetó la corrección de Ito en el GBM. El ganador en MAPE es provisional hasta confirmarlo con múltiples cortes, y la elección final pondera precisión, incertidumbre e interpretabilidad. La convergencia de las tres familias en la dirección de fondo refuerza la conclusión. Para robustecer el ranking más allá de un corte único, el siguiente capítulo desarrolla la validación cruzada temporal.

## Referencias

1. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
