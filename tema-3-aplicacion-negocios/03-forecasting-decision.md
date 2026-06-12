# 3.3 El forecasting como insumo de decisión

El forecasting, o pronóstico, es el producto más demandado de los modelos continuos en el ámbito de los negocios. Pero un pronóstico mal entendido o mal comunicado causa más daño que la ausencia de pronóstico, pues genera una falsa sensación de certeza. Este capítulo trata cómo convertir un pronóstico en un insumo de decisión honesto, evitando los errores que erosionan la confianza en proyectos reales.

## Un pronóstico es una distribución, no un número

El error más frecuente y más grave es tratar el pronóstico como un número. Cuando un modelo indica que el PIB de BRICS+ alcanzará cierto valor en 2035, esa cifra es solo el centro de una distribución de futuros posibles. La información verdaderamente útil para decidir reside en la dispersión, no en el centro. La guía de estudio lo reitera para Monte Carlo, y vale igual para el forecasting: siempre se reporta el intervalo junto al estimador puntual, nunca el punto aislado.

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
ic = np.exp(pred.conf_int(alpha=0.10))     # intervalo al 90%

resumen = pd.DataFrame({
    "central": media.values,
    "pesimista": ic.iloc[:, 0].values,
    "optimista": ic.iloc[:, 1].values,
}, index=media.index)
print(resumen.round(1))
```

El entregable no es la columna central, sino las tres columnas en conjunto. La distancia entre la pesimista y la optimista indica a quien decide cuánto margen de error debe tolerar su estrategia.

## Traducir incertidumbre a lenguaje de decisión

La guía de estudio propone métricas de riesgo para Monte Carlo que traducen una distribución a algo accionable: percentiles, probabilidad de quedar por debajo de un umbral y medidas de cola. Esa misma idea se traslada al forecasting. En lugar de entregar una banda muda, se responden preguntas de decisión.

```python
from statsmodels.tsa.arima.model import ARIMA

# Probabilidad de que el PIB supere un umbral usando la distribución del pronóstico
sim = modelo.simulate(nsimulations=10, anchor="end", repetitions=5000)
pib_2032 = np.exp(sim.iloc[-1].values)
umbral = 28.0
prob = np.mean(pib_2032 >= umbral)
p10, p50, p90 = np.percentile(pib_2032, [10, 50, 90])
print(f"P(PIB 2032 >= {umbral}) = {prob:.1%}")
print(f"Percentiles  P10={p10:.1f}  P50={p50:.1f}  P90={p90:.1f}")
```

Una formulación como existe un 70 por ciento de probabilidad de que el PIB supere cierto umbral en una década es infinitamente más útil para decidir que un número aislado. Es el mismo espíritu del error crítico de interpretación que advierte la guía: un valor esperado positivo puede convivir con una probabilidad alta de resultado adverso, y la decisión depende del apetito de riesgo de quien decide.

## El horizonte honesto

Cada modelo tiene un horizonte más allá del cual su pronóstico es ficción. Para datos anuales con pocas décadas de historia, proyectar treinta años es deshonesto: el intervalo se vuelve tan ancho que abarca cualquier resultado. La regla práctica es no proyectar más allá de un tercio de la longitud de la serie sin advertir con énfasis que, a partir de cierto punto, la proyección es ilustrativa, no operativa. Prometer precisión a un horizonte que el dato no soporta es la forma más rápida de perder credibilidad.

## El pronóstico se monitorea, no se entrega y se olvida

En producción, un pronóstico no es un entregable final, sino un sistema vivo. Cada año llega un dato nuevo, se reentrena y se compara el dato real con lo que el modelo había proyectado. Si el dato cae fuera del intervalo con mayor frecuencia de la que el nivel de confianza permite, el modelo está mal calibrado y debe revisarse. Esta disciplina de monitoreo es lo que separa un proyecto de ciencia de datos serio de un reporte de una sola vez.

```python
def dentro_del_intervalo(real, ic_inf, ic_sup):
    return (real >= ic_inf) & (real <= ic_sup)

# En el monitoreo anual, la tasa de cobertura debería acercarse al nivel nominal
# del intervalo (por ejemplo, 90% de los datos reales dentro del IC al 90%).
```

## La lectura para el caso geopolítico

Cuando el pronóstico recae sobre algo tan cargado como la transición de poder entre bloques, la tentación de sobrevender una conclusión, en cualquier dirección, es considerable. Existe tanto análisis triunfalista que da por hecho el sorpasso de los BRICS+ como análisis escéptico que lo niega de plano, y ambos suelen ignorar el intervalo. La posición que este trabajo adopta es que la honestidad sobre la incertidumbre no debilita el argumento de fondo, sino que lo fortalece: el ascenso del bloque emergente es lo bastante robusto para sostenerse sin necesidad de ocultar el error del modelo. Declarar cuánto se desconoce es parte de tomar el tema en serio.

## Bibliografía

Un pronóstico es una distribución de futuros, y su valor para decidir reside en la incertidumbre, no en el punto central. Se traduce a lenguaje de decisión con percentiles y probabilidades de umbral, se respeta un horizonte honesto, y se monitorea en el tiempo en lugar de entregarse y olvidarse. Con esto se cierra la parte conceptual del Tema 3 y se pasa al caso aplicado, donde todo lo anterior se emplea para analizar la participación de BRICS+ en el PIB mundial.

## Referencias

1. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
