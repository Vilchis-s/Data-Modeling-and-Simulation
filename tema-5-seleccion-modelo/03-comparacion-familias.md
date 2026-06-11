# 5.3 ODE contra series de tiempo contra GBM

Este es el capítulo donde pongo a competir las tres familias de modelos continuos que el árbol de decisión dejó como candidatas para el PIB: la ODE logística del Tema 1, el ARIMA del Tema 2 y el movimiento browniano geométrico, GBM, que aparece en la guía como el modelo de una variable continua con tendencia y ruido (Law, 2014). La comparación se hace sobre una moneda común, el error de pronóstico fuera de muestra, porque como vi en el capítulo anterior, el AIC no compara bien entre familias.

## Las tres filosofías en una tabla

Antes del código, conviene tener claro qué representa cada familia, porque no son lo mismo con distinta cara, son tres formas distintas de entender el fenómeno.

```
Familia    Qué supone                          Da incertidumbre   Da mecanismo
ODE log.   Motor de crecimiento + techo         No (determinista)  Sí, interpretable
ARIMA      Memoria lineal sobre serie estac.    Sí, intervalos     No
GBM        Crecimiento proporcional + ruido     Sí, distribución   Parcial (drift)
```

La guía resume el GBM con su fórmula y su lectura: los cambios son proporcionales al valor actual, el drift es la tasa de crecimiento esperada y la volatilidad mide la incertidumbre, con la corrección de Ito en el exponente (Law, 2014). Es el puente natural entre la ODE determinista y la serie de tiempo estadística, porque es una ODE con ruido.

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

Uso un corte temporal: entreno las tres familias con los datos hasta 2017 y mido su error sobre los años posteriores, que ninguna vio. Es la regla de validación del capítulo 2.8.

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

El GBM se calibra estimando el drift y la volatilidad de los log-retornos históricos, y se proyecta con su media teórica. Sigo la fórmula de la guía, cuidando la corrección de Ito (Law, 2014).

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

Respeto la advertencia de la guía: la media esperada del GBM usa `exp(mu*t)`, mientras que la corrección de Ito `sigma^2/2` aparece en el exponente de las trayectorias simuladas, no en la media (Law, 2014). Confundir esto es uno de los errores que la guía marca como típicos de examen.

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

El MAPE fuera de muestra es la moneda común que sí permite comparar familias distintas, algo que el AIC no podía hacer. El modelo con menor MAPE es el mejor para predecir este tramo, y esa es la evidencia dura que llevo a la decisión.

## Cómo leo el resultado sin tramposear

Aquí viene la honestidad metodológica. El modelo ganador en este corte específico no es necesariamente el mejor en general, porque un solo corte puede favorecer a uno por azar. Por eso el ranking de este capítulo es provisional, y lo confirmo con el backtesting de múltiples cortes del capítulo 5.4. Además, el MAPE no lo es todo: la ODE puede perder en MAPE pero ganar en interpretabilidad, y el ARIMA puede ganar en MAPE pero ser el único que da intervalos. La elección final pondera precisión, incertidumbre e interpretabilidad según la decisión del Tema 4.

## La lectura de fondo

Cada familia cuenta una historia distinta sobre la transición BRICS+. La ODE dice que el bloque sigue lejos de su techo, así que el crecimiento continúa. El GBM dice que, si el régimen de drift y volatilidad reciente se mantiene, el bloque crece de forma multiplicativa con un cono de incertidumbre que se ensancha. El ARIMA dice que la memoria de corto plazo de la serie proyecta cierta inercia con intervalos. Las tres apuntan en la misma dirección de fondo, el bloque sigue ganando, y esa convergencia de tres métodos distintos es, para mí, más convincente que cualquiera de ellos por separado.

## Cierre

Comparé ODE, ARIMA y GBM sobre el error de pronóstico fuera de muestra, la única moneda que compara familias distintas con justicia. Respeté la corrección de Ito en el GBM. El ganador en MAPE es provisional hasta confirmarlo con múltiples cortes, y la elección final pondera precisión, incertidumbre e interpretabilidad. La convergencia de las tres familias en la dirección de fondo refuerza la conclusión. Para robustecer el ranking más allá de un corte único, el siguiente capítulo desarrolla la validación cruzada temporal.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Hull, J. C. (2018). *Options, futures, and other derivatives* (10a ed.). Pearson.

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
