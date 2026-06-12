# 2.7 Diferenciación, tendencia y transformaciones

Este capítulo sistematiza las operaciones que convierten una serie cruda y no modelable en una serie estacionaria que un ARIMA puede tratar. Son tres herramientas: el logaritmo para la varianza, la diferenciación para la tendencia y la diferenciación estacional para el ciclo. El orden de aplicación importa, y aplicarlas en exceso es tan dañino como aplicarlas de menos.

## El logaritmo estabiliza la varianza

Las variables económicas casi siempre tienen varianza que crece con el nivel. Una recesión del 5 por ciento sobre una economía de 18 billones de dólares es un movimiento absoluto enorme comparado con el mismo 5 por ciento sobre una economía de 2 billones. El logaritmo corrige esto, pues convierte cambios porcentuales en cambios absolutos: la diferencia de logaritmos es, aproximadamente, la tasa de crecimiento.

```python
import numpy as np
import wbgapi as wb
import pandas as pd

pib = wb.data.DataFrame("NY.GDP.MKTP.CD", "CHN", time=range(1980, 2023)).iloc[0]
pib = pib.sort_index().astype(float) / 1e12
pib.index = range(1980, 2023)

log_pib = np.log(pib)
print("Desv. estándar de diferencias en niveles:", pib.diff().std())
print("Desv. estándar de diferencias en log:    ", log_pib.diff().std())
```

La diferencia de logaritmos tiene una desviación estándar mucho más estable a lo largo del tiempo que la diferencia en niveles, que es justamente el objetivo. Por ello, casi todos los modelos de PIB se ajustan sobre el logaritmo y los pronósticos se devuelven exponenciando.

## La diferenciación elimina la tendencia

Diferenciar una serie consiste en reemplazar cada valor por su cambio respecto al anterior:

```
diferencia_t = y_t - y_{t-1}
```

Una serie con tendencia lineal se vuelve estacionaria con una diferenciación. Una con tendencia cuadrática puede requerir dos. El orden de diferenciación es exactamente la d del ARIMA. La regla práctica es diferenciar lo mínimo necesario para superar la prueba ADF, pues cada diferenciación adicional introduce ruido y puede crear autocorrelación artificial.

```python
from statsmodels.tsa.stattools import adfuller

def cuantas_diferencias(serie, max_d=2):
    s = serie.dropna().copy()
    for d in range(max_d + 1):
        p = adfuller(s)[1]
        if p < 0.05:
            return d
        s = s.diff().dropna()
    return max_d

print("d sugerido para log(PIB):", cuantas_diferencias(log_pib))
```

Esta función diferencia de forma incremental hasta que la serie supera ADF y devuelve el número mínimo de diferenciaciones. Es la versión manual de lo que `auto_arima` hace internamente al elegir d.

## El peligro de sobrediferenciar

Diferenciar en exceso es un error sutil pero costoso. Si una serie ya es estacionaria y se diferencia, se introduce una autocorrelación negativa espuria en el primer rezago y se aumenta la varianza sin necesidad. La señal de sobrediferenciación es una ACF con un valor fuertemente negativo en el rezago 1. La regla de oro es: la d más pequeña que logre estacionariedad es la correcta, nunca más.

## La diferenciación estacional

Para series con estacionalidad, la diferenciación ordinaria no elimina el ciclo. Lo que lo elimina es la diferenciación estacional, que resta el valor del mismo periodo del ciclo anterior:

```
diferencia_estacional_t = y_t - y_{t-s}
```

donde s es el periodo, 4 para trimestres o 12 para meses. Esto compara cada trimestre con el mismo trimestre del año previo, eliminando el patrón que se repite. Es la D estacional del SARIMA. En series con tendencia y estacionalidad, a veces se aplican ambas, una diferenciación ordinaria y una estacional, con la precaución de no excederse.

## Transformación Box-Cox: generalizando el logaritmo

El logaritmo es un caso particular de una familia más amplia, la transformación Box-Cox, que tiene un parámetro lambda estimado de los datos. Cuando lambda es 0, Box-Cox equivale al logaritmo; cuando es 1, no transforma; valores intermedios dan transformaciones de potencia. Permitir que los datos elijan lambda a veces estabiliza la varianza mejor que el logaritmo impuesto.

```python
from scipy.stats import boxcox

# Box-Cox exige valores positivos, que el PIB cumple
transformada, lam = boxcox(pib.values)
print(f"lambda óptimo de Box-Cox: {lam:.3f}")
```

Si el lambda óptimo resulta cercano a 0, confirma que el logaritmo era la elección correcta, lo cual suele ocurrir con el PIB. Sirve como verificación: si Box-Cox sugiere un lambda lejano a 0, conviene reconsiderar la transformación logarítmica.

## El flujo completo de preprocesamiento

Se reúne todo en el orden correcto, que es la secuencia a aplicar antes de cualquier ARIMA:

```
1. Graficar la serie e inspeccionar varianza y tendencia.
2. Si la varianza crece con el nivel, aplicar logaritmo o Box-Cox.
3. Probar estacionariedad con ADF y KPSS.
4. Diferenciar el mínimo necesario para superar ADF.
5. Si hay estacionalidad, aplicar diferenciación estacional.
6. Verificar con ACF que no haya sobrediferenciación.
7. Modelar con ARIMA o SARIMA sobre la serie ya estacionaria.
```

Omitir este flujo es la causa más frecuente de pronósticos deficientes: ajustar ARIMA sobre series con tendencia o varianza explosiva produce intervalos sin sentido.

## Bibliografía

El logaritmo estabiliza la varianza, la diferenciación elimina la tendencia, la diferenciación estacional elimina el ciclo, y Box-Cox generaliza la idea del logaritmo. La clave es transformar lo justo y nunca en exceso, pues sobrediferenciar introduce ruido y autocorrelación falsa. Con la serie preparada y el modelo ajustado, resta lo más importante para que el modelo sirva: validarlo adecuadamente. Esto exige una validación que respete el orden temporal, tema de la siguiente página.

## Referencias

1. Box, G. E. P., & Cox, D. R. (1964). An analysis of transformations. *Journal of the Royal Statistical Society: Series B, 26*(2), 211-243. https://doi.org/10.1111/j.2517-6161.1964.tb00553.x
2. Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3a ed.). OTexts. https://otexts.com/fpp3/
