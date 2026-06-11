# 2.8 Validación temporal y métricas de error

Un modelo de series de tiempo no vale por lo bien que ajusta el pasado, sino por lo bien que predice lo que no vio. Esto suena obvio, pero la forma de medirlo en series de tiempo es distinta y más delicada que en el aprendizaje supervisado clásico, porque el orden temporal lo cambia todo. Este capítulo trata cómo validar honestamente y con qué métricas.

## Por qué no se puede usar validación cruzada normal

En un problema tabular barajo los datos y reparto en entrenamiento y prueba al azar. En una serie de tiempo eso es un error grave, porque significaría entrenar con datos del futuro para predecir el pasado, una fuga de información que infla artificialmente el desempeño. El principio inviolable es: el conjunto de prueba siempre debe ser posterior en el tiempo al de entrenamiento. Nunca se mira el futuro para predecir el pasado.

## La partición temporal simple

La forma más básica y correcta es cortar la serie en un punto: entreno con lo anterior, evalúo con lo posterior.

```python
import numpy as np
import wbgapi as wb
import pandas as pd
from statsmodels.tsa.arima.model import ARIMA

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "IND", time=range(1970, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = pd.period_range("1970", "2022", freq="Y")
log_pib = np.log(pib)

corte = "2014"
train = log_pib[:corte]
test = log_pib[corte:][1:]            # estrictamente posterior al corte

modelo = ARIMA(train, order=(1, 1, 1)).fit()
pred = modelo.forecast(steps=len(test))
```

El corte deja los últimos años fuera del entrenamiento para simular cómo se habría comportado el modelo si lo hubiera entrenado en ese momento y proyectado hacia adelante. Es lo más cercano a un experimento honesto que puedo hacer con datos históricos.

## Backtesting con ventana expansiva

La partición simple usa un solo corte, lo que la hace dependiente de qué tan típico fue ese tramo final. Una validación más robusta es el backtesting con ventana expansiva, también llamado validación cruzada de series de tiempo: hago varios cortes sucesivos, en cada uno entreno con todo lo anterior y predigo el siguiente paso, y promedio el error sobre todos los cortes.

```python
def backtesting_expansivo(serie, order, inicio, horizonte=1):
    errores = []
    for t in range(inicio, len(serie) - horizonte):
        train = serie.iloc[:t]
        real = serie.iloc[t + horizonte - 1]
        try:
            m = ARIMA(train, order=order).fit()
            pred = m.forecast(steps=horizonte).iloc[-1]
            errores.append(real - pred)
        except Exception:
            continue
    return np.array(errores)

errs = backtesting_expansivo(log_pib, (1, 1, 1), inicio=30)
print(f"Error medio absoluto en backtesting (escala log): {np.mean(np.abs(errs)):.4f}")
```

La ventana expansiva imita cómo se usaría el modelo en la vida real: cada año llega un dato nuevo, reentreno y vuelvo a proyectar. El error promedio sobre muchos cortes es una estimación mucho más confiable del desempeño futuro que un solo corte afortunado o desafortunado.

## Las métricas de error y cuándo usar cada una

No hay una métrica universal; cada una mide algo distinto y tiene un sesgo propio.

El MAE, error absoluto medio, es el promedio de los errores en valor absoluto. Es fácil de interpretar porque está en las unidades de la serie y es robusto a valores atípicos.

El RMSE, raíz del error cuadrático medio, penaliza más los errores grandes por elevarlos al cuadrado. Lo prefiero cuando un error grande es desproporcionadamente costoso, por ejemplo cuando subestimar una crisis es peor que muchos errores pequeños.

El MAPE, error porcentual absoluto medio, expresa el error en porcentaje, lo que permite comparar series de escalas distintas, como el PIB de China y el de Sudáfrica. Su defecto es que se vuelve inestable cuando la serie tiene valores cercanos a cero y que penaliza asimétricamente las sobreestimaciones.

```python
def metricas(real, pred):
    real, pred = np.asarray(real), np.asarray(pred)
    mae = np.mean(np.abs(real - pred))
    rmse = np.sqrt(np.mean((real - pred) ** 2))
    mape = np.mean(np.abs((real - pred) / real))
    return {"MAE": mae, "RMSE": rmse, "MAPE": mape}
```

## La línea base ingenua, el estándar a vencer

Antes de celebrar cualquier modelo, lo comparo contra la predicción más tonta posible: el pronóstico ingenuo, que predice que mañana será igual a hoy. Para una serie con tendencia uso el ingenuo con deriva, que extiende la última pendiente. Si mi ARIMA no le gana al pronóstico ingenuo, toda su complejidad es injustificada.

```python
def naive_drift(train, n):
    ultimo = train.iloc[-1]
    pendiente = (train.iloc[-1] - train.iloc[0]) / (len(train) - 1)
    return np.array([ultimo + pendiente * (i + 1) for i in range(n)])
```

Esta comparación es la versión en pronóstico de la regla de parsimonia de la guía del curso (Law, 2014). El modelo complejo tiene que ganarse su lugar superando a la alternativa trivial, no se le concede por defecto.

## Validar los residuos, no solo el error

Una última verificación que no se puede saltar: los residuos del modelo deben parecer ruido blanco. Si tienen autocorrelación, el modelo dejó estructura sin capturar. Esto se prueba con Ljung-Box sobre los residuos, como vi en el capítulo 2.3, y se complementa mirando que los residuos no muestren patrones ni varianza cambiante.

```python
from statsmodels.stats.diagnostic import acorr_ljungbox
res = ARIMA(log_pib, order=(1, 1, 1)).fit().resid
print(acorr_ljungbox(res, lags=[10], return_df=True))
```

## Cierre

Validar una serie de tiempo exige respetar el orden temporal: el futuro nunca entra al entrenamiento. La partición simple es el mínimo, el backtesting expansivo es lo robusto. MAE, RMSE y MAPE miden facetas distintas del error y hay que elegir según el costo del negocio. Todo modelo se compara contra el pronóstico ingenuo y se valida revisando que sus residuos sean ruido blanco. Con toda la maquinaria del Tema 2 lista, cierro con el caso completo: ajustar y validar un ARIMA sobre el PIB de BRICS+ y G7 para proyectar la transición.

## Referencias

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/

Bergmeir, C., & Benítez, J. M. (2012). On the use of cross-validation for time series predictor evaluation. *Information Sciences, 191*, 192-213. https://doi.org/10.1016/j.ins.2011.12.028

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.
