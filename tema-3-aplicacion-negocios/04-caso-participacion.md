# 3.4 Caso de negocio: participación BRICS+ en el PIB mundial

En este caso pongo el Tema 3 a trabajar sobre una variable de negocio concreta: la participación del bloque BRICS+ en el PIB mundial. La participación es más interesante que el PIB absoluto para una decisión, porque es una cuota de mercado: una fracción entre cero y uno que dice qué porción del producto global controla cada bloque. Modelarla y proyectarla es, literalmente, modelar el reparto del poder económico mundial.

## La decisión de negocio

Me pongo en el lugar de un equipo de estrategia de un banco de desarrollo del Sur Global. La decisión es cuánto peso dar al bloque BRICS+ en una cartera de exposición a diez años. La pregunta que el modelo debe responder es si la participación de BRICS+ en el PIB mundial seguirá creciendo, a qué ritmo, y con cuánta incertidumbre. Una participación creciente justifica aumentar la exposición; una que se estanca, no.

## Construcción de la variable de participación

```python
import wbgapi as wb
import numpy as np
import pandas as pd

brics = ["BRA", "RUS", "IND", "CHN", "ZAF", "EGY", "ETH", "IRN", "ARE"]

# PIB mundial total y PIB del bloque, ambos en dólares corrientes
mundo = wb.data.DataFrame("NY.GDP.MKTP.CD", "WLD", time=range(1995, 2023)).iloc[0]
mundo = mundo.sort_index().astype(float)
mundo.index = range(1995, 2023)

df = wb.data.DataFrame("NY.GDP.MKTP.CD", brics, time=range(1995, 2023))
df.columns = [int(c.replace("YR", "")) for c in df.columns]
pib_brics = df.sum(axis=0).sort_index()

participacion = (pib_brics / mundo).dropna()
participacion.index = pd.period_range(str(participacion.index.min()),
                                      str(participacion.index.max()), freq="Y")
print(participacion.round(3).tail())
```

La participación es una variable acotada en el intervalo cero a uno, lo que tiene una consecuencia técnica: no puedo modelarla directamente con un ARIMA, porque ARIMA no respeta los límites y podría proyectar valores fuera de rango. La solución estándar es transformarla con la función logit, que mapea el intervalo cero a uno a toda la recta real, modelar la serie transformada, y deshacer la transformación al final.

## La transformación logit

```python
def logit(p):
    return np.log(p / (1 - p))

def inv_logit(x):
    return 1 / (1 + np.exp(-x))

part_logit = logit(participacion)
```

Esta es la misma idea que usar el logaritmo para el PIB en niveles: transformar para que la serie viva en un espacio donde el modelo es válido, y volver al espacio original al reportar. El logit es a las proporciones lo que el logaritmo es a las cantidades positivas.

## Modelado y proyección

```python
from statsmodels.tsa.arima.model import ARIMA

modelo = ARIMA(part_logit, order=(1, 1, 0)).fit()
pred = modelo.get_forecast(steps=10)

media = inv_logit(pred.predicted_mean)
ic = inv_logit(pred.conf_int(alpha=0.10))

resultado = pd.DataFrame({
    "participacion_central": media.values,
    "pesimista": ic.iloc[:, 0].values,
    "optimista": ic.iloc[:, 1].values,
}, index=media.index)
print((resultado * 100).round(1))
```

Reporto la participación proyectada como porcentaje con su banda al 90 por ciento. Por construcción, todos los valores quedan dentro del rango válido, gracias al logit. La columna central es el escenario base, y la distancia entre pesimista y optimista es la incertidumbre que el equipo de estrategia debe tolerar.

## De la proyección a la métrica de decisión

Siguiendo el capítulo anterior, no entrego la banda muda, sino que la traduzco a una probabilidad de umbral relevante para la decisión.

```python
sim = modelo.simulate(nsimulations=10, anchor="end", repetitions=5000)
part_final = inv_logit(sim.iloc[-1].values)

umbral = participacion.iloc[-1]    # nivel actual del bloque
prob_crece = np.mean(part_final > umbral)
print(f"P(participación dentro de 10 años > nivel actual) = {prob_crece:.1%}")
```

Si esa probabilidad es alta, la recomendación de aumentar exposición tiene respaldo cuantitativo. Si fuera baja o ambigua, el modelo estaría diciendo que la tendencia ascendente no es tan robusta como el relato sugiere, y la prudencia obligaría a moderar la apuesta.

## La lectura económica y política

La participación de BRICS+ en el PIB mundial ha crecido de forma sostenida durante dos décadas, impulsada sobre todo por China e India, y el modelo recoge esa inercia. Pero quiero subrayar dos matices que un análisis honesto no puede esconder.

Primero, medido en dólares corrientes el ascenso se ve más lento que en paridad de poder adquisitivo, donde el bloque ya pesa más que el G7. La unidad de medida no es neutral, y lo retomo en el Tema 6.

Segundo, la participación agregada del bloque esconde una enorme heterogeneidad interna: China domina, mientras que otros miembros aportan poco al agregado. Tratar al bloque como una sola economía es una simplificación útil para la decisión de cartera, pero borra tensiones internas que importan políticamente. El modelo responde la pregunta de cartera que le hice, no la pregunta más fina sobre la cohesión interna del bloque.

## Cierre del Tema 3

Apliqué todo el aparato de los sistemas continuos a una decisión de negocio real: proyectar la participación de BRICS+ en el PIB mundial para informar una cartera de exposición. Usé la transformación logit para respetar los límites de una proporción, entregué incertidumbre cuantificada y la traduje a una probabilidad de decisión. El modelo no decide por el equipo de estrategia, pero convierte una intuición geopolítica en una cifra con margen de error sobre la cual se puede decidir con responsabilidad. El Tema 4 retrocede al principio del proceso para tratar con rigor la etapa que más determina el éxito: identificar y formular bien el problema de variable continua.

## Referencias

Law, A. M. (2014). *Simulation modeling and analysis* (5a ed.). McGraw-Hill.

Banco Mundial. (2024). *World Development Indicators* [Conjunto de datos]. https://databank.worldbank.org/source/world-development-indicators

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
