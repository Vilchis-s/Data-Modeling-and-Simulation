# 2.5 Modelos AR, MA y ARMA

Llego a la familia que define las series de tiempo clásicas. Los modelos AR, MA y su combinación ARMA son la base sobre la que se construye ARIMA, el caballo de batalla del pronóstico estadístico. Todos parten de la misma idea: el valor de hoy se explica como una combinación lineal de información pasada, ya sea de valores pasados de la serie o de errores pasados.

## El modelo autorregresivo AR

Un modelo autorregresivo de orden p, escrito AR(p), explica el valor de hoy como una combinación lineal de los p valores anteriores más un choque aleatorio:

```
y_t = c + phi_1 * y_{t-1} + phi_2 * y_{t-2} + ... + phi_p * y_{t-p} + e_t
```

Los coeficientes phi miden cuánta inercia arrastra la serie. Un AR(1) con phi cercano a 1 describe una serie muy persistente, donde los choques tardan mucho en disiparse. Esto tiene una lectura económica directa: las economías con alta persistencia, donde un buen o mal año arrastra a los siguientes, tienen un phi alto. La condición de estacionariedad de un AR(1) es que el valor absoluto de phi sea menor que 1; si fuera 1, tendríamos una raíz unitaria y la serie no sería estacionaria, justo el caso que la prueba ADF detecta.

```python
import wbgapi as wb
import pandas as pd
import numpy as np
from statsmodels.tsa.arima.model import ARIMA

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "IND", time=range(1970, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = pd.period_range("1970", "2022", freq="Y")

crecimiento = np.log(pib).diff().dropna()       # serie estacionaria

ar1 = ARIMA(crecimiento, order=(1, 0, 0)).fit()  # AR(1)
print(ar1.summary().tables[1])
```

El coeficiente autorregresivo de la tasa de crecimiento de India me dice cuánta memoria tiene su crecimiento de un año al siguiente. Un valor positivo y significativo significa que un año fuerte tiende a ir seguido de otro año fuerte, lo que en términos de la guía del curso es un lazo reforzador suave operando en la dinámica de corto plazo.

## El modelo de medias móviles MA

Un modelo de medias móviles de orden q, MA(q), explica el valor de hoy como una combinación de los choques aleatorios recientes, no de los valores de la serie:

```
y_t = c + e_t + theta_1 * e_{t-1} + ... + theta_q * e_{t-q}
```

La intuición es distinta a la del AR. En un MA, lo que persiste no es el nivel de la serie sino el efecto de las sorpresas. Un choque inesperado, como una sanción o una crisis, no afecta solo el año en que ocurre, sino que se arrastra durante q periodos a través de los términos theta. Para variables económicas esto modela bien cómo un evento puntual deja una estela que se desvanece en pocos años.

```python
ma1 = ARIMA(crecimiento, order=(0, 0, 1)).fit()  # MA(1)
print(ma1.summary().tables[1])
```

## Cómo elegir entre AR y MA

AR y MA capturan dos formas distintas de memoria, y la pregunta práctica es cuál usar para una serie dada. La respuesta no se adivina, se lee en los gráficos del capítulo 2.3. La regla es la que vimos ahí: si la PACF se corta de golpe, la serie pide un AR y el rezago de corte es su orden; si la ACF se corta de golpe, pide un MA. A veces las dos describen razonablemente bien la misma serie, así que no hay que obsesionarse con encontrar la única estructura correcta. Cuando ninguno de los dos gráficos se corta limpio, la señal es que conviene combinarlos en un modelo mixto, que es justo lo que sigue.

## El modelo combinado ARMA

Un ARMA(p, q) junta las dos ideas: el valor de hoy depende de p valores pasados de la serie y de q choques pasados.

```
y_t = c + sum(phi_i * y_{t-i}) + e_t + sum(theta_j * e_{t-j})
```

La ventaja de combinar es la parsimonia. Una serie que necesitaría un AR de orden 5 para describirse podría quedar bien capturada con un ARMA(1,1) de solo dos parámetros. Como menos parámetros significa menos riesgo de sobreajuste y estimaciones más estables, casi siempre prefiero un ARMA pequeño antes que un AR o MA grande.

```python
arma = ARIMA(crecimiento, order=(1, 0, 1)).fit()  # ARMA(1,1)
print(f"AIC ARMA(1,1) = {arma.aic:.2f}")
print(f"AIC AR(1)     = {ar1.aic:.2f}")
print(f"AIC MA(1)     = {ma1.aic:.2f}")
```

Comparo los tres con el criterio de información AIC, que penaliza la complejidad. El modelo con menor AIC es el preferido, y este criterio formaliza la búsqueda de parsimonia. Desarrollo AIC y BIC a fondo en el capítulo 5.2.

## El supuesto crítico: la serie debe ser estacionaria

Todo lo anterior exige que la serie sea estacionaria. Por eso en los ejemplos no modelé el PIB en niveles sino su log-diferencia. Si le paso a un ARMA una serie con tendencia, las estimaciones son basura, porque el modelo supone media constante y la serie no la tiene. La operación de diferenciar para volver estacionaria es lo que convierte un ARMA en un ARIMA, y ese paso, la integración, es el tema de la siguiente página.

## Cierre

AR explica el presente con valores pasados, MA lo explica con choques pasados, y ARMA combina ambos buscando parsimonia. Todos exigen estacionariedad. La forma de la ACF y la PACF sugiere los órdenes, y los criterios de información deciden entre candidatos. El paso que falta para modelar series reales con tendencia es incorporar la diferenciación dentro del modelo, lo que da lugar a ARIMA y su extensión estacional SARIMA.

## Referencias

Box, G. E. P., Jenkins, G. M., Reinsel, G. C., & Ljung, G. M. (2015). *Time series analysis: Forecasting and control* (5a ed.). Wiley.

Hamilton, J. D. (1994). *Time series analysis*. Princeton University Press.

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
