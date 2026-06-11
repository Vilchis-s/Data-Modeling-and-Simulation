# 3.3 El forecasting como insumo de decisión

El forecasting, o pronóstico, es el producto más demandado de los modelos continuos en el mundo de los negocios. Pero un pronóstico mal entendido o mal comunicado hace más daño que no tener ninguno, porque da una falsa sensación de certeza. Este capítulo trata cómo convierto un pronóstico en un insumo de decisión honesto, evitando los errores que vi destruir confianza en proyectos reales.

## Un pronóstico es una distribución, no un número

El error más común y más grave es tratar el pronóstico como un número. Cuando un modelo dice que el PIB de BRICS+ será de cierto valor en 2035, esa cifra es solo el centro de una distribución de futuros posibles. La información que de verdad sirve para decidir está en la dispersión, no en el centro. La guía del curso lo repite para Monte Carlo y vale igual para el forecasting: siempre se reporta el intervalo junto al estimador puntual, nunca el punto solo (Law, 2014).

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

El entregable no es la columna central, son las tres columnas juntas. La distancia entre la pesimista y la optimista le dice al que decide cuánto margen de error debe tolerar su estrategia.

## Traducir incertidumbre a lenguaje de decisión

La guía propone métricas de riesgo para Monte Carlo que traducen una distribución a algo accionable: percentiles, probabilidad de quedar por debajo de un umbral, y medidas de cola (Law, 2014). Llevo esa misma idea al forecasting. En vez de entregar una banda muda, respondo preguntas de decisión.

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

Una frase como hay 70 por ciento de probabilidad de que el PIB supere cierto umbral en una década es infinitamente más útil para decidir que un número pelado. Es el mismo espíritu del error crítico de interpretación que advierte la guía: un valor esperado positivo puede convivir con una probabilidad alta de resultado adverso, y la decisión depende del apetito de riesgo de quien decide (Law, 2014).

## El horizonte honesto

Cada modelo tiene un horizonte más allá del cual su pronóstico es ficción. Para datos anuales con pocas décadas de historia, proyectar treinta años es deshonesto: el intervalo se vuelve tan ancho que cubre cualquier cosa. Mi regla práctica es no proyectar más allá de un tercio de la longitud de la serie sin advertir con fuerza que más allá de cierto punto la proyección es ilustrativa, no operativa. Prometer precisión a un horizonte que el dato no soporta es la forma más rápida de perder credibilidad.

## El pronóstico se monitorea, no se entrega y se olvida

En producción, un pronóstico no es un entregable final, es un sistema vivo. Cada año llega un dato nuevo, reentreno y comparo el dato real contra lo que el modelo había proyectado. Si el dato cae fuera del intervalo con más frecuencia de la que el nivel de confianza permite, el modelo está mal calibrado y hay que revisarlo. Esta disciplina de monitoreo es lo que separa un proyecto de ciencia de datos serio de un reporte de una sola vez.

```python
def dentro_del_intervalo(real, ic_inf, ic_sup):
    return (real >= ic_inf) & (real <= ic_sup)

# En el monitoreo anual, la tasa de cobertura debería acercarse al nivel nominal
# del intervalo (por ejemplo, 90% de los datos reales dentro del IC al 90%).
```

## La lectura para el caso geopolítico

Cuando el pronóstico es sobre algo tan cargado como la transición de poder entre bloques, la tentación de sobrevender una conclusión es enorme, en cualquier dirección. He visto tanto análisis triunfalista que da por hecho el sorpasso de los BRICS+ como análisis escéptico que lo niega de plano, y ambos suelen ignorar el intervalo. Mi posición como analista es que la honestidad sobre la incertidumbre no debilita el argumento de fondo, lo fortalece: el ascenso del bloque emergente es lo bastante robusto como para sostenerse sin necesidad de esconder el error del modelo. Decir cuánto no sé es parte de tomar el tema en serio.

## Cierre

Un pronóstico es una distribución de futuros, y su valor para decidir vive en la incertidumbre, no en el punto central. Lo traduzco a lenguaje de decisión con percentiles y probabilidades de umbral, respeto un horizonte honesto, y lo monitoreo en el tiempo en lugar de entregarlo y olvidarlo. Con esto cierro la parte conceptual del Tema 3 y paso al caso aplicado, donde uso todo esto para analizar la participación de BRICS+ en el PIB mundial.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
