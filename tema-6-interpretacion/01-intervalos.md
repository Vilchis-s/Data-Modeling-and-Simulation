# 6.1 Intervalos de confianza y de predicción

Interpretar un modelo empieza por entender lo que casi todo el mundo confunde: la diferencia entre un intervalo de confianza y un intervalo de predicción. Son cosas distintas, miden incertidumbres distintas y se usan para decisiones distintas. Confundirlos lleva a subestimar el riesgo, que es el error más caro que un analista puede cometer al comunicar un pronóstico.

## Dos incertidumbres distintas

Cuando proyecto el PIB hay dos fuentes de incertidumbre que no hay que mezclar.

La primera es la incertidumbre sobre los parámetros del modelo. No conozco el valor exacto de la tasa de crecimiento o de los coeficientes del ARIMA; los estimé de una muestra finita, así que tienen error. Esta es la incertidumbre que captura el intervalo de confianza.

La segunda es la incertidumbre sobre la realización futura. Aunque conociera los parámetros perfectos, el futuro tiene ruido propio: choques, sorpresas, eventos que ningún modelo anticipa. Esta es la incertidumbre adicional que captura el intervalo de predicción.

La consecuencia práctica es que el intervalo de predicción siempre es más ancho que el de confianza, porque suma las dos incertidumbres. Para decidir sobre el futuro, el que importa es el de predicción, no el de confianza. Reportar un intervalo de confianza como si fuera de predicción hace que el pronóstico parezca mucho más preciso de lo que es.

## Verlo en el modelo del PIB

```python
import numpy as np
import wbgapi as wb
import pandas as pd
from statsmodels.tsa.arima.model import ARIMA

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "CHN", time=range(1990, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = pd.period_range("1990", "2022", freq="Y")

modelo = ARIMA(np.log(pib), order=(1, 1, 1)).fit()
pred = modelo.get_forecast(steps=10)

media = np.exp(pred.predicted_mean)
# get_forecast en statsmodels devuelve el intervalo de PREDICCIÓN
ic_pred = np.exp(pred.conf_int(alpha=0.05))

ancho = (ic_pred.iloc[:, 1] - ic_pred.iloc[:, 0]) / media.values
print("Ancho relativo del intervalo de predicción por horizonte:")
print((ancho * 100).round(1))
```

El ancho relativo crece con el horizonte, y esa es la lección visual más importante de todo el tema de interpretación: cuanto más lejos proyecto, menos sé, y el intervalo lo dice con honestidad. Un pronóstico cuyo intervalo no se ensancha con el horizonte es sospechoso, porque viola algo que sabemos del mundo.

## El intervalo se abre como un cono

La forma típica del intervalo de predicción es un cono que se abre hacia el futuro. Para el PIB, eso significa que el valor a un año tiene una banda estrecha, pero a diez años la banda es ancha. La decisión de cartera del Tema 4 tiene que vivir con ese cono: si la recomendación cambia según se tome el borde inferior o el superior del cono a diez años, entonces el modelo no es lo bastante preciso para esa decisión a ese horizonte, y hay que decirlo en lugar de esconderlo.

## El error de interpretación que la guía advierte

La guía del curso marca un error crítico de interpretación que aplica directo aquí: un valor esperado favorable puede convivir con una probabilidad alta de resultado adverso, y la decisión depende del apetito de riesgo (Law, 2014). Trasladado al PIB: el pronóstico central puede mostrar que el bloque BRICS+ crece, pero si el borde inferior del intervalo de predicción incluye un estancamiento, entonces existe un escenario plausible donde la transición se frena. Reportar solo el central oculta ese escenario. La interpretación honesta reporta el central y el rango, y deja que quien decide pondere su tolerancia al riesgo.

## La calibración del intervalo

Un intervalo solo es útil si está bien calibrado, es decir, si un intervalo al 95 por ciento de verdad contiene al valor real el 95 por ciento de las veces. Esto se verifica con el backtesting: cuento qué fracción de los datos reales cayó dentro de los intervalos proyectados en la validación. Si la cobertura observada es mucho menor que la nominal, el modelo es sobreconfiado y sus intervalos mienten por estrechos.

```python
def cobertura(reales, inf, sup):
    dentro = (reales >= inf) & (reales <= sup)
    return np.mean(dentro)

# En un backtesting, la cobertura debería acercarse al nivel nominal del IC.
# Cobertura muy por debajo del nominal = intervalos demasiado estrechos = riesgo subestimado.
```

Para variables económicas con colas pesadas, como vi en el EDA del capítulo 4.3, los intervalos basados en normalidad tienden a quedar cortos en los extremos, porque las crisis son más frecuentes de lo que una normal predice. Por eso desconfío de un intervalo que se ve demasiado limpio.

## Cierre

El intervalo de confianza mide la incertidumbre sobre los parámetros, el de predicción suma además la incertidumbre del futuro, y para decidir importa el de predicción, que es más ancho. El intervalo se abre como un cono con el horizonte, lo que es honesto, y debe estar bien calibrado, algo que se verifica con backtesting. Un valor esperado favorable no anula un borde inferior preocupante. Con esto claro, la siguiente página interpreta lo que significan los parámetros del modelo, no solo sus pronósticos.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
