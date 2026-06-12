# 2.6 ARIMA y SARIMA

ARIMA es el modelo de series de tiempo más utilizado de la historia y, con razón, el estándar frente al cual se mide todo lo demás. Su nombre describe sus tres ingredientes: AutoRegresivo, Integrado, de Medias móviles. El elemento nuevo respecto al capítulo anterior es la I, la integración, que es la forma elegante de incorporar la diferenciación dentro del propio modelo.

## Los tres órdenes de un ARIMA(p, d, q)

Un ARIMA tiene tres parámetros enteros.

p es el orden autorregresivo: cuántos valores pasados de la serie intervienen.

d es el orden de integración: cuántas veces hay que diferenciar la serie para volverla estacionaria.

q es el orden de medias móviles: cuántos choques pasados intervienen.

La d es la clave que conecta con la estacionariedad. En el capítulo 2.2 se observó que el PIB en niveles no es estacionario, pero su primera diferencia sí lo es. Un ARIMA(p, 1, q) modela exactamente eso: diferencia la serie una vez de forma interna, ajusta un ARMA(p, q) sobre la serie diferenciada y luego deshace la diferenciación para devolver pronósticos en la escala original. Ya no es necesario diferenciar manualmente: se indica d=1 y el modelo se encarga.

```python
import wbgapi as wb
import pandas as pd
import numpy as np
from statsmodels.tsa.arima.model import ARIMA

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "CHN", time=range(1980, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = pd.period_range("1980", "2022", freq="Y")

log_pib = np.log(pib)                       # estabiliza la varianza
modelo = ARIMA(log_pib, order=(1, 1, 1)).fit()
print(modelo.summary())
```

Se modela el logaritmo del PIB con un ARIMA(1,1,1): primero el logaritmo estabiliza la varianza creciente, luego la diferenciación interna estabiliza la media. Es la receta estándar para una variable económica en niveles.

## Pronóstico con intervalos

Un pronóstico sin medida de incertidumbre es casi inútil, y en este aspecto ARIMA destaca, pues entrega intervalos de predicción de forma natural. La guía de estudio es enfática en que nunca se reporta un estimador puntual sin su intervalo, y esa regla se aplica al pronóstico de forma directa.

```python
pred = modelo.get_forecast(steps=10)
media = np.exp(pred.predicted_mean)                  # vuelve a escala original
ic = np.exp(pred.conf_int(alpha=0.05))               # intervalo al 95%

import matplotlib.pyplot as plt
pib.plot(label="Observado")
media.plot(label="Pronóstico", color="red")
plt.fill_between(media.index.to_timestamp(),
                 ic.iloc[:, 0], ic.iloc[:, 1], alpha=0.2, color="red")
plt.legend()
plt.title("ARIMA(1,1,1) sobre PIB de China con intervalo al 95%")
plt.show()
```

El intervalo se ensancha conforme avanza el horizonte, lo cual es honesto: cuanto más lejana es la proyección, menor es el conocimiento disponible. Ese ensanchamiento es precisamente la incertidumbre que un modelo determinista como la ODE logística del Tema 1 no podía expresar, y constituye una de las grandes ventajas de la rama estadística.

## SARIMA: incorporando estacionalidad

Cuando la serie presenta un patrón estacional, ARIMA no basta y entra SARIMA, que agrega un bloque estacional con sus propios órdenes:

```
SARIMA(p, d, q)(P, D, Q, s)
```

Los primeros tres son los órdenes no estacionales de siempre. Los siguientes cuatro son sus análogos estacionales: P, D, Q son el AR, la integración y el MA estacionales, y s es el periodo de la estación, por ejemplo 4 en datos trimestrales o 12 en mensuales. La D estacional diferencia la serie respecto al mismo periodo del ciclo anterior, por ejemplo este trimestre frente al mismo trimestre del año previo, lo que elimina la estacionalidad.

El PIB anual carece de estacionalidad, de modo que, para ilustrar SARIMA, se cambia a una serie trimestral con estacionalidad genuina. Aquí se utilizan datos sintéticos, lo cual se indica de forma explícita, pues se requiere una serie trimestral con un patrón estacional limpio y controlado para que el ejemplo sea didáctico, no por falta de datos reales.

```python
# Datos sintéticos: serie trimestral con tendencia, estacionalidad y ruido
rng = np.random.default_rng(42)
n = 80
t = np.arange(n)
tendencia = 100 + 0.8 * t
estacional = 10 * np.sin(2 * np.pi * t / 4)         # ciclo de 4 trimestres
ruido = rng.normal(0, 3, n)
y = tendencia + estacional + ruido
idx = pd.period_range("2004Q1", periods=n, freq="Q")
serie_q = pd.Series(y, index=idx)

from statsmodels.tsa.statespace.sarimax import SARIMAX
sarima = SARIMAX(serie_q, order=(1, 1, 1),
                 seasonal_order=(1, 1, 1, 4)).fit(disp=False)
print(f"AIC SARIMA = {sarima.aic:.1f}")
```

SARIMA captura a la vez la tendencia creciente y el ciclo de cuatro trimestres. En el trabajo real se emplea para series trimestrales de comercio o producción de los bloques, donde el patrón estacional es fuerte y omitirlo arruina el pronóstico.

## auto_arima: búsqueda automática de órdenes

Elegir p, d, q manualmente con la ACF y la PACF funciona, pero resulta laborioso y subjetivo. En la práctica se utiliza `auto_arima` de la librería `pmdarima`, que busca la mejor combinación de órdenes minimizando un criterio de información, automatizando lo que de otro modo se haría por inspección.

```python
import pmdarima as pm

auto = pm.auto_arima(log_pib, seasonal=False, d=None,
                     information_criterion="aic",
                     stepwise=True, suppress_warnings=True)
print(auto.summary())
```

`auto_arima` no exime de criterio: sigue siendo responsabilidad del analista verificar que la d elegida tenga sentido, que los residuos superen Ljung-Box y que el modelo no esté sobreajustado. Pero ahorra la parte mecánica de la búsqueda y suele ofrecer un excelente punto de partida.

## Bibliografía

ARIMA reúne autorregresión, integración y medias móviles, y su orden d incorpora la diferenciación dentro del modelo, resolviendo la no estacionariedad sin trabajo manual. Entrega pronósticos con intervalos, lo cual es esencial para comunicar incertidumbre. SARIMA extiende todo a series estacionales, y `auto_arima` automatiza la selección de órdenes. Resta comprender mejor las transformaciones que vuelven modelable una serie, en particular la diferenciación y el logaritmo, que se sistematizan en la siguiente página.

## Referencias

1. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
