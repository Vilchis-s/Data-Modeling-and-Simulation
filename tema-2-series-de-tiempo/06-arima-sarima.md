# 2.6 ARIMA y SARIMA

ARIMA es el modelo de series de tiempo más usado de la historia y, con razón, el estándar contra el que se mide todo lo demás. Su nombre describe sus tres ingredientes: AutoRegresivo, Integrado, de Medias móviles. La pieza nueva respecto al capítulo anterior es la I, la integración, que es la forma elegante de incorporar la diferenciación dentro del propio modelo.

## Los tres órdenes de un ARIMA(p, d, q)

Un ARIMA tiene tres parámetros enteros:

p es el orden autorregresivo, cuántos valores pasados de la serie entran.

d es el orden de integración, cuántas veces hay que diferenciar la serie para volverla estacionaria.

q es el orden de medias móviles, cuántos choques pasados entran.

La d es la clave que conecta con la estacionariedad. En el capítulo 2.2 vi que el PIB en niveles no es estacionario pero su primera diferencia sí lo es. Un ARIMA(p, 1, q) modela exactamente eso: diferencia la serie una vez internamente, ajusta un ARMA(p, q) sobre la serie diferenciada, y luego deshace la diferenciación para devolver pronósticos en la escala original. Ya no tengo que diferenciar a mano: le digo d=1 y el modelo se encarga.

```python
import wbgapi as wb
import pandas as pd
import numpy as np
from statsmodels.tsa.arima.model import ARIMA

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "CHN", time=range(1980, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = pd.period_range("1980", "2022", freq="Y")

log_pib = np.log(pib)                       # estabilizo varianza
modelo = ARIMA(log_pib, order=(1, 1, 1)).fit()
print(modelo.summary())
```

Modelo el logaritmo del PIB con un ARIMA(1,1,1): primero el logaritmo estabiliza la varianza creciente, luego la diferenciación interna estabiliza la media. Es la receta estándar para una variable económica en niveles.

## Pronóstico con intervalos

Un pronóstico sin medida de incertidumbre es casi inútil, y aquí ARIMA brilla, porque entrega intervalos de predicción de forma natural. La guía del curso es enfática en que nunca se reporta un estimador puntual sin su intervalo (Law, 2014), y esa regla aplica al pronóstico tal cual.

```python
pred = modelo.get_forecast(steps=10)
media = np.exp(pred.predicted_mean)                  # vuelvo a escala original
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

El intervalo se abre conforme avanza el horizonte, lo cual es honesto: cuanto más lejos proyecto, menos sé. Esa apertura es exactamente la incertidumbre que un modelo determinista como la ODE logística del Tema 1 no podía expresar, y es una de las grandes ventajas de la rama estadística.

## SARIMA: incorporando estacionalidad

Cuando la serie tiene un patrón estacional, ARIMA no basta y entra SARIMA, que agrega un bloque estacional con sus propios órdenes:

```
SARIMA(p, d, q)(P, D, Q, s)
```

Los primeros tres son los órdenes no estacionales de siempre. Los siguientes cuatro son sus análogos estacionales: P, D, Q son el AR, la integración y el MA estacionales, y s es el periodo de la estación, por ejemplo 4 en datos trimestrales o 12 en mensuales. La D estacional diferencia la serie respecto al mismo periodo del ciclo anterior, por ejemplo este trimestre contra el mismo trimestre del año pasado, lo que elimina la estacionalidad.

El PIB anual no tiene estacionalidad, así que para mostrar SARIMA cambio a una serie trimestral con estacionalidad genuina. Uso datos sintéticos aquí, y lo digo de forma explícita, porque quiero una serie trimestral con un patrón estacional limpio y controlado para que el ejemplo sea didáctico, no por falta de datos reales.

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

SARIMA captura a la vez la tendencia creciente y el ciclo de cuatro trimestres. En el trabajo real lo uso para series trimestrales de comercio o producción de los bloques, donde el patrón estacional es fuerte y omitirlo arruina el pronóstico.

## auto_arima: búsqueda automática de órdenes

Elegir p, d, q a mano con la ACF y la PACF funciona, pero es laborioso y subjetivo. En la práctica uso `auto_arima` de la librería `pmdarima`, que busca la mejor combinación de órdenes minimizando un criterio de información, automatizando lo que de otro modo haría a ojo.

```python
import pmdarima as pm

auto = pm.auto_arima(log_pib, seasonal=False, d=None,
                     information_criterion="aic",
                     stepwise=True, suppress_warnings=True)
print(auto.summary())
```

`auto_arima` no me exime de pensar: sigue siendo mi responsabilidad verificar que la d elegida tenga sentido, que los residuos pasen Ljung-Box y que el modelo no esté sobreajustado. Pero me ahorra la parte mecánica de la búsqueda y suele dar un excelente punto de partida.

## Cierre

ARIMA junta autorregresión, integración y medias móviles, y su orden d incorpora la diferenciación dentro del modelo, resolviendo la no estacionariedad sin trabajo manual. Entrega pronósticos con intervalos, algo esencial para comunicar incertidumbre. SARIMA extiende todo a series estacionales, y `auto_arima` automatiza la selección de órdenes. Lo que falta para cerrar la mecánica es entender mejor las transformaciones que vuelven una serie modelable, en particular la diferenciación y el logaritmo, que sistematizo en la siguiente página.

## Referencias

Box, G. E. P., Jenkins, G. M., Reinsel, G. C., & Ljung, G. M. (2015). *Time series analysis: Forecasting and control* (5a ed.). Wiley.

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.
