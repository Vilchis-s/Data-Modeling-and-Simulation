# 2.9 Caso: ARIMA sobre el PIB de BRICS+ y G7

El Tema 2 cierra con el caso integrador de la rama estadística. Se construyen las dos series agregadas, BRICS+ y G7, se ajusta un modelo ARIMA a cada una con todo el flujo de preprocesamiento y validación, se proyectan ambas hacia adelante con intervalos, y se interpreta qué indica el modelo sobre el momento en que las trayectorias se cruzan. Es el espejo del caso de la ODE del Tema 1, ahora sin postular ninguna ley del sistema.

## Construcción de las dos series

```python
import wbgapi as wb
import numpy as np
import pandas as pd

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]
g7 = ["USA", "JPN", "DEU", "GBR", "FRA", "ITA", "CAN"]

def pib_bloque(paises, ini=1995, fin=2023):
    df = wb.data.DataFrame("NY.GDP.MKTP.CD", paises, time=range(ini, fin))
    df.columns = [int(c.replace("YR", "")) for c in df.columns]
    return (df.sum(axis=0) / 1e12).sort_index()

pib_brics = pib_bloque(brics)
pib_g7 = pib_bloque(g7)

panel = pd.DataFrame({"BRICS+": pib_brics, "G7": pib_g7})
panel.index = pd.PeriodIndex(panel.index, freq="Y")
print(panel.tail())
```

Se utiliza PIB a precios corrientes en dólares. Conviene una advertencia que se retoma en el Tema 6: medir en dólares corrientes favorece al G7, pues buena parte del producto de los BRICS+ se valora mejor en paridad de poder adquisitivo. Se mantiene así en este caso por simplicidad y se problematiza después, pues la elección de la unidad de medida es, en sí misma, una decisión cargada de implicaciones.

## Preprocesamiento y selección de orden

Se aplica el flujo del capítulo 2.7 (tema-2-series-de-tiempo/07-diferenciacion-transformaciones.md) a cada bloque: logaritmo para la varianza, prueba de estacionariedad y selección automática de orden.

```python
import pmdarima as pm
from statsmodels.tsa.stattools import adfuller

def ajustar_bloque(serie):
    log_s = np.log(serie)
    d_adf = adfuller(log_s.diff().dropna())[1]
    modelo = pm.auto_arima(log_s, seasonal=False,
                           information_criterion="aic",
                           stepwise=True, suppress_warnings=True)
    return log_s, modelo

log_brics, m_brics = ajustar_bloque(pib_brics)
log_g7, m_g7 = ajustar_bloque(pib_g7)
print("Orden BRICS+:", m_brics.order)
print("Orden G7:    ", m_g7.order)
```

Cada bloque puede recibir un orden distinto, y eso es correcto: no hay razón para imponer la misma estructura a dos economías con dinámicas diferentes. El bloque emergente, de crecimiento más volátil, suele requerir una estructura distinta a la del bloque maduro, más inercial.

## Validación antes de proyectar

No se proyecta sin antes medir el error fuera de muestra con un corte temporal, según el capítulo 2.8 (tema-2-series-de-tiempo/08-validacion-metricas.md).

```python
def validar(log_s, order, corte=2016):
    train = log_s[log_s.index.year <= corte]
    test = log_s[log_s.index.year > corte]
    from statsmodels.tsa.arima.model import ARIMA
    m = ARIMA(train, order=order).fit()
    pred = m.forecast(steps=len(test))
    real, fc = np.exp(test.values), np.exp(pred.values)
    mape = np.mean(np.abs((real - fc) / real))
    return mape

print(f"MAPE BRICS+: {validar(log_brics, m_brics.order):.2%}")
print(f"MAPE G7:     {validar(log_g7, m_g7.order):.2%}")
```

Si el MAPE fuera de muestra es bajo, se gana confianza para proyectar. Si fuera alto, el modelo no está listo y correspondería volver a etapas anteriores, que es la iteración que la guía de estudio describe como propia de la metodología de modelado.

## Proyección de las dos trayectorias

```python
from statsmodels.tsa.arima.model import ARIMA

def proyectar(log_s, order, pasos=15):
    m = ARIMA(log_s, order=order).fit()
    pred = m.get_forecast(steps=pasos)
    media = np.exp(pred.predicted_mean)
    ic = np.exp(pred.conf_int(alpha=0.05))
    return media, ic

proy_brics, ic_brics = proyectar(log_brics, m_brics.order)
proy_g7, ic_g7 = proyectar(log_g7, m_g7.order)

import matplotlib.pyplot as plt
ax = panel.plot(figsize=(10, 5))
proy_brics.plot(ax=ax, style="--", label="BRICS+ proyectado")
proy_g7.plot(ax=ax, style="--", label="G7 proyectado")
plt.ylabel("PIB (billones USD corrientes)")
plt.title("Proyección ARIMA: BRICS+ vs G7")
plt.legend()
plt.show()
```

## Lectura del cruce

La pregunta de fondo es cuándo, según el modelo, la trayectoria proyectada de BRICS+ alcanza a la de G7 medida en dólares corrientes.

```python
cruce = proy_brics[proy_brics.values >= proy_g7.values]
if len(cruce) > 0:
    print(f"El modelo proyecta el cruce hacia el año {cruce.index[0]}")
else:
    print("El modelo no proyecta cruce dentro del horizonte en dólares corrientes")
```

Aquí se requiere especial cautela, y es justamente el tipo de cautela que se desarrolla en el Tema 6. Medido en dólares corrientes, el cruce puede no aparecer en el horizonte, mientras que medido en paridad de poder adquisitivo el bloque BRICS+ ya superó al G7 hace años según varias fuentes. El modelo no engaña, pero responde exactamente la pregunta que se le formuló, y esa pregunta incluye la unidad de medida. Cambiar de dólares corrientes a PPP cambia la conclusión geopolítica, y un análisis honesto reporta ambas.

## ODE contra ARIMA: dos lentes sobre el mismo fenómeno

Conviene comparar lo que cada rama aportó sobre el mismo PIB de BRICS+. La ODE logística del Tema 1 ofreció un mecanismo interpretable, motor de crecimiento más techo estructural, con dos parámetros de significado económico, pero sin incertidumbre. El ARIMA de este capítulo no ofrece mecanismo, pero entrega pronósticos con intervalos que cuantifican el grado de desconocimiento, y se adapta a la dinámica observada sin imponer una forma. Ninguna es la verdadera. Son lentes complementarios, y en el Tema 5 se comparan formalmente para decidir cuándo conviene cada uno.

## Bibliografía del Tema 2

Se recorrió la rama estadística completa: de los componentes de una serie a la estacionariedad, de la ACF y PACF a los modelos AR, MA, ARMA, ARIMA y SARIMA, de las transformaciones a la validación temporal honesta, y de ahí a un caso real con datos del Banco Mundial sobre la transición BRICS+ frente a G7. Las dos ramas de los sistemas continuos, la mecanicista y la estadística, ya están sobre la mesa. El Tema 3 las conecta con lo que realmente importa en la práctica profesional: una decisión de negocio.

## Referencias

1. Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators
